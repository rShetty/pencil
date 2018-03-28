# Resiliency in Distributed Systems - Part 2

This is part 2 in discussions on Resiliency in Distributed systems, if you have not read Part 1 of this blog series. Please do so [here](), I will wait ...

In this article we will continue to discuss and explore more patterns in bringing resiliency/stability in our complex distributed systems architecture.

### Pattern[6] = Ratelimiting and Throttling

- What is rate limiting ?
- Importance of throtlling requests to external services (Maps API -> Google Service)
- Importance of rate limiting for security purposes (Login)
- Ratelimit algorithms (Leaky bucket)
- How it is useful ?

### Pattern[7] = Bulkheading

- What is bulkheading ?
- Bulkheading in services to provide failure isolation (Kong example)
- Bulkheading in connection pooling 
- Importance

### Pattern[8] = Queuing to decouple tasks from consumers

- What is Queuing ?
- How queuing is helpful decoupling from external services ?
- BTS
- SMS/PNS
- GOPAY

### Pattern[9] = Monitoring/alerting (Observability?)

- Monitoring/alerting setup in GOJEK
- How monitoring can help prevent faults turning into failures

### Pattern[10] = Canarying

- What are canary deployments ?
- Gradual rollout of features/algorithms
- Canarying
- Advantages
