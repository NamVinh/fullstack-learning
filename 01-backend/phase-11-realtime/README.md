# Phase 11 — Real-time Communication

Real-time features cho phép server push data đến client ngay khi có changes — không cần client polling. Node.js với event-driven architecture rất phù hợp cho real-time. Hai approach chính: WebSockets (bidirectional) và Server-Sent Events (server → client only).

---

## WebSocket vs SSE vs Polling

| | WebSocket | SSE | Long Polling |
|--|-----------|-----|-------------|
| Direction | Bidirectional | Server → Client | Server → Client |
| Protocol | WS/WSS | HTTP/HTTPS | HTTP/HTTPS |
| Reconnect | Manual | Automatic (built-in) | Manual |
| Browser support | Excellent | Good (no IE) | Universal |
| Overhead | Low (after handshake) | Low | High |
| Dùng khi | Chat, games, collab | Notifications, feeds | Legacy support |

---

## WebSockets với Socket.io

Socket.io là library phổ biến nhất cho WebSockets — tự động fallback sang polling, hỗ trợ rooms, namespaces, và acknowledgements.

```bash
npm install socket.io          # server
npm install socket.io-client   # client (optional, có thể dùng browser native)
```

### Server Setup

```js
const express = require('express')
const { createServer } = require('http')
const { Server } = require('socket.io')

const app = express()
const httpServer = createServer(app)

const io = new Server(httpServer, {
  cors: {
    origin: process.env.CLIENT_URL || 'http://localhost:3000',
    methods: ['GET', 'POST']
  },
  pingTimeout: 60000,    // disconnect sau 60s không ping
  pingInterval: 25000    // ping mỗi 25s
})

// Authentication middleware cho socket
io.use(async (socket, next) => {
  const token = socket.handshake.auth.token
  if (!token) return next(new Error('No token'))

  try {
    const payload = jwt.verify(token, process.env.JWT_SECRET)
    socket.userId = payload.userId
    socket.user = await db.user.findById(payload.userId)
    next()
  } catch {
    next(new Error('Invalid token'))
  }
})

// Connection handler
io.on('connection', (socket) => {
  console.log(`User ${socket.userId} connected: ${socket.id}`)

  // Join personal room (for direct messages)
  socket.join(`user:${socket.userId}`)

  // === Room Management ===
  socket.on('join-room', async (roomId, callback) => {
    const canJoin = await checkRoomAccess(socket.userId, roomId)
    if (!canJoin) return callback({ error: 'Access denied' })

    socket.join(roomId)
    socket.to(roomId).emit('user-joined', {
      userId: socket.userId,
      name: socket.user.name
    })
    callback({ success: true })
  })

  socket.on('leave-room', (roomId) => {
    socket.leave(roomId)
    socket.to(roomId).emit('user-left', { userId: socket.userId })
  })

  // === Messaging ===
  socket.on('send-message', async ({ roomId, text }, callback) => {
    if (!text?.trim()) return callback({ error: 'Empty message' })

    try {
      // Save to DB
      const message = await db.message.create({
        data: {
          roomId,
          authorId: socket.userId,
          text: text.trim()
        }
      })

      // Broadcast to room (including sender)
      io.to(roomId).emit('new-message', {
        id: message.id,
        text: message.text,
        author: { id: socket.userId, name: socket.user.name },
        createdAt: message.createdAt
      })

      callback({ success: true, messageId: message.id })
    } catch (err) {
      callback({ error: 'Failed to send' })
    }
  })

  // === Typing Indicators ===
  socket.on('typing-start', (roomId) => {
    socket.to(roomId).emit('user-typing', { userId: socket.userId, name: socket.user.name })
  })

  socket.on('typing-stop', (roomId) => {
    socket.to(roomId).emit('user-stop-typing', { userId: socket.userId })
  })

  // === Disconnect ===
  socket.on('disconnect', (reason) => {
    console.log(`User ${socket.userId} disconnected: ${reason}`)
    // Broadcast to all rooms this user was in
    socket.rooms.forEach(room => {
      if (room !== socket.id) {
        socket.to(room).emit('user-left', { userId: socket.userId })
      }
    })
  })
})

httpServer.listen(3000)
```

### Client (Browser)

```html
<script src="/socket.io/socket.io.js"></script>
<script>
const socket = io({
  auth: { token: localStorage.getItem('accessToken') }
})

socket.on('connect', () => {
  console.log('Connected:', socket.id)

  // Join a room
  socket.emit('join-room', 'general', (response) => {
    if (response.error) console.error(response.error)
    else console.log('Joined room')
  })
})

socket.on('new-message', ({ text, author, createdAt }) => {
  appendMessage(text, author.name, createdAt)
})

socket.on('user-typing', ({ name }) => {
  showTypingIndicator(name)
})

// Send message
function sendMessage(text) {
  socket.emit('send-message', { roomId: 'general', text }, (response) => {
    if (response.error) showError(response.error)
  })
}

socket.on('disconnect', () => console.log('Disconnected'))
socket.on('connect_error', (err) => console.error('Connection error:', err.message))
</script>
```

