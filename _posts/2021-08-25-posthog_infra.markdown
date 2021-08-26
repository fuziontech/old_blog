---
layout: post
title: "Future PostHog Platform"
date: 2021-08-25
categories: data quality 
draft: false 
---

# What does the future PostHog Platform look like?

**This is a rough outline of what I would build given unlimited resources currently.**

1. **Support.** - The most time consuming thing we will need SREs for is simply managing on prem PostHog deployments. If people are paying us for PostHog the future is going to be filled with N+1 users looking for:
   1. Supporting and debugging issues
   2. Helping with upgrades
   3. Performance tuning
   4. Migrations
   5. License management 🤣
   6. The more we can automate getting ahead of all of this, the better.
2. **K8s installs**
   1. Support each of the following environments _better_:
      1. AWS
      2. GCP
      3. Azure
      4. DO
      5. IBM
      6. Metal
   2. Each of these environments have different levers we can pull. Each one is relatively similar in shape, but the amount of work to research, test, and implement is relatively large if we want to support truly large deployments.
   3. Potentially split out ClickHouse from our chart.
      1. This would be relatively easy to do at first for a handful of customers (basically bespoke) but eventually it would be nice to have the same set of tools we build for our cloud available to customers for scaling up ClickHouse outside of K8s.
3. **Cloud infrastructure**
   1. Automate ClickHouse cluster management
      1. Resharding of sharded tables
         1. This is a big one as it is relatively bespoke the way that ClickHouse suggests running resharding jobs. Basically with ClickHouse Copier. Not exactly something we can run on customers datasets. I'd like to have this be automated. It may not be possible though.
      2. Backups and point in time restores
         1. Our backups still need a lot of work.
      3. Compliance
         1. SOC2 Type I
         2. SOC2 Type II
         3. HIPAA
         4. GDPR
            1. Offering Data Processing Addendums
         5. CCPA
            1. Offering Data Processing Addendums
      4. General security improvements (there are a lot)
         1. Restrict public access to production
         2. Proxy access to internal tools through a single auth source (GitHub, Google, etc)
         3. Proxy production access the same way
         4. Audit all access
      5. Single tenant support
         1. This is a big one at a similarly positioned company. They have a dedicated stack that they _do not want to manage_ and pay top dollar for a managed cloud version.
      6. Regional installations
         1. US
         2. EU
         3. China
         4. Russia
         5. Asian Pacific
