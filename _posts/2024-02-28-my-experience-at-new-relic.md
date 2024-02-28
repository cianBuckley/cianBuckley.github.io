---
layout: page
title: My Experience At New Relic
subtitle: 
tags: ["kstreams", "kafka", "kotlin", "elasticsearch", "telemetry"]
keywords: kstreams, kafka, kotlin, elastic, telemetry
comments: false
---

Boy did I learn a lot at this company. I will be forever grateful for the experience I had working along side some of the smartest people I've met. 

I started as a junior engineer and ended up leading the "entity" ingest team. How did that happen? Well, frankly some of it was simply the luck of being in the right place at the right time! The CEO and a select handful of engineers went off on a coding retreat for a week to build a prototype of what would become [New Relic One](https://newrelic.com/platform), New Relics answer to unifying it's 6 distinct (and heavily siloed) product offerings (APM, Dashboards, Infra etc). The prototype was destined to come to the at-the-time small Barcelona office, as the ideator was based here. And that simple fact changed everything for the Barcelona office for the positive, and consequently for me!

Me and my work-mate were some of the only "backend" engineers in the New Relic office at the time, so we were handed a very early stage prototype based on scraping APM urls and indexing them in an elastic cluster and then exposing them via an API. As you can imagine, every shortcut under the sun was taken to get the bare-bones prototype to work. It was our job to scale this scrappy 200 lines of code to a fully functional system!

So we started top down with the API and worked backwards, following the scrappy "fake it till you make it" approach the prototype had set for us and slowly extend it into actual functional and scalable parts. Naturally, we started with APM, our most used product. We knew we wanted a dedicated system that would read all existing APM entities and index them into the elastic cluster. We also knew we would need to scale this to Infra products, which had a much higher throughput compared to APM - just think of all those ephemeral telemetry sources popping up and dying out!

So we made our first major architectural decision - We would create an interface to our platform that would permit product owners (like the APM or Infra team) to index their entities and make them available to the New Relic One UI. That interface made most sense to take the shape of a kafka topic, one for the sheer scale we were looking at (we since scaled this to millions of messages per second) and additionally, most product owning teams had access to kafka producers. Also, from a reliability front, putting a queue in front of the database made a lot of sense, more so for elastic considering we didn't want to view it as a source of truth for the data. This permitted us to re-index the datastore when required without fear of loss of data. It also put the onus on the product owners to tell us about their entities, rather than us trying to find out by ourselves.

From there, things took off. We ended up with a dedicated indexer for each product domain, merging all of these entities with active alerts targeting these entities and other meta data like tags. After about a year of this growth, we hit a bottleneck in the architecture. We needed more and more entities to be indexed and didn't have the man power to create an indexer for each one. Additionally, all product teams had their own projects and goals that may not have been related to them migrating to New Relic One. With that, the next step would be to consume the entire 
telemetry pipelines of New Relic and run a set of rules against each data-point as they came in prior to being sent to their time-series databases. 
This way, we would be able to derive the existence of any entity type prior to their data even being persisted without relying on product specific indexers (nor have the need to build them)

Naturally, injecting code into the critical path of our edge telemetry pipelines was risky and had significant backlash. We needed this to scale. Processing O(n) data-points would be enough to bring these monstrous pipelines down to their knees and choke all of New Relics products. We needed to be very selective in the data points we process and index. We ended up with a [open source rule engine](https://github.com/newrelic/entity-definitions) where anyone could create the schema and rule on which the index would become active. These rules are then consumed a packaged in a library that is injected into numerous telemetry pipelines and run on matching data points. 

The downstream of this also needed to be scaled. We used a mix of Kafka KStreams (using Kotlin) to cache data for real-time lookup, bloom filters, in-memory caches and redis caches as filters to prevent each matched data point indexing an entity entry every time. You could add to that many off-shoot projects in Node.js and Golang 

Naturally, an infrastructure platform like comes with significant team growth as well. By the end of my journey at New Relic, we grew from 2 engineers into 3 teams. I was leading one team (the entity ingest team) and the other original engineer leading the persistence team. The entity platform is still live and is still the backbone for all New Relic product offerings. Growing the entity platform and entity teams was one of the most satisfying challenges of my career

If any of this sounds relevant to your work, please don't hesitate to reach out and contact!