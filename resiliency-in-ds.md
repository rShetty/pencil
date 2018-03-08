# Resiliency in Distributed Systems

Before jumping into discussing resiliency in distributed systems, Lets
quickly refresh some basic terminologies:

### Basic Terminologies

#### Resiliency 

It is the capacity of any system to recover from
difficulties.

#### Distributed Systems 

These are networked components which communicate
with each other by passing messages.

#### Availability 

Probability that the system is operating at time `t`.

#### Fault 

Fault is an incorrect internal state in your system.
Some common examples of fault in our systems include:

1. Slowing down of storage layer
2. Memory leaks in application
3. Blocked threads
4. Dependency failures
5. Bad data propogating in the system

#### Failure

Failure is inability of the system to perform its intended job.
Failure means loss of uptime and availability on our systems.

Faults if not contained from propogating can lead to failures.

Also, `Resiliency is all about preventing faults turning into failures`

### Why do we care about resiliency in our systems ?

Resiliency of your system is directly proportional to your uptime and
availability. The more resilient your systems are the more available you
are to serve your users.

Failing to be resilient can affect companies in many ways:
ex: GO-JEK going down can have these effects: 

1. It can lead to financial losses for the company
2. Losing customers to your competitors 
3. Affecting livelihood of drivers 
4. Bad publicity

Software running in health care sectors must be extremely resilient from
failures as it could potentially affect people lives.

### Resiliency in distributed systems is hard

We all by now understand that being available is critical. And to be
available, we need to build in resiliency from ground up in our systems so
that faults in our systems auto-heal.

But building in resiliency in a complex microservices architecture with
multiple distributed systems communicating with each other is difficult.
Some of the things which make this hard are:

1. The network is unreliable
2. Dependencies can always fail
3. Users behavior is unpredictable 

Though building resiliency is hard, it is not impossible. Following some
of the patterns while building distributed systems can help us acheive
high uptime across our services. We will discuss some of these patterns
going ahead:

### Pattern 0: nocode

`The best way to write reliable and secure applications is write no code
 at all`

Write Nothing and Deploy nowhere - Kelsey Hightower

Though being sarcastic, this pattern has a important message in it.
Writing less code means less reasons for your system to fail.

### Pattern 1: Timeouts

_Stop waiting for an answer_

The default Go http client has no HTTP timeout. This causes your application to leak goroutines. To avoid this problem, it is important that we add timeouts for every http integration point in your application.

Timeouts in application can help in following ways:

1. Preventing cascading failures 

Cascading failures are failures which propogate very quickly to other
parts of your system. These are very bad if unmanaged. 

Timeouts help us prevent these failures by failing fast. When downstream
services fail or are slower (violating their SLA), instead of waiting for
the answer forever, you fail early to save your system as well as the
systems which are dependent on yours.

2. Providing failure isolation
3. Failing fast

Timeouts must be based on the SLAs provided by your dependencies

### Pattern 2: Retries

_If you fail once, try again_

When we have a failure, our immediate instinct is to try again. To a degree, this translates to distributed systems. Retrying allows you to reduce your recovery time.

By doing this, you are...
1. Reducing recovery time

Idempotency is important when you're retrying. Non-idempotent write operations can cause your application to be in an inconsistent state if you are retrying.

### Pattern 3: Fallbacks

_Degrade gracefully_

1. Degrades gracefully
2. Saves critical flows from failure

Distance between origin and destination fallback.

### Pattern 4: Circuit Breakers

_Trip the circuit to protect your dependencies_

By doing this, you are...
1. Preventing cascading failures

Hystrix is a circuit breaker implementation by Netflix.


### Pattern 5: Testing

Resiliency testing.

_Find failure modes_

1. Create a test harness to break callers
2. Simulate failure modes
3. Inject failures [Chaos Monkey, Latency monkeys, the entire Simian
   Army

Conclusion: 

Though following some of these patterns will help you acheive resiliency,
these are no silver bullet. Systems do fail, it is important to keep the
impact of these failures have on your users to minimal and these patterns
help you acheive those.

`Design the system keeping failure in mind`

`Design for failure`
