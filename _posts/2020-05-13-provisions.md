---
layout: post
title: Consent Provisions
author: Mohammad Jafari
date: 2020-05-13 11:11:11 -0000
categories: [blog]
tags: [provisions, cds, change-request]
---

One of the most significant parts of the [Consent resource](https://www.hl7.org/fhir/consent.html) is to capture and record the preferences of the patient in the form of rules that specify under what conditions access must be denied or granted to the patient data. While the Consent resource currently supports referencing an external artifact that encodes such rules (using the `policy` attribute), the native mechanism for expressing and encoding rules is the use of `provision` attribute. Provisions capture the core logic of the rules that apply to the use of the patient's data, and therefore, defining clear and unambiguous semantics for them is crucial in correctly capturing the patient's preferences and enforcing them as intended. In other words, by ensuring that: 

- patient preferences are correctly captured in the form of provisions, and
- consent implementations correctly understand and interpret the provisions at the time of decision-making and enforcement.


### Current Model
Figure 1 shows an overview of the current provision model in the Consent resource:

![current-model][current-model]
*Figure 1. Current Model for Provisions in the Consent Resource*

- A base provision records the default `permit` or `deny` as well as overall applicable conditions such as validity period.
- The base provision may have a set of nested provisions each of which may in turn has its own nested provisions.
- The nested provisions state exceptions to the containing provisions. In other words, a match with a nested provision at one level, flips the higher-level decision (from `permit` or `deny` or vice versa). 
- Provisions at the same level are interpreted disjunctively (i.e., as OR), so matching _any_ nested provision constitutes an exception. 

Figure 2 shows a (hypothetical) example for this model. These provisions articulate that during the time period from 01/01/2020 to 31/12/2022, all access is permitted except:

- Any access for the purpose of *marketing*.
- Any access by *app1* to any restricted content.
- Any access by *Org1* for the purpose of *Payment*, unless:
    + The requested content is *Claims*, *Claim Responses*, or *Accounts*

![current-model-example1][current-model-example1]
*Figure 2. An example set of rules modelled using the current provision model*

### Problems with the Current Model
Aside from the ambiguities and inconsistencies in the current specifications and the accompanying examples, the experience of implementing the current provision model revealed a number of serious problems with the current design.

#### Complexity
The current provision structure is inherently complex: it is a tree with (theoretically) unlimited height and branching factor which can lead to exponential complexity as the number of provisions and the paths in the tree grows. Note that this complexity does not lead to more expressiveness, compared to a simpler, flat structure (proposed further below).

Because of this complexity, the nuances and side-effects of the rules and their arrangement in the tree is not very straightforward and there could be a lot of pitfalls. This makes the structure error-prone, both at the  time when human users translate rules from a paper form (or their mind) into a provision tree, as well as when a machine reads and interprets it. 

For example, consider the structure in Figure 3; it looks very similar to the example policy in Figure 2 but it has subtle differences. For example, the former would allow `Org1` to access anything for the purpose of `HLEGAL`, but the latter does not. This nuanced difference lies in the layering of the tree in a way that is not obvious and intuitive and can be pitfall in policy authoring.

![current-model-example2][current-model-example2]
*Figure 3. Another provision structure, similar to that of Figure 2 but with important differences*

#### Conflict
As will be discussed below, while the current structure does not prevent conflicts, it does not provide an explicit mechanism for conflict resolution either. Since the matching takes place by traversing down the tree structure, it is not impossible for more than one path to match a request. For example, consider the structure of Figure 4. If `Org1` requests access to `Claims` for the purpose of marketing (`HMK`), there will be a conflict. One branch denies any access for the purpose of marketing while another branch permits access for marketing purposes to `Claims` by `Org1`. It is not clear how such conflicts should be resolved and the current model provides no mechanism to specify this. Some possible resolutions could be:

- Assuming that the more general rule (shallower in the tree) overrides.
- Assuming that the more specific rule (deeper in the tree) overrides.
- Assuming that the order determines the precedence of the rules and the first matching provision prevails.

These are all poor choices since they are left to implications, and therefore, increase the risk of errors, unintended consequences, and poor interoperability because of the risk of different assumptions by interacting parties. Even if one of these alternatives is codified in the specifications, errors, both in modelling rules and in implementing the rule processor software are likely.

![current-model-example3][current-model-example3]
*Figure 4. A provision structure that could lead to conflicting decisions*


#### Superfluous Decision in Nested Provisions
The `type` attribute that enables each nested provision to specify a `deny`/`permit` decision currently seems superfluous; if provisions at each nested level are an exception to the previous level, the decision is implicitly the opposite of the decision of the previous level (as shown in the examples  above). So it is not clear how the value of this attribute should be interpreted in nested provisions.


### Alternative Model
Because of these issues, we propose an alternative model of provisions based on a much simpler structure with a more explicit mechanism for conflict resolution. Figure 5 shows an example.

The provisions are organized in a single flat array with no nesting and a new attribute `onConflict` records the expected behaviour if the decision from some provisions are inconsistent with others. A basic value set for this attribute can be: 

- `denyOverrides`, indicating that when in conflict, a decision to deny  must prevail,
- `permitOverrides`, indicating that when in conflict, a decision to permit must prevail, 
- `firstMatchOverrides`, indicating that the first decision, based on the order of provisions in the array, must prevail, and
- `invalid`, indicating that the author's intention has been to avoid conflicts, so, if a conflict occurs the Consent resource must be considered invalid. 


![proposed-model-example][proposed-model-example]
*Figure 5. An example for the proposed provision structure*

This structure is much easier to understand by human users and policy authors, leads to a less complicated evaluation algorithm, and is consistent with the best practices of existing policy languages, such as [XACML](http://docs.oasis-open.org/xacml/3.0/errata01/os/xacml-3.0-core-spec-errata01-os-complete.html#_Toc489959480) and [ODRL](https://www.w3.org/TR/odrl-model/#conflict).   






[current-model]: {{site.baseurl}}/assets/img/provisions-current-model.png "Figure 1. Current Model for Provisions in the Consent Resource"

[current-model-example1]: {{site.baseurl}}/assets/img/provisions-current-model-example1.png "An example set of rules modelled using the current provision model"

[current-model-example2]: {{site.baseurl}}/assets/img/provisions-current-model-example2.png "Figure 3. Another provision structure, similar to that of Figure 2 but with important differences"

[current-model-example3]: {{site.baseurl}}/assets/img/provisions-current-model-example3.png "Figure 4. A provision structure that could lead to conflicting decisions"

[proposed-model-example]: {{site.baseurl}}/assets/img/provisions-proposed-model-example.png "An example for the proposed provision structure"