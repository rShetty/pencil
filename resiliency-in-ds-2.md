# Resiliency in Distributed Systems - Part 2

This is part 2 in discussions on Resiliency in Distributed systems, if you have not read Part 1 of this blog series. 
Please do so [here](), I will wait ...

In this article we will continue to discuss and explore more patterns in bringing resiliency/stability in complex distributed systems.

## Pattern[6] = Ratelimiting and Throttling

```Ratelimiting/Throttling is about allowing only certain number of requests(n) in some time (t)```

This could be applied to requests which are coming into your system (ratelimiting access to your API),
or for the requests which you are leaving your system (when making calls to external third party services)

Ratelimiting/Throttling is an important resiliency pattern which can help protect systems from scripted attacks or sudden burst in traffic. 
It helps limit requests coming into your system in-turn allowing services to perform in normal mode.

At GO-JEK, this pattern is used at multiple places and for varied reasons, lets discuss few of those:

``` Ratelimiting requests coming into your system``

#### OTP Login
We ratelimit requests to OTP login from our mobile application to say 'n' attempts in some time 't'. 
This helps prevent a scripted attacker/a malicious user retrying logins more than necessary.

This approach helps in following ways:
- Prevent attackers from gaming the login system
- Saves cost on SMS/call

#### Promotions Platform
At times during promotions, we see a sudden surge in traffic. Ratelimiting requests coming to your promotions platform can help systems to work in normal mode even when there is a sudden surge of traffic.
Thus helping critical flows to not break.

```Throttling requests going out of the system```

#### Maps Service:
We use Google maps API for estimating distance between rider and driver, estimating price, mapping a route between driver and rider etc. 
We have certain limit(Quota) on the number of requests (QPS) which are allowed to be made to google at any time 't'. It is important for 
us to keep the requests going out from our system to google below that limit so that we don't get blocked by google from accessing the above mentioned services.

At GO-JEK, all the calls to google at GO-JEK go through Maps service which makes sure that requests to google are throttled at an appropriate level thus preventing catastrophic failures within our other systems.

#### SMS Providers:
We use multiple SMS providers for sending login OTP's, Marketing messages etc to our customers. We do have a Quota of requests allowed to be made to the SMS providers at any point in time (t). We heavily use throttling here to 
make sure we don't get blocked by our SMS providers.

We use Leaky Bucket algorithm at most places for ratelimiting/throttling requests.

## Pattern[7] = Bulkheading

- What is bulkheading ?
- Bulkheading in services to provide failure isolation (Kong example)
- Bulkheading in connection pooling 
- Importance

## Pattern[8] = Queuing to decouple tasks from consumers

- What is Queuing ?
- How queuing is helpful decoupling from external services ?
- BTS
- SMS/PNS
- GOPAY

## Pattern[9] = Monitoring/alerting (Observability?)

- Monitoring/alerting setup in GOJEK
- How monitoring can help prevent faults turning into failures

## Pattern[10] = Canarying

- What are canary deployments ?
- Gradual rollout of features/algorithms
- Canarying
- Advantages
