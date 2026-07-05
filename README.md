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
*   **BaaS Integration (Leaderboard):** To optimize performance, the frontend bypasses the Java backend for read-only operations, fetching real-time leaderboard data directly from the Game DB using the Supabase Data API.
*   **Static Assets (CDN):** Question images are securely stored in AWS S3 and delivered globally with low latency via AWS CloudFront directly to the client.

