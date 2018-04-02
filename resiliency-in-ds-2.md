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

``` Ratelimiting requests coming into your system```

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

```Bulkheading is the pattern of isolating elements into pools so that failure of one does not affect another```

The name `Bulkhead` comes from sectioned partitions in a ship, wherein if a parition is damaged/compromised=, only the damaged region fills with water and prevents from the whole ship sinking.
In a similar way, you can prevent failure in one part of your distributed system affecting and bringing down others.

Bulkhead pattern can be applied at multiple levels in a system.

Lets start at a higher level from an architecture point of view:

#### Physical redundancies

Bulkheading here is about providing physical redundancy for the same piece of software for different requirements/flows.
For ex: At GO-JEK, physical redundancy is provided at the Auth proxy, separating driver apis from customer apis. 
This bulkheading prevents failures in customer apis affecting and causing problems on the driver side.

It can also be envisoned at the connection pooling level:

Consider you have 2 downstream services B and C for a service A. Request handling threads in A process the requests to downstream services B and C.
When we have a common connection pooling config, problems in C can cause request handling threads to be blocked on A utilising all the resources, preventing requests depending on B to be served/fulfilled.
Isolating connection pooling config to B and C, can help prevent faults/slowdown in one dependent service affecting the whole system.

Hystrix implementation also provides Max concurrent requests per backend. Assume we have 25 request handling threads, and we have a limit of 10 on requests to C, which means at most 10 requests can hang,
when C has a slowdown. The other 15 can still be used to process requests for B.

Bulkheading thus provides failure isolation across your services, hence building resiliency.

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
