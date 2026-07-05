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
*   **Secure Access:** Full registration, login, and logout flows protected by JWT.
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
