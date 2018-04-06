# Resiliency in Distributed Systems - Part 2


This is part 2 in discussions on Resiliency in Distributed systems, if you have not read Part 1 of this blog series, do so here , I will wait …

In this article, we will continue to discuss and explore more patterns in bringing resiliency/stability in complex distributed systems.

## Pattern[6] = Rate-limiting and Throttling

```Rate-limiting/Throttling is only processing a certain number of requests in a fixed period of time and gracefully handling others```

Rate-limiting could be applied to requests which are coming into your system (rate-limiting access to your APIs), or for the requests which are leaving your system (when making calls to external party services)

We use Leaky Bucket algorithm at most places for rate-limiting/throttling requests.
<< Figure >>

Rate-limiting/Throttling can help protect systems in many scenarios: 
- scripted attacks to your system
- sudden bursts in traffic
- protecting your integration with third party external services

Usually when a rate-limit kicks in requests exceeding threshold are dropped and sent back a standard 429(Too many requests) response.
At GO-JEK, this pattern is used at multiple places and for varied reasons, let’s discuss few of those:

### Rate-limiting incoming requests

#### Case 1: OTP Login

We rate-limit requests to OTP (One time password) login from our mobile application. This rate-limit is per-user.
Users get only a fixed number of chances to log in for a fixed time window.
This helps prevent a scripted attacker/a malicious user retrying logins more than necessary.

This approach helps in following ways:
 - Prevent attackers from brute-forcing the login system by continuosly retrying login with different OTP’s
 - Saves cost on SMS/call

#### Case 2: Promotions Platform

Promotions platform is responsible for processing promotional codes which are sent to GO-JEK users for various services.
At times during promotions, we see a sudden burst in traffic. This sudden burst could crash services or prevent them from performing at their peak.
Rate-limiting requests, dropping excess ones or queuing them, can help promotions service as well as the dependent services to work normally. 
Thus protecting critical flows from failure.

### Throttling outgoing requests

#### Case 1: Maps Service

We use Google maps API for estimating the distance between rider and driver, estimating price, mapping a route between driver and rider etc. 
We have a certain limit(Quota) on the number of requests (QPS - Queries per second) which are allowed to be made to Google in a fixed period of time. 
It is important that we adhere to this limit so that we don’t get blocked by Google.

<< Figure >>

At GO-JEK, all the calls to Google go through a proxy service which makes sure that we do not exceed the limit set by Google by throttling requests that fall outside the limit.

## Pattern[7] = Bulkheading

```Bulkheading is isolating elements into pools so that failure of one does not affect another```

The name `Bulkhead` comes from sectioned partitions in a ship, wherein if a partition is damaged/compromised, only the damaged region fills with water and prevents the whole ship from sinking.

In a similar way, you can prevent failure in one part of your distributed system from affecting and bringing down other parts.
Bulkhead pattern can be applied at multiple ways in a system.

### Physical redundancies
Bulkheading here is about keeping components of the system physically separated. This could mean running on  separate hardware or different VM’s etc.

For example: At GO-JEK, We have driver and customer API’s physically separated on different clusters. 
Bulkheading done in this way prevents failures in customer APIs affecting and causing problems on the driver side.

### Categorized Resource Allocation (CRA)

```CRA is about splitting resources of a system into various buckets instead of a common pool```

At GO-JEK, we limit the number of outgoing HTTP connections which a server can make. The number of allowed connections is considered a connection pool. 

<<Figure>>

Consider ‘A’ supports 2 API’s API1 and API2. API1 requires a request to ‘B’ and API2 requires a request to ‘C’. When we have a common connection pool, problems in ‘C’ can cause request handling threads to be blocked on ‘A’ utilizing all the resources of ‘A’, preventing requests meant for ‘B’ to be served/fulfilled. This means failures on API1 causes API2 to also fail. Creating separate connection pools to ‘B’ and ‘C’ can help prevent this. This is an example of categorized  resource allocation:

Hystrix provides max concurrent requests setting  per backend. If we have 25 request handling threads, we can limit 10 connections to ‘C’, which means at most 10 connections can hang when ‘C’ has a slowdown/failiure. 
The other 15 can still be used to process requests for ‘B’.
Bulkheading thus provides failure isolation across your services, hence building resiliency.

