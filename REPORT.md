# Rough Opal Grant Report

*This document will give a comprehensive view of the state of the grant work as its duration comes to a close.*

---

## Background

[Rough-Opal](https://github.com/tabcat/rough-opal) is a grant accepted in October 2022. The duration spanned from September to December 2022.

### Deliverables and Development Roadmap

The [original proposal](https://github.com/filecoin-project/devgrants/issues/1031) containing grant deliverables has not since been edited.

Below the Deliverables and Development Roadmap from that post have been turned into a Table of Contents.
This document ends with a [Closing](#closing) included in the ToC.

The links for each item snap to a section that references prs/issues/commits related to them.
Items that are not linked have been included in the parent or child or are insignificant.
Crossed-out items will not be completed as part of this grant and include a written section for explanation.
All other ToC items have been or will be completed (see [Still Ongoing](#still-ongoing) for incomplete but still planned items).

### Changes Since Then

At the start of the grant, the protocol and implementation shared the name `opal`.
The grant proposal and the implementation's repo mentioned possible name changes.
Names have been changed for the protocol and implementation to `hldb` and `welo`, respectively.
The project now has its own Github org located at [github.com/hldb](github.com/hldb).

Two significant changes involve removing `Zzzync replicator` and `Heavy Testing` from this grant's roadmap.

## Table of Contents

- [Deliverables](#deliverables)
  - [`Opal` Base Feature Set](#opal-base-feature-set)
    - [Locally persisted Database replicas](#locally-persisted-database-replicas)
    - [~~Easy Custom Database States~~](#easy-custom-database-states)
    - [Pubsub Heads Exchange Replicator](#pubsub-heads-exchange-replicator)
    - [~~Zzzync Replicator integration~~](#zzzync-replicator-integration)
    - [~~IPLD Schema Validation  (if ready in javascript)~~](#ipld-schema-validation-if-ready-in-javascript)
    - [Automated Release: ci, change log, and api docs](#automated-release-ci-change-log-and-api-docs)
  - [`Zzzync` Development Plans](#zzzync-development-plans)
  - [`Opal-Spec` 1.0-beta]()
    - [Release Draft Spec for Opal and main modules](#release-draft-spec-for-opal-and-main-modules)
    - [Release Draft Spec for Live Replicator](#release-draft-spec-for-live-replicator)
    - [~~Release Draft Spec for Zzzync~~](#release-draft-spec-for-zzzync)
    - [Release completed Specifications for Opal ~~and Zzzync~~ 1.0-betas](#release-completed-specifications-for-opal-and-zzzync-10-betas)
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
    - [add live replicator (Libp2p pubsub + IPFS)](#add-live-replicator-libp2p-pubsub--ipfs)
    - [test replication and replicated states](#test-replication-and-replicated-states)
    - [write benchmarks](#write-benchmarks)
    - [automate release with generated API docs and changelog](#automate-release-with-generated-API-docs-and-changelog)
    - [release draft spec for Opal and main modules](#release-draft-spec-for-opal-and-main-modules-1)
    - [~~release alpha with expected public API changes~~](#release-alpha-with-expected-public-api-changes)
  - [~~Zzzync Replicator~~](#zzzync-replicator)
    - ~~begin design and spec of persistent replicator~~
    - ~~choose a design to build~~
    - ~~configure project repo~~
    - ~~write implementation (likely using web3.storage and w3name)~~
    - ~~write unit tests~~
    - ~~test interop with Opal~~
    - ~~write benchmarks~~
    - ~~automate release with generated API docs and changelog~~
    - ~~release draft spec~~
    - ~~release alpha with expected public API changes~~
  - [Release](#release)
    - ~~Heavy Testing~~
      - ~~network simulated testing with testground~~
      - ~~stress-test and benchmark replicators~~
      - ~~check for replication bugs and perf improvements~~
    - [Usage References](#usage-references)
      - [Opal and ~~Zzzync automated~~ (and nice-looking) API docs](#opal-and-zzzync-automated-and-nice-looking-api-docs)
      - [Write base FAQ.md document for common user questions](#write-base-faqmd-document-for-common-user-questions)
      - [Basic Tutorial document added to repo or blogged](#basic-tutorial-document-added-to-repo-or-blogged)
      - [NodeJS ~~and Create React App~~ examples](#nodejs-and-create-react-app-examples)
    - [Release Opal and Zzzync 1.0-beta](#release-opal)
      - [completed protocol specification](#completed-protocol-specification)
      - [typescript implementation (with public API locked until 1.0)](typescript-implementation)
- [Closing](#closing)
  - [Accomplishments](#accomplishments)
  - [Still Ongoing](#still-ongoing)
  - [Next Steps](#next-steps)
  - [Remarks](#remarks)

<br/>

---

<br/>

## Deliverables

### Opal Base Feature Set

Issue: https://github.com/cypsela/opal/issues/8

#### Locally persisted Database replicas

This required persisting the replicas and indexes of each database. Enables database reads (store index), and writes (replica) immediately upon being opened.

- https://github.com/hldb/welo/issues/12
- https://github.com/hldb/welo/commit/a9958dae8d105ca39f666487ee99bcb7cfee621f
- https://github.com/hldb/welo/commit/8bcbd35b8a935edffa13a9e540bd8554d3015255
- https://github.com/hldb/welo/commit/bcc880d197990688491b0bb6ecbb5ec4ffe135f8

#### ~~Easy Custom Database States~~

Creating custom databases is easy now but can still be much easier, so leaving this unfinished.
Necessary documentation will be added for this feature when it's ready.

Here is the [key-value store model](https://github.com/hldb/welo/blob/master/src/store/keyvalue/model.ts) showing what will be needed.
It includes a reducer, an init function (which supplies the initial value for the state), action creators, and selectors.
These are all similar to [Redux](https://redux.js.org/)'s design.

#### Pubsub Heads Exchange Replicator

This replicator is a reimplementation of the OrbitDB replicator.
Welo nodes join a shared pubsub channel unique per database.
When nodes see peers join the shared channel, nodes attempt to join a direct channel to advertise database updates.

Unlike OrbitDB, the replicator has a specification, and replication tests are [not flaky](https://github.com/hldb/welo/commit/3f60808cde11f90f4789a298426e6fdcf9603f98).

- https://github.com/hldb/welo/issues/17
- https://github.com/hldb/welo/issues/20

#### ~~Zzzync Replicator integration~~

Unfortunately, a Zzzync replicator was not added.
Building one will take more time than expected to do right. In this case *to do right* means the result is: relatively efficient/fast and general purpose (can be used apart from web3storage).

The proposal identified this as the most difficult part of the grant.

It's still possible to have a naive/experimental Zzzync replicator fairly quickly.
However, I didn't see this as worth it and am looking at designing a system for use long-term.

I did [inquire about this with Irakli](https://github.com/web3-storage/w3protocol/issues/154#issuecomment-1311854803) from web3storage, who always has some good information.

#### ~~IPLD Schema Validation  (if ready in javascript)~~

Validating the schema at the IPLD layer will likely be part of hardening the implementation in the nearer future.

#### Automated Release: ci, change log, and api docs

The release used is similar to js-libp2p's pipeline.
It creates a changelog inside a PR; after it's merged, the workflow cuts a new Github release and publishes it to npm.
A changelog is generated and the version is bumped by looking at the Conventional Commits in the commit messages.

The [API docs](https://github.com/hldb/welo/blob/master/API/welo.md) have been automated.
They look nice but can be improved in the future.
JSDoc is parsed by the tool to add useful information. The Typescript types are used to show parameters.

- https://github.com/hldb/welo/commit/97a7f072ee51a8be1b446c2c6b70c822ae8c4b87
- https://github.com/hldb/welo/issues/34
- https://github.com/hldb/welo/commit/b8c9b2a707a4825044c3f8fdd5757fdb05a5d7a6 

### Zzzync Development Plans

See [~~Zzzync Development Plans~~](https://github.com/tabcat/zzzync/issues/2)

### Opal-Spec 1.0-beta

#### Release Draft Spec for Opal and main modules

- https://github.com/hldb/specs/issues/2

#### Release Draft Spec for Live Replicator

- https://github.com/hldb/specs/issues/7

#### ~~Release Draft Spec for Zzzync~~

See [~~Zzzync Replicator integration~~](#zzzync-replicator-integration)

#### Release completed Specifications for Opal ~~and Zzzync~~ 1.0-betas

- https://github.com/hldb/specs/issues/1

## Development Roadmap

### Opal Repo Init

#### write unit tests

95% coverage according to [codecov.io](https://app.codecov.io/gh/hldb/welo)

There are more than a few testing-related commits; so I won't link them all.
Feel free to check out the [tests](https://github.com/hldb/welo/tree/master/test).
I plan on organizing them a bit more in the future.

#### prep for adding features

- https://github.com/hldb/welo/issues/10
- https://github.com/hldb/welo/issues/11

This work had to do with cleaning up code before adding new stuff.

##### build interfaces for manifest modules

Created Typescript interfaces for the different manifest modules.

##### rewrite manifest module registry

Fixed the module registry where modules referenced by the manifest are registered.

- https://github.com/hldb/welo/commit/fac797a8b36e51ee5eb4fd0a1f4fce5fd105339c

##### rework store module

- https://github.com/hldb/welo/commit/567646d872a09695544160b4666e566149f6b72
- https://github.com/hldb/welo/commit/e92a7410ec37c37c346c82ed0294b2c71db5241

##### rework classes to use Libp2p's startable interface

Libp2p's startable interface is a Typescript interface for managing the lifecycle of objects.
To make things easier when working with async start/stop I made [Playable](https://github.com/hldb/welo/blob/master/src/utils/playable.ts) that classes inside the project use.

- https://github.com/hldb/welo/commit/c9ec979387ae51c0ab14917f85212f0f98a585b
- https://github.com/hldb/welo/commit/1e88f90b4dcd56d749cab9e9a49895b73e5a168
- https://github.com/hldb/welo/commit/397009d5f604e2d3fd02e769982b5ae7b8e10c1
- https://github.com/hldb/welo/commit/7dd3ff883fce2ffd9b937ed062f39ead6f54eb1
- 65d549aca0967331aac2952172b73b63f5861801

#### make databases locally persistent

- https://github.com/hldb/welo/issues/12
- https://github.com/hldb/welo/commit/a9958dae8d105ca39f666487ee99bcb7cfee621f
- https://github.com/hldb/welo/commit/8bcbd35b8a935edffa13a9e540bd8554d3015255
- https://github.com/hldb/welo/commit/bcc880d197990688491b0bb6ecbb5ec4ffe135f8

### Opal Replication and Perf

#### add live replicator (Libp2p pubsub + IPFS)

- https://github.com/hldb/welo/issues/20
- https://github.com/hldb/welo/issues/21
- https://github.com/hldb/welo/issues/22
- https://github.com/hldb/welo/issues/23
- https://github.com/hldb/welo/issues/24
- https://github.com/hldb/welo/issues/25
- https://github.com/hldb/welo/issues/26
- https://github.com/hldb/welo/issues/27
- https://github.com/hldb/welo/issues/31

#### test replication and replicated states

- https://github.com/hldb/welo/issues/27

#### write benchmarks

This is not complete yet. It is part of [Still Ongoing](#still-ongoing).
Benchmarks will be added for read/write/replication, and metrics will be tracked over time with [github-action-benchmark](https://github.com/benchmark-action/github-action-benchmark).

- https://github.com/hldb/welo/issues/37

#### automate release with generated API docs and changelog

- https://github.com/hldb/welo/commit/b8c9b2a707a4825044c3f8fdd5757fdb05a5d7a6 
- https://github.com/hldb/welo/commit/97a7f072ee51a8be1b446c2c6b70c822ae8c4b87

See [Automated Release: ci, change log, and api docs](#automated-release-ci-change-log-and-api-docs) for more detail

#### release draft spec for Opal and main modules

- https://github.com/hldb/specs/issues/2

#### ~~release alpha with expected public API changes~~

The live replicator was unfinished at the time; as a result, an alpha was never released.

### Zzzync Replicator

See [~~Zzzync Replicator integration~~](#zzzync-replicator-integration)

### Release

#### ~~Heavy Testing~~

I'll be looking at doing the heavier testing with tools like Testground around the same time as performance and hardening. I did not quite reach this step during the grant.

#### Usage References

##### Opal ~~and Zzzync~~ automated (and nice-looking) API docs

Decided to use [api-documenter](https://api-extractor.com/pages/setup/generating_docs/) instead of [Typedoc](https://typedoc.org) because it had much nicer-looking generated markdown.

- https://github.com/hldb/welo/issues/34
- https://github.com/hldb/welo/commit/b8c9b2a707a4825044c3f8fdd5757fdb05a5d7a6 

See [Automated Release: ci, change log, and api docs](#automated-release-ci-change-log-and-api-docs) for more detail

##### Write base FAQ.md document for common user questions

https://github.com/hldb/welo/issues/39

##### Basic Tutorial document added to repo or blogged

https://github.com/hldb/welo/issues/40

##### NodeJS ~~and Create React App~~ examples

Welo has not been tested with browsers yet.
Support is tracked in [browser tests issue](https://github.com/hldb/welo/issues/42).

- https://github.com/hldb/welo/issues/41

#### Release Opal ~~and Zzzync~~ 1.0-beta

The 1.0.0 release was prematurely cut while trying out different release pipelines.

- https://github.com/hldb/welo/releases/tag/v1.0.0

Latest release is now [v1.0.2](https://github.com/hldb/welo/releases/tag/v1.0.2)

##### completed protocol specification

- https://github.com/hldb/specs/issues/10

##### typescript implementation (with public API locked until 1.0)

The original plan was to turn the project into Typescript toward the end of the grant.
However, Steve, my partner on [Cypsela](https://github.com/cypsela), recommended I use Typescript immediately.

*Thanks Steve*

It helped a bunch and likely sped up development pretty considerably.

- https://github.com/hldb/welo/issues/3
- https://github.com/hldb/welo/commit/d548bf51f31771a7c6f1837d54dd5fbcad21502d

## Closing

### Accomplishments

Welo, pronounced "way-low", is now a working peer-to-peer database written in Typescript.
It's very similar to OrbitDB but does not need to load all entries into memory before opening a database
(which can take multiple seconds; IIRC 2 seconds for every 1000 entries).
Because Welo write replicas and store indexes to disk, databases do not need to load all entries into memory;
exposing read and write immediately regardless of database size.
It will be easy, starting from here, to lazily load as much of the database into memory as the user sees fit.

Publishing, testing, API docs, and checking/updating dependencies are now automated.
Similar CI/CD actions to the js-libp2p repo test and create releases on push to the master branch. 

I created [`copy-deps`](https://github.com/tabcat/copy-deps), a package that makes updating shared dependencies easier.
This package comes in handy when upgrading `ipfs-core` as it overlaps a handful of dependencies `welo` uses.

A specification for the Opalescent database is published.
This is a living specification for version `1.0-beta` and will continue to be updated as things are changed and added.

### Still Ongoing

*This section will be updated as items are completed*

- [x] https://github.com/hldb/welo/issues/37
- [x] https://github.com/hldb/specs/issues/1
- [ ] ~~https://github.com/hldb/welo/issues/38~~

### Next Steps

*This section outlines ideas to work on following this grant*

After completing what is included in the section above,
I plan to look at designing an IO-efficient replica structure in IPLD.
Intuitively this also seems relevant to Zzzync replicators.

Testing, performance, and hardening the implementation will be the main focuses afterward, in that order.
I'd also like to learn Typescript a bit better.
I know general use right now but more advanced knowledge will enable me to write even cleaner code that requires less testing.

The protocol is in a good state for now,
but look forward to adding a low-latency replication protocol after Zzzync.

Will explore these path options more in a follow-up grant proposal.

### Remarks

This was my first solo grant project.
The admins managing the grant procedure and communication were great.
It's always been a pleasure when interacting with anyone from the Protocol Labs umbrella.

There are some areas I can improve, like tracking progress and creating deliverables/roadmaps. Hopefully, these will get easier to judge.

I would like to see how transparent/auditable I can make grant work progress in the future by linking directly to issues/prs.

## Amendment 1

Zzzync is being added back with a demo web app to close out the grant. This list will link directly to the relevant issues for better transparency.

- Development Roadmap
  - [`Zzzync` Initial Implementation](https://github.com/tabcat/zzzync/issues/6)
    - [begin and choose design](https://github.com/tabcat/zzzync/issues/7)
    - [write spec](https://github.com/tabcat/zzzync/issues/8)
    - [configure project repo](https://github.com/tabcat/zzzync/issues/9)
    - [write implementation](https://github.com/tabcat/zzzync/issues/10)
    - [write tests](https://github.com/tabcat/zzzync/issues/11)
    - [test interop with Welo](https://github.com/tabcat/zzzync/issues/12)
    - [automate release and changelog](https://github.com/tabcat/zzzync/issues/13)
    - [automate API docs](https://github.com/tabcat/zzzync/issues/14)
    - [complete protocol spec](https://github.com/tabcat/zzzync/issues/15)
    - [release 1.0-beta](https://github.com/tabcat/zzzync/issues/16)
  - [Zzzync Replicator integration](https://github.com/hldb/welo/issues/65)
  - [Welo + Zzzync Demo Webapp and Blog post](https://github.com/tabcat/rough-opal/issues/1)

ETA: June 17th

Grant budget is still capped at 30K

This amendment will count as the second two milestones, with the first two marked as complete.
