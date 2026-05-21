# 🧵 Threadly — Social Media Backend

A production-ready RESTful API backend for **Threadly**, a Threads-inspired social media platform. Built with **Node.js**, **Express 5**, and **MongoDB Atlas**, it handles authentication, posts, comments, messaging, and notifications — all deployable to **Vercel** or any Node.js hosting provider.

---

## 🌐 Live Deployment

| Service | URL |
|---------|-----|
| **Backend API** | `https://thread-clone-backend-1-zwlq.onrender.com` |
| **Frontend** | `https://thread-clone-frontend-green.vercel.app` |

---

## 🚀 Tech Stack

| Category | Technology |
|----------|-----------|
| **Runtime** | Node.js (ES Modules) |
| **Framework** | Express 5.x |
| **Database** | MongoDB Atlas via Mongoose 9.x |
| **Authentication** | JSON Web Tokens (JWT) + bcryptjs |
| **Image Storage** | Cloudinary + Multer |
| **Session Handling** | Cookie-Parser (httpOnly JWT cookie) |
| **CORS** | Dynamic origin — credentials enabled |
| **Deployment** | Vercel (`@vercel/node`) |

---

## 📁 Project Structure

```
Social-Media-Backend/
│
├── server.js                  # App entry point — mounts all routes & DB connection
├── vercel.json                # Vercel deployment config (rewrites all routes → server.js)
├── package.json               # Dependencies & scripts
├── .env                       # Environment variables (not committed)
├── request.http               # HTTP request samples for manual API testing
│
├── Routes/
│   ├── authRoutes.js          # /api/auth-api — register, login, logout, me
│   ├── userRoutes.js          # /api/user-api — profile, follow/unfollow, search
│   ├── postRoutes.js          # /api/post-api — CRUD posts, likes, bookmarks, comments
│   ├── commentRoutes.js       # /api/comment-api — standalone comment operations
│   ├── messageRoute.js        # /api/messages — direct messaging conversations
│   └── notificationRoute.js   # /api/notifications — fetch & mark-read notifications
│
├── Controllers/
│   ├── messageController.js   # Business logic for messaging
│   └── notificationController.js  # Business logic for notifications
│
├── Middleware/
│   └── authMiddleware.js      # JWT verification (cookie + Bearer header fallback)
│
├── Models/
│   ├── userModel.js           # User schema
│   ├── postModel.js           # Post schema (with embedded comments & replies)
│   ├── commentModel.js        # Standalone Comment schema
│   ├── messageModel.js        # Direct Message schema
│   └── notificationModel.js   # Notification schema
│
├── Services/
│   └── authService.js         # register() & authentication() helper functions
│
├── config/
│   └── cloudinary.js          # Cloudinary + multer-storage-cloudinary setup
│
└── uploads/                   # Temporary local upload buffer (for fallback)
```

---

## ⚙️ Environment Variables

Create a `.env` file in the project root. **Never commit this file.**

```env
# Server
PORT=5000

# MongoDB Atlas connection string
DB_URL=mongodb+srv://<username>:<password>@<cluster>.mongodb.net/<dbname>?retryWrites=true&w=majority

# JSON Web Token secret (use a long random string)
JWT_SECRET=your_super_secret_jwt_key

# Cloudinary credentials (from cloudinary.com dashboard)
CLOUD_NAME=your_cloudinary_cloud_name
API_KEY=your_cloudinary_api_key
API_SECRET=your_cloudinary_api_secret
```

| Variable | Description |
|----------|-------------|
| `PORT` | Port the server listens on locally (default: `5000`) |
| `DB_URL` | MongoDB Atlas connection URI |
| `JWT_SECRET` | Secret key used to sign and verify JWT tokens |
| `CLOUD_NAME` | Your Cloudinary cloud name |
| `API_KEY` | Cloudinary API key |
| `API_SECRET` | Cloudinary API secret |

---

## 🛠️ Local Setup & Installation

### Prerequisites

