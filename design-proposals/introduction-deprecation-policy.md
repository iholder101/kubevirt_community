Disclaimer: this proposal is unusual in the sense that it's not exactly a "design" proposal,
but rather a proposal to change our the way APIs are introduces, deprecated, removed, and
graduated in Kubevirt. Even though it's not a design proposal per-se, this is an important
topic that demands a thorough discussion. Therefore, let's use this format to provide context,
outline the goals, and discuss the subject.

# Overview
Kubernetes is an API-driven system that uses a set of APIs called "API groups". The main idea is that
each API is versioned independently and that API versions fall into 3 main tracks: Alpha, Beta and GA.
This approach has many advantages like being fine-grained in the sense that every API can be versioned
independently, so a user can choose which APIs to use in production.

For more info on Kubernetes' deprecation policy, please look here: https://kubernetes.io/docs/reference/using-api/deprecation-policy/

Kubevirt technically uses the same approach, but it is not maintained in practice. Many of our APIs
rarely change their versions, features are usually added straight to GAed APIs instead to the next Alpha
version, our `core/v1` API (which is a legacy API group existing in Kubernetes for historical reasons and
aimed to shrink and perhaps eventually disappear in Kubernetes) is huge and keeps growing, etc.

Some work has already been done at this front [1][2][3], but as we dove deeper into the subject
many delicate questions started to rise which demanded a thorough discussion with the entire
community. A PR is not suited for this kind of discussion, hence this proposal was created.

Note: In this document I'm referring to all user-facing changes, e.g. API fields, API objects,
feature gates, field behavior change, and any other user-facing change.

[1] https://github.com/kubevirt/kubevirt/pull/7791

[2] https://github.com/kubevirt/kubevirt/issues/7745

[3] https://github.com/kubevirt/kubevirt/pull/9012

## Motivation
There are many advantages to using API groups properly and many problems that arise from not using them.

### Ability to deprecate
Currently, it's hard to deprecate APIs in Kubevirt. This is mainly caused by the lack of a clear policy
that would outline the deprecation process. This often leads to problematic situations, such as:

- Old APIs stay around for the sake of backward compatibility for a very long time, often longer
than necessary. Since Kubevirt does not have a well-defined deprecation policy, developers always
fear that users would be surprised by an API being removed / feature behaving differently, etc.
This leads to a situation where we have to maintain code that is no longer relevant and that we discourage
its use.
- The same APIs behave differently although the API group did not change. This is unlike how it's like
in Kubernetes, where the behavior changes only if the user changes the API version.
- Every API is being deprecated in a different way, therefore as a user it is very hard to understand
that something is deprecated.

A proper deprecation policy should be consistent and transparent to give users the opportunity to
prepare for the upcoming changes. In order to achieve that, the user has to know the exact time-lines
for when the feature would be completely removed. In addition, users need to be able to change API
versions at their own pace. As an example, if version `v1` of some API group is deprecated and would
be removed within a year, the user can choose to move to version `v2` whenever he feels ready to do so,
even before `v1` was completely removed. For some users this can allow early testing and integration of
the advanced versions that lead to certainty and confidence.

### Confidence in new features
As of today, APIs are often added to GAed API groups / versions without first landing to an alpha version
API. As an example, if a field needs to be added to the VMI object, usually it's being added to `v1` API
version, and not to `vXbetaY`, e.g. `v2beta1`.

With this approach, bugs can be found in a GAed API version. As an example, if the field we've mentioned
reaches `v1` API version and a bug is found in its behavior, it would affect users running Kubevirt
in production, although the admin did not choose to graduate to a newer API.

In Kubernetes, this is unacceptable, as new APIs must first reach an Alpha version, then graduate to Beta,
and only then become part of a GAed version. Not only that every api track (i.e. alpha, beta, ga) provides
different guarantees (for example, alpha API can be dropped without warning), it gives the community and
users time to influence how the new feature is going to be developed. For this reason Alpha implementations
in Kubernetes tend to be vague with a lot of open ends, since it's very hard to know how a feature would
be used, what the users/community members would like and how it will affect the solution as a whole.

For these reasons, in Kubevirt, we need to introduce new features as Alpha, and only then graduate them to
Beta and then GA. In addition, the policy needs to determine what are the requirements in order to graduate
between the tracks. For example, in Kubernetes, a KEP (Kubernetes Enhancement Proposal) needs to be merged
with a list of requirements in order to graduate the API. In addition, Kubernetes' policy requires a certain
amount of time to pass before graduating to a newer version. In Kubevirt we don't have to exactly follow
Kubernetes' time-lines and requirements, but it's a very good reference that can serve a base ground
for discussions.

### Fine-grained API groups
In Kubernetes, the `core` API group is treated as a legacy group that exists for historical reasons only.
There is an ongoing effort to remove APIs from it to different, finer-grained groups. In Kubevirt,
the `core` API group is huge, messy and contains many different and unrelated APIs.

