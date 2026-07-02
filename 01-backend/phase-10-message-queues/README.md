# Phase 10 — Message Queues & Background Jobs

Message queues cho phép decouple các parts của system — thay vì xử lý đồng bộ (user phải đợi), bạn push task vào queue và worker xử lý asynchronously. Điều này cải thiện response time, reliability, và scalability.

---

## Tại sao cần Message Queue?

```
Không có queue (synchronous):
User → POST /orders → [validate] → [save to DB] → [send email] → [generate invoice] → [update analytics] → Response
     ← ← ← ← ← ← ← ← ← ← (user đợi 3-5 giây) ← ← ← ← ← ← ← ← ← ← ← ← ←

Với queue (asynchronous):
User → POST /orders → [save to DB] → Response (< 100ms)
                                     ↓ (publish to queue)
                          Worker: [send email] [generate invoice] [update analytics]
```

**Use cases:**
- Gửi email/SMS sau action
- Resize ảnh upload
- Generate reports/invoices
- Sync data sang 3rd-party services
- Scheduled tasks (cron jobs)

---

## BullMQ — Redis-backed Job Queue

BullMQ là job queue mạnh nhất cho Node.js, dùng Redis để store jobs, hỗ trợ retry, delay, priority, cron.

```bash
npm install bullmq
# Redis cần đang chạy
```

### Producer — Thêm jobs vào queue

```js
const { Queue } = require('bullmq')

const connection = { host: 'localhost', port: 6379 }

// Tạo queues
const emailQueue = new Queue('emails', { connection })
const imageQueue = new Queue('images', { connection })
const reportQueue = new Queue('reports', { connection })

// Thêm job đơn giản
await emailQueue.add('send-welcome', {
  to: 'alice@example.com',
  userId: 42
})

// Thêm job với options
await emailQueue.add('send-order-confirmation', {
  orderId: 100,
  email: 'alice@example.com'
}, {
  attempts: 3,               // retry tối đa 3 lần
  backoff: {
    type: 'exponential',     // delay: 2s, 4s, 8s
    delay: 2000
  },
  delay: 5000,               // bắt đầu sau 5 giây
  priority: 1,               // ưu tiên cao (thấp = cao hơn)
  removeOnComplete: { count: 100 },  // giữ 100 completed jobs
  removeOnFail: { count: 50 }        // giữ 50 failed jobs
})

// Repeatable jobs (cron)
await reportQueue.add('weekly-report', { type: 'weekly' }, {
  repeat: { cron: '0 9 * * MON' }  // mỗi thứ 2 lúc 9:00 AM
})

// Job trong Express route
app.post('/users', async (req, res) => {
  const user = await prisma.user.create({ data: req.body })

  // Không đợi email — add to queue và return ngay
  await emailQueue.add('send-welcome', { userId: user.id, email: user.email })

  res.status(201).json(user)  // Response trong < 50ms
})
```

### Worker — Xử lý jobs

```js
const { Worker, QueueEvents } = require('bullmq')
const { sendEmail } = require('./email.service')

// Worker xử lý email queue
const emailWorker = new Worker('emails', async (job) => {
  const { userId, email, to } = job.data

  console.log(`Processing job ${job.id}: ${job.name}`)

  switch (job.name) {
    case 'send-welcome':
      await sendEmail({
        to: email,
        subject: 'Welcome to our platform!',
        template: 'welcome',
        data: { userId }
      })
      break

    case 'send-order-confirmation':
      await sendEmail({
        to: email,
        subject: `Order #${job.data.orderId} confirmed`,
        template: 'order-confirmation',
        data: { orderId: job.data.orderId }
      })
      break

    default:
      throw new Error(`Unknown job name: ${job.name}`)
  }

  // Return value được lưu trong job.returnvalue
  return { sentAt: new Date().toISOString() }

}, {
  connection: { host: 'localhost', port: 6379 },
  concurrency: 5  // xử lý 5 jobs cùng lúc
})

// Event listeners
emailWorker.on('completed', (job, result) => {
  console.log(`Job ${job.id} completed:`, result)
})

emailWorker.on('failed', (job, err) => {
  console.error(`Job ${job?.id} failed:`, err.message)
})

emailWorker.on('active', (job) => {
  console.log(`Job ${job.id} started`)
})

// Queue events (cross-process)
const queueEvents = new QueueEvents('emails', { connection: { host: 'localhost', port: 6379 } })
queueEvents.on('failed', ({ jobId, failedReason }) => {
  console.error(`Job ${jobId} failed: ${failedReason}`)
})
```

### BullBoard — Dashboard

```js
const { createBullBoard } = require('@bull-board/api')
const { BullMQAdapter } = require('@bull-board/api/bullMQAdapter')
const { ExpressAdapter } = require('@bull-board/express')

