# Scale Plan

## Data model/indexes

- Keep UNIQUE constraint on idempotency_key to prevent duplicate signal creation.
- Use index on (user_id, created_at) for efficient signal retrieval.
- Archive old records and partition large datasets if data volume increases.

## Idempotency across instances

- Use a database-level UNIQUE constraint on idempotency_key.
- Insert operations should rely on atomic database guarantees instead of check-then-insert logic.
- On duplicate key conflicts, return the existing resource associated with the idempotency key.

## Rate limiting across instances

- Current implementation uses in-memory storage and works only for a single instance.
- For production deployments, use Redis with atomic INCR and EXPIRE commands.
- Shared Redis ensures rate limits remain consistent across multiple application instances.

## Observability (logs/metrics/alerts)

- Structured application logs with request identifiers.
- Track request count, latency, error rate, and rate-limit violations.
- Monitor database health and connection pool usage.
- Configure alerts for elevated error rates and service degradation.

## Failure modes (DB down / partial outages / retries)

- Retry transient database failures using exponential backoff with jitter.
- Use circuit breaker patterns for repeated failures.
- Ensure retries do not create duplicate records by relying on idempotency keys.
- Return graceful 503 responses when dependencies are unavailable.

## 10k RPS design sketch (infra & cost ballpark)

- Deploy multiple Fastify instances behind a load balancer.
- Use Redis for distributed rate limiting and caching.
- Use PostgreSQL with connection pooling and read replicas.
- Offload asynchronous processing to Kafka or RabbitMQ.
- Run services in containers managed by Kubernetes.
- Scale horizontally based on CPU, memory, and request volume.
- Estimated infrastructure: load balancer, Redis cluster, application replicas, managed database, monitoring stack, and queueing system.
