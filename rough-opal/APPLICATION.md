
# Open Grant Proposal: `Rough Opal`

**Name of Project:** [Opal](https://github.com/cypsela/opal)

**Proposal Category:** `devtools-libraries`

**Proposer:** [@tabcat](https://github.com/tabcat)

**(Optional) Technical Sponsor:** Dietrich Ayala, [@autonome](https://github.com/autonome)

**Do you agree to open source all work you do on behalf of this RFP and dual-license under MIT, APACHE3, or GPL licenses?:** `Yes`

# Project Description

<!-- Please describe exactly what you are planning to build. Make sure to include the following: -->

<!-- - Start with the need or problem you are trying to solve with this project. -->

`Opal` is a peer-to-peer, local-first database. Its focus will be on providing web applications with dynamic and collaborative states. The core technologies used are:

 - IPLD - data reference
 - [Merkle-CRDTs](https://research.protocol.ai/publications/merkle-crdts-merkle-dags-meet-crdts/) - the replica data structure
 - IPFS, IPNS, and Libp2p - update advertisement and replication
 - IPFS and Filecoin - backup and reliable hosting of replicas

The project is written in typescript, and compiled into javascript. It will be robust, maintainable, and easy for developers to start using.
This project continues work done under a grant for [OrbitDB](https://github.com/orbitdb/orbit-db) that they ended up not accepting.
Opal is not a fork of OrbitDB; it's is a complete rewrite and repurpose focused on efficiently representing arbitrary states. Opal will not yet be interoperable with OrbitDB.

Merkle-CRDTs are still at the heart of the project.
This data-structure is a combination of [Merkle-DAGs](https://docs.ipfs.tech/concepts/merkle-dag/) and [CRDTs](https://en.wikipedia.org/wiki/Conflict-free_replicated_data_type).
It provides causal order and de-duplication of operations and ensures [strong eventual consistency](https://en.wikipedia.org/wiki/Eventual_consistency#Strong_eventual_consistency).

The project will provide these two abilities as part of this grant:

**Developers will be able to model custom states, similar to using Redux reducers.**
They can supply a way to reduce and read a state.
These states are computed by reading the replica's causal log of operations.
This log can be appended online or offline and then merged/synced at a later point.

**Developers will be able to pin user replicas to reliable storage backends.**
Each device can pin its replicas as CAR files and update its IPNS record.
These allow peers to replicate with nodes that have gone offline.
Data persistence has been an issue for peer-to-peer databases. Persistent replication is a big step forward.

These two abilities make it possible to create compelling, edge-computed apps.

<!-- - Describe why your solution is going to adequately solve this problem. -->

<!-- This section should be 2-3 paragraphs long. -->

## Value

<!-- Please describe in more detail why this proposal is valuable for the Filecoin ecosystem. Answer the following questions: -->

<!-- - What are the benefits to getting this right? -->

The architecture described previously combines Merkle-CRDTs with reliable hosting of the CRDT replicas for replication.
This design centers around edge computers with more general machines pinning and keeping the data available.
Applications this architecture is best-suited fall into the media or communication, like most of Google's app suite.

Building software this way has unique characteristics and goes hand in hand with delay-tolerant network designs like IPFS. The user has control over their data with the choice to self-host. The local replica is the source of truth, sometimes called local-first.

Using Opal, developers will be able to define the state reducer for a database.
A state reducer is a function that takes a state with an operation and returns a new state.
Collaborative UI components can use Opal to derive their state.
This collaboration can be between users or a user's devices.

A big issue with peer-to-peer databases like OrbitDB is that if no other devices are online, you can't replicate anything!
Persistent replication is needed, and there are two ways to do it.

The first is continuing the pinning service idea, where you have servers to keep online with a list of databases to replicate.
These servers run replication algorithms that work over pubsub and IPFS.
If those pinners are online, then the data is available and can be replicated.
This solution is not terrible as it has some benefits, but it is better called 'persisted replication' since pubsub messages are not persistent.

The second solution for persistent replication has to do with swapping pubsub for IPNS.
Instead of a node advertising the latest known heads over pubsub, IPNS becomes used in-place, with the IPNS records and IPFS data being pinned.
Another advantage here is that IPFS and IPNS are more general layers and don't require building up infrastructure and support.

Because the work done for OrbitDB under a previous grant was not accepted, doing this with OrbitDB would be very difficult due to some tight coupling.
Opal is much more modular when compared, especially with replicationy.

Opal also includes incremental traversal of the Merkle-DAG in either direction.
This type of traversal is not part of OrbitDB, and is probably the most significant change from that previous grant work.
Incremental traversal allows for database entries to be kept out of memory and streamed when needed by traversing a graph of CIDs.

The most challenging part of this grant will be building persistent replication. It involves uploading the replica to pinning services as it's updated. It's new, will involve CAR files and updating IPNS records.

<!-- - What are the risks if you don't get it right? -->

<!-- - What are the risks that will make executing on this project difficult? -->

<!-- This section should be 1-3 paragraphs long. -->

## Deliverables

<!-- Please describe in details what your final deliverable for this project will be. Include a specification of the project and what functionality the software will deliver when it is finished. -->

[`Opal`](https://github.com/cypsela/opal) and [`Zzzync`](https://github.com/tabcat/zzzync) are the deliverables. `Opal` is the Merkle-CRDT collaborative states piece, and `Zzzync` will be a replicator module for persistent replication.

The exact features that will be delivered of each are defined in the following issues:

[`Opal` Base Feature Set](https://github.com/cypsela/opal/issues/8)

[`Zzzync` Development Plans](https://github.com/tabcat/zzzync/issues/2)

There will also be a monthly status issue in the Opal repo. The monthly issues will track what is being worked on and completed. Here is [September's Status](https://github.com/cypsela/opal/issues/1).

## Development Roadmap

<!-- Please break up your development work into a clear set of milestones. This section needs to be very detailed (will vary on the project, but aim for around 2 pages for this section). -->

<!-- For each milestone, please describe: -->
<!-- - The software functionality that we can expect after the completion of each milestone. This should be detailed enough that it can be used to ensure that the software meets the specification you outlined in the Deliverables. -->
<!-- - The amount of funding required for each milestone -->
<!-- - How much time this milestone will take to achieve (using real dates) -->

**(SEPT 2022)** Opal Repo Init
  - set up linting/formatting
  - configure typescript
  - write unit tests
  - prep for adding features
    - build interfaces for manifest modules
    - rewrite manifest module registry
    - rework store module
    - rework classes to use Libp2p's startable interface
  - make databases locally persistent

**(OCT 2022)** Opal Replication and Perf
  - add basic replicator (Libp2p pubsub + IPFS)
  - test replication and replicated states
  - write benchmarks
  - automate release with generated api docs and changelog
  - release alpha with expected public api changes

**(NOV 2022)** Zzzync Replicator
  - begin design of replicator
  - choose a design to start building
  - set up linting/formatting
  - configure typescript
  - write implementation of design
  - write unit tests
  - test interop with Opal
  - write benchmarks
  - automate release with generated api docs and changelog
  - release alpha with expected public api changes

**(DEC 2022)** Heavy Testing
  - network simulated testing with testground
  - stress-test and benchmark replicators
  - check for replication bugs and perf improvements
  - release Opal and Zzzync beta with public api locked

## Total Budget Requested

<!--Sum up the total requested budget across all milestones, and include that figure here. Also, please include a budget breakdown to specify how you are planning to spend these funds. -->

## Maintenance and Upgrade Plans

<!-- Specify your team's long-term plans to maintain this software and upgrade it over time. -->

This grant will build a foundation that will define the base features and keep the project hyper-maintainable over the years.
After the project reaches this level of maintainability, the key is cultivating a user base.
Acquiring users will require exposure while providing documentation, a helpful community chat, and a valueable tool with great developer experience.

Following release there will still be room for improvement:

Nearer future (~2.0):
 - active replication: implementing the replication algorithm described in [Byzantine Eventual Consistency](https://arxiv.org/pdf/2012.00472.pdf); involves pushing data missed by a peer's bloom filter. useful for applications that want less latency, like messaging.
 - encrypted Merkle-CRDT: using a group encryption algorithm like [Key Agreement for Decentralized Secure Group Messaging](https://martin.kleppmann.com/2021/11/17/decentralized-key-agreement.html), which should fit quite nicely.
 - dynamic access control: update access control lists without affecting operation history
 - efficient predecessor referencing: allow quicker traversal and replication of the Merkle-DAG by picking references smartly, thanks to [science](https://kodu.ut.ee/~lipmaa/papers/thesis/thesis.pdf).
 - graphsync replicator: using graphsync to improve replicator performance.

Further future (~3.0):
 - dynamic topological sort: maintaining a topological sort of the DAG as entries get merged. not sure if it is possible to do deterministically. related: [A Dynamic Topological Sort Algorithm for
Directed Acyclic Graphs](https://www.doc.ic.ac.uk/~phjk/Publications/DynamicTopoSortAlg-JEA-07.pdf) and will need to revisit.
 - finality gadgets: this would look at the best ways to migrate databases without too many side effects.
 - CBOR CRDT: a CBOR state where each field in the CBOR object is fully mergeable.
 - cross-log causality: use randomness beacons like `drand` to provide a universal causality. seems useful for some applications?

# Team

## Team Members

Daniel, [@tabcat](https://github.com/tabcat)

## Team Website

<!-- Please link to your team's website here (make sure it's `https`) -->
[https://github.com/cypsela](https://github.com/cypsela)

## Relevant Experience

<!-- Please describe (in words) your team's relevant experience, and why you think you are the right team to build this project. You can cite your team's prior experience in similar domains, doing similar dev work, individual team members' backgrounds, etc. -->

Daniel, has been involved with OrbitDB since finding it in late 2018, shortly after diving into IPFS.

> In March 2020, I started working on a collaborative filesystem on top of IPFS using OrbitDB. In July 2020, a partner and I leveraged this to build [`sailplane`](https://cypsela.github.io/sailplane-web/), a p2p Dropbox-like web app that made us finalists in the first HackFS.
>
> In February 2021, I was contracted by `equilibrium.co` to maintain OrbitDB.
>
> In November, we signed an open source grant from Protocol Labs to fund the first part of development for OrbitDB 1.0. During the 6 month grant period, I worked on keeping the current version supporting the latest `js` and `go` IPFS versions, and a protocol spec and implementation for 1.0.
>
> That work ended up not being accepted by the owners of OrbitDB, so I'm continuing it with Opal.
>
> In July 2022, I began thinking more about persistent replication methods for OrbitDB involving IPFS and IPNS pinning. While on a trip during HackFS 2022, I learned about web3.storage and w3name. After the trip I hacked them together for a short demo project and won the IPFS/Filecoin first prize.

## Team code repositories

<!-- Please provide links to your team's prior code repos for similar or related projects. -->

Daniel:
  - [zzzync](https://github.com/tabcat/zzzync/tree/hackfs22): 2 day hack/concept using web3storage, first prize at hackfs2022 [[ref1]](https://ethglobal.com/showcase/zzzync-xk96u)
  - [sailplane](https://github.com/cypsela/sailplane-web): collaborative filesystem web app built with orbitdb and ipfs, finalist at hackfs2020 [[ref1]](https://showcase.ethglobal.com/hackfs/sailplane-web) [[ref2]](https://filecoin.io/blog/posts/meet-the-hackfs-teams-vol.3/)
  - [orbit-db-fsstore](https://github.com/tabcat/orbit-db-fsstore): a custom orbitdb database representing a filesystem
  - [orbitdb](https://github.com/orbitdb): community maintainer and formerly full-time maintainer

# Additional Information

How did you learn about the Open Grants Program?

`from previously working on an open source grant from protocol labs for orbitdb`

Please provide the best email address for discussing the grant agreement and general next steps.

`tabcat00@proton.me`

<!-- Please include any additional information that you think would be useful in helping us to evaluate your proposal. -->

---

`Opal` is a temporary name/code-name, until I find something better. Always open to hearing naming ideas! 😁