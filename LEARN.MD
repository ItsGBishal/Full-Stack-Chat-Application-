<h1 align="center">✨ Full-Stack Chat App with Auth & Emails ✨</h1>

<p align="center">
  A production-ready, real-time messaging application built with Node.js, Express, Socket.io, MongoDB, React, Tailwind CSS, DaisyUI, and Zustand.
</p>

![Demo App](/frontend/public/screenshot-for-readme.png)

## 🚀 Highlights

- 🔐 **Custom JWT Authentication** (stored in secure HTTP-only cookies, no 3rd-party auth dependency)
- ⚡ **Real-time Messaging** via WebSockets (Socket.io)
- 🟢 **Online/Offline Presence Tracking** across active sessions
- 🔔 **Notification & Typing Audio Effects** (with toggle & persistence in LocalStorage)
- 📨 **Transactional Emails** (Welcome emails on signup via Resend)
- 🗂️ **Cloud Media Storage** (Image uploads via Cloudinary)
- 🧰 **REST API Architecture** with Node.js & Express
- 🧱 **MongoDB & Mongoose** data persistence with schemas & timestamps
- 🚦 **Security & API Rate-Limiting** powered by Arcjet (Shield, Bot Detection, Sliding Window)
- 🎨 **Modern Responsive UI** with React 19, Tailwind CSS & DaisyUI
- 🧠 **Global State Management** using Zustand
- ⚡ **Optimistic UI Updates** for instant chat message responsiveness
- 🚀 **Production Ready** with single-command build and SPA static serving

---

## 🧪 .env Setup

### Backend (`/backend/.env`)

```bash
PORT=3000
MONGO_URI=your_mongo_uri_here

NODE_ENV=development

JWT_SECRET=your_jwt_secret

RESEND_API_KEY=your_resend_api_key
EMAIL_FROM=your_email_from_address
EMAIL_FROM_NAME=your_email_from_name

CLIENT_URL=http://localhost:5173

CLOUDINARY_CLOUD_NAME=your_cloudinary_cloud_name
CLOUDINARY_API_KEY=your_cloudinary_api_key
CLOUDINARY_API_SECRET=your_cloudinary_api_secret

ARCJET_KEY=your_arcjet_key
ARCJET_ENV=development
```

---

## 🔧 Running Locally

### 1. Run the Backend

```bash
cd backend
npm install
npm run dev
```

### 2. Run the Frontend

```bash
cd frontend
npm install
npm run dev
```

---

# 🎓 Comprehensive Interview Preparation Guide

This section contains top technical interview questions and deep-dive answers specifically tailored to this project. Review these questions to master full-stack, real-time web application engineering concepts.

---

