# Resiliency in Distributed Systems - Part 2

This is part 2 in discussions on Resiliency in Distributed systems, if you have not read Part 1 of this blog series. 
[Please do so here](https://github.com/rShetty/pencil/blob/master/resiliency-in-ds.md), I will wait ...

In this article, we will continue to discuss and explore more patterns in bringing resiliency/stability in complex distributed systems.

## Pattern[6] = Ratelimiting and Throttling

```Ratelimiting/Throttling is only allowing a certain number of requests in fixed time```

Rate-limiting could be applied to requests which are coming into your system (rate-limiting access to your APIs),
or for the requests which are leaving your system (when making calls to external party services)

We use Leaky Bucket algorithm(https://en.wikipedia.org/wiki/Leaky_bucket) at most places for rate-limiting/throttling requests.

Rate-limiting/Throttling is an important resiliency pattern which can help protect systems in multiple ways: 
1) scripted attacks to your system
2) sudden bursts in traffic
3) protecting your integrations with third party external services

Usually when a rate-limit kicks in requests exceeding threshold are dropped and sent back a standard 429(Too many requests) response.

At GO-JEK, this pattern is used at multiple places and for varied reasons, let's discuss few of those:

### Ratelimiting requests coming into your system

#### OTP Login
We rate-limit requests to OTP login from our mobile application to say 'n' attempts in some time 't'. 
This means if the rate-limit configuration is set to allow 3 login request attempts from a customer in 15 minutes, customer gets 3 chances for logging in for that 15 minute window.
This helps prevent a scripted attacker/a malicious user retrying logins more than necessary.

This approach helps in following ways:
- Prevent attackers from gaming the login system by continuosly retrying login with different OTP's
- Saves cost on SMS/call

#### Promotions Platform
Promotions platform is responsible for processing promotional codes which are sent to GO-JEK users on various services.

At times during promotions, we see a sudden burst in traffic on our promotions platform. This sudden burst could potentially affect services involved in fulfilling this request from not performing at their peak.
Rate-limiting requests (dropping execess ones) coming to promotions platform can help promotions service as well as the dependent services to work in normal mode.
Thus helping critical flows to not break.

### Throttling requests going out of the system

#### Maps Service:
We use Google maps API for estimating the distance between rider and driver, estimating price, mapping a route between driver and rider etc. 
We have a certain limit(Quota) on the number of requests (QPS - Queries per second) which are allowed to be made to Google in a fixed period of time. It is important for
us to keep the requests going out from our system to google below that limit so that we don't get blocked by Google from accessing the above-mentioned services.

<< Figure >>

At GO-JEK, all the calls to google at GO-JEK go through Maps service which makes sure that requests to Google are throttled at an appropriate level before making external calls thus preventing catastrophic failures within our other systems.

#### SMS Providers:
We use multiple SMS providers for sending login OTP's, Marketing messages etc to our users. We do have a Quota of requests allowed to be made to the SMS providers in a fixed time period. We heavily use throttling here to make sure we don't get blocked by our SMS providers.

## Pattern[7] = Bulkheading

```Bulkheading is the pattern of isolating elements into pools so that failure of one does not affect another```

The name `Bulkhead` comes from sectioned partitions in a ship, wherein if a partition is damaged/compromised=, only the damaged region fills with water and prevents the whole ship sinking.
In a similar way, you can prevent failure in one part of your distributed system affecting and bringing down others.

Bulkhead pattern can be applied at multiple levels in a system.

Let's start at a higher level from an architecture point of view:

#### Physical redundancies

Bulkheading here is about providing physical redundancy for the same piece of software for different requirements/flows.
For example: At GO-JEK, physical redundancy is provided at the Auth proxy, separating driver APIs from customer APIs. 
This bulkheading prevents failures in customer APIs affecting and causing problems on the driver side.

It can also be envisioned at the connection pooling level:

#### Connection Pooling

Consider you have 2 downstream services 'B' and 'C' for a service 'A'. Request handling threads in A process the requests to downstream services 'B' and 'C'.
When we have a common connection pooling config, problems in 'C' can cause request handling threads to be blocked on 'A' utilizing all the resources, preventing requests depending on B to be served/fulfilled.
Isolating connection pooling config to 'B' and 'C' can help prevent faults/slowdown in one dependent service affecting the whole system.

