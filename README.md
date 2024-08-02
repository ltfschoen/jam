# JAM Implementers Prize

### Table of Contents:

* [Polkadot vs JAM](#polkadot-vs-jam)
* [Glossary](#glossary)
* [References](#references)

## Polkadot vs JAM <a id="polkadot-vs-jam"></a>

* Userland level (i.e. Parachains)
	* L1 (parachains/shards) and L2 (cores) are all entirely different applications, hence why Polkadot is a heterogeneous sharded blockchain
		* Each stores its blockchain definition as bytecode in the state of the blockchain itself
			* Polkadot 1.0 used WASM for bytecode
			* JAM uses PVM/RISC-V as its VM of choice

  * **CoreChains Service**
    * First service deployed in JAM will likely be called CoreChains Service (or Parachains Service) and support allowing existing Polkadot-2-style parachains to be executed on JAM.

  * **Further Services**
    * Additional services may be deployed on JAM that communicate with the existing **CoreChains Service**

* Kernel level
	* Polkadot ingests one L0 (relay chain) block and one L1 (parachain) block per one L2 Core, per time-slot
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

  * PVM
    * Efficient metering
    * Ability to pause and resume execution, which is important for CorePlay

* Hardware level (i.e. Cores)
	* Data Availability (DA) Layer (In-Core)
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

## Glossary <a id="glossary"></a>

* Blockchain
  * State-Transition Function (STF_)
* Data Availability (DA)
  * Ability for Polkadot validators to commit to having some data available for some period of time, and providing it to other validators
* In-Core Execution
  * Operations inside a core. Abundance, scalable, as secure as on-chain execution through crypto-economics and ELVES.
* JAM
  * Upgrade stages:
    * Separate Polkadot update to "Userland" layer by the ["Minimal Relay RFC"](https://github.com/polkadot-fellows/RFCs/blob/main/text/0032-minimal-relay.md) that would migrate to "Userland" layer into System Parachain(s) that would be called "CoreChains Service" the DOT token, its transferability, staking, governance, etc.
    * Replace only the "Kernel" layer of the Polkadot Relay Chain with JAM to make it more general purpose
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
* On-Chain Execution
  * Operations of all validators. Secured by default through economically secured validators. More expensive and constrained, as everyone is executing everything.
* Proof of Validity (PoV) is the L2's state proof in Polkadot 1.0
* Parachain Validation Function (PVF) is the combination of the state proof and the parachain block
* Service
  * Described by 3 Entry Points
    * `fn refine()` is Join (from JAM), which describes what the Service does In-Core.
      * When all Polkadot Cores work all in parallel, for different services, Join is when data is distilled into a smaller subset, then passed to the next stage, Accumulate, where the result of all the aforementioned are accumulated into main JAM state, which is the On-Chain Execution part
    * `fn accumulate()` that describes what the service does On-Chain

## References <a id="references"></a>

* https://x.com/kianenigma/status/1812789950741381567
