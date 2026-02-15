# System Architecture

PingMyDb is designed as a distributed monitoring system consisting of three main layers: the User Interface, the API Server, and the Monitoring Engine.

## 1. High-Level Flow
1. **User Management**: Users sign up and configure database monitors (PostgreSQL, MySQL, or MongoDB).
2. **Task Scheduling**: The API server schedules "heartbeat" jobs using BullMQ.
3. **Execution**: Distributed workers pick up jobs from Redis and attempt to connect to the target database.
4. **Data Aggregation**: Results (latency, status) are stored in the main PostgreSQL database.
5. **Alerting**: If a check fails, the notification service evaluates the user's alert channels (Email, Discord, Slack) and dispatches alerts.

## 2. Component Breakdown

### Frontend (React + Vite)
- **SPA Architecture**: Handles all routing and UI rendering.
- **State Management**: Uses Context API for authentication and specialized stores (Zustand) for notifications.
- **Theme Engine**: Custom Tailwind-based system for dark/light mode persistence.

### Backend (Node.js + Express)
- **RESTful API**: Secure endpoints for managing monitors, teams, and profiles.
- **Middleware Layer**: Handles JWT verification and rate limiting.
- **Paywall Integration**: Razorpay SDK manages plan-specific usage limits.

### Monitoring & Queuing (BullMQ + Redis)
- **Job Persistence**: Jobs are stored in Redis to survive server restarts.
- **Concurrency**: Multiple workers can scale to handle thousands of concurrent pings.
- **Encryption**: CryptoJS (AES-256) is used to decrypt database URIs only at the moment of the check.

## 3. Data Flow Diagram
```text
[User] <--> [Frontend] <--> [Express API] <--> [PostgreSQL]
                                   |
                                [Redis/BullMQ]
                                   |
                                [Monitoring Workers] --> [External DBs]
```
