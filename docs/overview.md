# AIChat Project Overview

This document provides a **high-level technical overview** of AIChat (forked from LibreChat). It explains how the **frontend and backend interact**, along with a **database schema overview** to help contributors quickly understand the system.

---

## 1. Code Flow: Frontend ↔ Backend Interaction

AIChat follows a client-server architecture:

* **Frontend:** Built with React, communicates via REST APIs & WebSockets.
* **Backend:** Node.js + Express server, acts as the API gateway, connects to DB & AI providers.
* **Database:** MongoDB (stores users, messages, presets, conversations).

### Sequence Diagram

```mermaid
sequenceDiagram
    participant U as User (Browser)
    participant F as Frontend (React)
    participant B as Backend (Express API)
    participant DB as Database (MongoDB)
    participant AI as AI Provider (OpenAI / Others)

    U->>F: Opens AIChat UI
    F->>B: GET /config (Fetch settings)
    B->>DB: Query config & presets
    DB-->>B: Return settings
    B-->>F: Send JSON config
    F->>B: POST /chat (User sends message)
    B->>DB: Store message in conversation
    B->>AI: Forward message to selected AI model
    AI-->>B: Return AI response
    B->>DB: Store AI reply
    B-->>F: Send AI reply
    F-->>U: Display AI response in chat UI
```

This flow repeats for each user message. Streaming (if enabled) sends partial tokens to the UI in real-time.

---

## 2. Database Schema Overview

AIChat uses **MongoDB**, with collections focused on user data, chat history, and configuration.

### Entity-Relationship Diagram

```mermaid
erDiagram
    USERS ||--o{ CONVERSATIONS : "has"
    CONVERSATIONS ||--o{ MESSAGES : "contains"

    USERS {
        string _id
        string username
        string email
        string passwordHash
    }
    CONVERSATIONS {
        string _id
        string userId
        string title
        date createdAt
    }
    MESSAGES {
        string _id
        string conversationId
        string sender "values: user | ai"
        string text
        date timestamp
    }
    PRESETS {
        string _id
        string name
        json config
    }
```

### Key Points:

* **USERS** → One user can have many conversations.
* **CONVERSATIONS** → Each conversation stores multiple messages.
* **MESSAGES** → Linked to conversations for full history.
* **PRESETS** → Stores reusable model configurations.

---

## 3. Notes

* Supports multiple AI providers (OpenAI, Azure, Anthropic, etc.).
* Backend handles authentication, rate-limiting, and persistence.
* Frontend is provider-agnostic: it only cares about responses from API.

---

**This file should be kept updated** whenever the schema or major data flow changes.
