--
layout: post
title: "You wouldn't trust bad advice. Don't trust bad data!"
date: 2020-11-17T00:21:42-08:00
categories: data quality 
draft: false 
---

"Why Software Is Eating the World" - [Marc Andreessen](https://a16z.com/2011/08/20/why-software-is-eating-the-world/)

"Data Scientist: The Sexiest Job of the 21st Century" - [DJ Patil](https://hbr.org/2012/10/data-scientist-the-sexiest-job-of-the-21st-century)


Software is in fact eating the world. But why is it eating the world? It's because generally software reduces friction points in almost any process or product that doesn't have software. Teams building software are looking to build products that make users lives' easier.

So how do we build products that make users lives' easier? Well before software you would have focus groups, surveys, and user interviews. Theses were effective means to gain a sample of understanding of how users are using your product. Learning where the friction points are. What was confusing.

Software is eating the world though. So it is only a matter of time before all of this is automated as well. Meaning that eventually most of this will be explicitly collected by software which means the scale of the data collected explodes. We collect product events, interactions, intents and store them for later. So that we can build better products.


### What value does data bring?
- Ask any Product Manager and their answer will be to better understand how people are using the product.
  - How to get users to convert to paying users
  - How to get paying users to stick around and get value out of the product
  - How to stem the bleeding of churned users


Sound good? Do I have you sold yet? But what if, just like software, something breaks in your data? Incorrect concusions are drawn. The product gets worse. The software loses some ground. A user is frustrated and churns.


This is why having a means to monitor the quality of your data is so important. As with any system: `Garbage in = Garbage out`. So how does a team monitor their data for quality? What even is quality when it comes to data. We all know how to check if a banana at the supermarket is bad. Or if the bread in the kitchen is bad. What characteristics does good data have and bad data lack?


## What is data quality?
- Latency
- Accuracy
- Precision
- Completeness


### Data Latency
Data latency is the easiest feature to define. It's how old is the data we are looking at? What is the delta between now and state of the data being represented?


### Accuracy
Accuracy is how faithfully does the data represent reality. Example of bad accuracy would the timestamp field says that an event happened at X time but in reality it happened 4 hours before then because of timezones or a bad clock. For web collected data there are about a million ways things can go sideways between the browser and wherever data is getting stored so you may find a significant % of data has malformed DOM elements if you are recording page interaction.


### Precisions
Precision of data is does the data have the correct data type to represent the data correctly. If you run a HFT firm and you are studying transactions you need to have the transaction data have the correct precision for timestamps. Cutting timestamps off at the seconds is a dealbreaker. The data is Accurate, but not Precises. Meaning that it is bad data for that use case.


### Completeness
Completeness of data is whether or not the data is lossy. Fields missing or entire rows missing would be incomplete data. 


## How do you set expectations about data quality?
- SLA - Service Level Agreement
  - Latency - Latency Guarantees
  - Accuracy - Tests Pass Rate
  - Precision - Type checking & Tests Pass Rate
  - Completeness - Tests Pass Rate


## Ensuring Quality

For inspiration on how we can ensure that teams have quality data we can look at a group of people who have been struggling with this problem since the very beginning: Software Engineers. Building software is the exact same as building data. It is riddled with bugs, mistakes, and breaking changes. What you have to do in order to make any progress in this world is to code defensively. You know you are going to make a mistake. Instead of trying to detect that mistake yourself build some software that will detect if for you! and that's how tests were born.

So what the hell do tests for _data_ look like?

Well to be frank, quite a bit like tests for software.

### Kinds of tests
- Rule Based Tests
  - Completeness
  - Precision
  - Latency
- Static Analysis
  - Precision
- Unit Test
  - Accuracy
  - Precision
  - Completeness


### Rule Based Tests

Rule based tests are the simplest of all of these tests. They basically just check to see if things are within bounds. Is the `created_at` field in the period that you expect? Like is it within the last 15 minutes? Does every column contain values? Are there any gaps of rows where there shouldn't be? This is the kind of test where you can apply machine learning and infer what the distribution of data _should_ look like and so based on the rules of the model you can alert on any outlier.

Rules based tests can also be used to ensure the precision of the data. By testing basic rules like if a UInt64 always has the precision of a UInt32, it might have been downsampled upstream and need to be flagged. Or if a varchar(255) always has exactly 255 characters used you might be truncating your string value.


### Static Analysis

This one is relatively easy to implement because you are only really looking at the schema. You want to make sure that all columns and fields have the datatype that is necessary for correct Precision. Is a UInt64 field always a UInt64 field? Does the query logic being used to fill keep the precision correct from start to end? If it doesn't you might be trusting a UInt64 field that was really downsampled to a UInt16 field and then brought back up.


### Unit Tests

This is probably the most valuable of the tests, just like with building software. Since most data being build is basically taylor made along with the software that's producing it no one knows better about what looks right or wrong than the person building the logic! This is why unit tests are so great in software. As the engineer you know that for your `add(x, y)` function `add(1,1)` will always return `2`. You can write that up as a rule protecting you from your future self. This is what we need in data.

A data team who invests the time in creating these tests will be saving themselves loads of time later trying to track down bugs. It also reduces the time to detecting issues since the test suite will pick up the regression or the bug almost immediately if it is setup correctly. As a newly onboarded data team member having these tests reduce the barier to entry since there is a security blanket associted with a test suite. You can push a change with the confidence of knowing that what you just pushed will not silently break something.


## So why do we want to do all of this?

There are so many reasons why you should invest in your Data Quality infrastructure. At it's core though, it is about confidence in the data. Just as with people, trust is hard earned and easily lost. If your team loses confidence in the data being collected they won't use it. It will no longer be a trusted ally but a deadbeat that never has your back.


## Products with Data Quality Management features
- [BigEye](https://www.bigeye.com/)
- [Soda.io](https://www.soda.io/)
- [great_expectations](https://greatexpectations.io/)
- [Socrata](https://www.socrata.com/)
- [talend](https://www.talend.com/)
- [mobyDQ](https://github.com/ubisoft/mobydq)
- [AWS Glue DataBrew](https://aws.amazon.com/glue/features/databrew/) <- more munging of data
- [DBT](https://getdbt.com)


### Appendix:
- [Data SRE](https://medium.com/bigeye/seven-principles-for-reliable-data-pipelines-e82a82810e4f)
- [Trust @ Uber](https://dribbble.com/shots/4748117-Trust-Data-Quality-Alerting-System/attachments/1069568?mode=media)
- [DBT Tests](https://docs.getdbt.com/docs/building-a-dbt-project/tests/)