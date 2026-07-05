# Real-Time Multiplayer Quiz Game | Auth Service

## Overview

The **Real-Time Multiplayer Quiz Game** is a distributed application that allows users to compete against each other in real-time. The platform is built on a microservices architecture, consisting of two independent backend services and a frontend client.

This repository contains the **Auth Service**. It acts as the security gateway for the platform, strictly handling user registration, authentication, and secure access management by issuing JSON Web Tokens (JWT).

## Architecture Diagram

Below is the high-level architecture of the Real-Time Multiplayer Quiz Game, demonstrating the separation of concerns, the Database-per-Service pattern, and the integration of cloud services.

![Architecture Diagram](./assets/quiz-architecture.png)

### Component Interaction & Data Flow:

*   **Frontend (Next.js):** Communicates with the Auth Service via REST API for user registration and login. It connects to the Game Service via WebSockets for real-time gameplay synchronization.
*   **Auth Service (This Repo):** Processes credentials and issues stateless JSON Web Tokens (JWT). It maintains loose coupling by reading and writing exclusively to its own Auth DB.
*   **Game Service:** Handles core game logic and validates JWTs locally (without synchronous calls to the Auth Service), ensuring high performance and fault tolerance. It interacts exclusively with the Game DB.
*   **BaaS Integration (Leaderboard & Stats):** To optimize performance, the frontend bypasses the Java backend for read-only operations, fetching real-time leaderboard data directly from the Game DB using the Supabase Data API.
*   **Static Assets (CDN):** Question images are securely stored in AWS S3 and delivered globally with low latency via AWS CloudFront directly to the client.

## Platform Features

The project is packed with features designed for a seamless, competitive, and real-time user experience.

### Authentication & User Management
*   **Secure Access:** Full registration, login, and logout flows protected by JWT. *(Note: This specific repository **Auth Service** is strictly responsible for handling these security and identity operations).*
*   **Public Profiles:** Click on any player to view their detailed profile, including Total Games, Total Score, Total Wins, etc.

### Game Lobby
*   **Customizable Matches:** Players can select the number of questions (from 3 to 15) and choose a specific topic or opt for a random one.
*   **Flexible Capacity:** Matches support 2 to 4 players.
*   **Smart Countdown:** The game initiates a countdown as soon as 2 players join. If players leave and the room drops to 1 person, the countdown automatically aborts to prevent solo games.

### Real-Time Gameplay (WebSockets)
*   **Synchronized State Machine:** The game seamlessly transitions through strict states: `WAITING`, `COUNTDOWN`, `ACTIVE`, `ROUND_FINISHED`, and `FINISHED`.
*   **Live UI Updates:** Players see real-time updates of who is online/offline, the current round number, an active timer, and visual indicators of who has already answered.
*   **Round Mechanics:** Once all active players submit their answers, the round stops immediately, revealing correct/incorrect answers and updating the current score dynamically.

### Edge Cases
*   **Technical Victories:** If opponents use the "Quit" button or disconnect, leaving only 1 player in the room, the remaining player is awarded a technical victory.
*   **Auto-Cleanup:** To prevent memory leaks and zombie rooms, games are automatically deleted after 30 seconds if all players abandon the session.

