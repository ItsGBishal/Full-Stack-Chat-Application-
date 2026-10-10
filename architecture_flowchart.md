# Chatify Application Architecture & System Flowcharts

This document provides a comprehensive visual guide to the **Chatify** full-stack real-time chat application architecture, user authentication flow, real-time messaging pipeline, and database relationships.

---

## Visual Architecture Overview

![Chatify Architecture Diagram](C:/Users/ghosh/.gemini/antigravity-ide/brain/b187e191-80dc-4ab5-8c4a-3b17ec34ee14/chatify_architecture_diagram_1785858449010.png)

---

## 1. High-Level System Architecture

```mermaid
flowchart TB
    subgraph Client ["Frontend (React / Vite + Tailwind + DaisyUI)"]
        UI["React Pages (SignUp, Login, Chat)"]
        AuthStore["Zustand Auth Store (useAuthStore.js)"]
        ChatStore["Zustand Chat Store (useChatStore.js)"]
        SocketClient["Socket.IO Client"]
    end

    subgraph Backend ["Backend (Node.js + Express + Socket.IO)"]
        Server["Express App / HTTP Server (server.js)"]
        SocketServer["Socket.IO Engine (socket.js)"]
        
        subgraph Middlewares ["Middlewares"]
            AuthMW["Auth Middleware (JWT Cookie)"]
            ArcjetMW["Arcjet Security / Rate Limiter"]
            SocketAuthMW["Socket Auth Middleware"]
        end

        subgraph Controllers ["API Controllers"]
            AuthController["Auth Controller (signup, login, checkAuth)"]
            MsgController["Message Controller (getMessages, sendMessage)"]
        end
    end

    subgraph Storage ["Database & External Services"]
        MongoDB[("MongoDB (Mongoose)\nUser & Message Docs")]
        Cloudinary["Cloudinary API\n(Profile & Chat Image Hosting)"]
        Resend["Resend Email API\n(Welcome Emails)"]
    end

    UI --> AuthStore
    UI --> ChatStore
    AuthStore <-->|REST API / Axios| Server
    ChatStore <-->|REST API / Axios| Server
    SocketClient <-->|WebSocket Connection| SocketServer

    Server --> ArcjetMW
    ArcjetMW --> AuthMW
    AuthMW --> Controllers

    SocketServer --> SocketAuthMW

    Controllers --> MongoDB
    Controllers -->|Upload Base64 Images| Cloudinary
    Controllers -->|Send Email Async| Resend
```

---

## 2. User Authentication & Registration Flow

```mermaid
sequenceDiagram
    autonumber
    actor User
    participant Frontend as React App (SignUpPage)
    participant Backend as Express Backend (/api/auth)
    participant DB as MongoDB (User Model)
    participant Resend as Resend API

    User->>Frontend: Fills Signup Form (Name, Email, Password)
    Frontend->>Backend: POST /api/auth/signup
    Backend->>Backend: Validate Input & Hash Password (bcrypt)
    Backend->>DB: Save New User Document
    DB-->>Backend: Saved User Object
    Backend->>Backend: Generate JWT & Set HTTP-Only Cookie
    Backend-->>Frontend: HTTP 201 Created (User Data + Auth Cookie)
    
    par Async Email Dispatch
        Backend->>Resend: sendWelcomeEmail(email, name)
        Resend-->>Backend: Email Dispatched
    end

    Frontend->>Frontend: Update Zustand state & Redirect to Chat
```

---

## 3. Real-Time Messaging & Image Sharing Flow

```mermaid
sequenceDiagram
    autonumber
    actor Sender as User A (Sender)
    actor Receiver as User B (Receiver)
    participant ClientA as Sender React UI
    participant Backend as Express API (/api/messages)
    participant Cloudinary as Cloudinary API
    participant DB as MongoDB
    participant SocketServer as Socket.IO Engine
    participant ClientB as Receiver React UI

    Sender->>ClientA: Types text or selects image
    
    alt Image Attached
        ClientA->>Backend: POST /api/messages/send/:id (Text + Base64 Image)
        Backend->>Cloudinary: Upload Image
        Cloudinary-->>Backend: Secure Image URL
    else Text Only
        ClientA->>Backend: POST /api/messages/send/:id (Text)
    end

    Backend->>DB: Save Message Document (senderId, receiverId, text, image)
    DB-->>Backend: Saved Message Object

    Backend-->>ClientA: HTTP 201 Response (Message saved)

    Backend->>SocketServer: getReceiverSocketId(receiverId)
    SocketServer->>ClientB: emit("newMessage", message)
    ClientB->>Receiver: Render new message in real-time UI
```

---

## 4. Database Schema Relationships (ER Diagram)

```mermaid
erDiagram
    USER ||--o{ MESSAGE : "sends (senderId)"
    USER ||--o{ MESSAGE : "receives (receiverId)"

    USER {
        ObjectId _id PK
        String email UK
        String fullName
        String password
        String profilePic
        Date createdAt
        Date updatedAt
    }

    MESSAGE {
        ObjectId _id PK
        ObjectId senderId FK
        ObjectId receiverId FK
        String text
        String image
        Date createdAt
        Date updatedAt
    }
```
