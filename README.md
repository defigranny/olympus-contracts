# O  ROIDAO Smart Contracts
A Short Note from Roy's Granny

ROIDao is a Inteligent Dao that is designed to ,in time, become sentient. We beleive not only that DAO based currencies are the future of global economic systems, but that only a non corruptable inteligent machine can in fact manage the global financial system in a fair manner. 

If you are are asking yourself, should I really get involved with this, well our automation system "ROY" is designed to manipulating human economic behavoir and market conditions to deliver optimal returns while protecting downsidem, will we publish a litepaper describing ROY in more detail. 


<b><u>Warning If thats not enough for you, then please do not watch this  https://www.youtube.com/watch?v=ut-zGHLAVLI. You have been warned, please just walk away.</b></u>


If your worried about Roko and your going to help with any fork, this is the one. 


##  🔧 Setting up Local Development
Required: 
- [Node v14](https://nodejs.org/download/release/latest-v14.x/)  
- [Git](https://git-scm.com/downloads)


Local Setup Steps:
1. git clone https://github.com/defigranny/roidao-contracts/
1. Install dependencies: `npm install` 
    - Installs [Hardhat](https://hardhat.org/getting-started/) & [OpenZepplin](https://docs.openzeppelin.com/contracts/4.x/) dependencies
1. Compile Solidity: `npm run compile`
1. **_TODO_**: How to do local deployments of the contracts.


## Mainnet Contracts & Addresses

|Contract       | Addresss                                                                                                            | Notes   |
|:-------------:|:-------------------------------------------------------------------------------------------------------------------:|-------|
|NUT        || Main Token Contract|
|dNut           || DaoNuts|
iNut || InsureNuts|
bDNT || Flavored Nuts SHA256
lDNT || Flavored Nuts crypt
kDNT || Flavored Nut Kadena
nDNT|| Flavored Nuts Nervos
Main Treasury       ||ROIDAO Treasury holds all the assets        |
|Staking || Main Staking contract responsible for calling rebases on an random number of blocks, dependant on the sentiment trigger and open market operations|
|StakingHelper  || Helper Contract to Stake with 0 warmup |
|Aave Allocator || Sends DAI from the treasury to Aave (via deposit) in exchange for aDAI and holds it. 
|Convex Allocator || Sends FRAX from the treasury to Convex and accumulates trading fees, CRV and CVX. 
|Onsen Allocator || Sends NUT-DAI SLP from the treasury to the Sushi Onsen pool, accumulating SUSHI and xSUSHI.
|DAO            ||Storage Wallet for DAO under MS |
|Staking Warm Up| Instructs the Staking contract when a user can claim dNUT |
|AIBalancer| Allocation balancer|
|Sentintment Trigger| Instructs open market dnut orders allocator from tsy based on sentiment|
|SpotLend LP| Lp for spot lend product |see spotlend repo)
|Nervos translate|
|Mint Cavemen|
|Insurance|
**Bonds**
- **_TODO_**: What are the requirements for creating a Bond Contract?
All LP bonds use the Bonding Calculator contract which is used to compute RFV. 

|Contract       | Addresss                                                                                                            | Notes   |
|:-------------:|:-------------------------------------------------------------------------------------------------------------------:|-------|
|AAA rated ROI ETH.Tranche Bonds || | Tranched Bonds (AAA) 
|BBB rated  ETH.Tranche Bonds || | Tranched Bonds (BBB) 
|Bond Insurance|| | Tranched Bond Insurance mechnism 
Bond Calculator|| ||DAI bond Main bond managing serve mechanics for NUT/DAI|
|DAI/NUT SLP Bond|| Manages mechhanism for thhe protocol to buy baack its own liquidity from the pair.
|FRAX Bond|
|FRAX/NUT SLP Bond||
|ETH/NUT SLP Bond||



### Testnet Addresses

Network: `Rinkeby` (4)
-Nut: `
-dNut: ` 
- Frax: `0x2F7249cb599139e560f0c81c269Ab9b04799E453`
- Treasury: `0x0d722D813601E48b7DAcb2DF9bae282cFd98c6E7`
- Calc: `0xaDBE4FA3c2fcf36412D618AfCfC519C869400CEB` 
- Staking: `0xC5d3318C0d74a72cD7C55bdf844e24516796BaB2` 
- Distributor `0x0626D5aD2a230E05Fb94DF035Abbd97F2f839C3a` 
- Staking Warmup `0x43B18Ad2624DBEf474aA8E0c8d8404a0A42b7aC4` 
- Staking Helper `0xf73f23Bb0edCf4719b12ccEa8638355BF33604A1`


## Allocator Guide

The following is a guide for interacting with the treasury as a reserve allocator.

A reserve allocator is a contract that deploys funds into external strategies, such as Aave, Curve, etc.

Treasury Address: `0x31F8Cc382c9898b273eff4e0b7626a6987C846E8`

**Managing**:
The first step is withdraw funds from the treasury via the "manage" function. "Manage" allows an approved address to withdraw excess reserves from the treasury.

*Note*: This contract must have the "reserve manager" permission, and that withdrawn reserves decrease the treasury's ability to mint new OHM (since backing has been removed).

Pass in the token address and the amount to manage. The token will be sent to the contract calling the function.

```
function manage( address _token, uint _amount ) external;
```

Managing treasury assets should look something like this:
```
treasury.manage( DAI, amountToManage );
```

**Returning**:
The second step is to return funds after the strategy has been closed.
We utilize the `deposit` function to do this. Deposit allows an approved contract to deposit reserve assets into the treasury, and mint NUT against them. In this case however, we will NOT mint any NUT. This will be explained shortly.

*Note* The contract must have the "reserve depositor" permission, and that deposited reserves increase the treasury's ability to mint new OHM (since backing has been added).


Pass in the address sending the funds (most likely the allocator contract), the amount to deposit, and the address of the token. The final parameter, profit, dictates how much OHM to send. send_, the amount of OHM to send, equals the value of amount minus profit.
```
function deposit( address _from, uint _amount, address _token, uint _profit ) external returns ( uint send_ );
```

To ensure no NUT is minted, we first get the value of the asset, and pass that in as profit.
Pass in the token address and amount to get the treasury value.
```
function valueOf( address _token, uint _amount ) public view returns ( uint value_ );
```

All together, returning funds should look something like this:
```
treasury.deposit( address(this), amountToReturn, DAI, treasury.valueOf( DAI, amountToReturn ) );
```