const serverAdapter = new ExpressAdapter()
serverAdapter.setBasePath('/admin/queues')

createBullBoard({
  queues: [
    new BullMQAdapter(emailQueue),
    new BullMQAdapter(imageQueue)
  ],
  serverAdapter
})

app.use('/admin/queues', serverAdapter.getRouter())
// Truy cập: http://localhost:3000/admin/queues
```

---

## RabbitMQ — Message Broker

RabbitMQ phù hợp cho complex routing scenarios — nhiều consumers, routing patterns, dead letter queues.

```bash
docker run -d \
  --name rabbitmq \
  -p 5672:5672 \
  -p 15672:15672 \
  rabbitmq:3-management-alpine

# Management UI: http://localhost:15672 (guest/guest)
```

```js
const amqp = require('amqplib')

// === Publisher ===
const conn = await amqp.connect(process.env.RABBITMQ_URL || 'amqp://localhost')
const channel = await conn.createChannel()

// Declare queue (idempotent)
await channel.assertQueue('tasks', {
  durable: true,     // survive broker restart
  arguments: {
    'x-dead-letter-exchange': 'tasks-dlx'  // dead letter queue
  }
})

// Send message
channel.sendToQueue(
  'tasks',
  Buffer.from(JSON.stringify({ type: 'send-email', payload: { to: 'a@b.com' } })),
  {
    persistent: true,          // survive broker restart
    contentType: 'application/json',
    messageId: crypto.randomUUID()
  }
)

// === Consumer ===
await channel.assertQueue('tasks', { durable: true })
channel.prefetch(1)  // xử lý 1 message tại một thời điểm

channel.consume('tasks', async (msg) => {
  if (!msg) return

  try {
    const { type, payload } = JSON.parse(msg.content.toString())

    if (type === 'send-email') {
      await sendEmail(payload)
    }

    channel.ack(msg)   // acknowledge — xóa khỏi queue
  } catch (err) {
    console.error('Failed to process:', err)
    // Nack — reject message, requeue = false (đẩy vào DLQ)
    channel.nack(msg, false, false)
  }
})
```

### Exchange Types

```js
// Direct exchange — routing key exact match
await channel.assertExchange('orders', 'direct')
channel.publish('orders', 'order.created', Buffer.from(JSON.stringify(order)))
channel.publish('orders', 'order.cancelled', Buffer.from(JSON.stringify(order)))

// Fanout exchange — broadcast tới tất cả queues
await channel.assertExchange('broadcasts', 'fanout')
channel.publish('broadcasts', '', Buffer.from(message))  // routing key bị ignore

// Topic exchange — pattern matching
await channel.assertExchange('events', 'topic')
channel.publish('events', 'user.created.admin', Buffer.from(data))

// Subscribe với pattern
await channel.bindQueue(queue, 'events', 'user.*')       // user.created, user.deleted
await channel.bindQueue(queue, 'events', '*.created.*')  // any created event
```

---

## Khi nào dùng gì?

| | BullMQ | RabbitMQ |
|--|--------|----------|
| Infrastructure | Redis (thường đã có) | Dedicated broker |
| Use case | Job queues, cron, retry | Complex routing, multiple consumers |
| Complexity | Thấp — dễ setup | Cao hơn — concepts nhiều hơn |
| Dashboard | BullBoard | Management UI built-in |
| Dùng khi | Single app, cần simple queues | Microservices, complex messaging patterns |

---

## Bài tập thực hành

1. **Email queue**: Implement registration flow — khi user register, push email job vào BullMQ queue, worker gửi email (console.log thay send thật). Thêm retry logic.

2. **Cron jobs**: Dùng BullMQ repeatable jobs để implement: daily digest email, weekly report generation, cleanup expired tokens mỗi giờ.

3. **Dead letter queue**: Implement DLQ với RabbitMQ — messages fail sau 3 retries được move vào dead letter queue, implement separate consumer để inspect và manually retry.

4. **Job monitoring**: Setup BullBoard dashboard, implement webhook notification khi job fail (gọi Slack webhook hoặc log ra file).

---

## Resources

- [BullMQ docs](https://docs.bullmq.io) — Official BullMQ documentation
- [RabbitMQ tutorials](https://www.rabbitmq.com/tutorials) — RabbitMQ guides
- [amqplib docs](https://amqp-node.github.io/amqplib/) — Node.js AMQP library
