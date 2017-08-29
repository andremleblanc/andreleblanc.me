---
layout: post
title: Announcing DruidDB-Ruby
excerpt: The gem was originally written to leverage the Tranquility Core library by including the JAR inside the gem, which dictated using JRuby...
tags: ruby druid opensource druiddbruby project
---

## What Does it Do

[druiddb-ruby](https://github.com/andremleblanc/druiddb-ruby) provides a client for your Ruby application to push data to Druid leveraging the [Kafka Indexing Service](http://druid.io/docs/latest/development/extensions-core/kafka-ingestion.html). The client also provides an interface for querying and performing management tasks. It will automatically find and connect to Kafka and the Druid nodes through ZooKeeper, which means you only need to provide the ZooKeeper host and it will find everything else.

## A Brief History of the Gem

### jruby-druid v1
The gem was originally written to leverage the [Tranquility Core](https://github.com/druid-io/tranquility) library by including the JAR inside the gem, which dictated using JRuby. At the time, making the gem dependent on JRuby was acceptable because our web app is dependent on JRuby. After getting everything working, tests included, we went to deploy the application and tragedy struck - we had conflicting JAR dependencies that didn't show up in development. We spent a couple of days trying to fix the conflicts with the JARs, but it became clear that the road to fixing the conflict was longer than switching how we pushed data to Druid.

**Lessons Learned:** Don't use JRuby. Also, make smaller iterations and get code into production ASAP (yeah, that whole Agile thing).

### jruby-druid v2
Around this time, Druid released an experimental version of the [Kafka Indexing Service](http://druid.io/docs/latest/development/extensions-core/kafka-ingestion.html). After experimenting with it, we decided to go this route as we wanted to use Kafka in other parts of our technology stack. We quickly rewrote the gem to leverage Kafka, breaking all our tests in the process. The refactor mainly changed how writes were performed but didn't touch the management and query side of the gem. Fortunately, this refactor worked beautifully and ran stable without any attention for almost a full year.

### druiddb-ruby v1.0
And then one day, we had another conflict. We wanted to write services in the web app that consume from and produce to Kafka so we brought in [ruby-kafka](https://github.com/zendesk/ruby-kafka). This conflicted with the [jruby-kafka](https://github.com/joekiller/jruby-kafka) gem used by `jruby-druid`. With the lessons learned from part one - don't use JRuby - it seemed like the right approach was to change which kafka gem `jruby-druid` used instead of forcing the app to use the JRuby version. This also made it obvious that the gem no longer had JRuby dependencies and could be converted to MRI - *win!* It actually only took a couple of hours to get all the work done on converting the gem and pushing it out to [rubygems.org](https://rubygems.org/gems/druiddb).

### druiddb-ruby v1.2
Since that change, everything has been running smoothly again. We process over 3 million events a day using this gem with Druid. Since the project has stabilized, it seemed like a good time to start adding those tests back. The challenge with testing this gem is that it requires a fairly complex dependency ecosystem including all the Druid nodes, Kafka, ZooKeeper, and Derby. I think that deserves a post of it's own, but bottom line, we got [CI running with Travis CI](https://travis-ci.org/andremleblanc/druiddb-ruby) and will be expanding the test coverage of the gem.

## Enjoy

Hopefully this gem saves you some time, I'd love some help on making it better.