### Statistics & History
*   **Global Leaderboard:** Real-time ranking of the top players across the platform.
*   **Deep Match History:** Users can browse through a chronological list of their recent games (or any other player's games). Clicking on a past match reveals historical details: end date, participants, their final scores, and the ultimate winner.

## Tech Stack

**Core & Architecture**
*   **Java 21**
*   **Spring Boot** (RESTful API)

**Security**
*   **Spring Security**
*   **JWT** (JSON Web Tokens) for stateless authentication
*   **BCrypt** for secure password hashing

**Database & ORM**
*   **PostgreSQL** (Hosted on Supabase)
*   **Spring Data JPA / Hibernate**

**Build & Deployment**
*   **Maven**
*   **Docker**

## Project Structure

The service follows a standard layered architecture to separate concerns and maintain clean code.

```text
.
├── src/main/java/eugenestellar/
│   ├── config/       # Spring Security, REST client, and JWT filter configurations
│   ├── controller/   # REST API endpoints (AuthController, InfoController)
│   ├── exception/    # Global exception handler (@ControllerAdvice) and custom exceptions
│   ├── model/        # JPA Entities (User), Enums (Role), and DTOs
│   ├── repository/   # Spring Data JPA interfaces (UserRepo)
│   ├── service/      # Core business logic (AuthService, InfoService)
│   └── util/         # Helper classes (JwtUtil for token generation/validation)
└── src/main/resources/
    └── application.yml # Service configuration and database connection settings
```

## Database Schema

Following the Database-per-Service microservice pattern, the Auth Service has its own isolated PostgreSQL database. It is strictly responsible for credentials and contains a single table.

**Table: `users`**
```sql
-- WARNING: This schema is for context only and is not meant to be run.
-- Table order and constraints may not be valid for execution.

CREATE TABLE public.users (
  id bigint GENERATED ALWAYS AS IDENTITY NOT NULL,
  password character varying NOT NULL,
  username character varying NOT NULL UNIQUE 
    CHECK (char_length(username::text) >= 3 AND char_length(username::text) <= 20),
  CONSTRAINT users_pkey PRIMARY KEY (id)
);
```

## Local Development

### Prerequisites
Before running the project, ensure you have the following installed:
*   **Java 21**
*   **Maven** (or use the provided Maven wrapper)
*   **Docker** (optional, for containerization)
*   **PostgreSQL** (local instance or Supabase connection)

### Configuration & Launching

1.  **Set Environment Variables:**
    Configure the required database, security, and service routing variables. You can set them in your terminal, IDE, or an `.env` file (matching the production Render environment):
    ```env
    DB_URL=jdbc:postgresql://<host>:5432/<db_name>
    DB_USER=your_db_user
    DB_PASSWORD=your_db_password
    SECRET_WORD=your_super_secret_jwt_key_here
    FRONTEND_URL=http://localhost:3000      # For CORS configuration
    GAME_SERVICE_URL=http://localhost:8081  # For synchronous profile creation
    ```

2.  **Start the Application (Maven):**
    Run the service using the included Maven wrapper.
    *   Mac/Linux: `./mvnw spring-boot:run`
    *   Windows: `mvnw.cmd spring-boot:run`

3.  **Start the Application (Docker):**
    Alternatively, build and run the image using Docker:
    ```bash
    docker build -t quiz-auth-service .
    docker run -p 8080:8080 -env-file .env quiz-auth-service
    ```

### Usage (API Examples)

Once the service is running (default port is `8080`), you can interact with the REST API.

**Register a new user:**
```bash
curl -X POST http://localhost:8080/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"username": "player1", "password": "securepassword"}'
```

## Potential Improvements & Architectural Trade-offs

While the current architecture successfully separates domains, there are a few technical trade-offs that are planned to be addressed in future iterations:

*   **Tight Coupling on Registration (Synchronous Communication):** 
    *   *Current State:* When a new user registers, the Auth Service makes a synchronous REST call to the Game Service (`GAME_SERVICE_URL`) to create a player profile. If the Game Service is temporarily down, the registration process might fail or result in inconsistent data.
    *   *Planned Solution:* Introduce an Event-Driven Architecture using a message broker (e.g., **Apache Kafka** or **RabbitMQ**). The Auth Service will simply publish a `UserRegisteredEvent`, and the Game Service will consume it asynchronously, ensuring eventual consistency and high availability.
*   **Token Revocation & Logout:**
    *   *Current State:* The application uses stateless JWTs. While great for performance, stateless tokens cannot be easily invalidated on the server side before they naturally expire (e.g., during a manual logout).
    *   *Planned Solution:* Integrate **Redis** to maintain a fast, in-memory "blacklist" for revoked tokens, or implement a stricter short-lived Access Token + rotating Refresh Token mechanism.
 
---

## Related Repositories

*   **Auth Service** *(This Repository)*
*   [Game Service](https://github.com/eugene-stellar/quiz-backend-game-service) - Manages WebSockets, game state, and round logic.
*   [Frontend Client](https://github.com/mildoss/quiz-frontend) - Next.js User Interface.

## License & Info

*   **License:** MIT
*   **Author:** [Eugene Bielichenko](https://github.com/eugene-stellar)
*   **Last Updated:** July 2026
