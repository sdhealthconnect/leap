---
layout: post
title: Future Directions for the LEAP Consent Project
author: Mohammad Jafari, Duane DeCouteau 
date: 2021-10-04 11:11:11 -0000
categories: [blog]
tags: [architecture, computable-consent, leap-cds]
---

The ONC LEAP Consent project ended on September, 30th 2021 after completing all of the goals set by the project plan. The project proved how _computable consents_, based on the FHIR `Consent` resource, can be used to capture, manage, and enforce patients' privacy preferences in a wide range of use cases, including exchange of patient information between providers, research, treatment, and advance healthcare directives, as well as across different technologies including HL7v2.0 messaging, eHealth Exchange, Direct Exchange, and FHIR. 

There is plenty of work to do in establishing a scalable consent management architecture and this post briefly discusses some of the future work that can directly follow from the LEAP Consent project and build upon this work. These future directions are organized in four broad categories: additional technology by further software development, new application areas where the architecture can be further developed and tested, further work in standardization to more formally record and communicate the findings of the project, and further testing and proof of the architecture using pilots and integration with production systems.

### Technology

#### OAuth Integration
By integrating the LEAP Consent Decision Service (LEAP-CDS) with an OAuth 2.0 server, the consent decision can be incorporated into the decision process for issuing of an OAuth token. In other words, patient consent can act as the policy based on which the OAuth server determines whether or not to issue a token for a client. Additional restrictions that are communicated in the form of _obligations_ by the LEAP-CDS can be translated into the OAuth _scopes_ granted to the client. 

Developing this integration can enable clients, especially web and mobile applications, to take advantage of existing OAuth 2.0 integrations for consent decision enforcement. It can also enable recording OAuth 2.0 consents (that often take place when a patient authorizes an app to access their information) in the form of FHIR `Consent` resources to further consolidate consent management and enforcement. Integrating this flow with [SMART App Launch Framework](http://hl7.org/fhir/smart-app-launch/index.html) and [SMART Backend Services](https://hl7.org/fhir/uv/bulkdata/authorization/index.html) can further integrate the Consent Decision Service with existing FHIR application ecosystems.

#### Questionnaire Directory and Consent Generation API
Since it is purely based on FHIR, the LEAP Consent Management models consent forms as FHIR `Questionnaire` resources and captures the raw patient responses to these forms as a FHIR `QuestionnaireResponse` resources. Based on the semantics of the form, it then generates a computable FHIR `Consent` resource based on the `QuestionnaireResponse`. The logic for this transformation is highly dependent on the type of form (i.e., the `Questionnaire` resource) and is currently embedded within the source code of of the LEAP Consent Management web application.

A future step in making this architecture more flexible and more widely available is to develop and provide: 
 - A directory of standard consent forms from different states and jurisdictions in the form of FHIR `Questionnaire` resources, 
 - A Computable Consent Generation API for generating a computable `Consent` resource from a `QuestionnaireResponse` resource.

 This can make it significantly easier for application developers to rapidly integrate consent management functions into their app by retrieving the right consent form from the directory, rendering it graphically for the patient, and then, submitting the patient's response to the Computable Consent Generation API to have the response turned into a computable and automatically enforceable consent without having to develop the code for consent generation.

 A further step in this direction can enable the Consent Management Service to provide a graphical representation of the consent form, directly renderable within an application (e.g., via a modal window or an embedded form similar to the ones provided by payment gateways for entering credit card information), capturing the consent, and directly storing the resulting computable `Consent` in the FHIR Consent Store. This can minimize the effort for capturing patient consents to a few lines of codes for any web or mobile application.

#### Consent Management API
FHIR provides a basic standard API for managing consents by defining how consents can be created, updated, deleted, and retrieved. The standard FHIR API, however, is designed generically based on a REST paradigm without Consent Management specifically in mind, therefore, some of the basic consent management functions depend on complex FHIR queries or require multiple requests.

Developing a standard set of FHIR custom [`Operations`](https://www.hl7.org/fhir/operations.html) specifically for Consent Management functions can significantly simplify developing Consent Management applications and integrations. Some examples are: specific custom operations to revoke or re-enact a Consent, search for consents based on a patient identifier, search for all `AuditEvent` resources tied to a particular consent, FHIR `Subscription` for notifying clients about Consent updates, etc.  
Most of these functions can be defined as short-hands and aliases for existing FHIR API, but some more advanced operations may depend on running multiple inter-dependent FHIR API calls or more complex queries and functionality. Providing a standard implementation and cataloguing these functions as the Standard FHIR Consent Management API is a future work that can significantly simplify implementing consent management solutions.

#### Advanced Security Labeling System
All of the LEAP Consent Enforcement Services rely on a [simple Security Labeling Service (SLS)](https://github.com/sdhealthconnect/leap-sls) that scans the patient information (in various formats, including HL7v2.0, CCDA, and FHIR) and assigns [sensitivity](https://terminology.hl7.org/ValueSet-v3-InformationSensitivityPolicy.html) and [confidentiality labels](https://terminology.hl7.org/ValueSet-v3-Confidentiality.html), such as _substance abuse information sensitivity_ (`ETH`) or _restricted_ (`R`) according to the [FHIR Security Labels Module](https://www.hl7.org/fhir/security-labels.html) and [FHIR Data Segmentation for Privacy Implementation Guide](https://build.fhir.org/ig/HL7/fhir-security-label-ds4p/index.html).

The LEAP-SLS can be expanded to become a pure FHIR-based service by supporting more advanced security labeling rules that are stored directly as FHIR resources. This enables more sophisticated and interoperable labeling rules that can support more complex policies and can be managed and shared using the FHIR API. 

### More Applications
While the project was initially focused on supporting the generic use cases of exchange, research, treatment, and advance healthcare directives, more specific application areas have their own details and nuances that require further examination and fine-tuning of the LEAP consent architecture. Behavioral health (BH) and social-determinants of health (SDH) application areas are two major examples that emerged in the course of the project where there is already a pressing need for automated consent management and enforcement.

Focusing on specific application areas and examining the nuances and complexities of these use cases will improve the LEAP Consent architecture in covering the specifics of each application. It will also bring valuable feedback to further enhance the FHIR `Consent` resource and its maturity. 

### Standardization
The LEAP Consent project resulted in valuable findings and best practices including details of using the FHIR `Consent` resource for implementing computable consents, using the FHIR `AuditEvent` and `Provenance` resources in consent management and enforcement use cases, and using the [CDS Hooks API](https://cds-hooks.org) for requesting and communicating consent decisions.Some of these findings have been reported back to the HL7 Community-Based Care and Privacy (CBCP) Working Group as change requests to the core specifications of the FHIR `Consent` resource. 

Although the documentation, code artifacts, and example FHIR resource capture the findings of the project, the more formal way to record and communicate these findings is by developing a formal FHIR Implementation Guide and balloting it through the HL7 process. This enables the results of the project (including the future findings resulting from the work on the future directions listed in this post) to be more widely and more formally communicated, and more thoroughly examined by the community.

### Testing
The LEAP consent architecture was tested extensively with synthetic test data, so an  important natural next step is to engage in pilots and conduct testing with production data in real use cases. This will enable identifying and implementing additional improvements to the architecture including more practical issues such as load testing and scaling, patient identity matching, integration with legacy systems, etc.
