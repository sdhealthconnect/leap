---
layout: post
title: High-Level Architecture    
author: Mohammad Jafari
date: 2021-09-30 11:11:11 -0000
categories: [blog]
tags: [architecture, computable-consent, leap-cds, leap-ces]
---

The goal of the LEAP Consent Project is to provide a scalable architecture for managing and enforcing patient consents based on the [FHIR Consent resource](https://www.hl7.org/fhir/consent.html). 

A high-level overview of the architecture is depicted in Figure 1. This figure shows a full arc from capturing patients' preferences in the form of patient consents, all the way to enforcing these preferences in different application contexts and workflows. The dependencies between all of the components are based on standard FHIR API and FHIR-related protocols with no tight dependency or proprietary links that could tie the system to a specific implementation.

![high-level][high-level]
*Figure 1. LEAP Computable Consent Architecture*

### Consent Stores
FHIR Consent Stores are FHIR Servers where FHIR `Consent` resources (and other related artifacts such as `Patient`, `Organization`, `Questionnaire`, `QuestionnaireResponse`, `Provenance`, and `AuditEvent`) are stored and are accessible via the FHIR API. Each Consent Store may be operated by a different organization that collects and maintains patient consents.

### Consent Management Service
The Consent Management Service enables capturing, managing, and maintaining patient consents. This usually entails a graphical user interface (GUI) to assist the patient in expressing their privacy preferences, handling record-keeping matters such as signatures and indexing, management features such as editing, revoking, and re-activating consents, as well as various reporting features such as viewing a log of access events authorized by a given consent. 

The LEAP Consent Management Service is an example of such a service implemented purely based on FHIR –with no dependency on any proprietary database or persistence layer. Various types of consent forms are modeled as FHIR `Questionnaire` resources. The GUI enables patients to walk through a consent form and express their privacy preferences by responding to questions. The raw outcome which includes the original answers by the patient is captured as a FHIR `QuestionnaireResponse` resource which is then turned into a computable `Consent` resource hosted by a FHIR Consent Store. The link between the `Consent` and the corresponding `QuestionnaireResponse` resource is recorded by a `Provenance` resource.

### LEAP Consent Decision Service (LEAP-CDS)
The LEAP Consent Decision Service (LEAP-CDS) is a decision engine capable of parsing and understanding patient consents and deciding, according to the patient's preferences encoded in the consent, whether an instance of access or activity should be permitted.

This service provides a simple API interface for checking consent decisions about activities in the context of a workflow, such as a record exchange, research, or treatment. 

After receiving a query, the LEAP-CDS consults with various Consent Stores and retrieves and processes all the `Consent` resources applicable to the context of the query. Based on this, it issues a decision as to whether or not the requested activity is permitted based on patient consents, and potentially whether any obligations apply. Additionally, it may also include a pointer to the `Consent` resource based which the decision was made to assist the client with audit and record keeping.

### LEAP Consent Enforcement Service (LEAP-CES)
Consent Enforcement Services (CES) are software components that reside at various workflows or application contexts and are in charge of:
- intercepting activities to identify the activities that require patient consent, 
- capturing the context of the activity to form a consent decision request, 
- obtaining the consent decision by communicating with the LEAP-CDS, and
- enforcing the decision in the workflow or application context. 

A CES is tightly coupled to the application context where it resides and is able to collect all the relevant attributes from the context of the workflow to form the consent decision request. It can also interfere in local workflows to enforce the consent decision, for example, by rejecting an end-user's request, blocking access to a data item, terminating a workflow, or modifying the workflow path based on the decision and obligations received from the CDS. 

This relationship between the LEAP-CDS and the LEAP-CES is depicted in Figure 1.

![architecture][architecture]
*Figure 2. Consent Decision vs. Enforcement*

The LEAP project provides proof-of-concept implementations of Consent Enforcement Services for a number of application contexts. This includes:
- [CES Implementations for Legacy Exchange Technologies](https://github.com/sdhealthconnect/leap-demos), including HL7 v2.0 Messaging, eHealth Exchange, and Direct Exchange
- [A proxy-based CES for FHIR](https://github.com/sdhealthconnect/leap-fhir-ces)
- [An embedded FHIR CES](https://github.com/sdhealthconnect/leap-hapi-fhir-ces-embedded) for the [HAPI FHIR Server](https://hapifhir.io/).


### Decentralized Architecture

As shown in Figure 1, the LEAP Consent architecture is inherently decentralized.  

- There is no tight dependency between the LEAP-CDS and Consent Stores; Consent Stores can be controlled by different organizations and as long as they are known to the LEAP-CDS, consent resources stored in them can be retrieved and processed using the standard FHIR API.

- There is no tight dependency between the Consent Management Services and Consent Stores; as long as a Consent Management Service generates standard FHIR resources and communicates via the standard FHIR API, it can connect to any Consent Store.

- There is no tight coupling between the LEAP-CDS and any particular Consent Management Services; as long as the Consent Management Service eventually produces standard FHIR `Consents`, the consent resources originating from any Consent Management Service (potentially owned and operated by different organizations) can be processed by the LEAP-CDS.

### Computable Consent
A _computable consent_ is a representation of patient consent in which privacy preferences are encoded in the form of machine-readable rules. Such rules can be processed by a decision engine (such as the LEAP-CDS) to adjudicate whether the consent permits a specific given activity, such as sharing the patient information with a requester, or enrolling the patient in a research project. 

Since they enable automated decision-making and enforcement based on a patient's privacy preferences, computable consents are an essential part of a scalable consent enforcement architecture.


[high-level]: {{site.baseurl}}/assets/img/architecture.png "Figure 1. LEAP Computable Consent Architecture"

[architecture]: {{site.baseurl}}/assets/img/cds-ces.png "Figure 2. Consent Decision vs. Enforcement"
