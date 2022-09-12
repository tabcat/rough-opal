

# Open Grant Proposal: `Rough Opal`

**Name of Project:** [Opal](https://github.com/cypsela/opal)

**Proposal Category:** `devtools-libraries`

**Proposer:** [@tabcat](https://github.com/tabcat)

**(Optional) Technical Sponsor:** Dietrich Ayala, [@autonome](https://github.com/autonome)

**Do you agree to open source all work you do on behalf of this RFP and dual-license under MIT, APACHE2, or GPL licenses?:** `Yes`

# Project Description

<!-- Please describe exactly what you are planning to build. Make sure to include the following: -->

<!-- - Start with the need or problem you are trying to solve with this project. -->

`Opal` is a peer-to-peer, local-first database. It's focus will be on providing web applications with dynamic and collaborative states. The core technologies used are:

 - IPLD - data reference
 - [Merkle-CRDTs](https://research.protocol.ai/publications/merkle-crdts-merkle-dags-meet-crdts/) - the replica data structure
 - IPFS, IPNS and Libp2p - update advertisement and replication
 - IPFS and Filecoin - backup and reliable hosting of replicas

The project is being written in typescript and is meant to be robust, maintainable, and easy for developers to start using. It is inspired by [OrbitDB](https://github.com/orbitdb/orbit-db) but will not be interoperable with it.

Merkle-CRDTs are at the heart of the project.
This data-structure is a combination of [Merkle-DAGs](https://docs.ipfs.tech/concepts/merkle-dag/) and [CRDTs](https://en.wikipedia.org/wiki/Conflict-free_replicated_data_type).
It provides causal order and de-duplication of operations and it ensures [strong eventual consistency](https://en.wikipedia.org/wiki/Eventual_consistency#Strong_eventual_consistency).

The project will provide these two abilities as part of this grant:

**Developers will be able to model custom states, similar to how Redux reducers are used.**
These states are computed by reading the replica's causal log of operations. This log can be updated online or offline and then merged/synced at a later point.

**Developers will be able to pin user replicas to reliable storage backends.**
Each device can pin their replica and update their own IPNS record. This allows peers to replicate with nodes that have gone offline.

With these two abilites, it's possible to create compelling, edge-computed apps.

<!-- - Describe why your solution is going to adequately solve this problem. -->

<!-- This section should be 2-3 paragraphs long. -->

## Value

<!-- Please describe in more detail why this proposal is valuable for the Filecoin ecosystem. Answer the following questions: -->

<!-- - What are the benefits to getting this right? -->

The architecture described previously combines Merkle-CRDTs with reliable hosting of the CRDT replicas for replication.
It can be seen as centered around the edge computers; with more general machines pinning the data to keep it available.
Applications this architecture is best-suited usually fall into the media or communication category; things like most of Google's app suite.

Building software this way has a few nice characteristics and goes hand in hand with delay-tolerant network designs like IPFS. The user has control over their data with the choice to pretty easily self-host. Any updates made locally are treated as the source of truth, sometimes called local-first.

While local-first applications can provide excellent UX, there are still areas that may need to be handled carefully by developers. When working with operation logs like this one, it can take time to merge with a peer that has been offline for a while or is just joining the database, depending on the number of entries that have been made.

There are also fundamental limits, and some applications cannot be modelled due to invariant confluence. To be invariant confluent an appliction must be able to apply any 2 messages asynchronously without invalidating the applications invarients.
For example, an invariant of blockchains is that no account can have a negative balance. This invariant is not confluent since there could be 2 messages subtracting from a balance; where separately applied the balance stays above 0, but applying them together results in a negative balance.

The most difficult part to deliver for this grant will be the persisted replication piece. This is the part that involves uploading the replica, as its updated, to pinning services. It's totally new, will likely use CAR files and involve updating IPNS records.

<!-- - What are the risks if you don't get it right? -->

<!-- - What are the risks that will make executing on this project difficult? -->

<!-- This section should be 1-3 paragraphs long. -->

## Deliverables

<!-- Please describe in details what your final deliverable for this project will be. Include a specification of the project and what functionality the software will deliver when it is finished. -->

[`Opal`](https://github.com/cypsela/opal) and [`Zzzync`](https://github.com/tabcat/zzzync) are the deliverables. `Opal` is the Merkle-CRDT collaborative states piece, and `Zzzync` will be a replicator module for persistent replication.

The exact features to be delivered of each are defined in the following issues:

[`Opal` Base Feature Set](https://github.com/cypsela/opal/issues/8)

[`Zzzync` First Release](https://github.com/tabcat/zzzync/issues/2)

There will also be a status issue every month in the Opal repo which will track what is being worked on and completed. Here is [September's Status](https://github.com/cypsela/opal/issues/1).

## Development Roadmap

<!-- Please break up your development work into a clear set of milestones. This section needs to be very detailed (will vary on the project, but aim for around 2 pages for this section). -->

<!-- For each milestone, please describe: -->
<!-- - The software functionality that we can expect after the completion of each milestone. This should be detailed enough that it can be used to ensure that the software meets the specification you outlined in the Deliverables. -->
<!-- - The amount of funding required for each milestone -->
<!-- - How much time this milestone will take to achieve (using real dates) -->

**(SEPT 2022)** Opal Repo Init
 - set up automation for docs, linting, formatting
 - prep for adding features
 - make databases locally persistent

**(OCT 2022)** Opal Replication and Perf
 - add basic replicator (Libp2p pubsub + IPFS)
 - test replication and replicated database states
 - write benchmarks

**(NOV 2022)** Zzzync Replicator
 - replicator using IPFS/IPNS pinning on web3.storage
 - write tests for Zzzync and interopt tests for Opal

**(DEC 2022)** Heavy Testing
 - testing inside network simulation with testground
 - stress-test and benchmark replicators
 - check for replication bugs and perf improvements

## Total Budget Requested

<!--Sum up the total requested budget across all milestones, and include that figure here. Also, please include a budget breakdown to specify how you are planning to spend these funds. -->

## Maintenance and Upgrade Plans

<!-- Specify your team's long-term plans to maintain this software and upgrade it over time. -->

The plan for this grant is to build a foundation that will define the base features and keep the project hyper-maintainable over the years. After the project reaches this level of maintainability, the key is to cultivate a userbase.
To do this it will be important to gain some exposure while providing good docs and a helpful community chat.

Following the inital release there will still be room for improvement:

Nearer future (~2.0):
 - active replication: implementing the replication algorithm described in [Byzantine Eventual Consistency](https://arxiv.org/pdf/2012.00472.pdf), involves pushing data that is missed by a peers bloom filter. good for applications that want less latency, e.g. messaging.
 - encrypted Merkle-CRDT: using a group encryption algorithm like [Key Agreement for Decentralized Secure Group Messaging](https://martin.kleppmann.com/2021/11/17/decentralized-key-agreement.html), which should fit quite nicely.
 - dynamic access control: update access control lists without affecting operation history
 - graphsync replicator: using graphsync to improve replicator performance.

Further future (~3.0):
 - dynamic topological sort: would involve maintaining a topological sort of the DAG as entries are merged. unsure if this can be done deterministically but willing to do some digging. the only paper I've found on this is [A Dynamic Topological Sort Algorithm for
Directed Acyclic Graphs](https://www.doc.ic.ac.uk/~phjk/Publications/DynamicTopoSortAlg-JEA-07.pdf) and will need to revisit.
 - finality gadgets: this would look at the best ways to migrate databases without too many side-affects.
 - CBOR CRDT: a CBOR state where each field in the CBOR object is fully merge-able
 - cross-log causality: use randomness beacons like `drand` to provide a universal causality. seems useful for some applications?

Following the full 1.0 release there will still be room for improvement:

# Team

## Team Members

Daniel, [@tabcat](https://github.com/tabcat)

## Team Website

<!-- Please link to your team's website here (make sure it's `https`) -->
[https://github.com/cypsela](https://github.com/cypsela)

## Relevant Experience

<!-- Please describe (in words) your team's relevant experience, and why you think you are the right team to build this project. You can cite your team's prior experience in similar domains, doing similar dev work, individual team members' backgrounds, etc. -->

Daniel, also known as Anders, has been involved with OrbitDB since finding it in late 2018, shortly after diving into IPFS.

> In March of 2020 I started working on a collaborative filesystem on top of IPFS using OrbitDB. In July 2020 a partner and I leveraged this to build [`sailplane`](https://cypsela.github.io/sailplane-web/), a p2p Dropbox-like web app that made us finalists in the first HackFS.
>
> In February 2021 I was contracted by `equilibrium.co` to maintain OrbitDB.
>
> In November we signed an open source grant from Protocol Labs to fund the first part of development for OrbitDB 1.0. During the 6 month grant period, I worked on keeping the current version supporting the latest `js` and `go` IPFS implementations, as well as a protocol spec and implementation for 1.0.
>
> That work ended up not being accepted by the owners of OrbitDB so I'm continuing it with Opal.
>
> In July 2022 I began thinking more about persistant replication methods for OrbitDB which involved IPFS and IPNS pinning. While on a trip during HackFS 2022 I learned about web3.storage and w3name which were exactly what I needed. After coming back from the trip I hacked them together for a short demo project and won the IPFS/Filecoin first prize.

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

`Opal` is a temporary name/code name, until I find something better. Always open to hearing naming ideas! 😁