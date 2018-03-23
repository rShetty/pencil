# Resiliency in Distributed Systems - Part 2

If you have not read Part 1 of this blog series. Please do so [here]()

6) Rate limiting and Throttling
- What is rate limiting ?
- Importance of throtlling requests to external services (Maps API -> Google Service)
- Importance of rate limiting for security purposes (Login)
- Ratelimit algorithms (Leaky bucket)
- How it is useful ?

7) Bulkheading
- What is bulkheading ?
- Bulkheading in services to provide failure isolation (Kong example)
- Bulkheading in connection pooling 
- Importance

8) Queuing to decouple tasks from consumers
- What is Queuing ?
- How queuing is helpful decoupling from external services ?
- BTS
- SMS/PNS
- GOPAY

9) Monitoring/alerting (Observability?)
- Monitoring/alerting setup in GOJEK
- How monitoring can help prevent faults turning into failures

10) Canary Deployments 
- Gradual rollout of features
- Of algorithms
- Deployments to only part of the service
- Advantages
