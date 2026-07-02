# Phase 6.2 — MongoDB + Mongoose

MongoDB là document database — lưu data dưới dạng JSON-like documents (BSON). Mongoose là **ODM** (Object Document Mapper) — KHÔNG phải ORM, vì MongoDB không phải relational database. Mongoose thêm schema validation, middleware, và helper methods lên MongoDB driver.

---

## MongoDB trong Docker (dev)

```bash
docker run -d \
  --name mongodb \
  -p 27017:27017 \
  -e MONGO_INITDB_ROOT_USERNAME=admin \
  -e MONGO_INITDB_ROOT_PASSWORD=password \
  mongo:7-jammy

# Connection string
MONGODB_URI=mongodb://admin:password@localhost:27017/mydb?authSource=admin
```

---

## Mongoose Setup và Connection

```js
const mongoose = require('mongoose')

async function connectDB() {
  try {
    await mongoose.connect(process.env.MONGODB_URI, {
      maxPoolSize: 10,  // connection pool size
    })
    console.log('MongoDB connected')
  } catch (err) {
    console.error('MongoDB connection error:', err)
    process.exit(1)
  }
}

// Graceful disconnect
async function disconnectDB() {
  await mongoose.disconnect()
}

// Connection events
mongoose.connection.on('error', (err) => console.error('MongoDB error:', err))
mongoose.connection.on('disconnected', () => console.log('MongoDB disconnected'))
```

---

## Schema Definition

```js
const { Schema, model } = require('mongoose')

// User Schema
const userSchema = new Schema({
  email: {
    type: String,
    required: [true, 'Email is required'],
    unique: true,
    lowercase: true,
    trim: true,
    match: [/^\S+@\S+\.\S+$/, 'Invalid email format']
  },
  name: {
    type: String,
    required: true,
    minLength: [1, 'Name too short'],
    maxLength: [100, 'Name too long'],
    trim: true
  },
  password: {
    type: String,
    required: true,
    select: false  // không include trong queries mặc định
  },
  age: {
    type: Number,
    min: [0, 'Age must be >= 0'],
    max: [150, 'Age must be <= 150']
  },
  role: {
    type: String,
    enum: {
      values: ['user', 'admin', 'moderator'],
      message: '{VALUE} is not a valid role'
    },
    default: 'user'
  },
  avatar: String,
  isActive: {
    type: Boolean,
    default: true
  }
}, {
  timestamps: true,           // tự thêm createdAt và updatedAt
  toJSON: { virtuals: true }, // include virtuals khi convert sang JSON
  toObject: { virtuals: true }
})

// Index
userSchema.index({ email: 1 })        // single field
userSchema.index({ role: 1, isActive: 1 })  // compound

// Virtual property — không lưu vào DB
userSchema.virtual('fullProfile').get(function() {
  return `${this.name} <${this.email}>`
})

// Model methods
userSchema.methods.comparePassword = async function(plaintext) {
  return bcrypt.compare(plaintext, this.password)
}

// Static methods
userSchema.statics.findByEmail = function(email) {
  return this.findOne({ email: email.toLowerCase() })
}

// Pre-save middleware
userSchema.pre('save', async function(next) {
  if (!this.isModified('password')) return next()
  this.password = await bcrypt.hash(this.password, 12)
  next()
})

// Post-save middleware
userSchema.post('save', function(doc, next) {
  console.log(`User saved: ${doc.id}`)
  next()
})

const User = model('User', userSchema)
module.exports = User
```

---

## Mongoose Validators

Các validators có sẵn:
- **Tất cả SchemaTypes**: `required`, `default`, `validate`
- **Number**: `min`, `max`
- **String**: `enum`, `match` (regex), `maxLength`, `minLength`, `lowercase`, `uppercase`, `trim`

```js
// Custom validator
const postSchema = new Schema({
  tags: {
    type: [String],
    validate: {
      validator: (arr) => arr.length <= 10,
      message: 'Too many tags (max 10)'
    }
  }
})
```

---

## CRUD Operations

