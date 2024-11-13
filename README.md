# JAM Implementers Prize

### Table of Contents:

* [JAM Demo (TypeScript)](#jam-demo)
* [Polkadot vs JAM](#polkadot-vs-jam)
* [Multiple JAM implementations](#multiple-implementations)
* [Questions & Answers](#questions-answers)
* [Graypaper Summary Notes](#graypaper-summary-notes)
* [Glossary](#glossary)
* [Tools](#tools)
* [References](#references)

## JAM Demo <a id="jam-demo"></a>

Follow the instructions [here](./packages/jam/README.md).

## Polkadot vs JAM <a id="polkadot-vs-jam"></a>

* Userland level (i.e. Parachains)
	* L1 (parachains/shards) and L2 (cores) are all entirely different applications, hence why Polkadot is a heterogeneous sharded blockchain
		* Each stores its blockchain definition as bytecode in the state of the blockchain itself
			* Polkadot 1.0 used WASM for bytecode
			* JAM uses PVM/RISC-V as its VM of choice https://x.com/paritytech/status/1788847203622043974
        * RISC-V is an open-source hardware specification, instruction set architecture
          * e.g. any software that can compile it to RISC-V (any code since LLVM can target RISC-V), then run it on-chain and it should just work
        * PVM is translatable from RISC-V, then create and run a PVM instance with gas (lower gas in JAM since more efficient and more parallelism), then save its state and restore that state later, useful for long-running services

  * **CoreChains Service**
    * First service deployed in JAM will likely be called CoreChains Service (or Parachains Service) and support allowing existing Polkadot-2-style parachains to be executed on JAM.

  * **Further Services**
    * Additional services may be deployed on JAM that communicate with the existing **CoreChains Service**

* Kernel level
	* Polkadot ingests one L0 (relay chain) block and one L1 (parachain) block per one L2 Core, per time-slot
  * Polkadot has low composability with limited opportunity for collaboration between its ecosystems similar to bridged blockchains but with a different security profile
	* JAM State is On-Chain Operation (Polkadot's main block and STF) and interacts with Accumulate (uses PVM)
	* Accumulate (uses PVM) is On-Chain Operation and gathers data from a Distributed Data Lake
	* Operations
		* Normal Execution
			* Blockchain (is a State-Transition Function STF)
			* On-Chain Operation (typical blockchain computation) where Main blockchain state updated as outcome of validators executing work
	
  * Execution Sharding
    * Fully Coherent (Synchronous) vs Fully Incoherent (Asychronous) Systems
      * Graypaper section 2.4
        * Synchronous (coherent) systems have growth limitations before compromising generality, accessibility, and resilience
        * Asynchronous (incoherent) systems is a major drawback of permanently sharded systems (e.g. Polkadot 1, Polkadot 2, existing Ethereum L2s)
    * Polkadot uses XCM language for parachain communication that is asynchronous, where a message is sent, and its reply cannot be waited upon.
    * Semi-Coherent System
      * JAM instead introduces Multiple Properties to achieve a Semi-Coherent system as a novel middle-ground, where sub-systems that communicate may often have a chance at creating a coherent environment with one another, whilst not enforcing the entire system to be coherent. https://www.youtube.com/watch?t=1378&v=O3kRAVBTkfs&embeds_referring_euri=https%3A%2F%2Fblog.kianenigma.nl%2F
      * JAM will be sharded, heterogenous, AND will offer a Semi-Coherent System where the boundaries of those shards may be determined flexibly https://t.co/tjAboJL9IA
        * Properties that enable this are:
          * In-Core Execution that is state-less and parallel to be provided, where different services can only interact Synchronously with other services that reside on the same core in that particular block
          * On-Chain Execution where a service has access to the outcome of all services across all cores.
          * JAM does not enforce any particular scheduling of services
          * Services that talk to each other frequently can create an economic incentive for their Sequencers to create WorkPackages that contain WorkItems of Services that often communicate. This enables them to reside in the same core, and in practice, talk to each other as-if they were in a Synchronous environment.
          * JAM services have access to the DA layer and can use it as an ephemeral, yet extremely cheap Data Accessibility Layer (DA). Once a data is placed in the DA, it is eventually propagated to all cores, but is guaranteed to be available in that same core immediately
          * JAM services through scheduling themselves into the same core in consecutive blocks can enjoy a much higher degree of access to data.
          * Protocol Layer that interface with JAM may be asynchronous in theory may act synchronously in practice through elegant abstractions and incentives (e.g. CorePlay)
  * Data Sharding
    * Semi-Coherence
      * Fully-Coherent (Synchronous) system (e.g. deploying on smart contract system like Solana or Ethereum L1) is better in principle, but does not scale, where all application data is stored on-chain for perfect programmability and easily accessible to other applications
      * Fully-Incoherent (Asynchronous) system (e.g. Polkadot and the Ethereum rollup model) is not desirable, extremely scalable, but not great for composability, where application data is kept outside the L1 in different isolated shards
      * Semi-Coherent model (e.g. JAM) poses a new possibility by providing:
        * Both Full-Coherent and Full-Incoherent options
        * Allows programmers to post arbitrary data into the JAM DA layer(middle-ground between on-chain and off-chain data)
        * Allows a novel category of applications to be written while leveraging the DA layer for majority of the application data
        * Allows only persisting what is absolutely crucial into the JAM state.
  * Scalability
    * Scaling Up (Vertical) (e.g. Solana)
      * Hyper optimize the code in a monolithic blockchain, and high performance hardware to achieve maximum throughput
    * Scaling Out (Horizontal) (e.g. Ethereum and Polkadot)
      * Reducing the number of machines that have to executing all the work
        * In traditional distributed systems you add more replicated machines
        * In blockchains, the "computation machine" is the entire validator set of a network so we want to reduce the workload on the entire validator set as a whole to scale out the system, which is achieved by either:
          * Split work between them (e.g. using ELVES, which are Polkadot's cynical rollups), or;
          * Optimistically discount their duties (e.g. using Optimistic Rollups)
          * SNARK-based rollups
  * Transactions
    * JAM is transactionless, everything is triggered by the servers used by users

  * PVM (PolkaVM)
    * Efficient metering
    * Ability to pause and resume execution, which is important for CorePlay

  * RISC-zero
    * RISC-V is open-source virtual machine platform for producing SNARKs of computation
    * RISC-V benchmark of proof generation alone in 2024 takes over 61,000x longer than simply recompiling and executing even when executing on 32 times as many cores, using 20,000x more RAM and additional GPU, according to PolkaVM benchmarking in 2024 when compared to RISC-zero's benchmark
    * Cost multiplier of proving using RISC-zero is 66,000,000x the cost to execute using a RISC-V recompiler, according to hardware rental agents https://cloud-gpus.com/
    * SNARK proof generation slowdowns could reduce to 50,000x from regular native execution and could mostly be parallelized according to Thaler 2023 through advancements in cryptographic techniques, but still several orders of magnitude greater required to compete on a cost-basis with established crypto-economic techniques such as ELVES 

* Hardware level (i.e. Cores)
	* Data Availability (DA) Layer (In-Core)
    * Distribute DA floats/hangs there without having to go to a distributed storage solution nor go on on-chain, it can just be there ready for usage for up to 28 days
		* Distributed Data Lakes (DDL) are the DA layer and are In-Core and interact with multiple Cores (uses PVM)
		* Cores (uses PVM) each are In-Core and `refine` (aka Join) Work Items from their each of their Work Packages
		* L2s automatically use Polkadot's native Data Availability (DA) Layer to keep their execution evidence available for a period of time, where that execution evidence posted that is needed to re-execute an L2 block is a blob that is fixed
		* L1s (parachains/shards) code never reads from the L2s Data Availability (DA) Layer
		* Core is an environement where a subgroup of validators work in coordination to re-execute a single L2 (parachain/shard) block
			* In-Core Execution (novel blockchain computation) where Core receives Data (L2 block + subset of L2 state) that is gossipped to the subgroup of validators (constituting the core) and used to execute L2 block then perform consensus tasks
			* Core validators share Data needed for re-execution so it is available to other validators (outside the core)
				* Other Core validators (outside the original core) may decide to re-execute the L2 block based on ELVES rules
				* Polkadot L0 (relay chain) main state is updated and Polkadot block provided with a small commitment of the L2 latest state that is executed by all Polkadot validators
    * **CorePlay**
      * New model for programming smart contracts that uses JAM's flexible primitives to create a synchronous and scalable smart contract environment with a very flexible programming interface
      * Deployment of actor-based smart contracts directly on JAM cores in the CorePlay Service of the "Userland" layer, and enabling them to enjoy a synchronous programming interface whereby they can be coded as a normal `fn main()`, in which they can communicate using `let _result = other_coreplay_actor(data).await?`, where the call is synchronous if `other_coreplay_actor` is in the same core in that JAM block, otherwise the actor is paused and will be resumed in a later JAM block. This is precisely possible because of JAM services and their flexible scheduling, and PVM's properties.
      * CorePlay Service will have access to In-Core and On-Chain APIs, but not its smart contracts that are merely a `fn main()` and a `fn on_message()` (called when a message from another Service arrives)
      * CorePlay Service will handly all the scheduling
      * **TODO** CorePlay RFC by Dr Gavin Wood https://github.com/polkadot-fellows/RFCs/tree/gav-coreplay

## Multiple JAM Implementations <a id="multiple-implementations"></a>

### Technical Decentralization

* Single implementation is highly centralizing
* Resilience
* Avoid same bug across implementations (e.g. Shanghai attack on Ethereum in 2016 where Geth clients went down but not Parity clients)

### Intellectual Decentralization

* Many implementers under different economic umbrellas contributing

* Reference:
  * https://x.com/paritytech/status/1788847203622043974

## Questions & Answers <a id="questions-answers"></a>

* What is "audit work" shown in diagram #2 at https://hackmd.io/0gSmXyElT2iKawymS5jJKw?view
  * [ ] Answer:
* What do "Guarantors" do as shown in diagram #3 at https://hackmd.io/0gSmXyElT2iKawymS5jJKw?view
  * [ ] Answer:
* Where is "Guaranteeing", "Import DA", "Audits DA", "Auditing & Judgements", and "Safrole" as shown in System diagram #4 at https://hackmd.io/0gSmXyElT2iKawymS5jJKw?view
  * [ ] Answer:
* What is role of "Proxy nodes" shown in Networking diagram #5 at https://hackmd.io/0gSmXyElT2iKawymS5jJKw?view
  * [ ] Answer:

## Graypaper Summary Notes <a id="graypaper-summary-notes"></a>

* JAM
	* Smart-contract functionality similar to Ethereum
	* Secure and scalable in-core/on-chain dualism
  * Service that provides an abstraction closer to the actual computation model generated by the validator nodes its economy incentives that was not offered previously

* Goals
	* Resilience - resistant from being stopped, corrupted and censored
	* Generality - able to perform Turing-complete computation
	* Performant - able to perform fast computation at low cost
	* Coherent - causal relationships possible between different parts of state impacting how well individual applications may be composed
	* Accessibility - no negligible barriers to innovation, easy, fast, cheap and permissionless.

* Size-synchrony Antagonism 
	* Describes the combination of the "performance" and "coherence" goals according to information theoretic principles
	* Example:
		* Fully-coherent (synchronous) system
			* As the state-space of information systems grow and utilise for its data processing, the more space the state must occupy and the greater the mean and variance of distances between state components causing interactions to become slower requiring subsystems to consider that distances between interdependent components of state could be significantly different, which requires asynchrony, such that the system necessarily becomes less synchronous
		* Fully-incoherent (asynchronous) system
			* Achieved by ignoring security for the moment, and sacrificing coherency, and applying the divide and conquer maxim and fragmenting the state of the system to create two independent smaller-state systems rather than the previous single large-state system, such that this pattern applies a step-curve to the principle behind meta-networks Polkadot or a scaled Ethereum, where;
				* Intra-system processing: low size, high synchrony
				* Inter-system processing: high size, low synchrony
		* Semi-coherent system
			* Explored by JAM, the middle-ground in the antagonism, which avoids persistent fragmentation of the statespace of the system as with existing approaches, achieved by using a new model of computation that pipelines a highly scalable element to a highly synchronous element, but whilst asynchrony is not avoided, it does increase granularity over how it is traded against size.
			* Fragmentation can be made ephemeral rather than persistent, drawing upon a coherent state and fragmenting it only for as long as it takes to execute any given piece of processing on it.
			* Unlike with snark-based L2-blockchain techniques for scaling, this model draws upon crypto-economic mechanisms and inherits their low-cost and high-performance profiles and averts a bias toward centralization.

* Service Profile
	* Declare a general-purpose rules-based service profile for a general-purpose service of a resilient Web3 digital system that offers strong guarantees to be unstoppable and Turing complete and allows for the use-cases.
	* It needs a general-purpose rule-set to deliver to a massively multiuser application platform

* Service
	* Offers collect/refine/join/accumulate model of computation
	* Offers CoreChains service that is Polkadot-compatible and where model largely based on architecture of Polkadot

* Permissionless - allowing anyone to deploy code as a service on it for a fee commensurate with the resources this code utilizes

* Execution - induces execution of the deployed code through the procurement and allocation of core-time, which is a metric of resilient and ubiquitous computation similar to purchasing gas in Ethereum

* Scaling approaches in blockchain - TODO section 2

* Polkadot Virtual Machine (PVM)
	* TODO section 4
	* TODO - additional material important for the protocol definition of pvm in appendices A & B

* Safrole consensus protocol - TODO section 4

* Common clock - TODO section 4

* Formalism - TODO section 4

* Full protocol definition 
	* Part 1 - correct on-chain state-transition formula
	  * helpful for all nodes wishing to validate the chain state
	* Part 2 - in sections 14 and 19 the honest strategy for the off-chain actions of any actors who wield a validator key

* Performance characteristics of the protocol - in section 20 

* Serialization and Merklization
	* TODO - in appendices C & D, it contains various additional material important for the protocol definition

* Cryptography
	* TODO - in appendices E, G, it contains various additional material important for the protocol definition

* Glossary
	* TODO - in appendix H

* Values of all simple constant terms used in JAM
	* TODO - in appendix I

## Glossary <a id="glossary"></a>

* Blockchain
  * State-Transition Function (STF_)
* Blockspace
  * Computation that is done without the need to trust someone, where its quality level is the amount you don't need to trust someone, and what it could be used for, and how much there is in terms of throughput and cost
* Data Availability (DA)
  * Ability for Polkadot validators to commit to having some data available for some period of time, and providing it to other validators
* ELVES
  * Competes with crypto-economic techniques such as SNARKs
  * Offers utilization of non-redundant partitioning, where combining this with a proposal-and-auditing game which validators play to weed out and punish invalid computations, and then require that the finality of one network be contingent on all causally-entangled networks. This is the most secure and economically efficient solution since its mechanism recognizes invalid transitions and corrects them before their effect is finalized across the ecosystem of networks. There is an upper limit on the number of networks which may be added.

* Ethereum
  * Dank Sharding Availability System
    * Ethereum protocol that offers an improvement by splitting responsibility for its storage amongst the validator base
    * Use of SNARKs in PLONK as part of the Dank Sharding availability system uses a form of erasure coding centered around polynomial commitments over a large prime field in order to allow SNARKs to get acceptably performant access to subsections of data. Compared
    to alternatives, such as a binary field and Merklization in
    the present work, it leads to a load on the validator nodes
    orders of magnitude higher in terms of CPU usage.
    * SNARKs
      * Present no great escape from decentralization and the need for redundancy in addition to their basic cost, leading to further cost multiples
      * Verifiable nature of SNARKs avoids requiring benefits of staked decentralization 
      * Incentivizing multiple parties to do the work is a requirement to ensure single party does not form a monopoly (or several not form
      a cartel). Proving an incorrect state-transition should be impossible, however service integrity may be compromised
      in other ways; e.g. 
        * Temporary suspension of proof-generation, even if only for minutes, could amount to major economic ramifications for real-time financial applications.
  * Sideband Strategy
    * Centre around snark-based rollups so the protocol is being evolved into a design that makes sense for this. Snarks are the product of an area of exotic cryptography which allow proofs to be constructed to demonstrate to a neutral observer that the purported result of performing some predefined computation is correct. The complexity of the verification of these proofs tends to be sub-linear in their size of computation to be proven and will not give away any of the internals of said computation, nor any dependent witness data on which it may rely
  * Scaling Strategy
    * Couple Data Availability (DA) with a private market of roll-ups, sideband computation facilities of various design, with zk-snark-based roll-ups being a stated preference, where each vendor's roll-up design, execution and operation comes with its own implications. Risk of a broad pattern of centralization occuring in the various roll-ups that is being mitigated

* Fragmented Meta-Networks for computation scalability
  * Fragmented approach (e.g. Cosmos, Avalanche)
    * System of scaling-by-fragmentation approach by networks of a homogenous consensus mechanic, yet staffed by separately motivated sets of validators.
    * No homogenous security—and therefore trustlessness—guarantees.
    * Homogeneity of fragmentation approach allows for reasonably consistent yet superficial messaging mechanics to present a fairly unified interface to the multitude of connected networks
    * Consistency is superficial since networks are trustless only if their validators operate correctly under a crypto-economic security framework enforced by economic incentives and punishments
  * Centralization approach
    * Polkadot (single validator set)
    * Ethereum (strategy of heterogenous rollup secured partially by same validator set operating under a coherent incentive framework)
* Heterogeneous
  * Allows each shard to be different blockchains (e.g. ETH, etc)
  * Communication properties (such as datagram latency and semantic range), security properties (e.g. costs for reversion, corruption, stalling and censorship) and economic properties (e.g. cost of accepting and processing incoming messages or transactions)
* In-Core Execution
  * Operations inside a core. Abundance, scalable, as secure as on-chain execution through crypto-economics and ELVES.
* JAM
  * JAM is 4th generation blockchain design (1:05 https://x.com/paritytech/status/1788847203622043974)
  * JAM protocol lifespan is only designed to last until mid-August of the year 2840 https://x.com/XiliangChen/status/1788898490825011646
  * JAM vs AWS but the cloud is not trustless and held in consensus (1:05:45 https://x.com/paritytech/status/1788847203622043974) so most comparable would be Golem (AWS powered by blockchain but it wasn't held in consensus where state remains persistent)
  * Polkadot was always been sharded, and fully heterogenous. Now, it will be sharded, heterogenous, AND the boundaries of those shards can be determined flexibly, what @gavofyork is referring to as semi-coherent system https://x.com/kianenigma/status/1790763921600606259 
  * Upgrade stages:
    * Separate Polkadot update to "Userland" layer by the ["Minimal Relay RFC"](https://github.com/polkadot-fellows/RFCs/blob/main/text/0032-minimal-relay.md) that would migrate to "Userland" layer into System Parachain(s) that would be called "CoreChains Service" the DOT token, its transferability, staking, governance, etc.
    * Drop-in replacement only of the "Kernel" layer of the Polkadot Relay Chain with JAM to make it more general purpose
      * Move Parachains Protocol from "Kernel" layer to "CoreChains Service" of the "Userland" layer
        * Note: "Kernel" layer currently comprises Parachains Protocol (rigid way to use Cores), DOT token, its transferability, staking, governance, etc.
    * Note: Unchanged is the "Cores" layer in stack that represents the "hardware", which provides the computation and DA
    * Applications would then written on top of the same Cores "Hardware" layer and "Kernel" layer (JAM)
  * JAM is fully compatible with Polkadot since the main product of Polkadot is Parachains in the agile-coretime fashion, and this product persists in JAM.
  * New protocol heavily inspired by and fully compatible with Polkadot
  * Builds on top of Polkadot 2, where Polkadot 2 makes deployment on L2s more flexible on cores
  * Aims to replace the Polkadot Relay Chain so its usage is un-opinionated
  * Increase accessibility of Polkadot Cores in more flexible and un-opinionated ways than Agile Coretime
  * Allow deploying any application on Polkadot Cores, even those not resembling a Blockchain (State-Transition Function STF) or L2
  * Exposes all the main 3 primitives to programmers including:
    * On-Chain
    * In-Core
    * Data Availability (DA) Layer
  * Allows both In-Core and On-Chain work to be fully programmable.
  * Allows arbitrary data to be read-from and written-into the Polkadot Data Availability (DA) layer.
  * Renaming of terminology to encapsulate use-cases beyond Blockchain/L2
    * Renames L2/Parachain -> Service
    * Renames Block/Transaction -> Work Packages or Work Items
  * Work Package is a group of Work Items
  * Work Packages belong to a Service
  * Work Items may specify exactly what code they execute In-Core, On-Chain, and if/how/what they read and write to/from the Distributed Data Lake.
  * JAM brings the ability to not have to have "Persisent Partitioning"
    * Scaling Out (Horizontal) is still required to parallelise, divide and conquer, separate workloads into different partitions and processes, but avoids those partitions having to be persistent, since only partition for an instant of time until it all returns back into the Distributed Data Lake of state, where different partitioning may occur at the next instant of time
  * JAM Analogy
      * JAM facilitates the thousand-person workforce single company analogy of how large companies really work, where they hire say a thousand people and have them work together, and split up the work they need to do as needed in an efficient way to bring new use-cases and opportunities. https://x.com/paritytech/status/1788847203622043974
      * Polkadot Analogy would be similar to if that company were to instead subcontract to a thousand smaller companies one-person companies that each have a high cost to interact with each other, since they would each need to handle contracts with other companies, and where you have company walls, and where employees those subcontracted companies stay an employee of that company, and where all the information about each of those companies resides with it and not anyone else. https://x.com/paritytech/status/1788847203622043974
        * e.g. A construction company would have a limited amount of things it could do if it was structured with a thousand single-person subcontracted companies and the skyscraper had to be partitioned with divide and conquer into a thousand separate work contracts for each company to be paid, where each company would bring its **interaction costs** including resources such as project manager, scheduler, payroll manager, legal and finance
        * e.g. In Ethereum with flashloans its possible to access all the state, ut in Polkadot you have to access different parachains using XCM and associated cross-chain tx fees that is asynchronous. JAM would open the possibilities of what could be deployed on the same machinery (more and new usage paradigms like "Synchronous Composability"), allowing different specialised parachains to synchronously interact with each other. It will unlock things like Accords/SPREE (smart contracts that govern inter-parachain behaviour), including a Tokens Accord with bilateral agreements to prevent parachains minting more tokens than they are meant to and can share tokens without having to go via some native chain or Asset Hub, and where Accords may help with XCM multiparty computation offering multilateral agreements for where more than two chains that are self-sovereign are involved (e.g. pay in currency on Chain A, transaction uses an NFT that sits on another chain, and sender and destination chain are different)
* JAM 2
  * Sequencing is not done by JAM as it is handled in the layer above, but the way it is designed will impact MEV opportunities https://x.com/paritytech/status/1788847203622043974
  * Current Cumulus and parachains does not provide any way for parachains to use "Synchronous Composability"
* JAM Service
  * Permissionless, powerful, and scalable on-chain smart contracts, where one of them will be the "Parachains Service" that will largely replicate the Polkadot's current functionality and will be targetable by Cumulus. Power cycle of decommissioning Polkadot relay chain and commissioning JAM chain then the parachains will start finalising blocks again.
  * Described by 3 Entry Points
    * `fn refine()` is Join (from JAM), which describes what the Service does In-Core.
      * When all Polkadot Cores work all in parallel, for different services, Join is when data is distilled into a smaller subset, then passed to the next stage, Accumulate, where the result of all the aforementioned are accumulated into main JAM state, which is the On-Chain Execution part
    * `fn accumulate()` that describes what the service does On-Chain
    * e.g.
      * zkSNARK Service with minimal compute to do but high storage required, so this zkSNARK Service could be paired with a compute-intense but storage-light service and share one core at a time
      * Storage Service like Filecoin that offers proofs for storage of data
* On-Chain Execution
  * Operations of all validators. Secured by default through economically secured validators. More expensive and constrained, as everyone is executing everything.
* Polkadot
  * Architecture of sharding with many blockchains (AppChains, domain-specific chains) running on WebAssembly that are able to handle specific tasks was the next step after Ethereum and was to be a hefty computation engine that can process a lot of transactions and works with consensus, and became resilient and secure. The Polkadot relay chain has been well architected, resilient, and has bootstrapped the security. Polkadot took the shape of a scalable heterogenous multichain, where it is a blockchain that delivers a lot of blockspace
* Proof of Validity (PoV) is the L2's state proof in Polkadot 1.0
* Parachain Validation Function (PVF) is the combination of the state proof and the parachain block

* SPREE - technical proposal that would utilize Polkadot's unique shared-security and improve composability, though blockchains would still remain isolated.

* Zk-SNARKs
  * Constraints - Trade-off between the proof's size, verification complexity and the computational complexity of generating it. Non-trivial computation, and especially the sort of general-purpose computation laden with binary manipulation which makes smart-contracts so appealing, is hard to fit into the model of snarks

## Tools <a id="tools"></a>

* FluffyLabs https://fluffylabs.dev/
* JAM Testnet https://github.com/jam-duna/jamtestnet

## References <a id="references"></a>

* JAM article Kian's Garden https://x.com/kianenigma/status/1812789950741381567
  * [X] Summarised here
* Gavin Wood on JAM: The next disruptor in Web3 https://x.com/paritytech/status/1788847203622043974
  * [X] Summarised here
* JAM Diagrams by @xlc https://hackmd.io/0gSmXyElT2iKawymS5jJKw?view
  * [X] Questions asked here
* JAM Polkadot Wiki
  * https://wiki.polkadot.network/docs/learn-jam-chain
  * https://wiki.polkadot.network/docs/learn-jam-faq
  * [ ] Summarised here
* Graypaper
  * https://graypaper.com/graypaper.pdf
* [ ] jam4s.org/blog
  * Appendix A.1. - Program Blob Structure
  * Appendix E of Graypaper - Merkle Tree Structure, Patricia Trie, MMR, Node Operator, Trace Operator (Merkle Proof)
* sub0 RESET Bangkok
  * [ ] sub0 RESET Day 3 JAM presentation by Kian
  * [ ] sub0 RESET Day 3 Jamixir in Elixir