Hystrix implementation also provides Max concurrent requests per backend. Assume we have 25 request handling threads, and we have a limit of 10 on requests to 'C', which means at most 10 requests can hang
when 'C' has a slowdown. The other 15 can still be used to process requests for 'B'.

Bulkheading thus provides failure isolation across your services, hence building resiliency.

## Pattern[8] = Queuing to decouple tasks from consumers

Queuing is the process of buffering messages in a queue so as to decouple producers of messages from consumers.

Queuing plays an important role in decoupling multiple services. It also can act as a buffer which helps smoothen out intermittent heavy loads which can cause service to fail.
It helps in increasing availability as well as scalability of your system.

A typical queuing architecture looks like this:

<< Figure >>

At GO-JEK, we heavily use queuing pattern to process some of the flows asynchronously without affecting our customers.

#### BTS (Booking Termination Service)

This service plays an important role in terminating all bookings in GO-JEK. There are a bunch of activities you need to complete to make sure a booking has completed.
For example, If a customer has booked a ride using GOPAY, you need to debit customer balance, and credit driver balance, update a service to release driver back to the pool of available drivers,
Mark the state of booking as completed, record history of this transaction, reward driver with points and much more.

Now more the tasks you have to do, more the failure modes and ways in which a service or an API call can fail. It is potentially a very bad experience if you asked your customer to wait until you clean up
various states in your system. God forbid, if there is a failure in some system, both your customer and driver are stuck from completing the transaction and getting on with their own livelihood.

In this case, we queue the request for completion of booking and process all the tasks required to complete this booking asynchronously from the main booking flow.

Queuing decouples this flow to provide much more resiliency in the whole architecture, wherein you eventually tend towards this state, potentially with retries at places.

#### SMS (Notification Service)

We heavily use SMS for communicating with our customers/drivers/merchants. The major chunk of which is for OTP login into GOJEK application. We use multiple SMS providers to be resilient against failures on a single SMS provider.
SMS Notification service is the service which interfaces with various SMS providers(external to GOJEK) to help deliver SMS to our customers/drivers/merchants.

To decouple ourselves from failures on our SMS providers, we use queuing to deliver messages. We retry SMS with a different provider when our primary provider fails to meet a certain SLA for delivering SMS.

## Pattern[9] = Monitoring/alerting

```Monitoring is about measuring some metric over a period of time. And alerting is about getting a notification when some metric violates its threshold```

Monitoring can help you measure the current state/health of your system as well as over historical periods. It becomes extremely crucial for resiliency to know when faults have made their way into our systems.
For example: Without monitoring/alerting you might not be able to detect a memory leak in your production application beforehand and fix it before it actually crashes and impact users.

At GO-JEK we collect various kinds of metrics which includes system metrics, app metrics, business metrics etc. Also, alerts with thresholds are set for various kinds of metrics, so that when a fault happens, we know
beforehand and take corrective measures.

Monitoring with alerting according to me can help in 2 ways:

- It notifies you of potential faults in your system, helping you recover from it before it causes failure in all of your system. (Proactive)
- It can help diagnose the issue and pinpoint the root cause to help recover your systems from failure(MTTR - Mean time to recovery). (Active)

## Pattern[10] = Canary releases

Last but not the least for this post comes Canary releases. More [here](https://martinfowler.com/bliki/CanaryRelease.html)

```It is a way to release a certain new feature/version of a software to only a certain set of people in order reduce the risk of failures```

Canary releases at GO-JEK are done at various places:

#### Backend Deployments

Every deployment on backend involving critical changes (like the algorithmic change to find a driver) goes through a canary release. Wherein the new version of the software is deployed only to a single node or selected set of nodes.
Usually, this is coupled with monitoring for failures, latencies etc on the node. Only after monitoring the system for a particular period of time, is the new version of the software released to all users.

#### Mobile releases

Even new app rollouts go through this phase of rolling out to a small set of users which is again coupled with gathering metrics of usage, collecting feedback on the new features etc. Only after the team is confident enough of the impact of the new version on the end users do they go with the full release.

Canary releases help you avoid large-scale risk to bring in new features to end users with higher confidence, in-turn helping resiliency of systems.

All the above patterns can help us with maintaining stability across our systems. The key being decoupling services. 

```The more coupled your services are, the more they are together in failures```

In conclusion, Failures will happen, we need to design systems to be resilient against these failures.

