<p align="center">
  <h1 align="center">🧠 ThinkBase</h1>
  <p align="center">
    <strong>AI-Powered RAG (Retrieval-Augmented Generation) Platform as a Service</strong>
  </p>
  <p align="center">
    Build intelligent, context-aware AI chatbots powered by your own documents — no ML expertise required.
  </p>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/NestJS-11-E0234E?logo=nestjs&logoColor=white" alt="NestJS 11" />
  <img src="https://img.shields.io/badge/Next.js-15-000000?logo=next.js&logoColor=white" alt="Next.js 15" />
  <img src="https://img.shields.io/badge/PostgreSQL-pgvector-4169E1?logo=postgresql&logoColor=white" alt="PostgreSQL" />
  <img src="https://img.shields.io/badge/Gemini-2.5%20Flash-4285F4?logo=google&logoColor=white" alt="Gemini" />
  <img src="https://img.shields.io/badge/LangChain-0.3-1C3C3C?logo=langchain&logoColor=white" alt="LangChain" />
  <img src="https://img.shields.io/badge/Docker-Containerized-2496ED?logo=docker&logoColor=white" alt="Docker" />
  <img src="https://img.shields.io/badge/Azure-Container%20Apps-0078D4?logo=microsoftazure&logoColor=white" alt="Azure" />
  <img src="https://img.shields.io/badge/GraphQL-Apollo-E10098?logo=graphql&logoColor=white" alt="GraphQL" />
</p>

---

## 📑 Table of Contents

