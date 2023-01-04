# Rough Opal Grant Report

*This document will give a comprehensive view into the state of the grant work its duration comes to a close.*

---

## Background

[Rough-Opal](https://github.com/tabcat/rough-opal) is a grant that was accepted in October 2022. The duration of the grant spanned from September to December 2022.

### Deliverables and Development Roadmap

The original post including the grant deliverables exists in this [issue](https://github.com/filecoin-project/devgrants/issues/1031).
That post has been editted once, the same day it was posted, and has not been editted since.

Below the Deliverables and Development Roadmap from that post have been turned into a Table of Contents.
This document ends with a [Closing](#closing) section which is included in the ToC.

The links for each item snap to a sections which references prs/issues/commits that relate to them, with a written section if necessary.
Items which do not have a link have been included in the parent or child or are not significant.
Items which have been crossed out have not be completed and will include a written section for explanation.
All other items have been completed or will be completed (see [Still Ongoing](#still-ongoing) for incomplete but still planned items).

### Changes Since Then

At the start of the grant, the protocol and implementation shared the name `opal`.
The grant proposal and the implementation's repo mentioned possible name changes.
Names have been changed for the protocol and implementation to `opalsnt` and `welo`, respectively.
The project also now has it's own Github org located at https://[github.com/opalsnt](github.com/opalsnt).

Some planned work has also been canceled for reasons that will be explained further below. The two largest changes have to do with the `Zzzync replicator` and `Heavy Testing` which involved using Testground and other tools.

## Table of Contents

- [Deliverables](#deliverables)
  - [`Opal` Base Feature Set](#opal-base-feature-set)
    - [Locally persisted Database replicas](#locally-persisted-database-replicas)
    - [~~Easy Custom Database States~~](#easy-custom-database-states)
    - [Pubsub Heads Exchange Replicator](#pubsub-heads-exchange-replicator)
    - [~~Zzzync Replicator integration~~](#zzzync-replicator-integration)
    - [~~IPLD Schema Validation  (if ready in javascript)~~](#ipld-schema-validation-if-ready-in-javascript)
    - [Automated Release: ci, change log, and api docs](#automated-release-ci-change-log-and-api-docs)
  - [`Zzzync` Development Plans]()
    - begin design of replicator
    - ~~choose a design to start building~~
    - ~~set up linting/formatting~~
    - ~~configure typescript~~
    - ~~write implementation of design~~
    - ~~write unit tests~~
    - ~~test interop with Opal~~
    - ~~write benchmarks~~
    - ~~automate release with generated api docs and changelog~~
    - ~~release alpha with expected public api changes~~
    - ~~testing with Opal in testground network simulation~~
    - ~~stress-test and benchmark~~
    - ~~check for bugs and perf improvements~~
    - ~~release beta with public api locked~~
  - [`Opal-Spec` 1.0-beta]()
    - Release Draft Spec for Opal and main modules
    - Release Draft Spec for Live Replicator
    - ~~Release Draft Spec for Zzzync~~
    - Release completed Specifications for Opal ~~and Zzzync~~ 1.0-betas
- [Development Roadmap](#development-roadmap)
  - [Opal Repo Init](#opal-repo-init)
    - begin opal design and spec
    - configure project repo
    - [write unit tests](#write-unit-tests)
    - [prep for adding features](#prep-for-adding-features)
      - [build interfaces for manifest modules](#build-interfaces-for-manifest-modules)
      - [rewrite manifest module registry](#rewrite-manifest-module-registry)
      - [rework store module](#rework-store-module)
      - [rework classes to use Libp2p's startable interface](#rework-classes-to-use-libp2ps-startable-interface)
    - [make databases locally persistent](#make-databases-locally-persistent)
  - [Opal Replication and Perf](#opal-replication-and-perf)
    - begin design and spec of live replicator
    - add live replicator (Libp2p pubsub + IPFS)
    - test replication and replicated states
    - write benchmarks
    - [automate release with generated API docs and changelog](#automate-release-with-generated-API-docs-and-changelog)
    - release draft spec for Opal and main modules
    - release alpha with expected public API changes
  - ~~Zzzync Replicator~~
    - begin design and spec of persistent replicator
    - choose a design to build
    - configure project repo
    - write implementation (likely using web3.storage and w3name)
    - write unit tests
    - test interop with Opal
    - write benchmarks
    - automate release with generated API docs and changelog
    - release draft spec
    - release alpha with expected public API changes
  - Release
    - ~~Heavy Testing~~
      - ~~network simulated testing with testground~~
      - ~~stress-test and benchmark replicators~~
      - ~~check for replication bugs and perf improvements~~
    - Usage References
      - Opal and ~~Zzzync automated~~ (and nice-looking) API docs
      - Write base FAQ.md document for common user questions
      - Basic Tutorial document added to repo or blogged
      - NodeJS ~~and Create React App~~ examples
    - [Release Opal and Zzzync 1.0-beta](#release-opal)
      - completed protocol specification
      - [typescript implementation (with public API locked until 1.0)](typescript-implementation)
- [Closing](#closing)
  - Accomplishments
  - Unplanned Work
  - Next Steps
  - Remarks

<br/>

---

<br/>

## Deliverables

### Opal Base Feature Set

Issue: https://github.com/cypsela/opal/issues/8

#### Locally persisted Database replicas

Not just the database replicas are persisted but also the store indexes. This is so databases can be read (index), and updated (replica) immediately upon being opened.

- https://github.com/opalsnt/welo/issues/12
- https://github.com/opalsnt/welo/commit/a9958dae8d105ca39f666487ee99bcb7cfee621f
- https://github.com/opalsnt/welo/commit/8bcbd35b8a935edffa13a9e540bd8554d3015255
- https://github.com/opalsnt/welo/commit/bcc880d197990688491b0bb6ecbb5ec4ffe135f8

#### ~~Easy Custom Database States~~

While this is crossed out it is still possible to create custom databases easily.
It can still be made much, much easier so leaving this as unfinished.

Here is the [key-value store model](https://github.com/opalsnt/welo/blob/master/src/store/keyvalue/model.ts) which shows what will need to be supplied once a better way for supplying one is provided.
It includes a reducer, an init function (which supplies the initial value for the state), action creators, and selectors which are all similar to how [Redux](https://redux.js.org/) is used (or was used).

This ended up left out because there were bigger fish to fry but a good solution will be found.

#### Pubsub Heads Exchange Replicator

This replicator works very closely to how the OrbitDB replicator works.
Welo nodes join a shared pubsub channel unique per database.
When nodes see peers join the shared channel, nodes attempt to join a direct channel where updates for databases are advertised.

Unlike OrbitDB the replication has been specified and the replication tests are [not flaky](https://github.com/opalsnt/welo/commit/3f60808cde11f90f4789a298426e6fdcf9603f98).

- https://github.com/opalsnt/welo/issues/17
- https://github.com/opalsnt/welo/issues/20

#### ~~Zzzync Replicator integration~~

Unfortunately a Zzzync replicator was not added.
This will take more time than expected to do right. In this case *to do right* means the result is: relatively efficient/fast and general purpose (can be useful apart from web3storage).

This was identified as the most difficult part in the grant proposal.

It's still possible to have a naive Zzzync replicator for experimental usage fairly quickly.
However, I didn't see this as worth it and am looking at designing system for use longterm.

During the grant I did [chat about this with Irakli](https://github.com/web3-storage/w3protocol/issues/154#issuecomment-1311854803) from web3storage who always has some good information.

#### ~~IPLD Schema Validation  (if ready in javascript)~~

Have not looked at this much since the grant began.
Plan on hardening the implementation in the nearer future and this will likely be part of that.

#### Automated Release: ci, change log, and api docs

The release used is similar to js-libp2p's pipeline.
You can see from the commit message I tried release please first (wasn't happy with how it bumped the version of the package).
Release-please is pretty much exactly what I wanted.
It creates a changlog inside a PR; when the PR is merged it makes a new Github release and publishes to npm.
Changelog is generated and versions are bumped by looking at the Conventional Commits in the commit messages.

The [API docs](https://github.com/opalsnt/welo/blob/master/API/welo.md) generation has been automated.
They look nice but can still be improved in the future.
JSDoc is parsed by the tool to add useful information and the Typescript types are used to show parameters.

- https://github.com/opalsnt/welo/commit/97a7f072ee51a8be1b446c2c6b70c822ae8c4b87
- https://github.com/opalsnt/welo/issues/34
- https://github.com/opalsnt/welo/commit/b8c9b2a707a4825044c3f8fdd5757fdb05a5d7a6 

### Zzzync Development Plans

See [~~Zzzync Replicator integration~~](#zzzync-replicator-integration)

### Opal-Spec 1.0-beta

#### Release Draft Spec for Opal and main modules

- https://github.com/opalsnt/specs/issues/2

#### Release Draft Spec for Live Replicator

- https://github.com/opalsnt/specs/issues/7

#### ~~Release Draft Spec for Zzzync~~

See [~~Zzzync Replicator integration~~](#zzzync-replicator-integration)

#### Release completed Specifications for Opal ~~and Zzzync~~ 1.0-betas

- https://github.com/opalsnt/specs/issues/1

## Development Roadmap

### Opal Repo Init

#### write unit tests

95% coverage according to [codecov.io](https://app.codecov.io/gh/opalsnt/welo)

There are a few commits related to testing so I won't link them all.
Feel free to check out the [tests](https://github.com/opalsnt/welo/tree/master/test). Plan on organizing them a bit more in the future.

#### prep for adding features

- https://github.com/opalsnt/welo/issues/10
- https://github.com/opalsnt/welo/issues/11

This work mainly had to do with cleaning up code before adding new stuff.

##### build interfaces for manifest modules

Created Typescript interfaces for the different manifest modules.
Probably did this a bit earlier than necessary.

##### rewrite manifest module registry

Fixed the module registry where modules referenced by the manifest are kept.

- https://github.com/opalsnt/welo/commit/fac797a8b36e51ee5eb4fd0a1f4fce5fd105339c

##### rework store module


- https://github.com/opalsnt/welo/commit/567646d872a09695544160b4666e566149f6b72
- https://github.com/opalsnt/welo/commit/e92a7410ec37c37c346c82ed0294b2c71db5241

##### rework classes to use Libp2p's startable interface

Libp2p's startable interface is a Typescript interface for managing the lifecycle of objects.
To make things a bit easier when working with async start/stop I made [Playable](https://github.com/opalsnt/welo/blob/master/src/utils/playable.ts) which classes inside the project use.

- https://github.com/opalsnt/welo/commit/c9ec979387ae51c0ab14917f85212f0f98a585b
- https://github.com/opalsnt/welo/commit/1e88f90b4dcd56d749cab9e9a49895b73e5a168
- https://github.com/opalsnt/welo/commit/397009d5f604e2d3fd02e769982b5ae7b8e10c1
- https://github.com/opalsnt/welo/commit/7dd3ff883fce2ffd9b937ed062f39ead6f54eb1
- 65d549aca0967331aac2952172b73b63f5861801

#### make databases locally persistent

- https://github.com/opalsnt/welo/issues/12
- https://github.com/opalsnt/welo/commit/a9958dae8d105ca39f666487ee99bcb7cfee621f
- https://github.com/opalsnt/welo/commit/8bcbd35b8a935edffa13a9e540bd8554d3015255
- https://github.com/opalsnt/welo/commit/bcc880d197990688491b0bb6ecbb5ec4ffe135f8

### Opal Replication and Perf

#### add live replicator (Libp2p pubsub + IPFS)

- https://github.com/opalsnt/welo/issues/20
- https://github.com/opalsnt/welo/issues/21
- https://github.com/opalsnt/welo/issues/22
- https://github.com/opalsnt/welo/issues/23
- https://github.com/opalsnt/welo/issues/24
- https://github.com/opalsnt/welo/issues/25
- https://github.com/opalsnt/welo/issues/26
- https://github.com/opalsnt/welo/issues/27
- https://github.com/opalsnt/welo/issues/31

#### test replication and replicated states

- https://github.com/opalsnt/welo/issues/27

#### write benchmarks

This is not complete yet. It is part of [Still Ongoing](#still-ongoing).
Benchmarks will be added for read/write/replication and metrics will be tracked overtime with [github-action-benchmark](https://github.com/benchmark-action/github-action-benchmark).

- https://github.com/opalsnt/welo/issues/37

#### automate release with generated API docs and changelog

- https://github.com/opalsnt/welo/commit/b8c9b2a707a4825044c3f8fdd5757fdb05a5d7a6 
- https://github.com/opalsnt/welo/commit/97a7f072ee51a8be1b446c2c6b70c822ae8c4b87

See [Automated Release: ci, change log, and api docs](#automated-release-ci-change-log-and-api-docs) for more detail

#### release draft spec for Opal and main modules

- https://github.com/opalsnt/specs/issues/2

#### ~~release alpha with expected public API changes~~

The live replicator was not done on time so an alpha was never released.

### Zzzync Replicator

See [~~Zzzync Replicator integration~~](#zzzync-replicator-integration)

### Release

#### ~~Heavy Testing~~

Will be looking at doing the heavier testing with tools like Testground around the same time as performance and hardening. I did not quite reach this step during the grant.

#### Usage References

##### Opal ~~and Zzzync~~ automated (and nice-looking) API docs

Decided to use [api-documenter](https://api-extractor.com/pages/setup/generating_docs/) instead of [Typedoc](https://typedoc.org) because it had much nicer looking generated markdown.

- https://github.com/opalsnt/welo/issues/34
- https://github.com/opalsnt/welo/commit/b8c9b2a707a4825044c3f8fdd5757fdb05a5d7a6 

See [Automated Release: ci, change log, and api docs](#automated-release-ci-change-log-and-api-docs) for more detail

##### Write base FAQ.md document for common user questions

https://github.com/opalsnt/welo/issues/39

##### Basic Tutorial document added to repo or blogged

https://github.com/opalsnt/welo/issues/40

##### NodeJS ~~and Create React App~~ examples

Welo has not been tested with browsers yet.
This can be tracked in [browser tests issue](https://github.com/opalsnt/welo/issues/42).

- https://github.com/opalsnt/welo/issues/41

#### Release Opal ~~and Zzzync~~ 1.0-beta

The 1.0.0 release was created prematurely while trying out different release pipelines.

- https://github.com/opalsnt/welo/releases/tag/v1.0.0

Latest release is now [v1.0.2](https://github.com/opalsnt/welo/releases/tag/v1.0.2)

##### completed protocol specification

- https://github.com/opalsnt/specs/issues/10

##### typescript implementation (with public API locked until 1.0)

The original plan was to turn the project into typescript toward the end of the grant.
However Steve, my partner on [Cypsela](https://github.com/cypsela), recommended I use Typescript immediately.

*Thanks Steve*

Had not tried Typescript much before. It helped a bunch and likely sped up development pretty considerably.

- https://github.com/opalsnt/welo/issues/3
- https://github.com/opalsnt/welo/commit/d548bf51f31771a7c6f1837d54dd5fbcad21502d

## Closing

### Accomplishments

Welo, pronouced "way-low", is now a working peer-to-peer database written in Typescript.
It's very similar to OrbitDB but does not need to load all entries into memory before opening a database
(which can take multiple seconds; IIRC 2 seconds for every 1000 entries).
Because Welo writes the replica and store index to disk, databases don't need to be loaded into memory;
exposing read/write immediately regardless of database size.

Publishing, testing, API docs, even checking and updating dependencies have been automated.
Similar CI/CD actions to the js-libp2p repo are used to test and create releases on push to the master branch. 

Created [`copy-deps`](https://github.com/tabcat/copy-deps) which is a package that automates dependency updates when they are shared with another dependency.
Now comes in handy when upgrading `ipfs-core`.

A complete spec is published for the Opalescent database.
When any new replicator or extension to the spec is added it can be updated to include specs for them.

### Still Ongoing

*This section will be updated as items are completed*

- [ ] https://github.com/opalsnt/welo/issues/37
- [ ] https://github.com/opalsnt/specs/issues/1
- [ ] https://github.com/opalsnt/welo/issues/38

### Next Steps

*This section outlines ideas to work on following this grant*

After completing what is included in the section above,
I plan to look at designing an IO-efficient replica structure in IPLD.
Intuitively this also seems relevant to Zzzync replicators.

Performance, hardening and testing the implementation will be the main focuses afterward, in that order.
I'd also like to learn Typescript a bit better.
I know general use right now but more advanced knowledge will enable me to right even cleaner code that requires less testing.

The protocol is in a good state for now,
but look forward to adding a low latency replication protocol after Zzzync.

Will explore these path options more in a follow up grant proposal.

### Reflection and Remarks

This was my first solo grant project.
The admins managing the grant procedure and communication were great.
It's always been a pleasure when interacting with anyone from the Protocol Labs umbrella.

I can definitely improve in some areas. Specifically tracking progress and creating deliverables/roadmaps/timelines; which will hopefully get easier to judge.

Would like to see how transparent/auditable I can make grant work progress in the future by linking directly to issues/prs.
