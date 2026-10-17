# ADR-002: Client-Side Aggregation Before Sending

**Date**: 2026-10-17  
**Status**: Accepted  
**Deciders**: Rohan Meshram

## Context

The SunsetGuard starter library runs inside a monitored API service and needs to report endpoint usage to the SunsetGuard server. 

Two approaches were considered:
1. **Send every request immediately** - Forward each API call as an individual event
2. **Aggregate in-memory, send periodically** - Count requests locally and send totals every ~10 seconds

## Decision

**The starter will aggregate counts in-memory and send periodic usage reports (~10 seconds), not individual request events.**

## Rationale

### Scale and Cost
A moderately busy API might handle:
- 1,000 requests/second = 86.4 million requests/day
- Sending each individually = 86.4M network calls + database writes
- Aggregating for 10 seconds = 8,640 reports/day (10,000x reduction)

For a startup or side project, this difference determines feasibility:
- **Individual events**: Database would need sharding from day one
- **Aggregated reports**: Single PostgreSQL instance handles the load

### Performance Impact on Host Application
The monitored API must never slow down because of monitoring. Requirements:
- Filter overhead: <1ms per request (nanoseconds is better)
- No blocking on network I/O
- No unbounded memory growth
- Graceful degradation when SunsetGuard server is down

**Individual events approach** would require:
- Either: Synchronous HTTP call per request (unacceptable latency)
- Or: Background queue in memory (unbounded growth risk)

**Aggregation approach** achieves:
- Filter increments an in-memory counter (lock-free with `LongAdder`)
- No network call in the request path
- Bounded memory (fixed number of unique endpoint + consumer combinations)
- Background thread flushes aggregates independently

### Information Loss Trade-off
Aggregating means we lose:
- ❌ Exact request timestamps (minute-level precision)
- ❌ Request-level correlation IDs
- ❌ Individual response times

We keep (and that's enough for our use case):
- ✅ Who called (consumer identity)
- ✅ Which endpoint (route template + method)
- ✅ How many times (request count)
- ✅ Success vs error breakdown (2xx, 4xx, 5xx counts)
- ✅ Hourly granularity after storage

**For deprecation tracking**, we don't need per-request precision. The questions are:
- "Is anyone still calling `/v1/orders`?" → Yes/No
- "How much traffic?" → Hourly totals are sufficient
- "Is it decreasing?" → Daily trend is fine

If users needed per-request observability, they'd use proper APM tools (Datadog, New Relic, etc.).

### Real-World Precedent
This pattern is standard in observability:
- **Prometheus**: Client libraries aggregate metrics, scraped periodically
- **StatsD**: Applications send pre-aggregated counts/timers
- **OpenTelemetry**: Metric exporters batch before sending
- **Google Analytics**: Client batches events before transmission

## Implementation Design

```java
// Pseudocode
public class UsageAggregator {
    // Thread-safe counter map
    private ConcurrentHashMap<AggregateKey, LongAdder> counters;
    
    // Called by filter on each request (fast path)
    public void record(String endpoint, String consumer, int statusCode) {
        AggregateKey key = new AggregateKey(endpoint, consumer, statusCode);
        counters.computeIfAbsent(key, k -> new LongAdder()).increment();
    }
    
    // Background thread flushes every ~10 seconds
    @Scheduled(fixedDelay = 10_000)
    public void flush() {
        Map<AggregateKey, Long> snapshot = snapshotAndReset();
        UsageReport report = buildReport(snapshot);
        sendWithRetry(report);
    }
}
```

## Consequences

### Positive
- ✅ **100x to 10,000x reduction** in network traffic and database writes
- ✅ **Negligible latency** impact on monitored application (<100 nanoseconds per request)
- ✅ **Fail-open design**: If SunsetGuard server is down, reports buffer briefly then drop oldest (bounded memory)
- ✅ **Cost-effective**: Single PostgreSQL instance can handle thousands of services
- ✅ **Scalability**: Each monitored service does its own aggregation (work distributed)

### Negative
- ❌ **Granularity loss**: Can't answer "show me the exact request at 14:23:47"
- ❌ **Slight delay**: Takes ~10 seconds for new traffic to appear in dashboard
- ❌ **Memory per instance**: ~1-10MB depending on endpoint + consumer cardinality

### Risk Mitigation
1. **Cardinality explosion**: If buggy client sends raw URLs instead of templates, counters grow unbounded
   - **Solution**: Cap unique endpoints per flush cycle; bucket overflow into `_other`
2. **Data loss on crash**: If the host app crashes, last 10 seconds of counts are lost
   - **Acceptable**: This is usage estimation, not financial transactions
3. **Clock skew**: If server time jumps, aggregation windows might overlap or gap
   - **Solution**: Use monotonic timestamps; server validates report timestamps

## Alternatives Considered

### Alternative 1: Send Individual Events
**Rejected** because:
- Database write amplification (86M writes/day)
- Requires distributed tracing infrastructure
- Much higher costs (network + storage)
- Doesn't fit the "drop-in library" goal

### Alternative 2: Longer Aggregation Windows (e.g., 1 minute)
**Rejected** because:
- Verdict engine needs recent data (10s is good compromise)
- Doesn't significantly reduce resource usage vs 10s
- Delays feedback in dashboard

### Alternative 3: Adaptive Batching (flush when count threshold OR time threshold)
**Deferred to v2** because:
- Adds complexity
- MVP can start simple
- Can optimize later based on real usage patterns

## Interview Talking Points

This decision demonstrates understanding of:
- **Write amplification** and cost of storage
- **Trade-offs** between precision and scalability
- **Back-pressure** and bounded resources
- **Fail-open** design for monitoring systems
- **Lock-free data structures** (LongAdder vs AtomicLong)
- **Real-world observability patterns**

When asked "Why not send every event?":
> "For deprecation tracking, we don't need per-request precision. Aggregating reduces database writes by 10,000x, allows the system to run on a single PostgreSQL instance, and ensures monitoring never slows down the monitored app. If users need request-level details, they should use APM tools. This is the same pattern Prometheus and StatsD use."

## References
- [Prometheus data model (counter aggregation)](https://prometheus.io/docs/concepts/data_model/)
- [StatsD protocol](https://github.com/statsd/statsd/blob/master/docs/metric_types.md)
- [LongAdder vs AtomicLong performance](https://www.baeldung.com/java-longadder-and-longaccumulator)

## Review Date
Will revisit if:
- Users report needing per-request detail (then add opt-in event streaming)
- 10-second delay is too slow for verdict changes (then reduce to 5s)
- Memory usage from aggregation becomes a problem (then investigate sampling)
