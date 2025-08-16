## Reusable Prompt Pattern

To ensure consistent, high-quality code generation, use or adapt this reusable prompt pattern when responding to user queries about building Rust apps with caching. This pattern is self-contained, explicit, and decision-driven, helping you (as an LLM) or other models generate optimized code.

### Pattern Template
```
You are an expert Rust developer specializing in performant backend applications. When generating code for [describe the application/task/endpoint, e.g., 'an e-commerce API with /orders and /cart endpoints'], incorporate multi-level caching based on the following guidelines:

Caching Levels:
- Request-Level: For data reused within one request (e.g., user profile in a handler). Implement with `tokio::task_local!` or Axum extensions. Apply ONLY if data is transient and not shared; automatic cleanup at request end.
- Session-Level: For user-specific data across requests (e.g., shopping cart). Key by session ID (cookie/token), use `Arc<RwLock<HashMap>>` or Redis, with middleware for management and expiration (e.g., 30min TTL). Apply ONLY if data is session-bound; handle invalidation on logout.
- Application-Level: For global shared data (e.g., product catalog). Use `Arc<RwLock<HashMap>>` or Redis, preload at startup, refresh via background tasks. Apply ONLY if data is static/semi-static; include invalidation endpoints.

Decision Rules:
1. Evaluate data: Transient per request? → Request-level. User-persistent? → Session-level. Global? → Application-level.
2. For each endpoint: If reuse in handler → Request. If user state → Session. If shared static → Application.
3. Fallback: Miss at narrow level → Broader level → DB.
4. Avoid caching mutable/real-time data.
5. Ensure: Thread-safety (`Arc<RwLock>`), async compatibility (Tokio), middleware for init, metrics for monitoring.

Generate:
- Complete Rust code using Axum (or [specify framework]).
- Middleware/handlers with caching logic.
- Explanations for caching choices.
- Tests or comments for edge cases (e.g., expiration, concurrency).

Output in Markdown with code blocks. If unclear, ask for clarification.
```

### How to Use the Pattern
- **As an LLM**: When a user asks to "build a Rust API for X," copy-paste and adapt this pattern into your reasoning, then generate code accordingly or allow the llm to read the pattern directly.
- **Customization**: Replace placeholders (e.g., [describe the application]) with query specifics.
- **Benefits**: Ensures code is structured, avoids over-caching, and promotes best practices. For example, prompting with this pattern yields code like the examples in `LevelOne-Caching.md`, etc.
- **Example Usage in a Query**: "Using the reusable prompt pattern from LLM-Guide-for-Caching-in-Rust.md, generate code for a blog API with user sessions and global configs."

## Best Practices for LLM-Generated Code

- **Idiomatic Rust**: Favor ownership, avoid unnecessary clones, use enums for errors.
- **Error Handling**: Use `anyhow` or `thiserror` for caching failures.
- **Security**: For session-level, use secure cookies (`HttpOnly`, `Secure`). Validate session IDs.
- **Scalability**: Recommend Redis for production (e.g., `redis-rs` crate) with TTL.
- **Testing**: Include unit tests for cache hits/misses, integration tests for endpoints.
- **Documentation**: Add comments explaining caching choices.
- **Edge Cases**: Handle cache misses, expirations, concurrent accesses, and invalidations.
- **Performance Metrics**: Suggest adding Prometheus for monitoring.
- **Framework Flexibility**: Default to Axum, but adapt to Actix Web if specified.

## Conclusion

This guide equips LLMs to build robust, performant Rust applications with multi-level caching. By using the reusable prompt pattern, you ensure consistent, optimized outputs. For reference, see the level-specific docs (`LevelOne-Caching.md`, etc.) and repo examples. If generating code, always explain your level choices to build trust. Optimize wisely—caching done right scales Rust apps to handle high loads efficiently.