- Node.js v18+
- npm v9+
- A [MongoDB Atlas](https://www.mongodb.com/cloud/atlas) cluster
- A [Cloudinary](https://cloudinary.com/) account

### Steps

```bash
# 1. Clone the repository
git clone https://github.com/Deepesh-7879/Thread-Clone_Frontend.git
cd Thread-Clone_Frontend/Backend/Social-Media-Backend

# 2. Install dependencies
npm install

# 3. Create your environment file
cp .env.example .env
# Then fill in your values in .env

# 4. Start the development server (with auto-restart)
npx nodemon server.js

# OR start without auto-restart
npm start
```

The server will start at: `http://localhost:5000`

---

## 📡 API Reference

All endpoints are prefixed under the base URL (e.g., `http://localhost:5000`).

> 🔒 **Protected routes** require a valid JWT, sent either as:
> - An **httpOnly cookie** named `token`, or
> - An **`Authorization: Bearer <token>`** request header.

---

### 🔐 Auth — `/api/auth-api`

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `POST` | `/api/auth-api/register` | ❌ | Register a new user |
| `POST` | `/api/auth-api/login` | ❌ | Login and receive JWT cookie + token |
| `GET` | `/api/auth-api/me` | 🔒 | Get the currently authenticated user |
| `GET` | `/api/auth-api/logout` | ❌ | Clear the auth cookie and log out |

#### `POST /api/auth-api/register` — Request Body
```json
{
  "name": "John Doe",
  "username": "johndoe",
  "email": "john@example.com",
  "password": "securePassword123"
}
```

#### `POST /api/auth-api/login` — Request Body
```json
{
  "email": "john@example.com",
  "password": "securePassword123"
}
```

#### Login Response
```json
{
  "message": "user login success",
  "user": { "_id": "...", "username": "johndoe", "email": "john@example.com", ... },
  "token": "eyJhbGciOiJIUzI1NiIsInR5..."
}
```

---

### 👤 Users — `/api/user-api`

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `GET` | `/api/user-api/me` | 🔒 | Get current user (with followers/following populated) |
| `GET` | `/api/user-api/search?q=term` | ❌ | Search users by name, username, or bio |
| `GET` | `/api/user-api/suggested` | ❌ | Get up to 10 suggested users |
| `GET` | `/api/user-api/:username` | ❌ | Get user profile + their posts by username |
| `PUT` | `/api/user-api/profile` | 🔒 | Update name and/or bio |
| `POST` | `/api/user-api/profile-picture` | 🔒 | Upload a profile picture (multipart `image` field) |
| `POST` | `/api/user-api/:userId/follow` | 🔒 | Follow a user (auto-creates a follow notification) |
| `POST` | `/api/user-api/:userId/unfollow` | 🔒 | Unfollow a user |
| `GET` | `/api/user-api/:userId/followers` | ❌ | Get a user's followers list |
| `GET` | `/api/user-api/:userId/following` | ❌ | Get a user's following list |
| `PATCH` | `/api/user-api/soft-delete/:userId` | ❌ | Soft-deactivate a user account (`isActive = false`) |

> **Aliases**: `/api/users` also maps to the same user router for legacy compatibility.

---

### 📝 Posts — `/api/post-api`

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `POST` | `/api/post-api/` | 🔒 | Create a new post (supports up to 4 image uploads) |
| `GET` | `/api/post-api/feed` | 🔒 | Get personalized feed (own + following posts) |
| `GET` | `/api/post-api/all` | ❌ | Get all posts (public explore feed) |
| `GET` | `/api/post-api/user/:userId` | ❌ | Get all posts by a specific user |
| `GET` | `/api/post-api/:postId` | ❌ | Get a single post by ID |
| `POST` | `/api/post-api/:postId/like` | 🔒 | Toggle like/unlike on a post (auto-creates like notification) |
| `POST` | `/api/post-api/:postId/share` | 🔒 | Toggle share/unshare on a post |
| `POST` | `/api/post-api/:postId/bookmark` | 🔒 | Toggle bookmark on a post |
| `POST` | `/api/post-api/:postId/comment` | 🔒 | Add a comment to a post (auto-creates comment notification) |
| `POST` | `/api/post-api/:postId/comment/:commentId/reply` | 🔒 | Add a reply to a comment (auto-creates reply notification) |
| `DELETE` | `/api/post-api/:postId` | 🔒 | Delete a post (only the author can delete) |

> **Aliases**: `/api/posts` also maps to the same post router for legacy compatibility.

#### `POST /api/post-api/` — Request (multipart/form-data)
```
content: "Hello world!"
images: <file> (optional, up to 4 files, max 5MB each, jpeg/jpg/png/gif/webp)
```

---

### 💬 Comments — `/api/comment-api`

> Note: Posts also embed comments directly. This route handles **standalone** comment documents (legacy collection).

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `POST` | `/api/comment-api/add-comment` | ❌ | Add a new comment to a post |
| `GET` | `/api/comment-api/post/:postId` | ❌ | Get all comments for a specific post |
| `POST` | `/api/comment-api/likes/:commentId` | 🔒 | Toggle like/unlike on a comment |
| `DELETE` | `/api/comment-api/:commentId` | ❌ | Delete a comment by ID |

#### `POST /api/comment-api/add-comment` — Request Body
```json
{
  "post": "<postId>",
  "user": "<userId>",
  "text": "This is my comment!"
}
```

---

### 📨 Messages — `/api/messages`

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `GET` | `/api/messages/conversations` | 🔒 | Get all conversations for the current user |
| `POST` | `/api/messages/` | 🔒 | Send a direct message (auto-creates message notification) |
| `PUT` | `/api/messages/:otherUserId/read` | 🔒 | Mark all messages from a user as read |

#### `POST /api/messages/` — Request Body
```json
{
  "receiverId": "<userId>",
  "text": "Hey there!"
}
```

---

### 🔔 Notifications — `/api/notifications`

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `GET` | `/api/notifications/` | 🔒 | Get all notifications for the current user (latest 50) |
| `PUT` | `/api/notifications/read` | 🔒 | Mark all unread notifications as read |

Notification types: `like` · `follow` · `comment` · `reply` · `message`

---

## 🗃️ Database Schemas

### User
```
_id         ObjectId
name        String
username    String (unique, required)
email       String (unique, required)
password    String (hashed with bcrypt, required)
bio         String (default: "")
profileImage String (Cloudinary URL, default: "")
isActive    Boolean (default: true)
followers   [ObjectId → User]
following   [ObjectId → User]
createdAt   Date (auto)
updatedAt   Date (auto)
```

### Post
```
_id       ObjectId
author    ObjectId → User
content   String (required)
image     String (Cloudinary URL, optional)
likes     [ObjectId → User]
shares    [ObjectId → User]
bookmarks [ObjectId → User]
comments  [{
  user      ObjectId → User
  content   String (required)
  createdAt Date
  replies   [{
    user      ObjectId → User
    content   String (required)
    createdAt Date
  }]
}]
createdAt Date (auto)
updatedAt Date (auto)
```

### Comment *(standalone collection)*
```
_id       ObjectId
post      ObjectId → Post (required)
user      ObjectId → User (required)
text      String (required)
createdAt Date (auto)
updatedAt Date (auto)
```

### Message
```
_id        ObjectId
senderId   ObjectId → User (required)
receiverId ObjectId → User (required)
text       String (required)
read       Boolean (default: false)
createdAt  Date (auto)
updatedAt  Date (auto)
```

### Notification
```
_id        ObjectId
receiverId ObjectId → User (required)
senderId   ObjectId → User (required)
type       String — enum: ["like", "follow", "comment", "message"] (required)
read       Boolean (default: false)
postId     ObjectId → Post (optional, for like/comment notifications)
text       String (optional, first 50 chars of comment/message)
createdAt  Date (auto)
updatedAt  Date (auto)
```

---

## 🔑 Authentication Flow

```
Client                        Server
  │                              │
  │── POST /api/auth-api/login ──►│
  │                              │── bcrypt.compare(password, hash)
  │                              │── jwt.sign({ userId, email, role })
  │◄─ Set-Cookie: token=<jwt> ───│
  │◄─ { user, token } ───────────│
  │                              │
  │── GET /api/post-api/feed ────►│  (cookie sent automatically)
  │                              │── authMiddleware: jwt.verify(cookie.token)
  │                              │── req.userId = decoded.userId
  │◄─ [posts array] ─────────────│
```

- Token is signed with `JWT_SECRET`, expires in **1 hour**
- Stored as an **httpOnly** cookie (`sameSite: lax`, `secure: false` in dev)
- Middleware also accepts `Authorization: Bearer <token>` header as fallback (for cross-origin deployments)

---

## 📦 Image Upload

Images are handled via **Multer** + **Cloudinary**:

- **Max file size**: 5 MB per file
- **Allowed formats**: `jpeg`, `jpg`, `png`, `gif`, `webp`
- **Storage**: Uploaded directly to Cloudinary; the returned URL is stored in MongoDB
- **Endpoints that accept images**:
  - `POST /api/post-api/` — field name: `images` (up to 4)
  - `POST /api/user-api/profile-picture` — field name: `image` (1 file)

---

## 🚀 Deployment

### Vercel

The `vercel.json` at the project root routes all incoming traffic to `server.js`:

```json
{
  "version": 2,
  "builds": [{ "src": "server.js", "use": "@vercel/node" }],
  "routes": [{ "src": "/(.*)", "dest": "server.js" }]
}
```

Add all `.env` variables as **Environment Variables** in your Vercel project dashboard before deploying.

```bash
# Deploy to Vercel
npx vercel --prod
```

### Render / Railway / Any Node Host

```bash
npm start
# Runs: node server.js
```

Set all `.env` variables in the hosting platform's environment settings.

---

## 🧪 Manual API Testing

A `request.http` file is included in the root for use with the [REST Client VS Code extension](https://marketplace.visualstudio.com/items?itemName=humao.rest-client). It contains pre-written sample requests for all major endpoints.

---

## 🛡️ Error Handling

The global error handler in `server.js` covers:

| Condition | Status | Response |
|-----------|--------|----------|
| Mongoose `ValidationError` | `400` | Field-level validation message |
| Mongoose `CastError` | `400` | Invalid ID format message |
| MongoDB duplicate key (`code 11000`) | `409` | `"<field> <value> already exists"` |
| Custom errors with `.status` | matches `.status` | Custom error message |
| All other errors | `500` | Generic server error |

---

## 📜 Available Scripts

```bash
npm start          # Start with Node (production)
npx nodemon server.js  # Start with auto-reload (development)
```

---

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/your-feature`
3. Commit your changes: `git commit -m "feat: add your feature"`
4. Push to the branch: `git push origin feature/your-feature`
5. Open a Pull Request

---

## 📄 License

ISC License © 2024 Deepesh
