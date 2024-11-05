# Navigators Season 2: Growth Grants Proposal

## NFT as an account (NFTAA)

The project seeks to leverage the unique properties of NFTs to function as proxy accounts, enhancing interoperability, security, and flexibility across various blockchain networks.

## Project Background

NFT as an Account (NFTAA) is an innovative project aimed at revolutionizing the use of NFTs within the blockchain ecosystem, particularly in Substrate and EVM-based networks.
The project seeks to leverage the unique properties of NFTs to function as proxy accounts, enhancing interoperability, security, and flexibility across various blockchain networks.
By enabling NFTs to hold assets and interact with other web3 components similarly to standard accounts, NFTAA introduces a novel approach to web3 interactions.

This concept has already been implemented as a prototype dApp on an Ethereum-like chain, with its details discussed in two academic publications.
The first is a [conference full paper](https://ieeexplore.ieee.org/document/10634390) presented at the [2024 IEEE International Conference on Blockchain and Cryptocurrency in Dublin](https://icbc2024.ieee-icbc.org/).
The second is a [Bachelor's thesis](https://opac.crzp.sk/?fn=detailBiblioForm&sid=2D2C61E4341E5366536A1A3BFA9A) where this concept was originally developed and first implemented using Solidity smart contracts on Moonriver (an EVM-compatible testnet Polkadot parachain).

Our motivation for creating this project stems from the desire to address the inherent limitations and inefficiencies associated with traditional accounts and the nature of how private keys work.
Issues such as the illiquidity of accounts and the complexities involved in transferring ownership pose significant challenges.
This innovative approach opens new avenues for business opportunities and applications in DeFi and enterprise blockchain solutions, underscoring our commitment to advancing web3 technology.

The project will be used in Romi's Master's thesis focused on ZK tech in web3.

## Proposal Overview

### Problem

Changing ownership of digital assets in web3 is not a problem at all.
You just need to create a transaction, sign it, and if everything checks out, the assets will be transferred from the source account to the target destination after the transaction is published to the network and processed.

However, the issue arises when we want to change the owner of an account.
This could be necessary for various reasons: the account may hold a large number of different assets, have a good transaction history, be verified somewhere, or be involved in different smart contracts.
Such a change of ownership, however, is not possible given the nature and architecture of the blockchain.

This is mainly due to the way asymmetric cryptography works, which is essentially the foundation of this whole concept.
The advantage here is that no third party or anyone else can manage the account’s assets or sign anything on its behalf.
This is a benefit compared to regular Web2 accounts, where typically a third party has access to the data alongside the user.

This is possible thanks to the principle of asymmetric cryptography, which is based on a pair of keys: a private key and a public key.
But what if we want such an account to change its owner?
Transferring ownership of the private key is an impractical solution, as the new owner has no way of ensuring or verifying that they are the only one who knows the corresponding private key at any given time.

In short, the issue is the illiquidity of accounts themselves and the inability to make a secure change of ownership when security is defined as access integrity.

### Solution

The term "non-fungible token" refers to a unique and distinct asset that can represent various physical items with varying values.
NFTs primarily operate on the asset side of the owner-asset relationship and lack the potential to own other assets.
In contrast, accounts are used to represent ownership of assets. In an owner-asset relationship, the account can be on either side.

By combining these two relations, it is possible for an account and an NFT to be represented by a single object that is uniquely identifiable and has the nature of an asset that can be owned—meaning its ownership can change—and at the same time can own other assets.
We present a non-fungible token as an account, or NFTAA for short.

Most of Mina is based on a single-account architecture consistent with the NFTAA philosophy and facilitates connection with existing infrastructure.
We can easily exploit existing infrastructure for smooth communication if NFTAAs are handled as conventional accounts.
Furthermore, if needed, NFTAAs can function as asset owners or in a multi-sig scheme.

Another key characteristic of NFTAA is the ownership possibilities and the transferability of this ownership to any other account, potentially other NFTAAs.
This means that the private keys used to alter a certain NFTAA will change. This is feasible thanks to the well-known proxy smart contracts.
Figure nftaa-sc-proxy shows the relationships between NFTAA and other logical entities.

<img src="https://raw.githubusercontent.com/decenzio/NFTAA-Polkadot/286414435e0fdd51810c1093688daf67df59d010/proposal/prox.png" alt="nftaa-sc-proxy" width="20%" height="20%">

Some related sources to proxy solution are [here](https://ethereum.org/en/developers/docs/smart-contracts/upgrading/) and [here](https://docs.openzeppelin.com/contracts/5.x/api/proxy).

### Impact

This proposal will significantly enhance the Mina ecosystem by providing innovative solutions in several key areas, driving adoption, improving tools, and attracting a diverse range of users and developers.

By introducing NFT-based accounts that can manage assets more efficiently, the NFTAA project makes the Mina ecosystem more appealing to a broad range of users, including developers, end-users, and enterprises.
The project's focus on secure and flexible account management aligns well with Mina's lightweight, privacy-preserving principles, enhancing user trust and making Mina a more attractive platform for blockchain activities.

The flexibility provided by NFT-based accounts for managing and delegating assets makes the Mina ecosystem more developer-friendly and user-centric.
Wallet developers can integrate NFTAA functionalities to offer superior user experiences, and dApp developers gain access to secure account management tools, ultimately attracting more creators and users to build and interact within the Mina network.

### Audience

- Builders and developers: Those building zero-knowledge smart contracts on Mina can leverage NFTAA to enhance their applications with secure and flexible account management features.
- End-users and enterprises: People who hold assets on the Mina network or are interested in it can benefit from NFTAA’s simplified and secure asset management. Businesses and organizations with significant blockchain holdings, or who wish to explore Mina's privacy and zk-SNARK capabilities, can use NFTAA for secure and efficient account management.
- Researchers and academics: Those exploring new web3 innovations can study and expand upon the unique concepts introduced by NFTAA.

## Architecture & Design

### Detailed Design/Architecture

First, we will need an NFT. Unlike Ethereum and other blockchains, Mina does not have a standard for NFTs like ERC721.
However, this does not mean that NFTs do not exist within the Mina ecosystem. Our plan is to use an [existing project](https://minanft.io) that deals with NFTs.
For the NFT contract, we will utilize the [Mina NFT library](https://github.com/dfstio/minanft-lib), and if necessary, we will communicate with its author about possible improvements or changes.

The second component is the proxy account. This is the more challenging part because currently, nothing like this exists in Mina.

The need for a proxy account arises from the requirement that if we want the NFT to act as an account, it needs an address to function as the owner of other assets.
Therefore, we will create a proxy smart contract that will serve as an address; however, the authorization to interact with this smart contract will be conditioned by the ownership of the specific NFT linked to it.

These proxy contracts can be created using the NFTAA Factory contract, which will enable users to create NFTAA entities.
Our current design does not account for backward compatibility, and minting NFT tokens will be atomic along with linking to the proxy account, meaning that both entities will contain mutual references to each other.

Since predicting identifiers for NFTs is simpler, the proxy account will be deployed first, and the identifier of the relevant NFT and collection will be embedded into the proxy account through its constructor.
Afterward, the NFT token will be minted, storing the address of the already created proxy account.

<img src="https://raw.githubusercontent.com/decenzio/NFTAA-Mina/refs/heads/main/proposal/images/nftaa-positive-flow-sq.png?token=GHSAT0AAAAAACYGL2KVIPCNVZEAEIIU4MGIZX22DHA" alt="nftaa-positive-flow-sq" width="70%" height="70%">

Figure nftaa-positive-flow-sq shows hight level sequence diagram of the key elements of the architecture.

### Vision

Our small team is a big fan of DeFi, and as part of our mission, we love to push the boundaries of what DeFi can do and strive to bring innovations to the world of web3.
NFTAA is one example of how we are fulfilling our mission.

Additionally, we are very interested in ZK technology and excited about what it offers and how it can be effectively utilized in the web3 space.
For this reason, we are particularly interested in Mina and appreciate the simplicity with which ZK technology can be used (we find o1js more user-friendly than Rust).
That’s why we want to bring part of our work to the Mina ecosystem.
This is our first project within the Mina environment, and upon its completion, we will consider continuing our journey in this ecosystem.

We would be delighted if NFTAA brings real value to users and inspires others to explore new ways of utilizing DeFi.
By demonstrating the potential of NFT-based accounts and ZK technology within the Mina network, we hope to encourage more developers, enthusiasts, and innovators to rethink how DeFi can work and what possibilities it can unlock for the web3 community.

### Existing Work
For our existing work, please check the Project Background section.

When talking about related work, there is a similar idea in the Ethereum ecosystem—[EIP-6551](https://eips.ethereum.org/EIPS/eip-6551).
In our [conference paper](https://arxiv.org/pdf/2404.14074#subsection.4.3), in the section "NFTAA vs TBA," we discuss the differences.

### Production Timeline
The project will be completed on the testnet at the end of January 2025.
After a few weeks in the testnet and implementation feedback from users, it will be pushed to production in Q1 2025.

##  Budget & milestones

### Deliverables

- zkApp (smart contracts) created with o1js
- Web application to interact with NFTAA, allowing users to create, manage, and transfer NFTAAs
- Documentation and user guide

### Mid-Point Milestones
Final working version of all necessary smart contracts:
- Completion of the NFT smart contract
- Development of the proxy account mechanism
- Integration of NFT and proxy accounts (NFTAA factory contract)
- Initial testing on the Mina testnet

### Project Timeline
3M

### Budget Requested
42 000 MINA

### Budget Breakdown
- Development
    - Smart contracts - 14,000 MINA
    - Web app - 10,000 MINA
- Testing and quality assurance: 5,000 MINA
- Documentation and user guide: 2,000 MINA
- Project coordination - 6,000 MINA
- Marketing - 2,000 MINA
- Support and future maintenance - 3,000 MINA

### Wallet Address
B62qp9RzjdeFwhfhqJ8MeVg8f6Ufpu9F5iV6ah8GpcpFPNaF2QEY4A6

## Team Info
Romi has 2 years of experience as a Java backend developer in the banking domain. He participated in the development of a mobile banking wallet app and an app for managing loan applications used by bank back-office workers. Additionally, Roman was involved in the development of a DeFi aggregator, an app focused on integrating multiple DeFi functionalities, such as swap, portfolio tracking, loan usage, etc., on the Ethereum network. Roman studies Blockchain technology and completed a bachelor's thesis on NFTAA, with an article about this published by IEEE (check the Development status section). Currently, Roman is pursuing a Master's degree at FIIT STU and working on ZKP research. His further learning activities include a three-year program focusing on leadership. Recently, Roman co-founded Decenzio, a blockchain and web3 oriented IT company.

Kiko is an experienced blockchain architect with a lot of projects behind him, e.g., gold-backed cryptocurrency, cross-chain interoperability API protocol, an EU-wide blockchain platform for debt financing of SMEs by issuing bonds, a web3 social network, etc. He holds PhD in Computer Science from the Slovak University of Technology, with his dissertation thesis focused on interoperability between heterogeneous blockchain networks. One of the partial challenges in the dissertation thesis was on scalability issues, where he studied Layer-2 techniques, especially with the use of Zero-knowledge proofs. In the last 3 years, he has been working on practical use cases of blockchain networks in different domains of computer science research. Mentionable are smart grids, e-voting platforms, and asset-sharing platforms with privacy preservation. He serves as a Slovak Representative for European blockchain services infrastructure within the European Blockchain Partnership (European Commission) and is also active in cooperating on policy-making documents regarding blockchain technologies. Besides that, he serves as leader of the Blockchain & FinTech research group and has authored 25 academic publications.

### Proposer Github
[Romi](https://github.com/Roman-24)

### Team Members
- Romi - lead developer, responsible for smart contract development, web app development and project coordination
- Kiko - supervisor and advisor, providing expertise in blockchain architecture and research

### Proposer Experience and Achievements

#### Romi
Backend developer, Full-stack development, web3 development (Ethereum, Polkadot), Teaching OOP
- IEEE publication and Bachelor's thesis focus on NFTAA topic (check Project Background section)
- [ZK research focus on EVM](https://drive.proton.me/urls/1MECE570A8#LOhUBh2nd5Ga)

#### Kiko
[More than 20 publications](https://www.researchgate.net/profile/Kristian-Kostal) and here are some project examples:
- [A pool-based liquidity protocol based on Polkadot](https://github.com/fiit-ba/pool-based-liquidity-protocol)
- [An oracle network for Solana blockchain](https://github.com/fiit-ba/omniscient-blockchain-oracle)
- [Tokengram social network for NFT holders](https://github.com/fiit-tp7-2023)


## Risks & Mitigations

- What risks or dependencies do you foresee with building this project?

  The risks are the technical difficulty of the project and my lack of experience in the Mina ecosystem.

- What are your mitigations, if any?

  To address this problem, I have a supervisor for this project, and I communicate closely with people from the Mina Foundation.
