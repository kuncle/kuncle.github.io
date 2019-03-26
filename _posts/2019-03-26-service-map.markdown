---
layout: post
title:  "service-map"
date:   2019-03-26 14:55:00
categories: Other
tags: Other
---
#### metrics-tracing-and-logging
* I think that the defining characteristic of metrics is that they are aggregatable: they are the atoms that compose into a single logical gauge, counter, or histogram over a span of time. As examples: the current depth of a queue could be modeled as a gauge, whose updates aggregate with last-writer-win semantics; the number of incoming HTTP requests could be modeled as a counter, whose updates aggregate by simple addition; and the observed duration of a request could be modeled into a histogram, whose updates aggregate into time-buckets and yield statistical summaries.

* I think that the defining characteristic of logging is that it deals with discrete events. As examples: application debug or error messages emitted via a rotated file descriptor through syslog to Elasticsearch (or OK Log, nudge nudge); audit-trail events pushed through Kafka to a data lake like BigTable; or request-specific metadata pulled from a service call and sent to an error tracking service like NewRelic.

* I think that the single defining characteristic of tracing, then, is that it deals with information that is request-scoped. Any bit of data or metadata that can be bound to lifecycle of a single transactional object in the system. As examples: the duration of an outbound RPC to a remote service; the text of an actual SQL query sent to a database; or the correlation ID of an inbound HTTP request.
 
#### service map tools
* zipkin
* jaeger

#### paper
*https://ai.google/research/pubs/pub36356
*https://ai.google/research/pubs/pub40378

* refer
http://peter.bourgon.org/blog/2017/02/21/metrics-tracing-and-logging.html
https://logz.io/blog/zipkin-vs-jaeger/
