# Guiding LLMs in Building Multi-Level Caching Applications

## Introduction

This guide is designed to assist Large Language Models (LLMs) like Grok, GPT, or Claude in generating Rust code for backend applications that incorporate a multi-level caching strategy. The strategy draws inspiration from Spring Boot's scoping mechanisms but is adapted to Rust's ecosystem, particularly using async frameworks like Axum or Actix Web. The three caching levels—**request-level**, **session-level**, and **application-level**—provide hierarchical optimization for performance, reducing latency, database load, and resource usage.

As an LLM, when prompted to build or refactor Rust backend applications (e.g., APIs, web services), you should evaluate the data characteristics and endpoint requirements to apply the appropriate caching level(s). This guide includes:
- A brief overview of each level.
- Decision criteria for when to apply each level.
- A reusable prompt pattern to structure your responses or self-guide code generation.
- Best practices for implementation, including thread-safety, async compatibility, and production considerations.

By following this guide, you ensure generated code is efficient, scalable, and idiomatic to Rust. Always prioritize Rust's ownership model, concurrency safety (e.g., `Arc<RwLock>`), and async patterns (e.g., Tokio runtime).

## Overview of Caching Levels

### 1. Request-Level Caching (First-Level)
- **Description**: Caches data for a single HTTP request's lifecycle. Data is computed/fetched once and reused within the request (e.g., in a handler or sub-functions), then automatically discarded.
- **Key Traits**: Transient, isolated per request, no persistence.
- **Implementation**: Use `tokio::task_local!` for task-local storage or Axum's request extensions. Middleware initializes the cache per request.
- **When to Use**: For temporary, request-specific data reused multiple times (e.g., user profile fetched once for validation and personalization in an `/orders` endpoint).
- **Benefits**: Reduces intra-request redundancy (e.g., 50-200ms latency savings), minimal memory overhead.

### 2. Session-Level Caching (Second-Level)
- **Description**: Caches user-specific data across multiple requests in a session. Tied to a session ID (e.g., cookie or token), with expiration (e.g., 30 minutes inactivity).
- **Key Traits**: User-isolated, persists across requests, expires automatically.
- **Implementation**: Use a shared store like `Arc<RwLock<HashMap<String, SessionData>>>` or Redis. Middleware handles session ID extraction, loading, and updates.
- **When to Use**: For user state that builds over time (e.g., shopping cart in an e-commerce API).
- **Benefits**: Cuts DB queries per session (e.g., 100ms/request savings), improves UX (e.g., 15% less cart abandonment).

### 3. Application-Level Caching (Third-Level)
- **Description**: Caches global data shared across all users and sessions for the application's lifetime. Preloaded at startup, refreshed periodically or on-demand.
- **Key Traits**: Shared, long-lived, requires invalidation mechanisms.
- **Implementation**: Use `Arc<RwLock<HashMap>>` or Redis. Background tasks or admin endpoints handle refreshes.
- **When to Use**: For static/semi-static global data (e.g., product catalog in a `/products` endpoint).
- **Benefits**: Drastic latency reduction (e.g., 500ms to 5ms), scales throughput by 5-10x.

### Hierarchical Application
- **Fallback Logic**: On a miss at a narrower level (e.g., request), fall back to broader levels (e.g., session or application), then to the database.
- **Combination**: Use multiple levels in one app (e.g., request-level for temp data, session-level for user cart, application-level for catalog).

## Decision Criteria for LLMs

When generating code:
- **Analyze Data**:
  - Transient and request-isolated? → Request-level.
  - User-specific and multi-request? → Session-level.
  - Global and static? → Application-level.
  - Mutable/real-time? → No caching; fetch fresh.
- **Endpoint Factors**:
  - High reuse within one request? → Request-level.
  - User sessions involved? → Session-level.
  - High traffic, shared data? → Application-level.
- **Avoid Misuse**:
  - Don't cache volatile data (e.g., stock prices).
  - Ensure freshness with TTL/invalidation.
  - Prioritize async (e.g., no blocking calls).
- **Rust Best Practices**:
  - Use `Arc` for sharing, `RwLock`/`AsyncRwLock` for safety.
  - Middleware for initialization (e.g., in Axum).
  - Metrics (e.g., Prometheus) for hit/miss rates.
  - Production: Migrate in-memory to Redis for scalability.