### Scale với Redis Adapter

```js
const { createAdapter } = require('@socket.io/redis-adapter')
const { createClient } = require('redis')

const pubClient = createClient({ url: process.env.REDIS_URL })
const subClient = pubClient.duplicate()

await Promise.all([pubClient.connect(), subClient.connect()])

io.adapter(createAdapter(pubClient, subClient))
// Bây giờ nhiều Node.js instances có thể share socket.io state
```

---

## Server-Sent Events (SSE)

SSE đơn giản hơn WebSockets — chỉ server → client, nhưng auto-reconnect và HTTP native.

### Server

```js
// Express SSE endpoint
app.get('/api/events', authenticate, (req, res) => {
  // SSE headers
  res.setHeader('Content-Type', 'text/event-stream')
  res.setHeader('Cache-Control', 'no-cache')
  res.setHeader('Connection', 'keep-alive')
  res.setHeader('X-Accel-Buffering', 'no')  // Disable Nginx buffering
  res.flushHeaders()

  // Send helper
  const sendEvent = (event, data) => {
    res.write(`event: ${event}\n`)
    res.write(`data: ${JSON.stringify(data)}\n`)
    res.write(`id: ${Date.now()}\n\n`)  // ID cho reconnection
  }

  // Send initial data
  sendEvent('connected', { userId: req.user.userId, timestamp: Date.now() })

  // Subscribe to user's events
  const unsubscribe = eventBus.subscribe(`user:${req.user.userId}`, (event) => {
    sendEvent(event.type, event.data)
  })

  // Keepalive ping mỗi 30s (prevent proxy timeout)
  const keepAlive = setInterval(() => {
    res.write(': keepalive\n\n')  // Comment line (ignored by client)
  }, 30000)

  // Cleanup khi client disconnect
  req.on('close', () => {
    clearInterval(keepAlive)
    unsubscribe()
    console.log(`SSE client disconnected: ${req.user.userId}`)
  })
})

// Trigger events từ other routes
app.post('/posts', authenticate, async (req, res) => {
  const post = await db.post.create({ data: { ...req.body, authorId: req.user.userId } })

  // Notify followers in real-time
  const followers = await db.follows.findMany({ where: { followingId: req.user.userId } })
  followers.forEach(follow => {
    eventBus.publish(`user:${follow.followerId}`, {
      type: 'new-post',
      data: { post, author: { id: req.user.userId } }
    })
  })

  res.status(201).json(post)
})
```

### Client (Browser)

```js
// Browser native EventSource — no library needed
const eventSource = new EventSource('/api/events', {
  withCredentials: true  // send cookies
})

eventSource.onopen = () => console.log('SSE connected')
eventSource.onerror = (e) => console.error('SSE error:', e)

// Listen to specific event types
eventSource.addEventListener('new-post', (e) => {
  const { post, author } = JSON.parse(e.data)
  displayNewPost(post, author)
})

eventSource.addEventListener('notification', (e) => {
  const notif = JSON.parse(e.data)
  showNotification(notif)
})

// Cleanup
function disconnect() {
  eventSource.close()
}
```

---

## Ví dụ thực tế: Live Notifications

```js
// Simple in-process event bus
class EventBus extends EventEmitter {
  publish(channel, event) {
    this.emit(channel, event)
  }

  subscribe(channel, handler) {
    this.on(channel, handler)
    return () => this.off(channel, handler)  // unsubscribe function
  }
}

const eventBus = new EventBus()
module.exports = eventBus

// Trong route handler
const eventBus = require('./event-bus')

app.post('/comments', authenticate, async (req, res) => {
  const comment = await db.comment.create({
    data: { ...req.body, authorId: req.user.userId }
  })

  const post = await db.post.findById(comment.postId)

  // Notify post author
  if (post.authorId !== req.user.userId) {
    eventBus.publish(`user:${post.authorId}`, {
      type: 'new-comment',
      data: {
        comment,
        commenter: { id: req.user.userId, name: req.user.name },
        post: { id: post.id, title: post.title }
      }
    })
  }

  res.status(201).json(comment)
})
```

---

## Bài tập thực hành

1. **Chat app**: Build real-time chat với Socket.io — rooms, typing indicators, message history (load từ DB khi join room), online users list.

2. **Live notifications**: SSE endpoint cho notifications — khi user được tag trong comment, nhận notification ngay lập tức. Persist vào DB và mark as read.

3. **Real-time dashboard**: Admin dashboard hiển thị live metrics — active users, new orders per minute, error rate. Dùng SSE để push updates mỗi 5 giây.

4. **Socket.io scaling**: Setup 2 instances của app với Redis adapter, verify rằng message từ instance 1 reach clients ở instance 2.

---

## Resources

- [Socket.io docs](https://socket.io/docs/v4/) — Official documentation
- [MDN — Server-Sent Events](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events) — SSE API reference
- [WebSocket API — MDN](https://developer.mozilla.org/en-US/docs/Web/API/WebSocket) — Native WebSocket