- [What is ThinkBase?](#what-is-thinkbase)
- [How It Works](#how-it-works)
- [Architecture Overview](#architecture-overview)
- [Project Structure](#project-structure)
- [Tech Stack](#tech-stack)
- [Backend — Deep Dive](#backend--deep-dive)
  - [Module Architecture](#module-architecture)
  - [Authentication System](#authentication-system)
  - [Project & Document Management](#project--document-management)
  - [RAG Pipeline — The Core Engine](#rag-pipeline--the-core-engine)
  - [Chat System](#chat-system)
  - [API Key Management](#api-key-management)
  - [GraphQL API for Mobile](#graphql-api-for-mobile)
  - [REST API Reference](#rest-api-reference)
- [Frontend — Deep Dive](#frontend--deep-dive)
  - [Pages & Routing](#pages--routing)
  - [Landing Page](#landing-page)
  - [Dashboard](#dashboard)
  - [Project Workspace](#project-workspace)
  - [State Management](#state-management)
- [ThinkBase SDK (npm Package)](#thinkbase-sdk-npm-package)
  - [ThinkBaseClient API](#thinkbaseclient-api)
  - [Chat React Component](#chat-react-component)
- [Database Schema](#database-schema)
- [Deployment & CI/CD Pipeline](#deployment--cicd-pipeline)
- [Environment Variables](#environment-variables)
- [API Documentation (Swagger)](#api-documentation-swagger)
- [Security Considerations](#security-considerations)

---

## What is ThinkBase?

**ThinkBase** is a full-stack **RAG (Retrieval-Augmented Generation) platform** that allows users to create AI chatbots grounded in their own documents. Instead of generic AI responses, ThinkBase chatbots answer questions using the specific knowledge you provide — PDFs, text files, and other documents.

### The Problem It Solves

Traditional AI chatbots (like raw ChatGPT) hallucinate and don't know about your specific data. ThinkBase solves this by:

1. **Ingesting your documents** — upload PDFs or text files
2. **Chunking & embedding them** — splits documents into semantic chunks and generates vector embeddings using Google's Gemini Embedding model
3. **Storing embeddings in a vector database** — uses PostgreSQL with the `pgvector` extension for efficient similarity search
4. **Retrieving relevant context at query time** — when a user asks a question, the system finds the most relevant document chunks via cosine similarity
5. **Generating grounded responses** — feeds the retrieved context + conversation history to Gemini 2.5 Flash for accurate, context-aware answers

### Who It's For

| Use Case                    | Example                                                                              |
| --------------------------- | ------------------------------------------------------------------------------------ |
| **Customer Support**        | Upload your product docs → get an AI agent that answers product questions accurately |
| **Internal Knowledge Base** | Upload company policies, SOPs → employees get instant answers                        |
| **Education**               | Upload course materials → students interact with an AI tutor                         |
| **Documentation Q&A**       | Upload API docs → developers get contextual code help                                |
| **Legal / Compliance**      | Upload contracts, regulations → get precise answers from legal text                  |

---

## How It Works

```
┌─────────────────────────────────────────────────────────────────┐
│                        USER WORKFLOW                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. Sign Up / Login                                             │
│     ├── Email + Password (with magic-link email verification)   │
│     ├── Google OAuth 2.0                                        │
│     └── GitHub OAuth                                            │
│                                                                 │
│  2. Create a Project                                            │
│     └── Give it a name and description                          │
│                                                                 │
│  3. Upload Documents (PDF / Text)                               │
│     ├── Documents are parsed on the frontend (react-pdftotext)  │
│     └── Raw text is sent to the backend and stored              │
│                                                                 │
│  4. Configure & Process Documents                               │
│     ├── Adjust chunk size (default: 500 chars)                  │
│     ├── Adjust chunk overlap (default: 50 chars)                │
│     ├── Click "Process" to generate embeddings                  │
│     │   ├── Text → RecursiveCharacterTextSplitter (LangChain)   │
│     │   ├── Chunks → Gemini Embedding-001 model (3072-dim)      │
│     │   └── Vectors stored in PostgreSQL pgvector column        │
│     └── Document status: Need_to_configure → Processed          │
│                                                                 │
│  5. Customize System Prompt                                     │
│     └── Define how the AI should behave and respond             │
│                                                                 │
│  6. Generate API Keys                                           │
│     └── Each key is a 64-char hex token tied to a project       │
│                                                                 │
│  7. Test in Playground                                          │
│     └── Built-in chat interface to test your RAG chatbot        │
│                                                                 │
│  8. Integrate via SDK or API                                    │
│     ├── npm install thinkbase (React Chat widget)               │
│     ├── ThinkBaseClient JS class (programmatic access)          │
│     └── REST API with your API key                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Architecture Overview

```
                    ┌──────────────────────────┐
                    │       End Users /         │
                    │     Client Websites       │
                    │  (SDK Chat Widget / API)  │
                    └────────────┬─────────────┘
                                 │
                    ┌────────────▼─────────────┐
                    │     ThinkBase SDK         │
                    │   (npm: thinkbase)        │
                    │  • Chat React Component   │
                    │  • ThinkBaseClient Class   │
                    └────────────┬─────────────┘
                                 │ REST API
         ┌───────────────────────┼───────────────────────┐
         │                       │                       │
┌────────▼────────┐   ┌─────────▼─────────┐   ┌────────▼────────┐
│   Frontend      │   │    Backend API     │   │  Mobile Admin   │
│   (Next.js 15)  │   │   (NestJS 11)     │   │  App (GraphQL)  │
│   Port: 4000    │   │   Port: 3000      │   │                 │
│                 │   │                   │   │  Queries via    │
│  • Landing Page │   │  • Auth Module    │   │  Apollo Server  │
│  • Dashboard    │   │  • Project Module │   │                 │
│  • Project Mgmt │   │  • Chat Module    │   └────────┬────────┘
│  • Playground   │   │  • User Module    │            │
│  • Doc Upload   │   │  • Mobile Module  │            │
│  • Auth Pages   │   │  • Swagger Docs   │            │
└────────┬────────┘   └─────────┬─────────┘            │
         │                      │                       │
         │            ┌─────────▼───────────────────────▼─┐
         │            │       PostgreSQL + pgvector        │
         │            │                                    │
         └────────────►  • Users table                     │
                      │  • Projects table                  │
                      │  • Documents table                 │
                      │  • VectorDB table (3072-dim vecs)  │
                      │  • ApiKeyModel table               │
                      │  • Chat table                      │
                      └────────────────────────────────────┘
                                      │
                      ┌───────────────▼────────────────┐
                      │     Google Gemini AI APIs       │
                      │                                │
                      │  • gemini-embedding-001         │
                      │    (document & query embedding) │
                      │                                │
                      │  • gemini-2.5-flash             │
                      │    (chat response generation)   │
                      └────────────────────────────────┘
```

---

## Project Structure

```
ThinkBase/
├── .github/
│   └── workflows/
│       └── deploy.yml              # CI/CD: GitHub Actions → Docker Hub → Azure
│
├── backend/                        # NestJS 11 API Server
│   ├── prisma/
│   │   ├── schema.prisma           # Database schema (6 models)
│   │   └── migrations/             # 15 migration files
│   ├── src/
│   │   ├── main.ts                 # App bootstrap (Swagger, CORS, cookies)
│   │   ├── app.module.ts           # Root module (all imports)
│   │   ├── auth/                   # Authentication module
│   │   │   ├── auth.controller.ts  # 11 auth endpoints
│   │   │   ├── auth.service.ts     # Auth business logic
│   │   │   ├── auth.guard.ts       # JWT verification guard
│   │   │   ├── auth.module.ts      # Module configuration
│   │   │   ├── strategies/         # Passport strategies (Google, GitHub)
│   │   │   ├── mailService/        # Email service (Nodemailer)
│   │   │   └── giveAccsessSameSiteNone/ # Cookie interceptor
│   │   ├── project/                # Project & document management
│   │   │   ├── project.controller.ts  # 13 project endpoints
│   │   │   ├── project.service.ts     # Core RAG logic (embeddings, CRUD)
│   │   │   ├── project.guard.ts       # Project access guard
│   │   │   └── Dtos/                  # Data Transfer Objects
│   │   ├── chat/                   # AI chat module
│   │   │   ├── chat.controller.ts  # 4 chat endpoints
│   │   │   ├── chat.service.ts     # RAG query pipeline + LLM
│   │   │   └── chat.guard.ts       # API key validation guard
│   │   ├── user/                   # User management
│   │   │   ├── user.controller.ts  # User endpoints
│   │   │   ├── user.service.ts     # User CRUD
│   │   │   └── Dtos/               # CreateUser, LoginUser DTOs
│   │   ├── mobile-app/             # GraphQL module for mobile admin
│   │   │   ├── mobile-app.resolver.ts  # GraphQL resolvers
│   │   │   ├── mobile-app.service.ts   # Mobile data queries
│   │   │   └── data-types/              # GraphQL type definitions
│   │   ├── prisma/                 # Prisma service (DB connection)
│   │   ├── config/                 # App configuration
│   │   ├── middleware/             # Custom middleware
│   │   └── schema.gql             # Auto-generated GraphQL schema
│   ├── Dockerfile                  # Multi-stage Docker build
│   └── package.json
│
├── frontend/                       # Next.js 15 Web Application
│   ├── src/
│   │   ├── app/
│   │   │   ├── page.tsx            # Landing page (public)
│   │   │   ├── layout.tsx          # Root layout (navbar, global state)
│   │   │   ├── globals.css         # Global styles
│   │   │   ├── login/              # Login page
│   │   │   ├── signup/             # Signup page
│   │   │   ├── magic-link/         # Email verification page
│   │   │   ├── Dashboard/          # Protected dashboard area
│   │   │   │   ├── page.tsx        # Dashboard home (welcome, projects)
│   │   │   │   ├── layout.tsx      # Auth-protected layout wrapper
│   │   │   │   ├── project/        # Individual project workspace
│   │   │   │   └── settings/       # User settings
│   │   │   ├── Documentation/      # SDK documentation page
│   │   │   ├── ProductionChatComponentForClient/ # Embeddable chat demo
│   │   │   └── test/               # Test pages
│   │   ├── components/
│   │   │   ├── Chat.jsx            # Full chat interface component
│   │   │   ├── DocumentUpload.tsx  # Drag-and-drop file upload
│   │   │   ├── DocumentChunkAdjust.tsx # Chunk size/overlap config UI
│   │   │   ├── DocumentsList.tsx   # Document management list
│   │   │   ├── ApiKeyGenaration.tsx    # API key management UI
│   │   │   ├── PlaygroundAndCustomisation.tsx # Playground + prompt editor
│   │   │   ├── CodeSnippet.tsx     # SDK integration code snippets
│   │   │   ├── ProjectCard.tsx     # Project card component
│   │   │   ├── ProjectsSection.tsx # Projects grid section
│   │   │   ├── CreateNewProject.tsx    # New project modal
│   │   │   ├── ConfigureTab.tsx    # Document configuration tab
│   │   │   ├── LandingPageComponents/ # 10 landing page sections
│   │   │   │   ├── HeroSection.tsx
│   │   │   │   ├── WhatIsThinkBase.jsx
│   │   │   │   ├── Features.tsx
│   │   │   │   ├── UseCase.tsx
│   │   │   │   ├── StartBuilding.tsx
│   │   │   │   ├── Pricing.tsx
│   │   │   │   ├── ReadyTo.tsx
│   │   │   │   ├── Footer.tsx
│   │   │   │   └── BetaLabel.tsx
│   │   │   └── ui/                 # Radix-based UI primitives
│   │   ├── store/                  # Zustand global state
│   │   ├── lib/                    # Utility libraries
│   │   └── utils/                  # Helper functions
│   ├── public/                     # Static assets
│   ├── Dockerfile                  # Multi-stage Docker build
│   └── package.json
│
└── SDK/                            # ThinkBase npm package ("thinkbase")
    ├── src/
    │   ├── index.js                # Package entry (exports)
    │   ├── Client.js               # ThinkBaseClient class
    │   ├── Chat.jsx                # Embeddable React Chat widget
    │   └── HelloWorld.jsx          # Test component
    ├── rollup.config.js            # Rollup bundler config
    ├── dist/                       # Built output
    └── package.json                # Published as "thinkbase" on npm
```

---

## Tech Stack

### Backend

| Technology               | Version   | Purpose                                                  |
| ------------------------ | --------- | -------------------------------------------------------- |
| **NestJS**               | 11        | Node.js framework — modular architecture, DI, decorators |
| **Prisma ORM**           | 6.16      | Type-safe database client with migration support         |
| **PostgreSQL**           | —         | Primary database with `pgvector` extension               |
| **LangChain**            | 0.3       | RAG orchestration — text splitting, embeddings           |
| **Google Gemini**        | 2.5 Flash | LLM for chat responses                                   |
| **Gemini Embedding-001** | —         | 3072-dimensional vector embeddings                       |
| **Passport.js**          | 0.7       | Authentication strategies (JWT, Google, GitHub)          |
| **NestJS JWT**           | 11        | JWT token signing and verification                       |
| **Apollo Server**        | 5.2       | GraphQL server for mobile app                            |
| **Swagger**              | 11.2      | Auto-generated REST API documentation                    |
| **Nodemailer**           | 7         | Email service for magic-link verification                |
| **bcrypt**               | 6         | Password hashing (10 salt rounds)                        |
| **Express**              | 5         | HTTP server (via NestJS platform-express)                |

### Frontend

| Technology          | Version | Purpose                                              |
| ------------------- | ------- | ---------------------------------------------------- |
| **Next.js**         | 15.5    | React framework with Turbopack                       |
| **React**           | 19      | UI library                                           |
| **Tailwind CSS**    | 4       | Utility-first CSS framework                          |
| **Zustand**         | 5       | Lightweight state management                         |
| **Framer Motion**   | 12      | Declarative animations                               |
| **GSAP**            | 3.13    | Advanced animation library                           |
| **Radix UI**        | —       | Accessible UI primitives (scroll-area, select, slot) |
| **HeroUI**          | —       | Pre-built UI components (alert, slider)              |
| **Lucide React**    | —       | Icon library                                         |
| **react-pdftotext** | 1.3     | Client-side PDF text extraction                      |
| **react-dropzone**  | 14.3    | Drag-and-drop file upload                            |
| **Axios**           | 1.12    | HTTP client                                          |
| **Zod**             | 4       | Runtime type validation                              |
| **Lenis**           | 1.3     | Smooth scroll library                                |

### SDK

| Technology | Version | Purpose                        |
| ---------- | ------- | ------------------------------ |
| **Rollup** | 4       | Module bundler                 |
| **Babel**  | 7       | JSX transpilation              |
| **Axios**  | 1.12    | HTTP requests to ThinkBase API |

### DevOps & Infrastructure

| Technology               | Purpose                                       |
| ------------------------ | --------------------------------------------- |
| **Docker**               | Multi-stage container builds (Node 22 Alpine) |
| **Docker Hub**           | Container registry                            |
| **Azure Container Apps** | Production hosting (auto-scaling)             |
| **GitHub Actions**       | CI/CD pipeline                                |

---

## Backend — Deep Dive

### Module Architecture

The backend follows NestJS's modular architecture pattern. Each domain concern is encapsulated in its own module:

```
AppModule (Root)
├── AuthModule          → Authentication & authorization
├── UserModule          → User CRUD operations
├── ProjectModule       → Projects, documents, API keys, embeddings
├── ChatModule          → AI chat with RAG retrieval
├── MobileAppModule     → GraphQL API for mobile admin
├── ConfigModule        → Environment configuration (global)
└── GraphQLModule       → Apollo Server setup (auto-schema)
```

The `main.ts` bootstrap configures:

- **Swagger UI** at `/api` — interactive API documentation
- **Global ValidationPipe** — auto-validates DTOs (whitelist mode, rejects unknown fields)
- **Cookie Parser** — handles HTTP-only refresh token cookies
- **CORS** — allows all origins with credentials
- **Body size limits** — 50MB for document uploads, 100KB for everything else
- **Cookie Options Interceptor** — sets `SameSite=None; Secure` for cross-origin cookie support

### Authentication System

ThinkBase supports **four authentication methods**:

#### 1. Email + Password (with Magic Link Verification)

```
Signup Flow:
User submits email + password
  → Password hashed with bcrypt (auto salt)
  → User created in DB (isEmailVerified = false)
  → JWT magic-link token generated (2min expiry)
  → Verification email sent via Nodemailer
  → User clicks link → frontend /magic-link page
  → Token validated → isEmailVerified set to true
  → Refresh token (7-day) set as HTTP-only cookie

Login Flow:
User submits email + password
  → Lookup user by email
  → Verify email is confirmed
  → bcrypt.compare password
  → Issue 7-day refresh token as HTTP-only cookie
```

#### 2. Google OAuth 2.0

```
User clicks "Sign in with Google"
  → Redirected to Google consent screen
  → Google callback hits /auth/google/callback
  → Passport extracts profile
  → User created/updated in DB
  → Refresh token set as cookie
  → Redirect to /Dashboard
```

#### 3. GitHub OAuth

```
User clicks "Sign in with GitHub"
  → Redirected to GitHub authorization
  → GitHub callback hits /auth/github/callback
  → Passport extracts profile
  → User created/updated in DB
  → Refresh token set as cookie
  → Redirect to /Dashboard
```

#### 4. Magic Link Re-send

If the magic-link expires (2-minute window), users can request a new one. The system decodes the expired token (without verification) to extract the userId/email, then signs a fresh token.

#### Token Strategy

| Token                | Type | Expiry     | Storage                                    | Purpose             |
| -------------------- | ---- | ---------- | ------------------------------------------ | ------------------- |
| **Refresh Token**    | JWT  | 7 days     | HTTP-only cookie (`SameSite=None; Secure`) | Session persistence |
| **Access Token**     | JWT  | 15 minutes | In-memory (Zustand)                        | API authorization   |
| **Magic Link Token** | JWT  | 2 minutes  | URL parameter                              | Email verification  |

The `VerifyAuthGuard` handles token rotation:

1. Extract access token from `Authorization: Bearer <token>` header
2. Verify the access token
3. If expired → extract refresh token from cookies → verify → issue new access token
4. Attach `user` and `newAccessToken` to the request object

### Project & Document Management

A **Project** is the core organizational unit. Each user can have multiple projects, and each project has:

- A **name** and **description**
- A **system prompt** (defines AI behavior)
- Multiple **documents** (uploaded files)
- Multiple **API keys** (for external access)

#### Document Upload Flow

```
1. Frontend: User drags files into react-dropzone
2. Frontend: PDF → text extraction (react-pdftotext)
3. Frontend: Sends { projectId, userId, files: [{ fileName, fileText }] }
4. Backend:  Validates file count limit (max 10 per project)
5. Backend:  Creates Documents records (status: "Need_to_configure")
6. Backend:  Returns success
```

#### Document Processing (Embedding Generation)

```
1. User adjusts chunk size and overlap per document
2. User clicks "Process" → triggers EmberdingAndUpdateDocumentTable
3. For each document:
   a. Fetch raw text from Documents table
   b. Split into chunks via RecursiveCharacterTextSplitter
      • Configurable chunkSize (default: 500)
      • Configurable chunkOverlap (default: 50)
   c. For each chunk:
      • Generate 3072-dim embedding via Gemini Embedding-001
      • INSERT into VectorDB table (raw SQL for pgvector support)
      • Vector stored as pgvector column type
   d. Update document status to "Processed"
4. Transaction timeout: 30 minutes (for large documents)
```

#### Document Deletion

Cascading delete within a Prisma transaction:

1. Delete all VectorDB records linked to the document
2. Delete the Document record

#### Project Deletion

Full cascade within a transaction:

1. Delete all VectorDB records for the project
2. Delete all Documents for the project
3. Delete all ApiKeyModel records for the project
4. Delete the Project record

### RAG Pipeline — The Core Engine

This is the heart of ThinkBase. When a user sends a message, the following pipeline executes:

```
       User Message
            │
            ▼
   ┌──────────────────┐
   │  1. Query         │  Split user message into chunks
   │     Chunking      │  (RecursiveCharacterTextSplitter)
   │                   │  chunkSize: 1000, overlap: 200
   └────────┬─────────┘
            │
            ▼
   ┌──────────────────┐
   │  2. Query         │  Embed each chunk using
   │     Embedding     │  Gemini Embedding-001
   │                   │  → 3072-dim vector
   └────────┬─────────┘
            │
            ▼
   ┌──────────────────┐
   │  3. Vector        │  For each embedded chunk:
   │     Search        │  SQL cosine distance query
   │                   │  against VectorDB table
   │  WHERE projectId  │  (<=>) operator (pgvector)
   │  ORDER BY dist    │  Limit 5 results per chunk
   │  LIMIT 5          │  Filter: distance < 0.6
   └────────┬─────────┘
            │
            ▼
   ┌──────────────────┐
   │  4. Context       │  Collect all matching chunks
   │     Assembly      │  into a context string
   └────────┬─────────┘
            │
            ▼
   ┌──────────────────┐
   │  5. Conversation  │  Load chat history from DB
   │     Memory        │  (ordered by createdAt ASC)
   │                   │  Map to HumanMessage/AIMessage
   └────────┬─────────┘
            │
            ▼
   ┌──────────────────┐
   │  6. Prompt        │  Assemble final message array:
   │     Construction  │  [SystemMessage, ...history,
   │                   │   HumanMessage with context]
   │                   │
   │  System: {prompt} │  "Write a clear and concise
   │  + constraints    │   message. No markdown/HTML.
   │                   │   Under 500 chars."
   │                   │
   │  Human: "Context: │
   │   {retrieved docs} │
   │   Question: {msg}" │
   └────────┬─────────┘
            │
            ▼
   ┌──────────────────┐
   │  7. LLM Call      │  Invoke Gemini 2.5 Flash
   │                   │  temperature: 0
   │                   │  maxRetries: 2
   └────────┬─────────┘
            │
            ▼
   ┌──────────────────┐
   │  8. Persist       │  Save user message + AI response
   │     to DB         │  to Chat table
   │                   │  Update API key request count
   └────────┬─────────┘
            │
            ▼
      AI Response
```

#### Key Technical Details

- **Embedding Model**: `gemini-embedding-001` — produces 3072-dimensional vectors
- **LLM Model**: `gemini-2.5-flash` — fast, efficient, temperature=0 (deterministic)
- **Similarity Metric**: Cosine distance via pgvector's `<=>` operator
- **Relevance Threshold**: Only chunks with distance < 0.6 are included as context
- **Top-K Retrieval**: 5 most similar chunks per query chunk
- **Memory**: Full conversation history is loaded from the database on each request

### Chat System

The chat system tracks conversations using a `clientId` cookie:

| Mode              | Client Identification                 | Use Case                     |
| ----------------- | ------------------------------------- | ---------------------------- |
| **Web (non-SDK)** | `clientId` cookie + `apikeyId` filter | Built-in playground testing  |
| **SDK mode**      | `clientId` cookie only                | External website integration |

**Endpoints**:

- `POST /chat/sendMessage` — Send a message, get AI response (guarded by `ChatGuard`)
- `POST /chat/getAllMessagesByClientId` — Retrieve conversation history
- `POST /chat/deleteAllMessagesByClientId` — Clear conversation

The `ChatGuard` validates that the provided API key exists and is active before allowing chat requests.

### API Key Management

API keys are the gateway for external access to ThinkBase chatbots:

- **Generation**: 32-byte cryptographically random hex string (64 characters)
- **Storage**: Primary key in the `ApiKeyModel` table
- **Tracking**: Each key tracks `requestsCount` and `lastUsedAt`
- **Scoping**: Each key is bound to a specific project
- **Deletion**: When an API key is deleted, associated chat records have their `apikeyId` set to null (soft disassociation)

### GraphQL API for Mobile

The `MobileAppModule` exposes a GraphQL API via Apollo Server for the ThinkBase Mobile Admin App:

**Schema** (`schema.gql`):

```graphql
type Query {
  getUsers: [User!]!
  getAlltProjects: [Project!]!
}

type Mutation {
  getEachUser(data: UserId!): User!
  getEachProject(data: ProjectId!): Project!
}
```

**Key features of the mobile API**:

- Returns deeply nested data including projects, documents, API keys, and chat statistics
- `uniqueClientCount` — count of distinct chat clients per API key
- Includes document processing status and configuration details
- Used by the React Native mobile admin application

### REST API Reference

#### Auth Endpoints (`/auth`)

| Method | Endpoint                     | Auth         | Description                          |
| ------ | ---------------------------- | ------------ | ------------------------------------ |
| `POST` | `/auth/signup`               | Public       | Register with email + password       |
| `POST` | `/auth/login`                | Public       | Login with credentials               |
| `POST` | `/auth/logout`               | Public       | Clear refresh token cookie           |
| `POST` | `/auth/verify`               | Bearer Token | Verify session, get new access token |
| `GET`  | `/auth/google`               | Public       | Initiate Google OAuth                |
| `GET`  | `/auth/google/callback`      | Google OAuth | Google OAuth callback                |
| `GET`  | `/auth/github`               | Public       | Initiate GitHub OAuth                |
| `GET`  | `/auth/github/callback`      | GitHub OAuth | GitHub OAuth callback                |
| `POST` | `/auth/magic-link/validate`  | Public       | Validate email magic link            |
| `POST` | `/auth/reSendEmail`          | Public       | Resend verification email            |
| `POST` | `/auth/homePageVerification` | Cookie       | Check if user is logged in           |

#### Project Endpoints (`/project`) — All guarded by `ProjectGuard`

| Method | Endpoint                                   | Description                             |
| ------ | ------------------------------------------ | --------------------------------------- |
| `POST` | `/project/createProject`                   | Create a new project                    |
| `POST` | `/project/getProjectsByUserIdAndProjectId` | Get specific project                    |
| `POST` | `/project/getProjects/userId`              | Get all projects for a user             |
| `POST` | `/project/document/upload`                 | Upload documents (50MB limit)           |
| `POST` | `/project/documents/getall`                | Get all documents for a project         |
| `POST` | `/project/documents/getAll/forConfig`      | Get unprocessed documents               |
| `POST` | `/project/updatedDocuments/emberdings`     | Process documents (generate embeddings) |
| `POST` | `/project/update/systemPrompt`             | Update project system prompt            |
| `POST` | `/project/get/prompt`                      | Get project system prompt               |
| `POST` | `/project/apikeys/getall`                  | List all API keys                       |
| `POST` | `/project/apikeys/create`                  | Generate new API key                    |
| `POST` | `/project/apikeys/detele`                  | Delete an API key                       |
| `POST` | `/project/document/delete`                 | Delete a document                       |
| `POST` | `/project/delete`                          | Delete entire project                   |

#### Chat Endpoints (`/chat`)

| Method | Endpoint                            | Auth                | Description                   |
| ------ | ----------------------------------- | ------------------- | ----------------------------- |
| `POST` | `/chat/sendMessage`                 | API Key (ChatGuard) | Send message, get AI response |
| `POST` | `/chat/getAllMessagesByClientId`    | Cookie              | Get conversation history      |
| `POST` | `/chat/deleteAllMessagesByClientId` | Cookie              | Clear conversation            |

#### User Endpoints (`/user`)

| Method | Endpoint                | Description            |
| ------ | ----------------------- | ---------------------- |
| `GET`  | `/user/getEachUser/:id` | Get user details by ID |

---

## Frontend — Deep Dive

### Pages & Routing

```
/                           → Landing page (public, server-rendered)
/login                      → Login page (email/password, OAuth buttons)
/signup                     → Registration page
/magic-link?token=xxx       → Email verification handler
/Dashboard                  → Protected dashboard home
/Dashboard/project/[id]     → Individual project workspace
/Dashboard/settings         → User settings
/Documentation              → SDK documentation
/ProductionChatComponentForClient → Embeddable chat demo
```

### Landing Page

The landing page (`/`) is server-rendered and consists of 10 animated sections:

1. **BetaLabel** — Beta version indicator
2. **HeroSection** — Main headline, CTA buttons
3. **WhatIsThinkBase** — Product explanation
4. **Features** — Key feature highlights
5. **StartBuilding** — Getting-started guide
6. **UseCase** — Industry use case examples
7. **ReadyTo** — CTA section
8. **Footer** — Links, copyright

The page uses:

- **GSAP + Framer Motion** for scroll-triggered animations
- **Lenis** for smooth scrolling
- Premium dark theme with blue accent gradients
- Glassmorphism blur effects (`blur-[300px]` on gradient orbs)

### Dashboard

The Dashboard is a **protected area** that requires authentication:

**Auth Flow in Dashboard Layout:**

```
1. Component mounts
2. Check if userId exists in Zustand store
3. If no userId → call POST /auth/verify with Bearer token
4. If token valid → store userId and new access token
5. If token invalid → redirect to /login
6. Show loading spinner during verification
```

**Dashboard Home (`/Dashboard`)**:

- Welcome message with user's name
- Projects grid showing all user projects
- Each project card displays:
  - Project name
  - Total processed documents count
  - Total API request count (sum across all API keys)
- "Create Project" overlay modal

### Project Workspace

The project workspace (`/Dashboard/project/[id]`) is the main working area with multiple tabs:

#### Documents Tab

- **DocumentUpload** — Drag-and-drop file upload using `react-dropzone`
  - Supports PDF (parsed via `react-pdftotext`) and text files
  - Files are read client-side and text is extracted before sending
  - Max 10 documents per project
- **DocumentsList** — Shows all uploaded documents with status indicators
  - Status: `Need_to_configure` (orange) or `Processed` (green)
  - Delete button per document
- **ConfigureTab** — Shows unprocessed documents
  - **DocumentChunkAdjust** — Slider UI to adjust chunk size and overlap per document
  - Chunk size range: configurable
  - Overlap range: configurable
  - "Process" button to trigger embedding generation

#### Playground & Customisation Tab

- **PlaygroundAndCustomisation** — 978-line component with:
  - **System prompt editor** — textarea to customize AI behavior
  - **API Key management** — generate, view, delete API keys
  - **Chat playground** — embedded chat interface for testing
  - **CodeSnippet** — shows integration code for the SDK

#### Chat Component

- **Chat.jsx** — Full chat interface (21KB):
  - Message list with user/AI bubbles
  - Input field with send button
  - Loading states
  - Conversation history
  - Delete conversation functionality
  - Timestamp display
  - Auto-scroll to latest message

### State Management

The frontend uses **Zustand** for global state management via a single store (`store/globalStore`):

| State       | Type             | Purpose                          |
| ----------- | ---------------- | -------------------------------- |
| `token`     | `string \| null` | Current access token (in-memory) |
| `userid`    | `string \| null` | Authenticated user ID            |
| `useName`   | `string \| null` | User's display name              |
| `userEmail` | `string \| null` | User's email                     |

Methods: `setToken`, `setUserId`, `setUserName`, `setUserEmail`

---

## ThinkBase SDK (npm Package)

The SDK (`npm: thinkbase`) allows developers to integrate ThinkBase chatbots into their own websites. Published on npm and bundled with Rollup.

### Package Exports

```javascript
export { Chat } from "./Chat"; // React Chat widget component
export { default as ThinkBaseClient } from "./Client"; // Programmatic client
export { HelloWorld } from "./HelloWorld"; // Test component
```

### ThinkBaseClient API

The `ThinkBaseClient` class provides programmatic access to ThinkBase chatbots:

```javascript
import { ThinkBaseClient } from "thinkbase";

const client = new ThinkBaseClient({
  apiKey: "your-api-key-here",
  baseUrl: "https://api.think-base.dev", // optional, this is the default
});

// Send a message and get AI response
const response = await client.sendMessage("What is your return policy?");
console.log(response);
// { aiMessage: "Based on our policy...", time: "2026-02-17T...", status: true }

// Get all messages in the conversation
const messages = await client.getMessages();
console.log(messages);
// [{ id, message, sender, time, status }, ...]

// Clear conversation history
const result = await client.deleteMessages();
// { status: true }
```

**Client Methods**:

| Method              | Returns                                   | Description                                 |
| ------------------- | ----------------------------------------- | ------------------------------------------- |
| `sendMessage(text)` | `{ aiMessage, time, status }`             | Send a user message, receive AI response    |
| `getMessages()`     | `[{ id, message, sender, time, status }]` | Retrieve full conversation history          |
| `deleteMessages()`  | `{ status }`                              | Delete all messages for this client session |

**How it works internally**:

- Uses `axios` with `withCredentials: true` for cookie-based session tracking
- The `clientId` cookie is automatically managed by the backend
- Sets `SDK: true` flag to differentiate from playground requests

### Chat React Component

The SDK also exports a drop-in React `<Chat>` component that provides a complete chat UI:

```jsx
import { Chat } from "thinkbase";

function App() {
  return <Chat apiKey="your-api-key" baseUrl="https://api.think-base.dev" />;
}
```

The Chat component includes:

- Full chat interface with message bubbles
- Input field with send button
- Loading indicators
- Conversation history
- Styled UI (styled-jsx)
- Auto-scroll behavior
- Error handling

---

## Database Schema

The database uses **PostgreSQL** with the **pgvector** extension for vector similarity search.

### Entity Relationship Diagram

```
┌─────────────┐     1:N     ┌──────────────┐     1:N     ┌──────────────┐
│    User      │────────────▶│   Project     │────────────▶│  Documents   │
├─────────────┤             ├──────────────┤             ├──────────────┤
│ id (cuid)   │             │ id (cuid)     │             │ documentsId  │
│ email (uniq)│             │ name          │             │ name         │
│ name        │             │ description   │             │ text         │
│ password    │             │ files (JSON)  │             │ chankSize    │
│ photo       │             │ systemPrompt  │             │ overlap      │
│ refreshToken│             │ createdAt     │             │ status       │
│ isEmailVer. │             │ updatedAt     │             │ createdAt    │
│ createdAt   │             │ userId (FK)   │             │ updatedAt    │
│ updatedAt   │             └──────┬───────┘             │ projectId(FK)│
└─────────────┘                    │                      └──────┬───────┘
                                   │                             │
                            1:N    │                      1:N    │
                                   │                             │
                            ┌──────▼───────┐             ┌──────▼───────┐
                            │ ApiKeyModel   │             │  VectorDB    │
                            ├──────────────┤             ├──────────────┤
                            │ apikey (PK)  │             │ id (cuid)    │
                            │ projectId(FK)│             │ type         │
                            │ name         │             │ chunk        │
                            │ createdAt    │             │ geminiFlash  │
                            │ requestCount │             │  Embeddings  │
                            │ lastUsedAt   │             │  (vector     │
                            └──────┬───────┘             │   3072)      │
                                   │                     │ documentId   │
                            1:N    │                     │  (FK)        │
                                   │                     │ projectsid   │
                            ┌──────▼───────┐             │  (FK)        │
                            │    Chat      │             └──────────────┘
                            ├──────────────┤
                            │ id (cuid)    │
                            │ clientId     │
                            │ message      │
                            │ sender       │
                            │ createdAt    │
                            │ apikeyId(FK) │
                            └──────────────┘
```

### Models Detail

#### User

| Field             | Type          | Notes                                          |
| ----------------- | ------------- | ---------------------------------------------- |
| `id`              | String (CUID) | Primary key, auto-generated                    |
| `email`           | String        | Unique constraint                              |
| `name`            | VarChar(255)  | Optional                                       |
| `password`        | String        | Optional (null for OAuth users), bcrypt hashed |
| `photo`           | String        | Profile photo URL (from OAuth)                 |
| `refreshToken`    | String        | Stored on email verification                   |
| `isEmailVerified` | Boolean       | Default: false                                 |

#### Project

| Field          | Type          | Notes                                          |
| -------------- | ------------- | ---------------------------------------------- |
| `id`           | String (CUID) | Primary key                                    |
| `name`         | VarChar(255)  | Required                                       |
| `description`  | Text          | Optional                                       |
| `files`        | JSON          | Legacy field (documents now in separate table) |
| `systemPrompt` | Text          | Default: "You are a helpful assistant..."      |
| `userId`       | String (FK)   | References User.id                             |

#### Documents

| Field         | Type          | Notes                                   |
| ------------- | ------------- | --------------------------------------- |
| `documentsId` | String (CUID) | Primary key                             |
| `name`        | VarChar(255)  | File name                               |
| `text`        | Text          | Full extracted text content             |
| `chankSize`   | Int           | Chunk size for splitting (default: 500) |
| `overlap`     | Int           | Chunk overlap (default: 50)             |
| `status`      | VarChar(100)  | `Need_to_configure` or `Processed`      |
| `projectId`   | String (FK)   | References Project.id                   |

#### VectorDB

| Field                   | Type          | Notes                                     |
| ----------------------- | ------------- | ----------------------------------------- |
| `id`                    | String (CUID) | Primary key                               |
| `type`                  | VarChar(100)  | Currently always "text"                   |
| `chunk`                 | Text          | The text chunk                            |
| `geminiFlashEmbeddings` | vector(3072)  | pgvector column — Unsupported Prisma type |
| `documentId`            | String (FK)   | References Documents.documentsId          |
| `projectsid`            | String (FK)   | References Project.id                     |

#### ApiKeyModel

| Field           | Type         | Notes                            |
| --------------- | ------------ | -------------------------------- |
| `apikey`        | String       | Primary key (64-char hex)        |
| `projectId`     | String (FK)  | References Project.id            |
| `name`          | VarChar(255) | Optional display name            |
| `requestsCount` | Int          | Incremented on each chat request |
| `lastUsedAt`    | DateTime     | Updated on each chat request     |

#### Chat

| Field       | Type          | Notes                                    |
| ----------- | ------------- | ---------------------------------------- |
| `id`        | String (CUID) | Primary key                              |
| `clientId`  | VarChar(255)  | Cookie-based client identifier           |
| `message`   | Text          | Message content                          |
| `sender`    | String        | `"user"` or `"ai"`                       |
| `createdAt` | DateTime      | Auto-set                                 |
| `apikeyId`  | String (FK)   | References ApiKeyModel.apikey (nullable) |

---

## Deployment & CI/CD Pipeline

ThinkBase uses a fully automated CI/CD pipeline powered by **GitHub Actions**, deploying to **Azure Container Apps** via **Docker Hub**.

### Pipeline Trigger

Deployments are triggered on push to the `deployement_t1` branch.

### Pipeline Steps

```
GitHub Push (deployement_t1 branch)
        │
        ▼
┌─────────────────────────────────────────┐
│         GitHub Actions Workflow          │
├─────────────────────────────────────────┤
│                                         │
│  1. Checkout repository                 │
│  2. Login to Azure (service principal)  │
│  3. Login to Docker Hub                 │
│                                         │
│  ═══ BACKEND ═══                        │
│  4. Build backend Docker image          │
│     (hashalagayendra/thinkbase-backend) │
│  5. Push to Docker Hub                  │
│  6. Configure Azure Container App       │
│     registry (Docker Hub credentials)   │
│  7. Update Azure Container App with:    │
│     • New image                         │
│     • All environment variables         │
│  8. Update ingress (port 3000, ext.)    │
│  9. Show container app URL              │
│                                         │
│  ═══ FRONTEND ═══                       │
│  10. Build frontend Docker image        │
│      (with build-args for env vars)     │
│      (hashalagayendra/thinkbase-        │
│       frontend)                         │
│  11. Push to Docker Hub                 │
│  12. Update Azure Container App         │
│  13. Update ingress (port 4000, ext.)   │
│                                         │
└─────────────────────────────────────────┘
```

### Docker Build Strategy

Both services use **multi-stage Docker builds** with `node:22-alpine`:

**Backend Dockerfile**:

```
Stage 1 (builder): npm install → prisma generate → nest build (1GB memory)
Stage 2 (production): npm install --only=production → copy dist + prisma client
Exposes: port 3000
CMD: node dist/main.js
```

**Frontend Dockerfile**:

```
Stage 1 (build): npm install → next build (with env vars as build args)
Stage 2 (production): copy .next + node_modules + public
Exposes: port 4000
CMD: npm start
```

### Infrastructure

| Component   | Azure Service                  | Port |
| ----------- | ------------------------------ | ---- |
| Backend API | Azure Container App            | 3000 |
| Frontend    | Azure Container App            | 4000 |
| Database    | Supabase PostgreSQL (pgvector) | —    |

---

## Environment Variables

### Backend (`backend/.env`)

| Variable                        | Description                                            |
| ------------------------------- | ------------------------------------------------------ |
| `DATABASE_URL`                  | PostgreSQL connection string (pooled/transaction mode) |
| `DIRECT_URL`                    | PostgreSQL direct connection string (for migrations)   |
| `JWT_SECRET`                    | Secret key for JWT signing                             |
| `JWT_EXPIRATION_TIME`           | Access token expiry (e.g., `900s`)                     |
| `REFRESH_TOKEN_EXPIRATION_TIME` | Refresh token expiry (e.g., `7d`)                      |
| `FRONTEND_URL`                  | Frontend base URL (for OAuth redirects, email links)   |
| `BACKEND_URL`                   | Backend base URL                                       |
| `Gemini_API_KEY`                | Google Gemini API key (for embeddings + chat)          |
| `GOOGLE_CLIENT_ID`              | Google OAuth client ID                                 |
| `GOOGLE_CLIENT_SECRET`          | Google OAuth client secret                             |
| `GIT_HUB_CLIENT_ID`             | GitHub OAuth client ID                                 |
| `GIT_HUB_CLIENT_SECRET`         | GitHub OAuth client secret                             |
| `GIT_HUB_CALLBACK_URL`          | GitHub OAuth callback URL                              |
| `EMAIL_USER`                    | SMTP email address (for Nodemailer)                    |
| `EMAIL_PASS`                    | SMTP email password/app password                       |
| `PORT`                          | Server port (default: 3000)                            |

### Frontend (`frontend/.env`)

| Variable                        | Description                        |
| ------------------------------- | ---------------------------------- |
| `NEXT_PUBLIC_BACKEND_BASE_URL`  | Backend API URL (baked into build) |
| `NEXT_PUBLIC_FRONTEND_BASE_URL` | Frontend URL (baked into build)    |

---

## API Documentation (Swagger)

The backend auto-generates interactive API documentation using **Swagger/OpenAPI**:

- **URL**: `https://<backend-url>/api`
- **Features**:
  - Interactive try-it-out for all endpoints
  - Request/response schemas
  - Bearer auth support
  - DTO validation documentation

All endpoints are decorated with `@ApiTags`, `@ApiOperation`, `@ApiBody`, and `@ApiCreatedResponse` for comprehensive documentation.

---

## Security Considerations

| Concern                 | Implementation                                                             |
| ----------------------- | -------------------------------------------------------------------------- |
| **Password Storage**    | bcrypt with auto-generated salt                                            |
| **Token Storage**       | Refresh tokens in HTTP-only, Secure, SameSite=None cookies                 |
| **Access Token**        | Short-lived (15min), stored only in-memory (Zustand)                       |
| **API Key Generation**  | `crypto.randomBytes(32)` — 256 bits of entropy                             |
| **Input Validation**    | NestJS ValidationPipe with whitelist mode                                  |
| **CORS**                | All origins allowed with credentials (for SDK support)                     |
| **Request Size Limits** | 50MB for document uploads, 100KB for all other routes                      |
| **Email Verification**  | Required before login (magic-link with 2min expiry)                        |
| **Cookie Security**     | `httpOnly: true`, `secure: true`, `sameSite: 'none'`                       |
| **OAuth**               | Passport.js strategies for Google and GitHub                               |
| **Route Guards**        | `VerifyAuthGuard` (JWT), `ProjectGuard` (ownership), `ChatGuard` (API key) |
| **Database**            | Prisma parameterized queries (except raw pgvector inserts)                 |

---

<p align="center">
  <strong>Built with ❤️ by the ThinkBase Team</strong>
</p>
<p align="center">
  <em>Powered by Google Gemini • LangChain • NestJS • Next.js • PostgreSQL + pgvector</em>
</p>
