# 1. Overview
## 1.1 Project Goal 
DREx aims to open up the renewables investment sector in the LatAm region by making it easier for investors to (1) invest in renewable energy assets and (2) interface with and track their renewable energy investments. 
* <ins>Aim 1</ins>: Onboard renewable IoT devices to the blockchain such that sensors provide an immutable and verifiable way of tracking energy usage, thus providing investors with transparent and digital ownership of their investments
* <ins>Aim 2</ins>: Use the revenue stream generated by investors to create a conservation fund and creation of I-REC certificates

## 1.2 Approach 
We will conduct a pilot study where a solar minigrid is constructed in Guayaquil, Ecuador. The minigrid sensors will be onboarded to the Robonomics blockchain in the Kusama ecosystem, and data will be passed via an [off-chain worker](https://docs.substrate.io/reference/how-to-guides/offchain-workers/) from Robonomics to the Moonriver blockchain periodically (currently once every day). Once sensor data has been passed to Moonriver, smart contracts will allocate an amount of DREx tokens (equivalent to kiloWatt-hours generated by the minigrid per day) to an NFT contract pool, which users only have access to with ownership of an associated NFT. 

![Figure: DREx pipeline overview](./imgs/overview_figure.png "*Figure 1*: DREx pipeline overview")

# 2.  Hackathon
### 2.1 Goal
The goal of this hackathon was to establish the foundations upon which we could construct a minimum viable product. As such, work was consolidated into five sectors:

1. Construction of a solar minigrid based out of Guayaquil, Ecuador
2. Onboarding solar minigrid sensor data to Robonomics blockchain
3. Cross-chain messaging between Robonomics and Moonriver 
4. Smart contract development for Moonriver
5. Front-end and back-end development 
6. Integration of a decentralized on-chain API via Subquery for Moonriver smart contracts

## 2.2 Implementation
We break down each of the above tasks into their technical components:

### *2.2.1. Construction of a solar minigrid based out of Guayaquil, Ecuador*
  A solar minigrid setup with the following technical specifications will be bought and installed:
  1. Two [Jinko solar PV panels](https://www.solarmaxstore.com/jinko-solar-450-watt-tiger-bifcial-mono-perc-solar-panel-clear-frame-white-backsheet-bow-156-half-cell.html) at 450Wp each 
  2. [Micro inverter WVC-600 240 Vac Output](https://www.amazon.com/MarsRock-Waterproof-Inverter-AC80-160V-Efficiency/dp/B075M8J35S)
  3. [SEM ONE - Pickdata 10A current transformer](https://www.pickdata.net/sites/default/files/Manual_SEM_One_V08-191218-EN.pdf)
  4. [Raspberry Pi 3](https://www.raspberrypi.com/products/raspberry-pi-3-model-b/) with industrial modbus pe2a

  and a deployment script which reads the voltage and ampere registers from the modbus map of the energy meter, SEM ONE will compute the accumulated energy at minute intervals.

### *2.2.2 Onboarding solar minigrid data to Robonomics blockchain*
Robonomics software will be flashed via a prepared image onto an Ubuntu operating system and installed on a Raspberry Pi such that sensor data can be detected and sent to Robonomics. A deployment script will be written to facilitate interaction with the sensor data.

### *2.2.3. Message passing between Robonomics and Moonriver via off-chain workers*
Robonomics will eventually open its HRMP channel to Moonriver, but for the moment, we will utilize an off-chain worker, hosted on a local development parachain, to relay data from Robonomics to Moonriver smart contracts. As a safety solution, we will implement a Chainlink oracle node to pass the data.

### *2.2.4. Smart contract development for Moonriver*
Smart contracts will be written in Solidity and then deployed to Moonriver. The smart contract logic is as follows: once receiving the amount of tokens passed from Robonomics, the smart contracts deployed on Moonriver will assign NFTs their proportional share of tokens i.e. number of DREx tokens generated divided by the number of NFTs minted for a given project. 
 
### *2.2.5. Front-end and back-end development*
React.js with JSX, along with @polkadot/api, mapbox-gl, recharts, and scrollreveal libraries will be used to construct the user interface. A REST API will be created to allow the front-end to pull sensor data from the back-end off the Raspberry Pi. The back-end services will be hosted on AWS and implemented in Java. 

### *2.2.6. Subquery Integration*
We will write our own manifest file for decentralized indexing of our smart contracts. 

## 2.3. Achievements
Our team was able to make major progress in all tasks, notably:
1. A solar minigrid was built in Guayaquil, Ecuador
2. Minigrid sensor data was onboarded successfully to Robonomics 
3. A Chainlink oracle was used to facilitate data transfer of Robonomics sensor data to Moonriver, after exploring other potential cross-chain messaging solutions e.g. off-chain workers
4. Solidity smart contracts written to facilitate NFT minting and DREx token generation and claim logic
5. Front-end and back-end development concluded to produce working user interface, connected to Metamask wallet
6. Subquery decentralized API integration for transaction processing

## 2.4. Challenges 
The major challenge that we faced during this hackathon was the difficulty of getting our sensor data from Robonomics to Moonriver. As an HRMP channel does not yet exist, we initially explored an idea by which we could transfer sensor data tokens via XCM from Robonomics to Statemine, and then utilize a scheduler palette on Statemine to again via XCM send the sensor data tokens to Moonriver. We discovered quickly that Statemine has no existing scheduler palette, and decided to integrate an off-chain worker. While this was successful on a locally hosted blockchain, we experienced configuration difficulties when attempting to adapt the development environment to a Cumulus parachain. With no other way of getting our sensor data from Robonomics to Moonriver, we set up a Chainlink node and accomodating oracle as a sensor data feed from Robonomics. While we recognize that this presents a centralized point of failure in our pipeline, for the remaining duration of this hackathon, we saw no other alternative.

In integrating Subquery for our Moonriver smart contracts, we were able to set up an API for transaction data, but ran into issues when trying to index minted NFTs for their metadata, as the documentation here was lacking. 

## 2.5. Future Work
With Polkadot Proposal [#119](https://moonriver.polkassembly.network/referendum/119) likely to pass in several days, we are optimistic that we will soon be able to easily integrate the initial vision of a decentralized end-to-end electricity tokenization pipeline, outlined above. The opening of a bidirectional HRMP channel between Robonomics and Moonriver should greatly expedite development and make our approach much more straightforward.

It is not lost on us that historical data access will also be important in the future. As such, we plan to integrate a decentralized data storage provider so that users can reference historical energy data. Currently we are considering partnerships with Robonomics group [Merklebot](https://merklebot.com/) as well as [Source Network](https://source.network/), a decentralized database solution. 

We have also plans to port our Moonriver NFTs to RMRK, as their framework will allow us to easily transfer NFTs cross-chain once XCMv3 is enabled. 

# 3. Installation
## Directories
* <code>[./robonomics_onboarding](./robonomics_onboarding)</code>: Code for grid sensor onbaording and minigrid sensor deployment scripts
* <code>[./asset_transfer](./asset_transfer)</code>: Code for explored approaches for asset transfer with XCM, OCW, oracles 
* <code>[./moonriver_dApp](./moonriver_dApp)</code>: Moonriver smart contract solidity files as well as deployment code
* <code>[./front_end](./front_end)</code>: Code for UI/UX deployment, connected via REST API to back-end
* <code>[./back_end](./back_end)</code>: Code for back-end deployment
* <code>[./subquery_integration](./subquery_integration)</code>: Subquery decentralized API feed
* <code>[./imgs](./imgs)</code>: Reference images

## Getting Started
Please refer to the READMEs in each directory for installation and setup.

## Deployment Details
* 0xd4d9ac4ecf5ddf18056ce6a91d0a8e7b0c910cce : Chainlink oracle
* 0x7625dAd074438ae964107FE927cbdbCE8c6355c8 : Chainlink node
* 0x7736ae714f3c53029e7f9ac0f9E1143f8883378e : [AdvancedCollectible.sol](./moonriver_dApp/contracts/AdvancedCollectible.sol)
* 0x466242d0E92f2924383a8464356334ABfA9655af : [Our_ERC20.sol](./moonriver_dApp/contracts/Our_ERC20.sol) 

All contracts are deployed on Moonbase Alpha. 

## Support
For any questions contact us at drexallnet@gmail.com.

## License
See the [LICENSE](./LICENSE) file for license rights and limitations (Apache License).
