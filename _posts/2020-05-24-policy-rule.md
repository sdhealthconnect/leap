---
layout: post
title: policyRule values
author: Mohammad Jafari
date: 2020-05-24 11:11:11 -0000
categories: [blog]
tags: [cds, change-request]
---

Based on the current specifications, `policyRule` is a required attribute unless the consent points to an external representation of the policy (using the `policy` attribute). In other words, if a system chooses to use native FHIR encoding of consent rules it must specify a value for this attribute. This attribute sets the base rule for the consent and is essentially the *default* decision for any situation not explicitly addressed as an exception using the provision structure.  

Currently, the value for this attribute is bound to [an extensible value set](https://www.hl7.org/fhir/valueset-consent-policy.html) which contains a number of pre-defined policies. For efficient processing of the consent rules, any consent processor software must be able to determine the default rule of the consent in the form of either a *deny* (opt-in) or a *permit* (opt-out), so that if a request context does not match any of the exceptions specified by the provisions, this default decision is followed. If this default decision is not explicitly encoded in the consent resource itself, the consent consumer must either query an outside service or maintain a mapping internally in order to determine which policies from the value set default to opt-in and which ones default to opt-out. Besides the complexity, this can also lead to inconsistencies in interpretation, as well as inability to process the consent if a particular policy value is not known to the consent processor. 

Thus, we propose that the specs require one of the following values to be present alongside the policy code in the `policyRule.coding` array to ensure that the consent processor can determine the default rule of the consent without ambiguity:

```
{
  "system": "http://terminology.hl7.org/CodeSystem/v3-ActCode",
  "code": "OPTIN"
}
```
```
{
  "system": "http://terminology.hl7.org/CodeSystem/v3-ActCode",
  "code": "OPTOUT"
}
```

Having the additional code as part of the array is consistent with the intentions of the `CodeableConcept` data type which permits having more than one code in case the consumer does not recognize some of them. 

Moreover, this seems to be consistent with the previous versions of the consent resource since some of the example consents currently in the specs use the above codes for `policyRule`.