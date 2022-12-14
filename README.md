# OverPass (Course Project in EE4032)

## Table of Contents

1. [Introduction](#section-1-introduction)
2. [Theoretical background](#section-2-Theoretical-background)
3. [Prerequisites](#section-3-prerequisites)
4. [How to interact](#section-4-how-to-interact)
5. [Remarks](#section-5-remarks) <br>
   5.1 [Installing Python3 Environment](#section-5.1-phython-environment) <br>
   5.2 [Installing Node16](#section-5.2-node16) <br>
   5.3. [Installing Virtual Environment](#section-5.3-virtual-environment) <br>
6. [Acknowledgement](#section-6-acknowledgement)
7. [Competitors](#section-7-competitors)

## Section 1: Introduction <a name="section-1-introduction"></a>

Smart contracts provide trust computing. Once a smart contract has been engaged, it will complete exactly how it was coded and no parties can interfere or change the result. However, the computation is usually expensive and slow. There exists a set of problems where there is signs showing that verifying answers is cheaper than searching for the answer. Hence well-defined incentive mechanism can be applied to delegate searching computation to untrusted computational power (off-chain server) while the completeness and soundness of the computation is still guaranteed:

-  User delegate task to the smart contract with an reward and life-time limit which contains a trusted verification function for incoming advices
- Some untrusted machines (Miners) would listen to the task event and provide answers/advice with proof so it can be verified by the smart contract efficiently. 
- Smart contract verifies the answer:
  - answer is correct AND more optimal than the exsisting best answer: save as the best answer so far
  - otherwise: reject
- Once life-time is past, the best answer can be retrieved by the User and the provider of the best answer so far can retrieved the incentive. 

Our project can optimize smart contracts by providing cheaper gas consumption for complex computation problems combining the pros of both EVM and un-trusted computational power as shown below:

|     Function      |      Pros      |    Cons    |
| :---------------: | :------------: | :--------: |
| local conputation | fast and cheap | un-trusted |
|  smart contract   |    trusted     |    slow    |

An overview of the overpass is as follows:

![Sequence Diagram](http://www.plantuml.com/plantuml/proxy?cache=no&src=https://raw.githubusercontent.com/EE4032Project/OverPass/main/image/overpass_overview.puml)

Un-trusted miners provide computation power to OverPass and the correctness of execution is guaranteed by the verification algorithm on OverPass(smart contract). The detailed theoretical background would be introduced in section 2.

## Section 2: Theoretical background <a name="section-1-Theoretical-background"></a>

The theory of computation is shaped by the Interactive Proof (IP) system [\[Goldwasser, Micali & Rockoff, 1989\]](https://people.csail.mit.edu/silvio/Selected%20Scientific%20Papers/Proof%20Systems/The_Knowledge_Complexity_Of_Interactive_Proof_Systems.pdf), where a strong, possibly malicious, prover interact with a weak verifier, and at the end of the computation, the client can output answer achiving [completeness](<https://en.wikipedia.org/wiki/Completeness_(cryptography)>) and [soundness](https://en.wikipedia.org/wiki/Zero-knowledge_proof#:~:text=Completeness%3A%20if%20the%20statement%20is,except%20with%20some%20small%20probability.). It's proved that [IP=PSPACE](<https://en.wikipedia.org/wiki/IP_(complexity)>) and hence many programs running on Turing machine have a polynomial interactive proof scheme. There are signs that many problems has cheaper verification algorithm than search algorithm(e.g., NP-complete, sorting). This triggered a novel idea on trust-worthy computation that not all work should be done on the trusted slow "computer", or verifier, as long as the un-trusted computational power can provide proof for the answer to the verifier. This would expand the computational power of consented computers (e.g. EVM) tremendously while the trust of the computation is maintained.  The class of problem we focus are problems with doubly efficient IP scheme [\[Goldwasser, Kala, & Rothblum\]](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/12/2008-DelegatingComputation.pdf)[\[Goldreich, 2017\]](https://www.wisdom.weizmann.ac.il/~oded/VO/de-ip.pdf) and some P problems with cheap verification system than existing search algorithm.

An incentive mechanism is made such that miners have the incentive to mine by executing the protocol honestly and the verifier has the incentive to participate and save gas fees from achieving the same goal. An example of the incentive mechanism is as follows:

| client \ advisor  |                         Honest                          |               Cheat                |
| :---------------: | :-----------------------------------------------------: | :--------------------------------: |
|   use OverPass    | (Answer_incentive - gas_verify-incentive, incentive - gas_verify) |         (0, - gas_verify)          |
| not user OverPass |           (Answer_incentive - gas_search, 0)            | (Answer_incentive - gas_search, 0) |

Given that gas for searching is much higher than gas for verifier, and a wide incentive is set by the client, the client use OverPass and Advisor be honest, is the Nash Equilibrium. Note that the mechanism would work under the assumption that the client is a rationale and at least one advisor is rationale and selfish, which is very robust.

This project introduces a standard for OverPass, see `overpass.sol` and also implements a demo project for the Longest Common Subsequence Problem(LCS) for two strings, which has O(mn) search algorithm and O(m+n) verification algorithm, where m and n are the lengths of the two strings respectively.

## Section 3: Prerequisites <a name="#section-2-prerequisites"></a>

### Mac Users:

1. Homebrew (Mac Users only)
2. Python 3 environment
3. Node 16
4. Ganache CLI

### Windows Users:

1. WSL and Ubuntu.
   Download WSL and Ubuntu by using the following site: https://ubuntu.com/tutorials/install-ubuntu-on-wsl2-on-windows-10#1-overview.
   Note that we recommend Ubuntu 20.04 LTS for the Ubuntu Version and it is suffiecient to follow until steps 4.
2. Latest version of Node Version Manager (nvm) and pick Node 16.
   Download nvm by using the following site: https://blog.devsharma.live/setting-up-nodejs-with-nvm-on-wsl-2
   After installing nvm, you can type the following command to install Node 16 and its relevant npm

```sh
nvm install 16
nvm use <node version>
```

3. Python 3 (Should be installed automatically for Ubuntu. We can check by typing: python3 --version)
4. Ganache CLI (See [Remarks](#section-4-remarks))
5. For opening new terminals, tap the windows menu, type ubuntu and enter

## Section 4: How to interact <a name="#section-4-how-to-interact"></a>

1. Clone the github repository and enter into the directory.

```sh
git clone https://github.com/EE4032Project/OverPass.git && cd OverPass
```

2. Even though it is optional, we recommend activating a virtual environment to
   localise all the installation of the python libraries into the folder. Refer to the
   remarks for instructions on how to activate a virtual environment.

3. Install the required python libraries

```sh

pip3 install -r requirements.txt

```

### Hands-On session

4. Open up 3 terminals. The first terminal (`Terminal 1`) will be used to simulate the server, the second terminal will be used to simulate the user (`Terminal 2`) and the third terminal will be used to simulate the miner (`Terminal 3`).

5. On `Terminal 1`, we will load the local ganache server using the following code. You need to specify the number of accounts that you would like to generate. For example, if I want to start 100 accounts, I will indicate 100.

```sh

cd testnet # Enter the testnet directory
sh start_gananche_testnet.sh 100 # Start up the ganache test net


```

Upon successful set up, you should see the following

```sh

Accounts and keys saved to ./keys.json

HD Wallet
==============
Mnemonic:
Base HD Path:

Default Gas Price
==============

BlockGas Limit
==============

Call Gas Limit
==============

ChainId
==============

RPC Listening on

```

6. To simulate the user, we would need to run the python file `demo.py` with the argument `LCSOverPass` on `Terminal 2`.

```sh
$ python3 demo.py LCSOverPass

```

Upon successful execution, you should see the following.

```sh

contract address: 0xe78A0F7E598Cc8b0Bb87894B0F60dD2a88d6a8Ab
times_to_delegate:

```

7. Before we indicate the number of times to delegate, we need to have a miner to listen to the contract address. Therefore, on `Terminal 3`, we can simulate the role of a miner by running the `demo.py` file with the `miner` argument.

```sh

$ python3 demo.py miner

```

Upon successful execution, you should see the following code.

```sh

Available orders:
1. listen <contract_address>
2. unlisten <contract_address>
3. min_incentive <min_incentive>
4. maximum_duration <maximum_duration>
5. get_incentive

```

8. Copy the contract address from `Terminal 2`. On `Terminal 3`, we run the following command to listen to the address block.

```sh

Order: listen 0xe78A0F7E598Cc8b0Bb87894B0F60dD2a88d6a8Ab # The address is from Terminal 2

```

Upon successful execution, we should receive the following code.

```sh

Start listening on address: 0xe78A0F7E598Cc8b0Bb87894B0F60dD2a88d6a8Ab
Available orders:
1. listen <contract_address>
2. unlisten <contract_address>
3. min_incentive <min_incentive>
4. maximum_duration <maximum_duration>
5. get_incentive


```

9. Return to `Terminal 2`. As the user, specify the number of times you would like to delegate the task.

```sh

contract address: 0xe78A0F7E598Cc8b0Bb87894B0F60dD2a88d6a8Ab
times_to_delegate: 1

```

After you press enter, you should see something like this on `Terminal 3`.

```sh

test case: 1
s1 len: 40
s2 len: 90
Gas used:  479193
LCSOverPass: Average Gas Used for 1 testcases is: 479193.0
Approximate Weis for one test one testcast is: 3.9773019e+16

```

On `Terminal 1`, you should see the following.

```sh

  Transaction: 0xc761a8adfcb9fc0650b6ed00d278b14c70b3ef0119ad2471eacff25223cdcf54
  Gas usage: 145104
  Block number: 3
  Block time: Thu Nov 10 2022 00:07:30 GMT+0800 (Singapore Standard Time)

```

Miners are incentivised to provide a more efficient solution. Supposed the solution is more effective than the current solution, they can execute `get_incentive` to get the rewards.

```sh

Order:get_incentive
totalIncentiveGot: 100000000000000000 # In wei
Available orders:
 1. listen <contract_address>
 2. unlisten <contract_address>
 3. min_incentive <min_incentive>
 4. maximum_duration <maximum_duration>
 5. get_incentive

```

## Section 5: Remarks<a name="section-4-remarks"></a>

### Section 5.1. Installing Python3 Environment <a name="section-5.1-phython-environment"></a>

If you do not have a Python3 environment, we recommend installing anaconda using the following command.

```sh
brew install --cask anaconda # Mac Environment: Installed using homebrew
```

### Section 5.2. Installing Node16<a name="section-5.2-node16"></a>

```sh
brew install node # Mac Environment: Installed using homebrew
```

### Section 5.3. Installing Virtual Environment<a name="section-5.3-virtual-environment"></a>

```sh

python3 -m venv venv # Create a virtual environment in the directory called venv
. venv/bin/activate # Activate the virtual environment. You should see a (venv) in terminal

```

If you encounter an issue where the `ganache-cli` is not found, execute the following command on
your terminal.

```sh

sudo npm install -g ganache # If you are installing npm

```

## Section 6: Acknowledgement<a name="section-6-acknowledgement"></a>

For production environments...

```sh

```

## Section 7: Competitor Products <a name="section-7-competitors"></a>

There are currently 2 competitor products, which are Solana and Arbitrum. First, Solana is a layer 1 blockchain that uses Proof of Stake (PoS) with Proof of History (PoH) to accelerate its average block validation times, which is around 10 times faster than Ethereum [\[Tyson, 2022\]](https://www.infoworld.com/article/3666736/solana-blockchain-and-the-proof-of-history.html). It also has a lower gas fee per transaction, which is around $0,0000014 per transaction [\[Vasile & Iulia, 2022\]](https://beincrypto.com/learn/solana-vs-ethereum/#:~:text=Ethereum%20has%20a%20higher%20transaction,0%2C0000014). Second, Arbitrum is a layer 2 blockchain that is built on top of Ethereum. It uses Interactive Fraud Proofs, which is a type of Interactive Proofing. The basic idea of Interactive Fraud Proofing is that both the prover and verifier will try to check/bisect each other answers until they disagree to one or more things. This will, then, be run on the main Ethereum network. On the other hand, our project will be focusing on Doubly Efficient Interactive Proofing and not Interactive Fraud Proofing. We can see section 2 (Theoretical Background) for the explanation of Doubly Efficient Interactive Proofing. <br>
