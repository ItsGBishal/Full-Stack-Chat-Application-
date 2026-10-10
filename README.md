# ⚡ Chatify — Real-Time Chat Application

<p align="center">
  A full-stack, real-time messaging platform built with <b>Node.js</b>, <b>Express</b>, <b>Socket.io</b>, <b>MongoDB</b>, <b>React 19</b>, and <b>Zustand</b>.
</p>

<p align="center">
  <img src="https://img.shields.io/badge/React_19-20232A?style=for-the-badge&logo=react&logoColor=61DAFB" alt="React 19" />
  <img src="https://img.shields.io/badge/Node.js-339933?style=for-the-badge&logo=nodedotjs&logoColor=white" alt="Node.js" />
  <img src="https://img.shields.io/badge/Express.js-000000?style=for-the-badge&logo=express&logoColor=white" alt="Express" />
  <img src="https://img.shields.io/badge/Socket.io-010101?style=for-the-badge&logo=socketdotio&logoColor=white" alt="Socket.io" />
  <img src="https://img.shields.io/badge/MongoDB-4EA94B?style=for-the-badge&logo=mongodb&logoColor=white" alt="MongoDB" />
  <img src="https://img.shields.io/badge/Tailwind_CSS-38B2AC?style=for-the-badge&logo=tailwind-css&logoColor=white" alt="Tailwind CSS" />
  <img src="https://img.shields.io/badge/Zustand-orange?style=for-the-badge" alt="Zustand" />
</p>

---

## ✨ Features

- ⚡ **Real-Time Messaging:** 1-to-1 chat via persistent Socket.io connections.
- 🟢 **Online Presence:** Real-time online/offline status tracking across sessions.
- 🔐 **JWT Auth in HTTP-Only Cookies:** Secure token storage protecting against XSS and CSRF.
- 🚀 **Optimistic UI:** Instant message rendering before server confirmation.
- 🖼️ **Image Sharing:** Cloud media upload and hosting powered by Cloudinary.
- 🔔 **Sound Alerts:** Audio chime on new incoming messages with user-toggle persistence.
- 🛡️ **Edge Security:** Rate limiting and bot protection using Arcjet.
- 📨 **Welcome Emails:** Automated transactional emails via Resend.

---

## 🛠️ Tech Stack

- **Frontend:** React 19, Vite, Tailwind CSS, DaisyUI, Zustand, Axios
- **Backend:** Node.js, Express.js, Socket.io, Mongoose, Cookie-Parser
- **Database & Cloud:** MongoDB Atlas, Cloudinary, Resend, Arcjet

---

## 🚀 Quick Start

### 1. Configure Environment (`backend/.env`)
```env
PORT=3000
MONGO_URI=your_mongodb_connection_string
JWT_SECRET=your_secret_key
CLIENT_URL=http://localhost:5173
NODE_ENV=development

CLOUDINARY_CLOUD_NAME=your_cloud_name
CLOUDINARY_API_KEY=your_api_key
CLOUDINARY_API_SECRET=your_api_secret

RESEND_API_KEY=your_resend_api_key
EMAIL_FROM=onboarding@resend.dev
EMAIL_FROM_NAME=Chatify

ARCJET_KEY=your_arcjet_key
ARCJET_ENV=development
```

### 2. Run Locally

```bash
# Backend (Terminal 1)
cd backend
npm install
npm run dev

# Frontend (Terminal 2)
cd frontend
npm install
npm run dev
```

App runs at `http://localhost:5173` with the backend at `http://localhost:3000`.

---

## 📚 Technical Interview & Architecture Notes

Detailed architecture flowcharts, database design, and 25+ technical interview questions covering this project can be found in **[LEARN.MD](./LEARN.MD)**.

---

## 👤 Author

- **GitHub:** [@ItsGbishal](https://github.com/ItsGbishal)
- **Repository:** [Full-Stack-Chat-Application-](https://github.com/ItsGbishal/Full-Stack-Chat-Application-)
