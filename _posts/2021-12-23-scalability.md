---
layout: post
title: Scalability in Consent Management    
author: Mohammad Jafari
date: 2021-12-23 11:11:11 -0000
categories: [blog]
tags: [architecture]
---

The goal of the LEAP Consent Project was to provide a _scalable_ architecture for capturing, managing, and enforcing patient consents based on the [FHIR Consent resource](https://www.hl7.org/fhir/consent.html). This post briefly discusses different aspects of scalability in consent management.

From the application perspective, a scalable architecture for consent management should accommodate the following dimensions of growth:

-	Growing number of consents from a growing number of enrolled consumers as electronic health records become more ubiquitous.
-	A growing landscape of health applications where the integration of consent capturing, consent management, and consent enforcement is a requirement; this leads to: 
	- a growing number of consent management transactions, and
	- a growing number of consent enforcement queries.
-	Growing level of sophistication in consumers’ control over their health data and care process, enabling them to express more complex rules and preferences about the user of their data.

These aspects of scalability lead to a number of major requirements for a consent management solution and have been considered as major principles in the design of the LEAP Consent architecture:

-	_Decentralization_ to allow a diversity of methods, systems, and organizations for consent capture and management.
-	_Interoperability_ to ensure that all the elements and participants in the decentralized architecture remain compatible and capable of seamless integration and collaboration.
-	_Computable Consents_ to enable automatic enforcement. A computable consent is a representation of consent in which privacy preferences are encoded in the form of machine-readable rules (as will be discussed further below).

### Computable Consent
From a technical standpoint, capturing, storing, and retrieving consents is rather similar to management of other types of documents and therefore not challenging per se. But consent enforcement is one of the bottlenecks where manual or proprietary solutions can impede scalability and interoperability.
 
To enable automatic and seamless integration of consent enforcement into different applications and workflows, it is imperative that consent rules are stated in machine-readable and interoperable form. Such rules can be processed by a decision engine to adjudicate whether the consent permits a specific given activity. This is one of the core features in ensuring a scalable consent architecture where large number of consents with complex rules can be efficiently enforced. Computable consents were one of the main focuses of the LEAP Consent project and one of its major contributions to the [HL7 FHIR Consent specifications](http://build.fhir.org/consent.html#6.2.1.1).