## Pattern[8] = Queuing

```Queuing is buffering of messages in a queue for later dispatch to consumers allowing producers to be decoupled from consumers```

Queuing brings in resiliency by:

- Decoupling Producers from consumers
- Smoothen out bursts of traffic by buffering
- Retries on intermittent failures

A typical queuing architecture looks like this:

<< Figure >>

At GO-JEK, we heavily use queuing to process asynchronous flows for our customers. Queuing can introduce latencies and so using it in a synchronous context should be carefully designed.

### Case 1: BTS (Booking Termination Service)

This service plays an important role in terminating all bookings at GO-JEK. There are a bunch of activities that need to complete before a booking is considered completed.
If a customer has booked a ride using GOPAY, you need to

- Debit customer balance
- Credit driver balance 
- Update a service to release driver back to the free pool
- Mark the state of booking as completed
- Record the history of this transaction
- Reward driver with points

The more the tasks you have to do, the more the failure modes and the more the ways in which a service or an API call can fail. All of the above tasks can be done asynchronously without asking customer or driver to wait. 
Without a queuing solution. if there is a failure in some system, both your customers and drivers are stuck waiting for their transactions to complete. Queuing can also help retry requests once the failure has been rectified.
Queuing decouples booking flow to provide much more resilience in the whole architecture.

### Case 2: SMS (Notification Service)

<<Figure>>

We heavily use SMS for communicating with our customers/drivers/merchants. To make our systems resilient against SMS provider failure, we integrate with multiple SMS providers. 
To decouple ourselves from failures on our SMS providers, we use queuing to deliver messages. Messages in the queue are removed when they are successfully sent through our SMS provider. 
We retry SMS with a different provider when our primary provider fails to meet a certain SLA for delivering SMS.

## Pattern[9] = Monitoring and alerting

```Monitoring is measuring some metric over a period of time```

```Alerting is sending a notification when some metric violates its threshold```

Resiliency does not mean that your systems will never fail rather it means that it recovers quickly and gracefully from faults.

Monitoring and alerting can help you measure the current state/health of your system so that you are alerted of the faults in your system as quickly as possible. 
This can prevent faults turning into failures.

For example: Without monitoring/alerting you might not be able to detect a memory leak in your production application before it crashes.
At GO-JEK, we collect various kinds of metrics which includes system metrics, app metrics, business metrics etc. Also, alerts with thresholds are set for various kinds of metrics, so that when a fault happens, we know beforehand and take corrective measures.

Monitoring with alerting can help in 2 ways:
- It notifies you of potential faults in your system, helping you recover from it before it causes failure in all of your system (Uptime)(Proactive)
- It can help diagnose the issue and pinpoint the root cause to help recover your systems from failure(MTTR — Mean time to recovery)(Active)

## Pattern[10] = Canary releases

```It is releasing a new feature/version of a software to only a certain set of users in order to reduce the risk of failures```

More about canary release [here](https://martinfowler.com/bliki/CanaryRelease.html)

Canary releases at GO-JEK are done at various places:

### Case 1: Back-end Deployments
Every deployment on backend involving critical changes (like the algorithmic change to find a driver) goes through a canary release. Wherein the new version of the software is deployed only to a single node or to a selected set of nodes.
Usually, this is coupled with monitoring for failures, latencies etc on that node. Only after monitoring the system for a particular period of time, is the new version of the software released to all users.

### Case 2: Mobile releases
Even new app rollouts go through this phase of rolling out to a small set of users which is again coupled with gathering metrics of usage, crashes, collecting feedback on the new features etc.
Only when the team is confident about the new version do they go with the full release.
Canary releases help you avoid risk of failures on new version/feature releases, in-turn helping resiliency of systems.

## Conclusion

All the above patterns can help us maintain stability across our systems. 

If there is one thing you take home from these patterns it should be …

```The more coupled your services are, the more they are together in failures```

In conclusion, Failures will happen, we need to design systems to be resilient against these failures.