But the situation is even worse. For years now, our one and only relevant version of the `core` API
group is `v1`. Every feature goes straight to `core/v1`, and features are removed from it as well.

This means that users cannot graduate API groups in a fine-grained manner, since a huge amount of the
APIs are under `core/v1`. Ideally, the user could choose exactly which set of APIs he wants to use. But
even the huge `core` API group itself cannot be updated, since we're stuck on `v1` for a very long time.

In Kubevirt, just like in Kubernetes, we should start the process of separating API groups from `core`.
In addition, we need to introduce new APIs into `v2alpha1` to start building the next API version,
hopefully in a much better way than now.

## Goals
* Formulate a clear and transparent deprecation / introduction policy with guaranteed time-lines that
would define exactly when an API will be removed / added and to which API track.
* Using the new policy, remove old APIs that are already deprecated for a very long time.
* Provide the users the ability to graduate APIs in a fine-grained manner.
* Provide confidence in new features by slowly graduating them to newer versions.

## Non Goals
* Discuss specific APIs that need to be graduated, deprecated, or separated to a different group.

## User Stories
* As a user, I want to have a consistent and clear way to know exactly which APIs are deprecated.
* As a user, I want to know when a deprecated feature would be removed.
* As a user, I expect the API groups and versions that I'm currently running to not change behavior
unless I choose to graduate them to a newer version.
* As a user / community member, I want to have the time to influence how new features are developed.
I wish to be able to test them and play with them (while they're Alpha) before the are GAed.
* As a user, I want to be able to update APIs to new versions in a fine-grained manner.
* As a developer, I want to remove old APIs that aren't being used / discouraged, etc and therefore
maintain less code and be able to develop faster.

## Repos
kubevirt/kubevirt

# Introduction / Graduation Policy
TBD

# Deprecation Policy
TBD

# Discussion phases
- [ ] General agreement on goals on motivation
- [ ] Introduction / Graduation Policy: Alpha requirements & guarantees
- [ ] Introduction / Graduation Policy: Beta requirements & guarantees
- [ ] Introduction / Graduation Policy: GA requirements & guarantees
- [ ] Deprecation Policy: GA deprecation
- [ ] Deprecation Policy: Beta deprecation
- [ ] Deprecation Policy: Alpha deprecation

This list would likely be edited and reshaped according to the discussions.

# Summary
TBD.

As a reference and a base ground, let me provide the following table that is kept empty for now:

| API Entity                        | Operation    | Version | Guarantees | Requirements / Time-Line |
|-----------------------------------|--------------|---------|------------|--------------------------|
| API Object                        | Introduction | Alpha   |            |                          |
| API Object                        | Graduation   | Beta    |            |                          |
| API Object                        | Graduation   | GA      |            |                          |
| API Object                        | Deprecation  | GA      |            |                          |
| API Object                        | Deprecation  | Beta    |            |                          |
| API Object                        | Deprecation  | Alpha   |            |                          |
| API Field                         | Introduction | Alpha   |            |                          |
| API Field                         | Graduation   | Beta    |            |                          |
| API Field                         | Graduation   | GA      |            |                          |
| API Field                         | Deprecation  | GA      |            |                          |
| API Field                         | Deprecation  | Beta    |            |                          |
| API Field                         | Deprecation  | Alpha   |            |                          |
| Feature Gate                      | Introduction | Alpha   |            |                          |
| Feature Gate                      | Graduation   | Beta    |            |                          |
| Feature Gate                      | Graduation   | GA      |            |                          |
| Feature Gate                      | Deprecation  | GA      |            |                          |
| Feature Gate                      | Deprecation  | Beta    |            |                          |
| Feature Gate                      | Deprecation  | Alpha   |            |                          |
| Feature Behavior                  | Introduction | Alpha   |            |                          |
| Feature Behavior                  | Graduation   | Beta    |            |                          |
| Feature Behavior                  | Graduation   | GA      |            |                          |
| Feature Behavior                  | Deprecation  | GA      |            |                          |
| Feature Behavior                  | Deprecation  | Beta    |            |                          |
| Feature Behavior                  | Deprecation  | Alpha   |            |                          |
| Metadata (labels/annotations/etc) | Introduction | Alpha   |            |                          |
| Metadata (labels/annotations/etc) | Graduation   | Beta    |            |                          |
| Metadata (labels/annotations/etc) | Graduation   | GA      |            |                          |
| Metadata (labels/annotations/etc) | Deprecation  | GA      |            |                          |
| Metadata (labels/annotations/etc) | Deprecation  | Beta    |            |                          |
| Metadata (labels/annotations/etc) | Deprecation  | Alpha   |            |                          |


# FAQ
TBH
