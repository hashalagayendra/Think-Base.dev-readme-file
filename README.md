<div align="center">

# 🧠 ThinkBase

### AI-Powered RAG Platform as a Service

_Upload your documents. Get an intelligent chatbot. Embed it anywhere._

[![Live](https://img.shields.io/badge/🌐_Live-think--base.dev-blue?style=for-the-badge)](https://think-base.dev)
[![SDK](https://img.shields.io/badge/📦_npm-thinkbase-red?style=for-the-badge)](https://www.npmjs.com/package/thinkbase)

---

**NestJS 11** · **Next.js 15** · **PostgreSQL + pgvector** · **Google Gemini 2.5** · **LangChain** · **Docker** · **Azure** · **GraphQL**

</div>

<br/>

## 💡 What is ThinkBase?

ThinkBase is an **end-to-end AI platform** that turns your documents (PDFs, text files) into an intelligent chatbot — powered by **Retrieval-Augmented Generation (RAG)**.

> **The core idea:** Instead of hallucinating, the AI reads YOUR documents, finds the most relevant sections, and generates grounded, accurate answers.

### What I Built — At a Glance

| What                       | Details                                                                                                          |
| :------------------------- | :--------------------------------------------------------------------------------------------------------------- |
| 🔧 **Full-Stack Platform** | Designed and built the entire system from scratch — backend, frontend, database, SDK, CI/CD                      |
| 🤖 **RAG Engine**          | Custom vector search pipeline: document chunking → Gemini embeddings → pgvector cosine similarity → LLM response |
| 🔐 **Multi-Auth System**   | JWT + Google OAuth + GitHub OAuth + email magic-link verification                                                |
| 📦 **Published npm SDK**   | `thinkbase` — a drop-in React chat widget + programmatic JS client                                               |
| 🚀 **Production CI/CD**    | GitHub Actions → Docker Hub → Azure Container Apps (auto-deploy on push)                                         |
| 📱 **Mobile API**          | GraphQL/Apollo Server for a companion React Native admin app                                                     |
| 📄 **Swagger Docs**        | Auto-generated interactive API documentation at `/api`                                                           |

<br/>

---

## 🛠️ Skills & Technologies Demonstrated

<table>
<tr>
<td width="50%" valign="top">

### Backend Engineering

- **NestJS 11** — Modular architecture, DI, guards, interceptors
- **Prisma ORM 6** — Type-safe queries, migrations, transactions
- **PostgreSQL + pgvector** — Vector similarity search at scale
- **LangChain 0.3** — RAG orchestration, text splitting, embeddings
- **Google Gemini APIs** — Embedding-001 (3072-dim) + Gemini 2.5 Flash
- **Passport.js** — Multi-strategy auth (JWT, Google, GitHub)
- **Apollo Server / GraphQL** — Schema-first API for mobile
- **Swagger / OpenAPI** — Auto-generated docs
- **Nodemailer** — Transactional email (magic links)
- **bcrypt** — Secure password hashing

</td>
<td width="50%" valign="top">

### Frontend Engineering

- **Next.js 15** — App Router, Server Components, Turbopack
- **React 19** — Hooks, state management, component architecture
- **Tailwind CSS 4** — Responsive, utility-first design
- **Zustand 5** — Lightweight global state management
- **Framer Motion + GSAP** — Advanced scroll & micro-animations
- **Radix UI + HeroUI** — Accessible component primitives
- **react-dropzone** — Drag-and-drop document upload
- **react-pdftotext** — Client-side PDF parsing
- **Axios** — HTTP client with interceptors
- **Zod** — Runtime schema validation

</td>
</tr>
<tr>
<td width="50%" valign="top">

### DevOps & Infrastructure

- **Docker** — Multi-stage production builds (Node 22 Alpine)
- **GitHub Actions** — Automated CI/CD pipeline
- **Docker Hub** — Container registry
- **Azure Container Apps** — Serverless container hosting
- **Environment management** — Secure secrets via GitHub Secrets

</td>
<td width="50%" valign="top">

### SDK & Developer Tools

- **npm package publishing** — `thinkbase` on npm registry
- **Rollup** — Module bundler with Babel + JSX support
- **API design** — Clean, documented REST + GraphQL endpoints
- **SDK design** — React component + programmatic client class

</td>
</tr>
</table>

<br/>

---

## 🏗️ System Architecture

```
┌──────────────────┐     ┌──────────────────┐     ┌──────────────────┐
│   Landing Page   │     │   SDK / Widget    │     │  Mobile Admin    │
│   + Dashboard    │     │   (npm package)   │     │  (React Native)  │
│   Next.js 15     │     │   thinkbase       │     │                  │
│   Port 4000      │     │                   │     │                  │
└────────┬─────────┘     └────────┬──────────┘     └────────┬─────────┘
         │ REST API               │ REST API                │ GraphQL
         └────────────────┬───────┘                         │
                          ▼                                 ▼
              ┌───────────────────────────────────────────────┐
              │              NestJS 11 Backend                │
              │              Port 3000                        │
              │                                               │
              │   Auth · Projects · Chat · Users · Mobile     │
              │   Swagger Docs · Guards · Interceptors        │
              └──────────────────┬────────────────────────────┘
                                 │
                    ┌────────────┴────────────┐
                    ▼                         ▼
          ┌──────────────────┐     ┌──────────────────┐
          │   PostgreSQL     │     │   Google Gemini   │
          │   + pgvector     │     │                   │
          │                  │     │  Embedding-001    │
          │  6 tables        │     │  (3072-dim vecs)  │
          │  Vector search   │     │                   │
          │  via <=> cosine  │     │  Gemini 2.5 Flash │
          │                  │     │  (chat LLM)       │
          └──────────────────┘     └──────────────────┘
```

<br/>

---

## ⚙️ How the RAG Pipeline Works

This is the **core intelligence** of ThinkBase — what happens when a user sends a message:

### Phase 1 — Document Ingestion (One-time setup)

```
Upload PDF/Text  →  Extract text (client-side)  →  Store in DB
                                                        │
User configures chunk size & overlap                    │
                                                        ▼
Split into chunks  →  Embed each chunk  →  Store 3072-dim vectors
(LangChain             (Gemini                (PostgreSQL pgvector
 Recursive              Embedding-001)         with cosine index)
 Splitter)
```

### Phase 2 — Query Pipeline (Every chat message)

```
User Question
      │
      ▼
 ① Chunk the query (1000 chars, 200 overlap)
      │
      ▼
 ② Embed each chunk → 3072-dim vector
      │
      ▼
 ③ Cosine similarity search against VectorDB
    • Top 5 matches per chunk
    • Filter: distance < 0.6 (relevance threshold)
      │
      ▼
 ④ Build prompt:
    [System Prompt] + [Chat History] + [Retrieved Context + Question]
      │
      ▼
 ⑤ Call Gemini 2.5 Flash (temp=0, deterministic)
      │
      ▼
 ⑥ Save conversation → Return AI response
```

> **Key technical decisions:**
>
> - **pgvector** over a separate vector DB (Pinecone/Weaviate) — keeps everything in one PostgreSQL instance, simpler infrastructure
> - **Cosine distance `<=>` operator** — native SQL similarity search, no external service needed
> - **Temperature 0** — deterministic, factual responses (no creative hallucination)
> - **Distance threshold 0.6** — only highly relevant chunks are used as context

<br/>

---

## 🔐 Authentication System

ThinkBase supports **4 authentication methods**, all working together:

| Method               | Flow                                                                             |
| :------------------- | :------------------------------------------------------------------------------- |
| **Email + Password** | Register → bcrypt hash → magic-link email (2min expiry) → verify → login         |
| **Google OAuth 2.0** | One-click → Google consent → callback → auto-create user → redirect to dashboard |
| **GitHub OAuth**     | One-click → GitHub auth → callback → auto-create user → redirect to dashboard    |
| **Magic Link**       | Email with JWT link → click → validate → issue session                           |

### Token Strategy

| Token         | Lifetime | Storage                 | Purpose              |
| :------------ | :------- | :---------------------- | :------------------- |
| Access Token  | 15 min   | In-memory (Zustand)     | API authorization    |
| Refresh Token | 7 days   | HTTP-only Secure cookie | Silent token renewal |
| Magic Link    | 2 min    | URL parameter           | Email verification   |

> **Security:** Refresh tokens are stored as `httpOnly`, `secure`, `SameSite=None` cookies — never accessible to JavaScript. Access tokens live only in memory and are lost on page refresh (refreshed silently via the cookie).

<br/>

---

## 🗄️ Database Design

**6 tables** in PostgreSQL with the **pgvector** extension:

```
User ─────1:N──── Project ─────1:N──── Documents
                     │                      │
                     │ 1:N                  │ 1:N
                     │                      │
                 ApiKeyModel            VectorDB
                     │               (3072-dim vectors)
                     │ 1:N
                     │
                   Chat
```

| Table           | Key Fields                                                                 | Purpose                                 |
| :-------------- | :------------------------------------------------------------------------- | :-------------------------------------- |
| **User**        | `email (unique)`, `password (bcrypt)`, `isEmailVerified`, `photo`          | User accounts (credentials + OAuth)     |
| **Project**     | `name`, `description`, `systemPrompt`, `userId` (FK)                       | RAG project container                   |
| **Documents**   | `text`, `chankSize`, `overlap`, `status`                                   | Uploaded documents & processing config  |
| **VectorDB**    | `chunk`, `geminiFlashEmbeddings` (vector 3072), `documentId`, `projectsid` | Embedding vectors for similarity search |
| **ApiKeyModel** | `apikey` (PK, 64-char hex), `requestsCount`, `lastUsedAt`                  | API access keys with usage tracking     |
| **Chat**        | `clientId`, `message`, `sender` (user/ai), `apikeyId`                      | Conversation history per client session |

> **15 Prisma migrations** track the full schema evolution from initial design to production.

<br/>

---

## 📦 ThinkBase SDK (`npm install thinkbase`)

I published a **JavaScript SDK** so developers can integrate ThinkBase chatbots into their own sites with just a few lines of code.

### Option 1 — Drop-in React Chat Widget

```jsx
import { Chat } from "thinkbase";

function App() {
  return <Chat apiKey="your-api-key" />;
}
```

A complete, styled chat interface — renders a message list, input box, send button, loading states, and auto-scroll. Zero config needed.

### Option 2 — Programmatic Client

```javascript
import { ThinkBaseClient } from "thinkbase";

const client = new ThinkBaseClient({ apiKey: "your-api-key" });

// Send a message → get AI response grounded in your documents
const { aiMessage } = await client.sendMessage("What is your return policy?");

// Get full conversation history
const messages = await client.getMessages();

// Clear the conversation
await client.deleteMessages();
```

### SDK Architecture

| Export            | Type            | Description                                   |
| :---------------- | :-------------- | :-------------------------------------------- |
| `Chat`            | React Component | Full chat UI widget with styling              |
| `ThinkBaseClient` | ES6 Class       | Programmatic API (send, get, delete messages) |

> **Built with:** Rollup (bundler) + Babel (JSX transpilation) + Axios (HTTP client)
> **Session tracking:** Uses HTTP-only cookies — each browser/device gets a unique `clientId` automatically.

<br/>

---

## 📱 GraphQL API (Mobile Admin)

A **GraphQL API** powered by Apollo Server serves as the backend for the React Native mobile admin app:

```graphql
type Query {
  getUsers: [User!]! # List all users
  getAlltProjects: [Project!]! # List all projects
}

type Mutation {
  getEachUser(data: UserId!): User! # Detailed user with nested projects, docs, API keys
  getEachProject(data: ProjectId!): Project! # Detailed project with docs, keys, chat stats
}
```

Returns deeply nested data — projects include documents, API keys, request counts, and unique client counts.

<br/>

---

## 🖥️ Frontend Pages

| Route                     | Access       | Description                                                                                |
| :------------------------ | :----------- | :----------------------------------------------------------------------------------------- |
| `/`                       | Public       | Landing page — hero, features, use cases, footer (10 animated sections)                    |
| `/login`                  | Public       | Email/password login + OAuth buttons                                                       |
| `/signup`                 | Public       | Registration with email verification                                                       |
| `/magic-link`             | Public       | Handles email verification token                                                           |
| `/Dashboard`              | 🔒 Protected | Welcome screen + project cards grid                                                        |
| `/Dashboard/project/[id]` | 🔒 Protected | **Project workspace** — document upload, chunk config, playground, API keys, code snippets |
| `/Dashboard/settings`     | 🔒 Protected | User settings                                                                              |
| `/Documentation`          | Public       | SDK integration docs                                                                       |

### Project Workspace — The Main Feature

The workspace for each project has multiple sections:

- **📄 Document Management** — Upload (drag-and-drop), view status, delete documents
- **⚙️ Configure** — Adjust chunk size and overlap with sliders, trigger embedding processing
- **💬 Chat Playground** — Test your chatbot in real-time with the built-in chat UI
- **✏️ System Prompt Editor** — Customize how the AI behaves and responds
- **🔑 API Keys** — Generate, view, and manage access keys
- **📋 Code Snippets** — Copy-paste integration code for the SDK

### Design & UX

- **Premium dark theme** with blue accent gradients and glassmorphism blur effects
- **Framer Motion + GSAP** scroll-triggered animations on the landing page
- **Lenis** smooth scrolling
- **Radix UI** accessible primitives (scroll areas, selects, slots)
- **Responsive** across desktop and mobile

<br/>

---

## 🚀 CI/CD & Deployment

Fully automated pipeline — every push to `deployement_t1` triggers a production deployment:

```
git push → GitHub Actions → Docker Build → Docker Hub → Azure Container Apps
```

| Step                    | What Happens                                                                 |
| :---------------------- | :--------------------------------------------------------------------------- |
| **1. Build Backend**    | Multi-stage Docker build (Node 22 Alpine) → Prisma generate → NestJS compile |
| **2. Push to Registry** | `hashalagayendra/thinkbase-backend:latest` → Docker Hub                      |
| **3. Deploy Backend**   | Azure Container App updated with new image + 14 environment variables        |
| **4. Build Frontend**   | Multi-stage Docker build with `NEXT_PUBLIC_*` build args baked in            |
| **5. Push to Registry** | `hashalagayendra/thinkbase-frontend:latest` → Docker Hub                     |
| **6. Deploy Frontend**  | Azure Container App updated, ingress on port 4000                            |

> **Infrastructure:** Backend on port 3000, Frontend on port 4000 — both as Azure Container Apps with external ingress. Database on Supabase PostgreSQL (with pgvector extension).

<br/>

---

## 🔒 Security

| Area               | Implementation                                                                             |
| :----------------- | :----------------------------------------------------------------------------------------- |
| Passwords          | bcrypt with auto-generated salt                                                            |
| Tokens             | HTTP-only, Secure, SameSite=None cookies (never in JS)                                     |
| API Keys           | 256-bit entropy (`crypto.randomBytes(32)`)                                                 |
| Input Validation   | NestJS `ValidationPipe` — whitelist mode, rejects unknown fields                           |
| Request Limits     | 50MB for document upload, 100KB for all other routes                                       |
| Route Protection   | 3 guard layers: `VerifyAuthGuard` (JWT), `ProjectGuard` (ownership), `ChatGuard` (API key) |
| Email Verification | Required before first login, 2-minute expiry links                                         |

<br/>

---

## 📁 Project Structure

```
ThinkBase/
├── backend/                     # NestJS 11 API
│   ├── src/
│   │   ├── auth/                # 4-method auth system (11 endpoints)
│   │   ├── project/             # Project + document + API key management (13 endpoints)
│   │   ├── chat/                # RAG chat engine (4 endpoints)
│   │   ├── user/                # User management
│   │   ├── mobile-app/          # GraphQL resolvers for mobile
│   │   ├── prisma/              # Database service
│   │   ├── config/              # Environment config
│   │   └── middleware/          # Custom middleware
│   ├── prisma/schema.prisma     # 6 models, 15 migrations
│   └── Dockerfile               # Multi-stage production build
│
├── frontend/                    # Next.js 15 (Turbopack)
│   ├── src/
│   │   ├── app/                 # 8 routes (App Router)
│   │   ├── components/          # 18 components + 10 landing page sections
│   │   ├── store/               # Zustand global state
│   │   └── utils/               # Helpers
│   └── Dockerfile               # Multi-stage production build
│
├── SDK/                         # npm: thinkbase
│   ├── src/
│   │   ├── Client.js            # ThinkBaseClient class
│   │   ├── Chat.jsx             # React Chat widget
│   │   └── index.js             # Package exports
│   └── rollup.config.js         # Bundler config
│
└── .github/workflows/
    └── deploy.yml               # CI/CD → Docker Hub → Azure
```

<br/>

---

## 📊 REST API Overview (28+ Endpoints)

| Module                   | Endpoints | Key Features                                                                                                     |
| :----------------------- | :-------- | :--------------------------------------------------------------------------------------------------------------- |
| **Auth** (`/auth`)       | 11        | Signup, login, logout, verify, Google OAuth, GitHub OAuth, magic-link validate/resend                            |
| **Project** (`/project`) | 13        | Create, get, delete projects; upload/delete documents; process embeddings; manage API keys; update system prompt |
| **Chat** (`/chat`)       | 4         | Send message (RAG), get history, delete history; guarded by API key                                              |
| **User** (`/user`)       | 1         | Get user details                                                                                                 |

> **Full interactive API docs** available at `/api` (Swagger UI) — try any endpoint directly from the browser.

<br/>
s
---

<div align="center">

### Built as a Full-Stack Production Application

**Backend** · **Frontend** · **Database Design** · **AI/ML Pipeline** · **SDK** · **DevOps** · **Security**

All designed, implemented, and deployed by a single developer.

---

_Powered by Google Gemini · LangChain · NestJS · Next.js · PostgreSQL + pgvector_

</div>
