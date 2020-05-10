---
layout: post
title: High-Level Architecture of LEAP Consent Decision Service (CDS)
author: Mohammad Jafari
date: 2020-04-29 11:11:11 -0000
categories: [blog]
tags: [architecture, cds]
---
The LEAP Consent Decision Service (CDS) provides a simple API interface for inquiring about a patient's consent in the context of a workflow, such as a record exchange. 

Once it receives a query, the CDS consults with various Consent Stores and after processing all the patient consent resources applicable to the context of the query, issues a decision as to whether or not the patient consent permits the activities in the requested context and whether any obligations apply. 

Consent Stores are FHIR servers where patient consents are stored. Each Consent Store may be operated by a different organization that collects and maintains patient consents.

The workflow context is captured by a Consent Enforcement Service (CES) and is sent to the CDS in the form of a query. The CES is tightly coupled to the application and is able to collect all the relevant attributes from the context of the workflow and also to interfere in the workflow to enforce the consent decisions, for example, by rejecting an end-user request,  terminating a workflow, or modifying the workflow path or data based on obligations received from the CDS. 

This model is depicted in Figure 1.

![architecture][architecture]
*Figure 1. High-Level Architecture of the Consent Decision Service*


### Consent Discovery Service
The Consent Discovery Service is in charge of finding and retrieving patient consent resources applicable to the context of the query from all known consent stores.

The query context in a CDS request requires specifying a patient identified by providing a set of patient identifiers. Multiple identifiers are allowed here since the patient may be known to different services by different [types of identifier](https://www.hl7.org/fhir/valueset-identifier-type.html), such as SSN, driver's license, passport number, etc. 
Using these identifiers, the Consent Discovery service searches for a matching patient at each of the consent stores, and if the patient exists, retrieves the local FHIR identifier for the patient resource. Using this local FHIR identifier, the Consent Discover Service can search for any consent resources pertaining to the patient. Additional query context attributes, such as the purpose of use, will be used at this stage to narrow down the search and retrieve only consent resources that are precisely applicable to the context at hand.

### Consent Processing Service
The Consent Processing Service is in charge of evaluating the request context against all (potentially) applicable consents retrieved by the Consent Discovery Service. The Consent Processing Service examines these consents one by one and determines the decision as to whether the activities in the context of the query are permitted and whether any obligations must be applied. The decision may also be that simply no applicable consent was found.

In case more than one applicable consent is found, the Consent Processing Service also adjudicates any potential conflicts to produce a single final decision based on a _decision combining strategy_. Currently, the only combining strategy implemented by the CDS is that in case of any conflicts, the most recent consent prevails.







[architecture]: {{site.baseurl}}/assets/img/general-architecture.png "Figure 1. General Architecture of the Consent Decision Service"
