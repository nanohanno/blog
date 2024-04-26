---
layout: post
title: "Watermarks"
subtitle: "Am I complete?"
date: 2024-04-19
background: '/images/default_post.jpg'
---
In stream processing, everything is based on event time, which gives events from different sources a notion of their place in time. This is important because in event streams we do not have the security that we have seen all events as we have when processing bounded streams like in batch data.

## Distributed chaos

Event streams can come from different sources, meaning actually that the producers could be different applications or running on different machines, they could be even edge devices running around all over the world. Next, the event queue, which holds and forwards these events, like Apache Kafka can be a distributed system, having multiple partitions over different servers. 

Eventually, all these distributed components can add specific latencies and instabilities to the system. A consumer, the stream processor, might read events from the same stream that come in via different routes which introduce different latencies. Eventually, the event time of these events is not in order anymore.

Often stream processing uses time windows to aggregate values, like counting logs or taking average of a metric from events in the last hour or 5 minutes. How does the stream processor know if we have all events that would fall into such a time window based on event time?

## Watermarks

The heuristic that is used to answer that question is called watermark. In principle, it checks the event times of the last ingested events and reports the latest event time as the current watermark. It means that the expectation is that all events previous to that time have been ingested. If we know that we have a system that has out of orderness in event times the watermark needs to be adjusted by a certain delay to reflect that we are only sure to have observed all events up to that delay before the latest observed event because some events could come in a little later.

## Tradeoff

There is an inherent tradeoff like so many in distributed systems. If we want to make sure to have observed all events in a time window we can increase that delay and wait a long time before the window gets processed. On the other hand, often, stream processing systems should be quick and not add a large latency. Thus, we want to have a short delay to process the window as soon as possible.