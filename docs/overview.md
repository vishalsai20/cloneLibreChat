# AIChat Overview

This document provides a **high-level technical overview** of AIChat (forked from LibreChat). It explains how the **frontend and backend interact**, along with a **database schema overview** to understand the entire system.

---

## 1. Code Flow: Frontend ↔ Backend Interaction

AIChat follows a client-server architecture:

* **Frontend:** Built with React, communicates via REST APIs & WebSockets.
* **Backend:** Node.js + Express server, acts as the API gateway, connects to Database & AI providers(OpenAI, Google, etc).
* **Database:** MongoDB (stores users, messages, presets, conversations).

1. **User Opens the App**  
   The browser loads the React frontend, which immediately requests initial configuration (available AI models, settings, etc.) from the backend.

2. **Frontend Fetches Config**  
   The React app calls `GET /config`.  
   The backend fetches app settings (presets, available providers, etc.) from MongoDB and returns them as JSON.

3. **User Sends a Message**  
   When the user types a message and clicks send, the frontend sends a `POST /chat` request to the backend.

4. **Backend Processes the Request**  
   The backend:
   * Stores the user's message in the **messages** collection.
   * Forwards the message to the selected **AI provider** (OpenAI, Anthropic, etc.).
   * Waits for the AI’s response.

5. **AI Response Returned**  
   The backend stores the AI response in the database and sends it back to the frontend.

6. **Frontend Updates the UI**  
   The React app updates the chat interface, displaying the AI’s reply to the user.

* We run the npm run background command on our terminal to start the application. Using the port:3080 starts the application.
* Frontend loads the React app assets from `client/` and fetches the config settings from `.env` like available AI providers, user settings.
* Backend server (Node.js) starts listening to this port. It connects Database for users, conversation history, messages data. Also connects to AI providers and loads middleware like auth.

* We lofin via the frontend forms (Credentials and user data are stored in DB).

* When we submit a prompt, the frontend sends a HTTP request to backend with the prompt, metadata, model, conversationID, etc
* Backend receives the request and validates it and checks for the previous messages in conversation (if any)
* Backend then sends message + context to the AI provider
* Once the backend recieves the response from the AI, it stores in the database and returns the response to the frontend and updates UI.


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

### Tables:

* **USERS** → One user can have many conversations.
* **CONVERSATIONS** → Each conversation stores multiple messages.
* **MESSAGES** → Linked to conversations for full history.
* **PRESETS** → Stores reusable model configurations.

* NOTE : There exists other tables also.

---

## 3. Notes

* Supports multiple AI providers (OpenAI, Azure, Anthropic, etc.).
* Backend handles authentication, rate-limiting, and persistence.
* Frontend is provider-agnostic: it only cares about responses from API.