## Table of Contents
1. [Architecture & System Design](#1-architecture--system-design)
2. [Real-time Communication & WebSockets (Socket.io)](#2-real-time-communication--websockets-socketio)
3. [Authentication, Authorization & Security](#3-authentication-authorization--security)
4. [Database & Data Modeling (MongoDB / Mongoose)](#4-database--data-modeling-mongodb--mongoose)
5. [Frontend Engineering & State Management (React & Zustand)](#5-frontend-engineering--state-management-react--zustand)
6. [Cloud Media, Emailing & Third-Party APIs](#6-cloud-media-emailing--third-party-apis)
7. [Scalability, Performance & Production Bottlenecks](#7-scalability-performance--production-bottlenecks)
8. [Behavioral & Code Quality Questions](#8-behavioral--code-quality-questions)

---

## 1. Architecture & System Design

### Q1. Can you explain the high-level architecture of this application?
- **Answer:**
  The project follows a decoupled client-server architecture:
  - **Frontend (Client):** A Single Page Application (SPA) built with React and Vite. It utilizes **Zustand** for centralized state management (auth state, chat state, online users, active conversations) and Tailwind CSS / DaisyUI for component styling.
  - **Backend (Server):** Built on Node.js and Express. It exposes RESTful endpoints for CRUD operations (signup, login, fetching chat partners, fetching message histories) and attaches a **Socket.io** server on top of the native Node HTTP server for bidirectional, low-latency communication.
  - **Persistence & Cloud Services:**
    - **MongoDB (via Mongoose):** Persists user accounts and message histories.
    - **Cloudinary:** Hosts user avatars and shared chat images.
    - **Resend:** Dispatches transactional onboarding emails asynchronously.
    - **Arcjet:** Acts as an edge security middleware providing rate limiting (sliding window), bot detection, and attack protection.

### Q2. Why did you choose a hybrid REST + WebSocket design instead of doing everything over WebSockets?
- **Answer:**
  - **Separation of Concerns:** REST is ideal for standard stateless CRUD operations, authentication checks, initial history loads, and profile updates. These benefit from standard HTTP status codes (200, 400, 401, 404, 500), caching headers, standard error handlers, and easy debugging.
  - **Connection Overhead:** Establishing a WebSocket connection requires an initial HTTP handshake and maintains an open TCP socket. Using REST for authentication and onboarding guarantees the user is authenticated *before* connecting to the WebSocket server.
  - **Reliability & Resilience:** If a WebSocket drops or reconnects, REST APIs continue to work independently (e.g. browsing older chats, changing account settings).

---

## 2. Real-time Communication & WebSockets (Socket.io)

### Q3. How does 1-to-1 private messaging work in this app?
- **Answer:**
  1. The sender inputs a message and clicks Send.
  2. The client fires a POST request to `/api/messages/send/:receiverId`.
  3. The backend validates the request, uploads any attached image to Cloudinary, and saves the `Message` document to MongoDB.
  4. Once persisted, the server checks the in-memory map `userSocketMap[receiverId]` (defined in `socket.js`) to find the receiver's active `socket.id`.
  5. If the receiver is connected, `io.to(receiverSocketId).emit("newMessage", newMessage)` pushes the payload exclusively to that user's socket connection.
  6. The HTTP response also returns the saved message to the sender, allowing the sender's client to confirm delivery.

### Q4. How do you track and broadcast online/offline user status?
- **Answer:**
  - In `backend/src/lib/socket.js`, an in-memory dictionary `userSocketMap` maintains `{ [userId]: socketId }`.
  - When a user connects:
    - Their authenticated `userId` is extracted from the handshake session.
    - `userSocketMap[userId] = socket.id` is registered.
    - `io.emit("getOnlineUsers", Object.keys(userSocketMap))` broadcasts the list of all online IDs to every connected client.
  - When a user disconnects:
    - The `socket.on("disconnect")` event fires.
    - `delete userSocketMap[userId]` removes the entry.
    - `io.emit("getOnlineUsers", Object.keys(userSocketMap))` notifies remaining clients immediately.

### Q5. What is a major limitation of using an in-memory `userSocketMap` and how do you solve it in production?
- **Answer:**
  - **Multi-Tab / Multi-Device Issue:** If a user logs in from both a laptop and a phone, storing a single `socket.id` per `userId` causes one connection to overwrite the other. When one device disconnects, the user may appear offline even if the other device is still connected.
    - *Fix:* Store an array or Set of socket IDs: `userSocketMap = { [userId]: Set<string> }`.
  - **Horizontal Scaling Issue:** When scaling the backend horizontally across multiple server instances (Node cluster or multiple containers behind a load balancer), each instance has its own isolated memory. A message received on Server A cannot reach a receiver connected to Server B via a local dictionary.
    - *Fix:* Use the **Socket.io Redis Adapter** (`@socket.io/redis-adapter`) with Redis Pub/Sub so events emitted on any server instance are published to Redis and fanned out to the appropriate socket connection across the entire cluster.

### Q6. How do you authenticate WebSocket connections?
- **Answer:**
  - In `backend/src/middleware/socket.auth.middleware.js`, we use `io.use(socketAuthMiddleware)` before any connection event is accepted.
  - During the initial WebSocket HTTP handshake, browser cookies are sent in `socket.handshake.headers.cookie`.
  - The middleware parses the `jwt` cookie, verifies the token using `jwt.verify(token, ENV.JWT_SECRET)`, fetches the user from MongoDB (excluding password), and attaches `socket.user` and `socket.userId` to the socket instance.
  - If the token is missing, expired, or invalid, `next(new Error("Unauthorized"))` aborts the connection before any events can be transmitted.

### Q7. How does the frontend handle incoming messages and prevent duplicate listeners?
- **Answer:**
  - In `frontend/src/store/useChatStore.js`, `subscribeToMessages` binds a listener: `socket.on("newMessage", callback)`.
  - Inside the callback:
    - It verifies if `newMessage.senderId === selectedUser._id` to ensure messages are only appended to the active conversation window.
    - If `isSoundEnabled` is true, an audio element (`new Audio("/sounds/notification.mp3")`) plays an alert sound.
  - When switching chats or unmounting the chat component, `unsubscribeFromMessages` calls `socket.off("newMessage")` to prevent memory leaks and duplicate audio alerts or duplicate message appends.

---

## 3. Authentication, Authorization & Security

### Q8. Why did you store JWTs in HTTP-Only cookies instead of browser `localStorage`?
- **Answer:**
  - **XSS Protection:** Data stored in `localStorage` or `sessionStorage` is accessible to any JavaScript executing on the page (including malicious scripts injected via third-party libraries or compromised CDN bundles).
  - **`httpOnly: true`:** Prevents JavaScript from reading `document.cookie`, completely neutralizing token theft via Cross-Site Scripting (XSS).
  - **`sameSite: "strict"`:** Protects against Cross-Site Request Forgery (CSRF) by preventing the browser from attaching the cookie to cross-origin requests.
  - **`secure: true` (in production):** Guarantees the cookie is only transmitted over encrypted HTTPS connections.

### Q9. Explain the signup flow and password hashing mechanism.
- **Answer:**
  - Input validation checks required fields, minimum password length (6 characters), and email syntax via regex.
  - Duplicate user check: `User.findOne({ email })`.
  - Password Hashing with **bcryptjs**:
    - Generates a cryptographic salt: `const salt = await bcrypt.genSalt(10);`.
    - Hashes the plain text password: `const hashedPassword = await bcrypt.hash(password, salt);`.
    - Salting ensures that even if two users choose identical passwords, their resulting hashes are completely different, preventing rainbow table attacks.
  - **Code Review Note:** The user is saved to MongoDB *before* generating and setting the auth cookie so that if database persistence fails, no orphaned session token is issued to the client.

### Q10. What is Arcjet and how does it protect this application?
- **Answer:**
  - Arcjet (`@arcjet/node`) is a security layer implemented as an Express middleware (`backend/src/middleware/arcjet.middleware.js`).
  - It configures:
    1. **Shield (`shield({ mode: "LIVE" })`):** Protects against common web vulnerabilities like SQL injection, cross-site scripting attempts, and malicious payloads.
    2. **Bot Detection (`detectBot({ mode: "LIVE", allow: ["CATEGORY:SEARCH_ENGINE"] })`):** Blocks automated scrapers, credential-stuffing bots, and malicious crawlers while allowing legitimate search engines.
    3. **Sliding Window Rate Limiting (`slidingWindow({ max: 100, interval: 60 })`):** Limits clients to 100 requests per 60-second sliding window, preventing brute-force attacks and Denial of Service (DoS).

---

## 4. Database & Data Modeling (MongoDB / Mongoose)

### Q11. Explain your database schema design for `User` and `Message`.
- **Answer:**
  - **User Schema (`models/User.js`):**
    - `email`: String, required, unique, validated format.
    - `fullName`: String, required.
    - `password`: String, required (hashed).
    - `profilePic`: String (URL pointing to Cloudinary), default empty.
    - `timestamps: true`: Automatically manages `createdAt` and `updatedAt`.
  - **Message Schema (`models/Message.js`):**
    - `senderId`: `ObjectId`, ref: `'User'`, required.
    - `receiverId`: `ObjectId`, ref: `'User'`, required.
    - `text`: String, trimmed, max length 2000 characters.
    - `image`: String (Cloudinary URL).
    - `timestamps: true`: Records exact timestamp for message ordering.

### Q12. How do you query conversation history between two users?
- **Answer:**
  - In `getMessagesByUserId`:
    ```javascript
    const messages = await Message.find({
      $or: [
        { senderId: myId, receiverId: userToChatId },
        { senderId: userToChatId, receiverId: myId },
      ],
    });
    ```
  - This retrieves all messages exchanged between the two users regardless of who initiated or replied.

### Q13. What database indexes would you add to optimize this query in production?
- **Answer:**
  - Without an index, the `$or` query performs a full collection scan (`COLLSCAN`), which deteriorates rapidly as millions of messages accumulate.
  - We should create compound indexes:
    ```javascript
    messageSchema.index({ senderId: 1, receiverId: 1, createdAt: 1 });
    messageSchema.index({ receiverId: 1, senderId: 1, createdAt: 1 });
    ```
  - This allows MongoDB to perform index-assisted lookups (`IXSCAN`) and fetch messages in chronological order without an in-memory sort.

### Q14. How does `getChatPartners` find recent conversations? How could it be optimized?
- **Answer:**
  - **Current Implementation:**
    Queries all messages where `senderId` or `receiverId` equals the logged-in user, collects unique partner IDs in JavaScript using `new Set()`, and queries `User.find({ _id: { $in: chatPartnerIds } })`.
  - **Optimization (Aggregation Pipeline):**
    Using a MongoDB `$facet` or `$group` pipeline at the database level eliminates fetching every raw message across the wire into Node memory:
    ```javascript
    Message.aggregate([
      { $match: { $or: [{ senderId: myId }, { receiverId: myId }] } },
      { $sort: { createdAt: -1 } },
      {
        $group: {
          _id: {
            $cond: [{ $eq: ["$senderId", myId] }, "$receiverId", "$senderId"],
          },
          lastMessage: { $first: "$$ROOT" },
        },
      },
      {
        $lookup: {
          from: "users",
          localField: "_id",
          foreignField: "_id",
          as: "partnerDetails",
        },
      },
    ]);
    ```

### Q15. How would you implement pagination for chat messages?
- **Answer:**
  - **Avoid Offset Pagination (`skip(offset).limit(limit)`):** Skipping large offsets causes performance degradation and causes duplicated or missed messages if new messages arrive while scrolling.
  - **Use Cursor-based Pagination:**
    - Query using the timestamp or `_id` of the oldest loaded message:
      `Message.find({ ...query, createdAt: { $lt: oldestMessageTimestamp } }).sort({ createdAt: -1 }).limit(30)`.
    - As the user scrolls up, fetch the next 30 older messages and prepend them to the chat window.

---

## 5. Frontend Engineering & State Management (React & Zustand)

### Q16. Why use Zustand over Redux Toolkit or React Context?
- **Answer:**
  - **Simplicity & Zero Boilerplate:** No reducers, action creators, or complex slice configurations; state and asynchronous actions coexist in a single hook (`useChatStore`, `useAuthStore`).
  - **No Context Re-render Cascades:** React Context re-renders all consuming components whenever any value in the context changes. Zustand allows granular selectors (`const authUser = useAuthStore(state => state.authUser)`), ensuring components only re-render when their selected slice changes.
  - **Outside React Tree Access:** Zustand stores can be accessed and manipulated outside React components (e.g. `useAuthStore.getState().socket`), which is crucial for socket listeners, interceptors, and background helpers.

### Q17. What is Optimistic UI and how is it implemented in this project?
- **Answer:**
  - **Concept:** Optimistic UI immediately updates the client interface assuming the backend request will succeed, providing instant feedback without waiting for server network round-trips.
  - **Implementation (`useChatStore.js:sendMessage`):**
    1. Creates a temporary message object with a temporary ID (`temp-${Date.now()}`), sender/receiver IDs, current timestamp, and text/image.
    2. Instantly updates state: `set({ messages: [...messages, optimisticMessage] })`. The sender sees the message bubble appear with 0ms latency.
    3. Dispatches the HTTP POST request to `/api/messages/send/:receiverId`.
    4. Upon HTTP success: Replaces or appends the confirmed server record with the real MongoDB `_id`.
    5. Upon HTTP error: Rolls back state (`set({ messages })`) and displays a toast error alerting the user.

### Q18. How are protected routes and authentication gates managed on the frontend?
- **Answer:**
  - In `App.jsx`, when the application mounts, `useEffect` triggers `checkAuth()` (`GET /api/auth/check`).
  - While `isCheckingAuth` is true, a full-screen `<PageLoader />` is displayed.
  - React Router routes are conditional:
    - `/` renders `<ChatPage />` if `authUser` exists, otherwise `<Navigate to="/login" />`.
    - `/login` and `/signup` render auth pages if `!authUser`, otherwise `<Navigate to="/" />`.

---

## 6. Cloud Media, Emailing & Third-Party APIs

### Q19. How does image uploading work and what are the trade-offs of Base64 vs Presigned URLs?
- **Answer:**
  - **Current Implementation:**
    The user selects an image, the frontend converts it to a Base64 data URL string via `FileReader.readAsDataURL()`, sends it in JSON payload (with `express.json({ limit: "5mb" })`), and the backend uploads it via `cloudinary.uploader.upload(image)`.
  - **Trade-offs:**
    - *Base64 Pros:* Easy to implement, works over standard JSON endpoints without configuring `multer` or multipart streams.
    - *Base64 Cons:* Base64 encoding increases payload size by ~33%, consumes server CPU during encoding/decoding, and routes large binary traffic through the Express server.
  - **Production Optimization:**
    Generate a **Cloudinary / S3 Presigned URL** on the backend. The client uploads the binary directly from browser to cloud storage bucket, saving server CPU and bandwidth.

### Q20. How is the welcome email handled and why is it wrapped in an isolated try-catch?
- **Answer:**
  - On signup, after user creation and cookie issuance, `sendWelcomeEmail(savedUser.email, savedUser.fullName, ENV.CLIENT_URL)` is called using the **Resend API**.
  - It is wrapped in an isolated `try...catch` block:
    ```javascript
    try {
      await sendWelcomeEmail(savedUser.email, savedUser.fullName, ENV.CLIENT_URL);
    } catch (error) {
      console.error("Failed to send welcome email:", error);
    }
    ```
  - **Critical Reason:** Email dispatch should be treated as a non-fatal side effect. If Resend's API is temporarily down or rate-limited, the user account has already been created and they should not receive a `500 Internal Server Error` preventing them from using the app.

---

## 7. Scalability, Performance & Production Bottlenecks

### Q21. If this app grew to 500,000 active users, what bottlenecks would arise first and how would you fix them?
- **Answer:**
  1. **Memory on Node Server (WebSockets):** Each open WebSocket connection consumes file descriptors and memory. A single Node process can realistically handle 10k–30k concurrent connections.
     - *Fix:* Horizontally scale with a load balancer (Nginx / AWS ALB) configured for WebSocket upgrades and sticky sessions, with a Redis Pub/Sub adapter.
  2. **MongoDB Connection Pool & Unindexed Queries:** Chat partner discovery and message lookups would choke database IOPS.
     - *Fix:* Add compound indexes, read-replicas for chat history fetching, and MongoDB connection pooling.
  3. **Event Loop Blocking (Synchronous operations):** Large Base64 uploads and email API calls can block the single-threaded Node.js event loop.
     - *Fix:* Offload background tasks (emails, video/image transcoding) to background workers using a job queue like **BullMQ** with Redis.
  4. **Active State & Ephemeral Data:**
     - *Fix:* Move active online presence from in-memory `userSocketMap` to **Redis Hashes or Redis Sets** with automatic expiration (heartbeats).

### Q22. How would you implement Group Chats in this system?
- **Answer:**
  - **Data Model:** Create a `Conversation` or `Room` schema (`participants: [ObjectId]`, `isGroup: Boolean`, `groupName: String`, `groupAdmin: ObjectId`).
  - **Socket.io Rooms:** When a user opens a group or connects, join them to socket rooms: `socket.join(conversationId)`.
  - **Message Dispatch:** Instead of `io.to(receiverSocketId).emit()`, emit to the room: `io.to(conversationId).emit("newGroupMessage", message)`.
  - The Redis adapter automatically handles fanning out the message to all members in that room across all server instances.

### Q23. How would you implement "Typing..." indicators and Read Receipts?
- **Answer:**
  - **Typing Indicators:**
    - Client emits `typing` and `stopTyping` events with `{ senderId, receiverId }` on input keypress (debounced by 1500ms).
    - Server forwards the event directly to receiver's socket without writing to MongoDB (ephemeral event).
  - **Read Receipts ("Seen"):**
    - Add `isRead: { type: Boolean, default: false }` and `readAt: Date` to the `Message` model.
    - When the receiver focuses the chat window, client emits `markMessagesAsRead({ senderId })`.
    - Server updates messages in DB (`updateMany({ senderId, receiverId: myId, isRead: false }, { isRead: true })`) and emits `messagesRead` event back to the original sender to update their checkmarks to blue double ticks.

---

## 8. Behavioral & Code Quality Questions

### Q24. Can you describe a technical challenge you encountered while building this project and how you solved it?
- **Answer:**
  - *Example Talking Point (Socket Authentication & Cookie Handling):*
    "A major challenge was authenticating WebSocket handshakes seamlessly with JWT. Because WebSockets initiate via an HTTP upgrade request, standard `Authorization: Bearer <token>` headers aren't easily sent from native browser WebSocket clients without URL query strings (which leak tokens in server access logs).
    I solved this by storing JWT in HTTP-Only cookies with `credentials: true` on both Express CORS and Socket.io client configs. In `socketAuthMiddleware`, I parsed `socket.handshake.headers.cookie` directly, verified the token, and attached the user profile to the socket instance before allowing the connection."

### Q25. What code review improvements were made to ensure data integrity?
- **Answer:**
  - In `auth.controller.js`, during user registration, the code previously generated the JWT token before `await newUser.save()` completed.
  - If the database query threw an error (such as a database connection timeout or validation issue), the auth cookie would still have been attached to the HTTP response header.
  - The fix refactored the flow: save the user to MongoDB first, verify persistence succeeded, and only then issue the JWT cookie and send the welcome email.

---

## 🛠️ Quick Reference: Key File Map

| Feature / Area | Primary File(s) | Key Symbols / Responsibilities |
|---|---|---|
| Server Entry Point | `backend/src/server.js` | Express app, HTTP server, routes, static asset serving |
| WebSocket Setup | `backend/src/lib/socket.js` | `Server`, `userSocketMap`, `getReceiverSocketId`, online status |
| Socket Authentication | `backend/src/middleware/socket.auth.middleware.js` | Handshake cookie parsing, JWT verification |
| HTTP Authentication | `backend/src/middleware/auth.middleware.js` | `protectRoute`, cookie verification |
| Auth Business Logic | `backend/src/controllers/auth.controller.js` | `signup`, `login`, `logout`, `updateProfile` |
| Message Controller | `backend/src/controllers/message.controller.js` | `sendMessage`, `getMessagesByUserId`, `getChatPartners` |
| Security & Rate Limiting | `backend/src/lib/arcjet.js`, `arcjet.middleware.js` | Shield, bot detection, sliding window limit |
| Email Service | `backend/src/emails/emailHandlers.js` | `sendWelcomeEmail` with Resend |
| Frontend Auth Store | `frontend/src/store/useAuthStore.js` | Zustand store: `authUser`, `login`, `connectSocket` |
| Frontend Chat Store | `frontend/src/store/useChatStore.js` | Zustand store: `sendMessage`, optimistic updates, sound alerts |
