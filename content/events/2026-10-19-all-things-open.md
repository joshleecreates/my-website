---
title: "All Things Open"
date: "2026-10-19T00:00:00Z"
slug: all-things-open-26
params:
  location: "Raleigh, North Carolina"
  eventDate: 2026-10-19T00:00:00Z
  eventURL: "https://2026.allthingsopen.org/"
talks:
- "Observability is Broken and Open-Source (will be) the Solution"
---

# Observability is Broken and Open-Source (will be) the Solution

The Observability industry is built on a fundamental promise: Send all of your telemetry data to your vendor of choice, and they will apply their magic sauce to turn that telemetry into "insights."

In the beginning this promise was predicated on fundamental truths inherent in all SaaS models of the era: the data being sent was not too large, and not too valuable. We were content to ship our data off to become part of our vendors' statistical data models.

But that balance has shifted. Data volumes have grown exponentially - observability data doubly so. With this increased volume has come an even larger increase in Observability costs that can no-longer be ignored. Every vendor now charges directly or based on some approximation of bytes-ingested.

And now we all dream of using our data to train our own models; wouldn't that be liberating? This is only possible if we retain our data, or if our vendors were to somehow open up APIs allowing us to access it within their walled gardens.

OpenTelemetry is just the beginning of this sea-change. First it solved the problem of massive duplicated effort and lack of interoperability in the instrumentation layer. Now organizations are beginning to adopt the OpenTelemetry Collector as a way to change telemetry destinations in an instant.

But Observability can never be truly vendor-neutral - and open - until the storage layer is open, standardized, and hostable anywhere.