```js
// === CREATE ===
const user = await User.create({
  email: 'alice@example.com',
  name: 'Alice',
  password: 'password123'
})

// Hoặc
const user = new User({ email, name, password })
await user.save()

// Bulk create
await User.insertMany([user1, user2, user3])

// === READ ===
// findById
const user = await User.findById(id)
if (!user) throw new NotFoundError('User')

// find với filters
const users = await User.find({
  isActive: true,
  age: { $gte: 18, $lte: 65 }  // MongoDB operators
})
  .sort({ createdAt: -1 })  // hoặc '-createdAt'
  .limit(10)
  .skip(0)
  .select('name email role')  // project fields
  .lean()  // trả về plain JS object thay vì Mongoose document (nhanh hơn)

// findOne
const user = await User.findOne({ email: 'alice@example.com' })

// Count
const total = await User.countDocuments({ isActive: true })

// Exists
const exists = await User.exists({ email })  // trả về null hoặc { _id }

// === UPDATE ===
// findByIdAndUpdate — trả về document MỚI nếu có { new: true }
const updated = await User.findByIdAndUpdate(
  id,
  { $set: { name: 'Bob' }, $inc: { loginCount: 1 } },  // MongoDB update operators
  { new: true, runValidators: true }
)

// updateMany
const result = await User.updateMany(
  { role: 'user', isActive: false },
  { $set: { deletedAt: new Date() } }
)
console.log(result.modifiedCount)

// === DELETE ===
const deleted = await User.findByIdAndDelete(id)
await User.deleteMany({ isActive: false, createdAt: { $lt: cutoffDate } })
```

---

## References và Populate

Mongoose `populate()` thay thế ObjectId reference bằng document thật — tương tự JOIN nhưng làm nhiều queries:

```js
const postSchema = new Schema({
  title: { type: String, required: true },
  content: String,
  author: {
    type: Schema.Types.ObjectId,
    ref: 'User',
    required: true
  },
  tags: [{ type: Schema.Types.ObjectId, ref: 'Tag' }],
  comments: [{
    user: { type: Schema.Types.ObjectId, ref: 'User' },
    text: String,
    createdAt: { type: Date, default: Date.now }
  }]
})

const Post = model('Post', postSchema)

// Populate single ref
const post = await Post.findById(id)
  .populate('author', 'name email avatar')  // chỉ lấy name, email, avatar
  .populate('tags', 'name')

// Populate nested
const post = await Post.findById(id)
  .populate({
    path: 'author',
    select: 'name email',
    populate: { path: 'profile', select: 'bio avatar' }  // nested populate
  })

// Populate comments.user
const post = await Post.findById(id)
  .populate('comments.user', 'name avatar')
```

---

## Aggregation Pipeline

Aggregation cho phép complex data transformations:

```js
// Đếm posts per user, chỉ lấy users có > 5 posts
const stats = await Post.aggregate([
  { $match: { published: true } },              // filter
  { $group: {
    _id: '$author',
    postCount: { $sum: 1 },
    avgLength: { $avg: { $strLenCP: '$content' } }
  }},
  { $match: { postCount: { $gt: 5 } } },        // filter sau group
  { $sort: { postCount: -1 } },                 // sort
  { $limit: 10 },
  { $lookup: {                                   // join với users collection
    from: 'users',
    localField: '_id',
    foreignField: '_id',
    as: 'author'
  }},
  { $unwind: '$author' },
  { $project: {                                  // shape output
    'author.name': 1,
    'author.email': 1,
    postCount: 1,
    avgLength: { $round: ['$avgLength', 0] }
  }}
])
```

---

## Bài tập thực hành

1. **Blog API**: Implement blog với Users, Posts, Comments, Tags — Schema với validators, pre-save hooks, virtual properties, populate.

2. **Aggregation**: Viết aggregation pipeline: top 10 authors by post count, monthly post statistics, tag popularity ranking.

3. **Transactions**: MongoDB 4+ support multi-document transactions. Implement "transfer" giữa hai users trong transaction — rollback nếu lỗi.

4. **Schema migration**: Simulate schema evolution — thêm field mới, migrate existing documents, handle backward compatibility.

---

## Resources

- [Mongoose docs](https://mongoosejs.com/docs/) — Official Mongoose documentation
- [MongoDB docs — Aggregation](https://www.mongodb.com/docs/manual/aggregation/) — Aggregation pipeline
- [MongoDB docs — CRUD](https://www.mongodb.com/docs/manual/crud/) — CRUD operations reference
