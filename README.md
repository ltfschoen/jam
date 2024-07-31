# JAM Implementers Prize

### Table of Contents:

* [Polkadot vs JAM](#polkadot-vs-jam)
* [Glossary](#glossary)
* [References](#references)

## Polkadot vs JAM <a id="polkadot-vs-jam"></a>

* Polkadot level
	* Polkadot ingests one L0 (relay chain) block and one L1 (parachain) block per one L2 Core, per time-slot
	* JAM State is On-Chain Operation (Polkadot's main block and STF) and interacts with Accumulate (uses PVM)
	* Accumulate (uses PVM) is On-Chain Operation and gathers data from a Distributed Data Lake
	* Operations
		* Normal Execution
			* Blockchain (is a State-Transition Function STF)
			* On-Chain Operation (typical blockchain computation) where Main blockchain state updated as outcome of validators executing work
	* Synchronous (Coherent) vs Asychronous (Incoherent) Systems
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

* Parachains level
	* L1 (parachains/shards) and L2 (cores) are all entirely different applications, hence why Polkadot is a heterogeneous sharded blockchain
		* Each stores its blockchain definition as bytecode in the state of the blockchain itself
			* Polkadot 1.0 used WASM for bytecode
			* JAM uses PVM/RISC-V as its VM of choice

* Cores level
	* Data Availability (DA) Layer (In-Core)
		* Distributed Data Lakes are In-Core and interact with multiple Cores (uses PVM)
		* Cores (uses PVM) each are In-Core and `refine` (aka Join) Work Items from their each of their Work Packages
		* L2s automatically use Polkadot's native Data Availability (DA) Layer to keep their execution evidence available for a period of time, where that execution evidence posted that is needed to re-execute an L2 block is a blob that is fixed
		* L1s (parachains/shards) code never reads from the L2s Data Availability (DA) Layer  
		* Core is an environement where a subgroup of validators work in coordination to re-execute a single L2 (parachain/shard) block
			* In-Core Execution (novel blockchain computation) where Core receives Data (L2 block + subset of L2 state) that is gossipped to the subgroup of validators (constituting the core) and used to execute L2 block then perform consensus tasks
			* Core validators share Data needed for re-execution so it is available to other validators (outside the core)
				* Other Core validators (outside the original core) may decide to re-execute the L2 block based on ELVES rules
				* Polkadot L0 (relay chain) main state is updated and Polkadot block provided with a small commitment of the L2 latest state that is executed by all Polkadot validators

* CorePlay
	* New model for programming smart contracts

## Glossary <a id="glossary"></a>

* JAM
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
* Service
  * Described by 3 Entry Points
    * `fn refine()` is Join (from JAM), which describes what the Service does In-Core.
      * When all Polkadot Cores work all in parallel, for different services, Join is when data is distilled into a smaller subset, then passed to the next stage, Accumulate, where the result of all the aforementioned are accumulated into main JAM state, which is the On-Chain Execution part
    * `fn accumulate()` that describes what the service does On-
* Blockchain
  * State-Transition Function (STF_)
* In-Core Execution
  * Operations inside a core. Abundance, scalable, as secure as on-chain execution through crypto-economics and ELVES.
* On-Chain Execution
  * Operations of all validators. Secured by default through economically secured validators. More expensive and constrained, as everyone is executing everything.
* Data Availability (DA)
  * Ability for Polkadot validators to commit to having some data available for some period of time, and providing it to other validators

## References <a id="references"></a>

* https://x.com/kianenigma/status/1812789950741381567
