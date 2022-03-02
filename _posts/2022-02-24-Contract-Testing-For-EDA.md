---
layout: post
title: Contract Testing for Event Driven Architectures - Can We, Do We and How Do We?
---
 

## Introduction

In recent past, I had the chance to involve deeply in setting up a developer testing strategy for microservices that follow Event Driven Architecture (EDA) patterns. As a part of this, we performed a deep dive analysis to understand how to introduce Contract Testing to EDA. 

While Contract Testing is an important testing concept which became extremely popular in the microservices world, it is mostly associated with the RESTful realm. Therefore when it comes to messaging, we wanted to evaluate how contract testing fits in the test pyramid. Can we actually perform Contract Testing? Are there any challenges? What value does it bring? Pressed reset button and we went into research mode as it required us to dig deeper. 

<!-- ![[Test-Pyramid.excalidraw]] -->
![Testing Pyramid](https://raw.githubusercontent.com/chinthakadd/chinthakadd.github.io/master/_posts/images/Test-Pyramid.png)

In this article I would like share what we learnt and the opinion that we built around it. As all opinions go, it is subjected to change, but given the current context, we think it is the right way to go.

**First of all, why are we even talking about Contract Testing?**

I am not going to bore you with my explanations around how Contract Testing typically work. Instead I have referenced some of the good literature around that in References section.

At a gist, Contract Testing is a verification mechanism to ensure that producer - consumer relationships that get formed due to API based integration are maintained in a healthy fashion. Given it is a Test that lives lowers in the test pyramid, it allows you to identify such potential breaking changes earlier in the development lifecycle. In essence, Contract Testing is all about testing the honesty of consumers and producers and evaluate how truthful they are to each other in this relationship they have built. It provides a platform for communication and collaboration and helps them built a long lasting relationships. If you want to learn Consumer Driven Contracts and also have a good laugh while doing so, I recommend watching the following presentation by Ben Sayers and Mauri Edo from 2016 [Video](https://www.youtube.com/watch?v=-6x6XBDf9sQ&list=PLiMP2R4ptDW3aFt2zN0qmgvdmJnXQKePh&index=7&t=662s)

Contract Testing is a well established pattern in the REST world. Specifically the Consumer Driven Contract (CDC) Testing is widely adopted. In CDC, Consumer writes the contracts that defines their expectations and publishes it. These contracts are verified against the Producer APIs. There are two main frameworks for Consumer Driven Contract Testing that are widely adopted.

Those are:

- Pact - https://docs.pact.io/

- Spring Cloud Contract - https://cloud.spring.io/spring-cloud-contract/reference/html/project-features.html

While the workflows provided by Pact and Spring Cloud Contract (SCC) have subtle differences, the concepts stands similar and can be applied nicely in the REST scenario.

Apart from Consumer Driven form of it, Contract Testing has grown in scope with introduction of various sub concepts such as Provider Driven Contract Testing, Bi-Directional Contract Testing and Schema Driven Contract Testing and etc.

On the other hand, Event Driven Architecture (EDA) and Messaging has a very very broad scope as well. I am not going to get into those details. Instead let me get back to what our original intention was and define the scope our EDA that is more specific and relatable to most industry standard implementations.

Assume the following:

- We have an microservice architecture which uses **Kafka** as the messaging platform. 
- These microservices are Event Driven and uses two main patterns: Events and Command-Reply. There are two forms of Producer - Consumer relationships.
	- Producer microservices raises events of interest to a Kafka Topic and Consumer microservices that are interested in those events will listen to that Kafka topic
	- Consumer microservices raises commands via a Kafka Topic and Producer microservices act on those commands and reply with a Reply Kafka Topic
- Kafka Messaging uses Avro as the Serialization mechanism (Or assume other schema driven serialization formats but definitely not JSON)
- We also have a Schema Registry in place to support schema collaboration and compatibility verification (assume Confluent Schema Registry)

<!-- ![[EDA-Highlevel.excalidraw]] -->
![EDA with Kafka & Avro](https://raw.githubusercontent.com/chinthakadd/chinthakadd.github.io/master/_posts/images/EDA-Highlevel.png)

If we are thinking of an Event Driven Architecture using Kafka as the Messaging Platform, this would be a common scenario. Next we need to look at how Contract Testing becomes applicable in this context...

## Is Contract Testing  applicable for Messaging?

Well, given Messaging is also a form of API, Contract Testing can apply to messaging and EDA as it also results in Producer - Consumer relationships very similar to REST. 

So let's try to define the requirement for Contract Testing in the messaging use-case. Let's start with Consumer Driven Contract Testing for event messaging and define our requirement.

```
Given:
I have a suite of Event Driven Microservices that communicate with each other using Event Messages

When:
A Producer - Consumer relationship is formed when a Consumer subscribed to Producer's Event Topic

Then:
I want the Consumer to define their expectations in the form of a Contract

And:
I want the Producer to comply with this contract and run verification tests against the Producer's Event Publication logic before it gets deployed to production

```

This looks a very reasonable requirement. Now how do we achieve this? To find answers to this question,  we needed to look at `Pact` and `Spring Cloud Contract` and see how can use those frameworks to support this requirement.

## Messaging Support in Pact & SCC

- How Spring Cloud Contract provides messaging support?

https://cloud.spring.io/spring-cloud-contract/reference/html/project-features.html#features-messaging

- How Pact provides messaging support? 

https://docs.pact.io/getting_started/how_pact_works#how-to-write-message-pact-tests

Diving deep into these two approaches, we came to the realization that these frameworks have very different ideologies when it comes to messaging support of contract testing (with different but valid reasons of course)

As I mentioned before, in the REST APIs, both Pact and SCC follow the same principles. Both frameworks have implemented Consumer Driven Contract Testing as a form of Integration Testing.
When I mean Integration Testing, I really mean it because it involves a wire protocol. On consumer side, the actual client layer is tested against a HTTP mock server. On producer side, the actual REST / HTTP API is invoked and response is validated against the consumer contract.

But when it comes to Messaging...

Spring Cloud Contract follows a consistent approach here as well and Contract Testing happens against the actual Broker like a typical integration test. But Pact instead recommends that we test the service layer not the real integration layer. More like a unit test.

```
"""We recommend that you split the code that is responsible for handling the protocol specific things - for example an AWS lambda handler and the AWS SNS input body - and the piece of code that actually handles the payload."""
```
Quoted from: https://docs.pact.io/getting_started/how_pact_works#how-to-write-message-pact-tests

If we follow the Pact Approach, it means that Contract Testing will follow different philosophies for messaging vs. REST. 

So we started asking ourselves, why should we make two different choices for two different protocols.
For Pact as a Framework, their design choice makes sense to them.

```
To reiterate: Pact does not know about the various message queueing technologies - there are simply too many! And more importantly, Pact is really about testing the messages that pass between them, you can still write your standard functional tests using other frameworks designed for such things.
```
Quoted from: https://docs.pact.io/getting_started/how_pact_works#how-to-write-message-pact-tests

But would that simply makes sense for us when we have standardized our architecture on one messaging protocol and that is Kafka? Therefore even though it is an approach that would definitely work, we were reluctant to subscribe to the idea of testing contracts in Service Layer instead. 

SCC on the other hand allows us to be consistent. So the use of SCC approach for messaging started to be bit more interesting and aligned with our thought process.

<!-- ![[SCC-Pact-Messaging.excalidraw]] -->
![Different Approaches of Pact and SCC for Messaging](https://raw.githubusercontent.com/chinthakadd/chinthakadd.github.io/master/_posts/images/SCC-Pact-Messaging.png)

Above diagram depicts the difference between the two approaches. But more to come...

When we talk about REST, we implicitly mean REST / JSON most of the time. But in messaging, JSON is not the standard. Remember the scope I outlined as well. We want to use a schema oriented serialization format, i.e. Avro. 

Does SCC support Kafka / Avro? In fact, does SCC support Kafka / Avro + Schema Registry? Because use of Avro in Kafka gets augmented a bit with the use of Schema Registry. The answer is No. There is no direct support for Avro or Schema Registry. 

This is where we came to a bit of dilemma. How do we go from here? So we started breaking the problem down. Let's focus on actual difference of REST vs. Messaging with Kafka. 

## REST vs. Messaging - They are not the same...

In REST APIs, we define a Specification using Open API. The term "specification" is the key here. It is not a real schema. We use JSON as the serialization format for request and responses which is not schema oriented. The lack of such schema orientation made Contract Testing absolutely vital in the context of REST. We had no other choice to validate if the specification is actually implemented in the producer side. 

Also most producer APIs were often poorly designed by our own mistakes breaking Single Responsibility Principle. We often made APIs do many things in one, and it was easy because there was no schema or a rule that prevented us from doing so. If the design was proper, the specification itself could have covered most of the contracts. But it never did.  On the other hand, specifications are not capable to upheld compatibility rules. They were not meant for it.

**Can we learn from REST and do somethings different in Kafka?**

Yes we can. In Kafka with Avro, we have one more additional tool at hand. That is the `Schema`. Schema is powerful than the specification in this respect. You can apply compatibility rules for a schema and make sure that producers can only change their production within a limited scope that doesn't break consumers. Schema Registry provides this verification capability and it can be done far early in the development cycle even before testing.

Next let's think about message modeling. In an Event Driven Architecture, messages in the form of Events and Commands are entitled to have a Strong Schema definition. Those must be defined with single responsibility in mind. For instance, we should not create a generic multi-purposed command messages grouping many commands. That easily creates confusions and many interpretations of the same schema introducing the need for different consumer contracts. 

For example, assume the case of transfers where there are two sub-types, i.e. Internal Transfers and External Transfers. External Transfers require to hold lot more information typically compared to internal transfers (ex: Bank's Routing Number). Defining a single command called `InitiateTransferCommand` or an event called `TransferEvent` for both sub-types would create the need for schema relaxations and different contract definitions. Therefore, creating a command / event for each type of transfers is recommended instead taking Single Responsibility Principle into consideration.

So if our architecture is based on the following:

- All message models and topics are defined with Single Responsibility Principle in mind. 
- We define such strong schema definitions with only small percentage of relaxed rules for attributes that are optional
- We apply proper compatibility rules for all schemas and verify them through schema registry

If we can upheld all of the above, is it still worth doing Consumer Contract Testing for messaging? This was a question that we gave serious serious thought into. 

Further more and finally. Contract Testing is most effective when you have a manageable number of consumer-producer relationships. Specifically Consumer Driven Contracts are most efficient when consumers are well-known and are less in number. But in the messaging and EDA, this could not be even practical. Events specifically are supposed to be open for consumption and subscribed by any other microservice interested in that event. So EDAs can easily lead to (Too) Many Consumers to One Producer relationships that makes things difficult to manage Consumer Driven Contract. 

So it becomes a trade-off between the value of testing vs. the maintainability cost of these tests. 

## Is CDC the only Option here for Messaging?

This was a question that we asked the community as well. [link](https://stackoverflow.com/questions/71028845/do-we-need-to-do-consumer-driven-contract-testing-kafka-driven-micro-services-wh)

An alternative would be to write Producer Driven Contract Tests instead. However at present, the same challenges presented above for CDC in messaging applies in here as well. But this is an approach that may be promising if we can have tooling for it.

## Conclusion

Therefore, given the current context, looking at the messaging and schema oriented serialization format support of Contract Testing tools, our team decided that this is not the right time to invest our efforts in writing Contract Testing for Messaging. 

Instead, we focus our energy on defining single responsibility driven messaging schemas, binding schema compatibility rules for our topics and messages and finally doing Schema Compatibility Testing leveraging the Schema Registry.

Like I said, we will keep an open mind about this. The community is very active in this area and a good amount of discussions are happening around the future of Contract Testing. So we hope we have better answers as we keep digging. But at this point, I wanted to share these thoughts hoping that it will benefit others who are going through the same research. I am sure this is a recurring problem. 

Of course if there are better alternatives, feel free to suggest and always happy to learn!!! Cheers..!!


## References

- Great introduction to Consumer Driven Contract Testing & Pact:
	- https://www.youtube.com/watch?v=-6x6XBDf9sQ&list=PLiMP2R4ptDW3aFt2zN0qmgvdmJnXQKePh&index=7&t=662s

- Discussion on how to do Contract Testing in EDA:
	- https://stackoverflow.com/questions/71028845/do-we-need-to-do-consumer-driven-contract-testing-kafka-driven-micro-services-wh

- Lack of Support for Avro/Schema Registry in Contract Testing Frameworks 
	- https://stackoverflow.com/questions/66654842/spring-cloud-contract-tests-for-avro-messages-using-schema-registry
	- https://github.com/pact-foundation/pact-jvm/issues/603
	- https://www.youtube.com/watch?v=75uA3X2lVqo

- Schemas vs. Contracts:
	- https://pactflow.io/blog/schemas-are-not-contracts/

- Schema Evolution and Compatibility:
	- https://www.confluent.io/blog/schemas-contracts-compatibility/

  