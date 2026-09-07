# Chat App

A real-time, multi-user chat room built with **Spring Boot** and native **WebSockets**. Users log in with a display name, join a shared chat room, and exchange messages instantly with all connected participants — with a live "online users" counter.

## Overview

This project demonstrates a full-stack, event-driven web application: a Spring MVC layer handles login/session state, while a raw `javax/jakarta.websocket` endpoint (not STOMP/SockJS) manages real-time bidirectional messaging. It's a compact but complete example of session-aware WebSocket authentication, in-memory connection management, and JSON message broadcasting.

## Features

- **Simple username login** — no password, session-based identity (`HttpSession`)
- **Real-time group chat** over a native WebSocket endpoint (`/messages`)
- **Live online-user count** broadcast to all clients with every message
- **Session-to-WebSocket handshake binding** — the logged-in username is securely attached to the WebSocket handshake via a custom `Configurator`, so the server always knows who is speaking without trusting client-supplied identity
- **Server-side broadcast** — every incoming message is fanned out to all currently connected sessions
- Server-rendered UI with **Thymeleaf** templates (login page + chat room)

## Tech Stack

| Layer | Technology |
|---|---|
| Language | Java 17 |
| Framework | Spring Boot 3.5.3 |
| Web | Spring MVC, Thymeleaf |
| Real-time | Jakarta WebSocket API (`@ServerEndpoint`), Spring `ServerEndpointExporter` |
| JSON | Jackson (server), Fastjson (dependency) |
| Frontend | jQuery, MDUI, vanilla WebSocket client JS |
| Build | Maven (with Maven Wrapper) |

## Architecture

```
Browser (login.html) --POST /login--> HomeController --stores username in HttpSession
Browser (chat.html)  --GET /chat----> HomeController --renders page with username

Browser  --WebSocket handshake--> ServerWebSocket.Configurator
                                   (reads username from HttpSession, attaches to WS session)

Browser  --send JSON message--> ServerWebSocket.OnMessage
                                   --deserializes UserChatMessage
                                   --builds ServerMessage (adds live online count)
                                   --broadcasts to every open Session
```

Key classes:
- `HomeController` — serves the login and chat pages, and handles the login POST that stores the username in the session
- `ServerWebSocket` — the `@ServerEndpoint` at `/messages`; tracks connected sessions in a `ConcurrentHashMap`, binds identity at handshake time, and broadcasts messages on receipt
- `Users` — exposes a shared, thread-safe map bean for tracking online sessions
- `WebSocketConfig` — registers the `ServerEndpointExporter` so the raw `@ServerEndpoint` is picked up by Spring's embedded server
- `UserChatMessage` / `ServerMessage` / `MessageType` — the message DTOs and the `SPEAK` / `CONNECT` / `DISCONNECT` message types exchanged between client and server

## Getting Started

### Prerequisites
- Java 17+
- No local Maven install required (Maven Wrapper included)

### Run locally

```bash
# Clone the repo
git clone https://github.com/AI-Abdulgawad/chat_app.git
cd chat_app

# Run with the Maven wrapper
./mvnw spring-boot:run
```

The app starts on `http://localhost:8080`.

1. Open `http://localhost:8080` and enter a username to log in.
2. You'll be redirected to `/chat`.
3. Open a second browser tab (or an incognito window) with a different username to see real-time messaging and the live online-user count in action.

## Project Structure

```
src/main/java/com/udacity/chat_app/
├── ChatAppApplication.java        # Spring Boot entry point
├── config/
│   ├── Users.java                 # Bean: thread-safe map of online sessions
│   └── WebSocketConfig.java       # Registers the WebSocket endpoint exporter
├── controller/
│   ├── HomeController.java        # Login / chat page routing
│   └── ServerWebSocket.java       # WebSocket endpoint: connect, message, disconnect
└── model/
    ├── MessageType.java           # SPEAK / CONNECT / DISCONNECT
    ├── RedirectResponse.java      # Login response payload
    ├── ServerMessage.java         # Outgoing broadcast payload
    └── UserChatMessage.java       # Incoming client message payload

src/main/resources/
├── templates/                     # Thymeleaf views (login.html, chat.html)
├── static/img/                    # Login page background
└── application.properties
```

## What This Project Demonstrates

- Building real-time features in Spring Boot **without** a messaging broker, using the native Jakarta WebSocket API directly
- Securely propagating authenticated session state (HTTP session → WebSocket session) via a custom `ServerEndpointConfig.Configurator`
- Managing concurrent client connections safely with `ConcurrentHashMap`
- Designing simple client/server message contracts (DTOs + enum message types) for a bidirectional protocol
- Server-rendered front end (Thymeleaf) paired with a lightweight JS WebSocket client

## Possible Extensions

- Persist chat history (currently messages are not stored)
- Add authentication (passwords, OAuth) instead of open username entry
- Support multiple chat rooms/channels
- Add typing indicators and delivery/read receipts
- Replace raw WebSocket protocol with STOMP over SockJS for broader browser fallback support

---

*This is a demo project (originally built as part of a Udacity course) intended to showcase WebSocket and Spring Boot fundamentals.*
