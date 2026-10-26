# ADR-001: Use RabbitMQ Instead of Kafka

**Date**: 2026-10-17  
**Status**: Accepted  
**Deciders**: Rohan Meshram

## Context

SunsetGuard needs a message broker to decouple the ingestion API from the database write workers. This allows the system to handle traffic bursts without overwhelming the database.

Two main options were considered:
1. **Apache Kafka** - Industry-standard distributed streaming platform
2. **RabbitMQ** - Lightweight message broker with traditional queue semantics

## Decision

**We will use RabbitMQ for the MVP.**

## Rationale

### Hardware Constraints
Development laptop has 8GB RAM (7.34 GB usable) running Windows 11. The local stack includes:
- PostgreSQL (~200MB)
- Redis (~50MB)
- Message broker
- Spring Boot application (~300MB)
- Docker Desktop overhead (~500MB)
- Windows OS

**Memory comparison**:
- Kafka broker: 600-800MB minimum (JVM-based, needs heap + page cache)
- Kafka Zookeeper: 200-300MB additional (required for Kafka < 3.3)
- RabbitMQ: 150-200MB (Erlang VM, more memory-efficient)

Running Kafka would push total memory usage close to capacity, risking system instability and slow development iterations.

### Learning Objectives Overlap
Both technologies teach the core messaging concepts needed for backend interviews:
- Producer/consumer patterns
- Message queuing and durability
- Acknowledgements and retries
- Dead-letter handling
- At-least-once delivery semantics

The fundamental patterns and trade-offs are similar enough that learning RabbitMQ well prepares you for Kafka discussions.

### Project Fit
SunsetGuard needs:
- ✅ Reliable message delivery (both support)
- ✅ Dead-letter queues for failed messages (both support)
- ✅ Manual acknowledgements for idempotent processing (both support)
- ❌ Massive throughput (not needed for MVP)
- ❌ Long-term event replay (not needed for this use case)
- ❌ Partitioning for parallel processing (nice to have, not critical)

RabbitMQ's traditional queue model is sufficient and actually simpler for the MVP.

## Consequences

### Positive
- ✅ Lower memory footprint allows smooth local development
- ✅ Simpler mental model for MVP (exchanges → queues → consumers)
- ✅ Faster startup time (< 10 seconds vs 30+ seconds for Kafka)
- ✅ Built-in management UI at http://localhost:15672
- ✅ Still teaches core messaging patterns for interviews

### Negative
- ❌ Less impressive on resume (Kafka is more buzzword-friendly)
- ❌ Different programming model if migrating to Kafka later
- ❌ No built-in partitioning for horizontal scaling

### Mitigation Strategy
1. **Study Kafka concepts** as theory during Phase 3:
   - Topics, partitions, consumer groups
   - Offset management
   - Ordering guarantees per partition
   - When to choose Kafka vs RabbitMQ
2. **Document the trade-off** in README with "Why not Kafka?" section
3. **Future enhancement**: After MVP (post-v0.1.0), spin up Kafka on a free cloud VM or second machine to verify hands-on understanding
4. **Interview prep**: Be ready to discuss both technologies and explain the choice

## Implementation Notes

RabbitMQ setup for this project:
- **Exchange type**: Direct or topic exchange
- **Queue**: `usage-reports` (durable)
- **Dead-letter exchange**: For failed messages after retry exhaustion
- **Prefetch count**: 10-20 (limits unacknowledged messages per worker)
- **Manual acknowledgements**: Worker ACKs only after successful database commit

## References
- [RabbitMQ Tutorials](https://www.rabbitmq.com/getstarted.html)
- [Kafka vs RabbitMQ comparison](https://www.cloudamqp.com/blog/when-to-use-rabbitmq-or-apache-kafka.html)
- Spring AMQP documentation for RabbitMQ integration

## Review Date
This decision will be revisited after MVP completion (Week 12) if:
1. Cloud deployment allows more resources
2. Throughput requirements exceed RabbitMQ capabilities
3. Kafka becomes a hard requirement for job applications