4. **Shared infrastructure** - Cloud and on prem improvements
   1. Managing and migrating table schemas
      1. In order to iterate quickly we are going to need some way to manage the schemas and the migrations of the tables on ClickHouse. We do already have migrations in PostHog, but that's only for simple and fast migrations. This would have to be something more robust to support long running queries on ClickHouse. Something that might need to use `ClickHouse Copier`
   2. Object stores 
      1. Person store
         1. Eventually we will need a person store as we grow out of Postgres. This won't happen for a long time, but it is very possible it will happen at some point in the future. It may also become an issue where an on prem deployment is unable to manage or scale up a postgres instance.
      2. Event store
         1. If we are ever to append persons to events to avoid joins we will need to have a cannonical event store that lives outside of ClickHouse. We explored this earlier this year and decided against it because we were using Dyanmodb, probably for the best.
      3. Both of these would most likely live in something like [ScyllaDB](https://scylladb.com/) but more research is needed. Maybe we use Vitess?
   3. Backups
      1. Point in time restores for paying customers. Basically have the ability to replay events from some storage:
         1. Block store like EBS (or equivalent)
         2. Blob store like S3 (or equivalent)
   4. Improve admin dashboard
      1. We should invest in better self service tooling for managing PostHog.
      2. The `internal_metrics` dashboard has been a great start, but there is so much more that we can do.
      3. Plugin service metrics
         1. Events metrics
         2. Task metrics
         3. Plugin metrics
         4. Event -> Plugin factor
      4. Events service metrics
         1. Events / second
         2. 200s/400s/500s
      5. Kafka metrics
         1. Latency
         2. Disk usage
         3. Topic metrics
         4. IO metrics
         5. Consumer Group latency
      6. ZK metrics
         1. If ZK is a critical point of failure.
      7. ClickHouse Operator metrics
      8. Status of backups
5. **Beam me up! / Beam me down!** - Moving to cloud and back
   1. Some seamless way to migrate **everything** from cloud and back. This includes
      1. Action definitions
      2. Cohort definitions
      3. Insights
      4. Dashboards
      5. Plugin configurations (if able)
      6. Session recordings
      7. Events
      8. Persons & Distinct IDs
      9. Feature Flags
      10. Experiments
      11. Organization and Teams w/ Users (if able)
6. **Testing of other infrastructure deployment vectors**
   1. We did not use K8s at Uber, so right now if an engineer wanted to setup PostHog to instrument an interal app they would be SOL. We should investigate other means to deploy PostHog. This will increase the complexity of managing our infrastructure, but if we are serious about PostHog as a product that will be deployed internally this might be something we will have to support. (open product deployment strategy discussion)
   2. Is just a big Terraform what we might need here? Is there a better way to deploy PostHog if K8s is not available?
7. **Prototyping different infrastructure options.** - The ecosystem of infrastructure options that exist to us is huge and constantly changing. We should be looking at better options as we go.
   1. Vectorized RedPanda
   2. New versions of ClickHouse
   3. ScyllaDB for object stores
   4. Vitess
   5. PostHog Operator for K8s?
8. **Leveraging PostHog** - for complete monitoring and alerting of infrastructure related issues. This is a big one but something we should be striving for long term. What we build here could be huge in the future. We talked about this a while back and dismissed it as too hard. If we are larger and allocate resources to this - it is feasible.
   1. Metrics by dimensions
   2. Outlier/Anomaly detection
   3. Forecasting
      1. Holt-winters
      2. Linear regression
      3. so much to do!
   4.  Triggering events/actions/alerts based on arbitrary rules
9.  **Stream Processing** - This piggy-backs on the last line item. If we build out the API needed for triggering events on arbitrary rules we can build out a stream processing pipeline.
    1.  Triggering events/actions/alerts to external systems based on arbitrary rules, aggregates, predictions, outliers
    2.  Consider building out [Distributed RPC](https://storm.apache.org/releases/current/Distributed-RPC.html) systems using plugins as a basis
        1.  Hit this API and get a wholistic view of a user from PostHog, Hubspot, salesforce, and some internal API
        2.  Submit a certain type of event and call a series of downstream RPC endpoints
    3.  Build out better examples of building PostHog into core streaming data workflows in apps.
10. **Platform Integrations** - More ways to onboard to PostHog. We should write as many docs as possible to help gain developer and product manager mindshare but the truth is the easier we make it to onboard to PostHog cloud from a place someone mine discover us the better.
    1.  [Vercel](https://vercel.com/integrations)
    2.  [Heroku](https://elements.heroku.com/addons#metrics-analytics)
    3.  [Netlify](https://docs.netlify.com/configure-builds/build-plugins/)
11. **Data Engineering tools for PostHog.** *<- We have SREs applying with DE backgrounds. Perfect.*
    1. We have talked about this earlier with plugin exports but this would be dedicated, opinionated examples on how to load data from things like:
       1. Hive - needs Java Client library as UDF
       2. Hadoop - needs Java Client library
       3. Spark - needs Java Client library
       4. Presto/Trino - needs Java Client library as UDF
       5. Druid - needs Java Client library
       6. Airflow - Python ✅
       7. Snowflake - Library TBD
       8. DBT (for modeling data into the correct schema)
       9. Luigi - Python ✅
       10. Snowplow - Library TBD
       11. S3 - Using a task orchestrator like Airflow/Luigi
    2. Solid documentation on:
       1. What the schema needs to be for Events and Persons (from Data Engineering Perspective)
       2. Updating person properties and attributes effectively
       3. Filling in event gaps by using your data warehouse
       4. Backfilling event data (rewriting events)
12. **Client Libraries** - We still have a few to build
    1.  Java (We have Android!)
    2.  Java UDFs for Data Warehouses
    3.  C/C++ - Critical for IoT
    4.  Rust
    5.  C#
    6.  Scala (using Java lib) - Just example and docs
    7.  Clojure (using Java lib) - Just example and docs
    8.  Kotlin (using Java lib) - Just example and docs
13. **Pluggable backends** - Remember, unlimited funding. Make it possible to replace the backend of PostHog with a different OLAP database. I'm still betting my money on ClickHouse, but to land certain deals this would be a deal-making option.
    1.  Presto/Trino
    2.  Druid
    3.  Snowflake
    4.  BigQuery
14. **ClickHouse Stewardship** - Contributing back to ClickHouse
    1.  Help steer the roadmap for ClickHouse to better align with our long term goals.
    2.  Speed up the development of features that are on the horizon that we need sooner than later
        1.  Window functions
        2.  Materialized Columns - force materialization
        3.  Better resharding and schema mutations
            1.  Change table engine in place
            2.  Change sort by keys in place
            3.  Change sample by key in place
        4.  Bug fixes
        5.  Performance improvements
    3.  Improve generic tooling around ClickHouse
        1.  ClickHouse Operator
        2.  ClickHouse Copier
        3.  ClickHouse Backup