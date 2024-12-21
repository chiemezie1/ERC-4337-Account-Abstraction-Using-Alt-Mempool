# Abstraction Using EIP-4337

In 2021, Stefan Thomas lost access to $220 million worth of Bitcoin because he couldn’t remember the password to his hardware wallet. This incident highlights a common vulnerability with traditional accounts that rely entirely on private keys. Ethereum accounts, known as Externally Owned Accounts (EOAs), are one such example. These accounts depend entirely on a private key to sign transactions. If you lose access to your private key, your funds and assets become irretrievably locked. This lack of flexibility in account management is a significant barrier to the mass adoption of blockchain technology.

Consider another problem, assuming a user (Alex) wants to buy a token, send some of it to another wallet, swap half of it for another asset, then supply liquidity on a decentralized market, and finally stake the remaining asset to earn interest. Each of these steps requires gas fees and individual transaction approvals. The total gas fee paid, the time required to sign each transaction, and the process of completing all these transactions can be overwhelming to the user.

***What if there is a solution to solve these problems?*** Account abstraction — EIP 4337 — offers a solution. It promises to make it **easier,** **safer,** and more **flexible** to handle user accounts and transactions on Ethereum and the blockchain in general. In this post, I offer a comprehensive overview of Ethereum account abstraction. It is divided into two parts, **Part 1** focuses attention on the fundamental concepts of Account Abstraction, while in **Part 2**, we look at a basic implementation of account abstraction using the Ethereum Foundation example.

---

### PART 1: What is Account Abstraction (AA)?

Account Abstraction (AA) is a concept that improves how blockchain accounts work by combining Externally Owned Accounts (EOAs) and Smart Contracts, resulting in what we now call Smart Contract Accounts (SCAs).

With traditional EOAs, private keys are the only way to access and authorize transactions. If you lose the private key, you lose access to your funds permanently. SCAs, on the other hand, hold assets in smart contracts that may be programmed with code to control how the account functions. This could include multi-signature approval, recovery procedures, and transaction automation.

By abstracting transaction management to a higher layer (smart contracts), SCAs make blockchain systems more flexible ****and resilient.

### EIP-4337: The Engine of Account Abstraction

Ethereum Improvement Proposal (EIP) 4337 enables account abstraction without **modifying Ethereum’s consensus layer** (or Ethereum protocol). Instead, it introduces a new transaction format, **UserOperation**, and a core smart contract, **EntryPoint**, to handle transaction validation and execution. For a better understanding of ERC-4337-compliant smart contracts, let’s break down the main components that play a role in the system:

### [***UserOperations***](https://eips.ethereum.org/EIPS/eip-4337#useroperation)

A **UserOperation** is a struct that describes a transaction to be sent on behalf of a user. This could include transferring funds, interacting with smart contracts, or initiating a social recovery process. It is similar to Ethereum’s transaction objects but includes additional fields specific to ERC-4337’s design. Fields such as `nonce` and `signature` are unique to each user's smart contract account (SCA), ensuring that transactions can be uniquely identified and validated. Users send **UserOperation** objects to the user operation mempool (a new mempool alternative called the alternative mempool). A specialized class of actors, known as bundlers, monitors the user operation mempool and creates bundle transactions.

### ***Bundlers***

Bundlers take multiple `UserOperation` objects from different users, combine them into a single transaction, and submit it to Ethereum through the EntryPoint contract.Before accepting a UserOperation, bundlers use the EntryPoint contract’s `simulateValidation` method to run a simulation.

This simulation checks:

- Whether the operation is valid.
- The accuracy of the maximum gas cost for the operation.
- Whether the account has a sufficient deposit in the EntryPoint.

If the simulation fails (e.g., due to an invalid signature or insufficient funds), the bundler **rejects the UserOperation** and excludes it from the mempool. The `UserOperation` is not allowed to access any information that changes between operation and execution, such as current block time, hash, or number.

Bundlers are rewarded with transaction fees, incentivizing them to stay active. Bundlers must carefully select which **UserOperations** to bundle, ensuring that the **UserOperations** do not fail due to the bundling of the transactions. At the same time, they must optimize for their profitability while ensuring the Ethereum network processes them efficiently.

### ***EntryPoint***

The **EntryPoint** contract is a core component of the ERC-4337 system. It handles both **Validation** and **Execution** of `UserOperation` objects. Once validated, it forwards the `UserOperation` to the user’s smart contract wallet (SCA) for execution.

**Key Functions:**

1. ***Execution**:* After bundlers submit `UserOperations`, the EntryPoint unpacks and executes them. If an operation fails, it can roll back all changes to maintain transaction integrity.
2. ***Validation**:* It checks the validity of each `UserOperation`, including signature validation and other authenticity checks.
3. ***Forwarding to Smart Contract**:* Once validated, the `UserOperation` is forwarded to the user's smart contract wallet (SCA) for execution.

Here is the EntryPoint Workflow:

**For verification Loop**: For each `UserOperation`, the EntryPoint:

1. Creates the account if it doesn’t exist, using the `initcode` in the operation.
2. Calculates the maximum fee based on gas limits and current gas prices.
3. Ensures the account has a sufficient deposit to cover the fee.
4. Calls `validateUserOp` on the account to verify the operation’s signature and validity.
5. Verifies the account’s deposit in the EntryPoint is sufficient for both validation and execution.

**For execution Loop**: Once verified, the EntryPoint:

1. Executes the operation’s calldata by calling the user’s account, which processes it as needed.
2. If the calldata starts with `IAccountExecute.executeUserOp`, the EntryPoint encodes and sends `executeUserOp(userOp, userOpHash)` to the account.
3. After execution, any unused gas is refunded.

### ***Paymaster***

Paymaster contracts allow the abstraction of gas, enabling a contract that is not the sender of the transaction to pay for the transaction fees. This is an optional feature in ERC-4337 and is useful when a third party (e.g., a dApp or service) wants to cover the user’s gas fees. This eliminates the need for users to have ETH in their wallet to pay for transaction fees, improving the overall user experience.

### ***Aggregators***

Aggregators are responsible for validating and processing signatures for multiple UserOperations in a batch. They allow multiple UserOperations to be bundled together, making the execution of multiple transactions more efficient and cost-effective.

!https://cdn-images-1.medium.com/max/880/1*UH1zBX_U863IRvwv7E34Kg.png

The image above shows, in a pictorial format, the execution process. Starting from the top right, the user submits a **UserOperation** to the mempool. The bundler collects, validates, and executes it, then submits it for miners to include in a block.

### Why Does Account Abstraction Matter?

Account Abstraction (AA) helps make the blockchain world more ***user-friendly***, ***secure***, and ***flexible***. Here’s why:

1. **Simplifying Blockchain for Everyone:** Setting up a crypto wallet and managing private keys can be nerve-wracking. One wrong move, and you could lose access to your funds forever, as mentioned at the outset. With Account Abstraction, wallets can include features like password recovery or multi-factor authentication. It’s like bringing the ease of traditional banking to crypto — something users are already familiar with — without sacrificing decentralization.
2. **Making Transactions Effortless:** Imagine Alex, who wants to perform about four transactions. For each step, he needs to give approval and pay a gas fee. Account Abstraction bundles all these actions into one, making the process smoother and faster for users.
3. **Solving the “Gas Fee” Problem:** Have you ever tried to send crypto or interact with a dApp, only to realize you don’t have enough ETH to pay the gas fee? It’s frustrating. Account Abstraction allows dApps or third parties to cover those fees on your behalf, so you can focus on what you’re doing instead of worrying about ETH balances.
4. **Improved Security:** While cryptographic signatures are secure, adding multi-signature functionality introduces another layer of security. This can also make accounts quantum-resistant as quantum computing continues to improve.

***Learn More***

1. **Official Documentation**: Dive deeper into the mechanics of EIP-4337 [here](https://eips.ethereum.org/EIPS/eip-4337).
2. **Implementation**: Explore the Ethereum Foundation’s implementation repository: [Account Abstraction GitHub](https://github.com/eth-infinitism/account-abstraction).

---

### PART 2: Implementing Account Abstraction

The Ethereum foundation has implemented a minimalistic example of an ERC-4337 compliant contract called [SimpleAccount.sol](https://github.com/eth-infinitism/account-abstraction/blob/develop/contracts/samples/SimpleAccount.sol). In this guide, we’ll modify the Ethereum Foundation implementation of an ERC-4337-compliant contract, deploy it, and interact with it to understand how account abstraction works.

### Setting Up the Development Environment

### **Prerequisites**

1. **Ethereum Wallet:** Install **MetaMask** to manage your wallet. Download it [here](https://metamask.io/download/).
2. **Node.js:** Install the latest recommended version of Node.js. For this project, we are using **v20.17.0**. To check your current version, run:

```
node -v
```

If your version is earlier than v20.17.0, it should work. Otherwise, install or update Node.js using [Node Version Manager (NVM)](https://github.com/nvm-sh/nvm#intro).

3. **Yarn:** Install the Yarn package manager for JavaScript by running:

```
npm install --global yarn
```

Ensure that your Yarn version is 1.22.22 or later by running

```
yarn --version
```

---

### **Cloning the Repository**

Clone the Ethereum Foundation’s implementation of **account abstraction**:

```
git clone https://github.com/eth-infinitism/account-abstraction.git
```

After cloning, navigate to the project directory and install its dependencies

```
cd account-abstraction

yarn install
```
---

### Configuring the Environment

Create a `.env` file in the project directory and replace the placeholders with your actual values:

```
INFURA_ID=your-infura-project-id
PRIVATE_KEY=your-private-key
ETHERSCAN_API_KEY=your-etherscan-api-key
SALT=0x90d8084deab30c2a37c45e8d47f49f2f7965183cb6990a98943ef94940681de9
```

- **`INFURA_ID`**: Create a project on [Infura](https://developer.metamask.io/) to obtain your API key.
- **`PRIVATE_KEY`**: Export your private key from MetaMask (you can find instructions for this in [MetaMask’s guide](https://support.metamask.io/managing-my-wallet/secret-recovery-phrase-and-private-keys/how-to-export-an-accounts-private-key))
- **`ETHERSCAN_API_KEY`** (optional): Sign up on [Etherscan](https://etherscan.io/) and get your API key for contract verification.
- **`SALT:`** Use a predefined salt value for deterministic deployment.

Install the necessary environment management library:

```
yarn add dotenv
```
---

### Deploying and Interacting with SimpleAccount.sol

### **SimpleAccount.sol**

The `SimpleAccount.sol` can be found at `contracts/samples` directory.

**`*contracts/samples/SimpleAccount.sol*`**

```
import "@openzeppelin/contracts/utils/cryptography/ECDSA.sol";
import "@openzeppelin/contracts/utils/cryptography/MessageHashUtils.sol";
import "@openzeppelin/contracts/proxy/utils/Initializable.sol";
import "@openzeppelin/contracts/proxy/utils/UUPSUpgradeable.sol";
import "../core/BaseAccount.sol";
import "../core/Helpers.sol";
import "./callback/TokenCallbackHandler.sol";

/**
  * minimal account.
  *  this is sample minimal account.
  *  has execute, eth handling methods
  *  has a single signer that can send requests through the entryPoint.
  */
contract SimpleAccount is BaseAccount, TokenCallbackHandler, UUPSUpgradeable, Initializable {
    address public owner;

    IEntryPoint private immutable _entryPoint;

    event SimpleAccountInitialized(IEntryPoint indexed entryPoint, address indexed owner);

    modifier onlyOwner() {
        _onlyOwner();
        _;
    }

    /// @inheritdoc BaseAccount
    function entryPoint() public view virtual override returns (IEntryPoint) {
        return _entryPoint;
    }

    // solhint-disable-next-line no-empty-blocks
    receive() external payable {}

    constructor(IEntryPoint anEntryPoint) {
        _entryPoint = anEntryPoint;
        _disableInitializers();
    }

    function _onlyOwner() internal view {
        //directly from EOA owner, or through the account itself (which gets redirected through execute())
        require(msg.sender == owner || msg.sender == address(this), "only owner");
    }

    /**
     * execute a transaction (called directly from owner, or by entryPoint)
     * @param dest destination address to call
     * @param value the value to pass in this call
     * @param func the calldata to pass in this call
     */
    function execute(address dest, uint256 value, bytes calldata func) external {
        _requireFromEntryPointOrOwner();
        _call(dest, value, func);
    }

    /**
     * execute a sequence of transactions
     * @dev to reduce gas consumption for trivial case (no value), use a zero-length array to mean zero value
     * @param dest an array of destination addresses
     * @param value an array of values to pass to each call. can be zero-length for no-value calls
     * @param func an array of calldata to pass to each call
     */
    function executeBatch(address[] calldata dest, uint256[] calldata value, bytes[] calldata func) external {
        _requireFromEntryPointOrOwner();
        require(dest.length == func.length && (value.length == 0 || value.length == func.length), "wrong array lengths");
        if (value.length == 0) {
            for (uint256 i = 0; i < dest.length; i++) {
                _call(dest[i], 0, func[i]);
            }
        } else {
            for (uint256 i = 0; i < dest.length; i++) {
                _call(dest[i], value[i], func[i]);
            }
        }
    }

    /**
     * @dev The _entryPoint member is immutable, to reduce gas consumption.  To upgrade EntryPoint,
     * a new implementation of SimpleAccount must be deployed with the new EntryPoint address, then upgrading
      * the implementation by calling `upgradeTo()`
      * @param anOwner the owner (signer) of this account
     */
    function initialize(address anOwner) public virtual initializer {
        _initialize(anOwner);
    }

    function _initialize(address anOwner) internal virtual {
        owner = anOwner;
        emit SimpleAccountInitialized(_entryPoint, owner);
    }

    // Require the function call went through EntryPoint or owner
    function _requireFromEntryPointOrOwner() internal view {
        require(msg.sender == address(entryPoint()) || msg.sender == owner, "account: not Owner or EntryPoint");
    }

    /// implement template method of BaseAccount
    function _validateSignature(PackedUserOperation calldata userOp, bytes32 userOpHash)
    internal override virtual returns (uint256 validationData) {
        bytes32 hash = MessageHashUtils.toEthSignedMessageHash(userOpHash);
        if (owner != ECDSA.recover(hash, userOp.signature))
            return SIG_VALIDATION_FAILED;
        return SIG_VALIDATION_SUCCESS;
    }

    function _call(address target, uint256 value, bytes memory data) internal {
        (bool success, bytes memory result) = target.call{value: value}(data);
        if (!success) {
            assembly {
                revert(add(result, 32), mload(result))
            }
        }
    }

    /**
     * check current account deposit in the entryPoint
     */
    function getDeposit() public view returns (uint256) {
        return entryPoint().balanceOf(address(this));
    }

    /**
     * deposit more funds for this account in the entryPoint
     */
    function addDeposit() public payable {
        entryPoint().depositTo{value: msg.value}(address(this));
    }

    /**
     * withdraw value from the account's deposit
     * @param withdrawAddress target to send to
     * @param amount to withdraw
     */
    function withdrawDepositTo(address payable withdrawAddress, uint256 amount) public onlyOwner {
        entryPoint().withdrawTo(withdrawAddress, amount);
    }

    function _authorizeUpgrade(address newImplementation) internal view override {
        (newImplementation);
        _onlyOwner();
    }
}
```

The `SimpleAccount` contract is a basic Ethereum wallet designed to handle transactions and manage funds efficiently. The contract is controlled by a single owner, who can use it to send transactions, either one at a time or in batches, using the `execute` and `executeBatch` functions. The `execute` function lets the owner send a single transaction to a specified destination, while `executeBatch` allows multiple transactions to be sent in one go.

The contract also allows the owner to deposit and withdraw funds to and from the **EntryPoint** with the `addDeposit` and `withdrawDepositTo` functions, making it easy to manage the funds required for transactions.

The contract uses signature validation to ensure that only the owner can authorize transactions. Additionally, it is upgradeable, allowing the contract’s functionality to be updated in the future without losing control or security.

---

### EntryPoint.sol

The `EntryPoint.sol` can be found in the `contracts/core/` directory. Here is a highlight of `_executeUserOp`, one of the important functions of `EntryPoint.sol.`

```
    function _executeUserOp(
        uint256 opIndex,
        PackedUserOperation calldata userOp,
        UserOpInfo memory opInfo
    )
    internal
    returns
    (uint256 collected) {
        uint256 preGas = gasleft();
        bytes memory context = getMemoryBytesFromOffset(opInfo.contextOffset);
        bool success;
        {
            uint256 saveFreePtr;
            assembly ("memory-safe") {
                saveFreePtr := mload(0x40)
            }
            bytes calldata callData = userOp.callData;
            bytes memory innerCall;
            bytes4 methodSig;
            assembly {
                let len := callData.length
                if gt(len, 3) {
                    methodSig := calldataload(callData.offset)
                }
            }
            if (methodSig == IAccountExecute.executeUserOp.selector) {
                bytes memory executeUserOp = abi.encodeCall(IAccountExecute.executeUserOp, (userOp, opInfo.userOpHash));
                innerCall = abi.encodeCall(this.innerHandleOp, (executeUserOp, opInfo, context));
            } else
            {
                innerCall = abi.encodeCall(this.innerHandleOp, (callData, opInfo, context));
            }
            assembly ("memory-safe") {
                success := call(gas(), address(), 0, add(innerCall, 0x20), mload(innerCall), 0, 32)
                collected := mload(0)
                mstore(0x40, saveFreePtr)
            }
        }
        if (!success) {
            bytes32 innerRevertCode;
            assembly ("memory-safe") {
                let len := returndatasize()
                if eq(32,len) {
                    returndatacopy(0, 0, 32)
                    innerRevertCode := mload(0)
                }
            }
            if (innerRevertCode == INNER_OUT_OF_GAS) {
                // handleOps was called with gas limit too low. abort entire bundle.
                //can only be caused by bundler (leaving not enough gas for inner call)
                revert FailedOp(opIndex, "AA95 out of gas");
            } else if (innerRevertCode == INNER_REVERT_LOW_PREFUND) {
                // innerCall reverted on prefund too low. treat entire prefund as "gas cost"
                uint256 actualGas = preGas - gasleft() + opInfo.preOpGas;
                uint256 actualGasCost = opInfo.prefund;
                emitPrefundTooLow(opInfo);
                emitUserOperationEvent(opInfo, false, actualGasCost, actualGas);
                collected = actualGasCost;
            } else {
                emit PostOpRevertReason(
                    opInfo.userOpHash,
                    opInfo.mUserOp.sender,
                    opInfo.mUserOp.nonce,
                    Exec.getReturnData(REVERT_REASON_MAX_LEN)
                );

                uint256 actualGas = preGas - gasleft() + opInfo.preOpGas;
                collected = _postExecution(
                    IPaymaster.PostOpMode.postOpReverted,
                    opInfo,
                    context,
                    actualGas
                );
            }
        }
    }
```

**`_executeUserOp:`** This function is responsible for executing a user operation. It processes a UserOp by first preparing the relevant data, then calling an internal method to handle the operation, and finally managing errors that may occur during execution.

---

### Deploy the Contract

We will be deploying this to Sepolia. Before we can deploy this contract, let’s make some changes to the Repo.

- Let’s head to the root directory of our project and edit the file `hardhat.config.ts`. Replace all contents with the following code snippet.

```
import '@nomiclabs/hardhat-waffle';
import '@nomiclabs/hardhat-ethers';
import 'hardhat-deploy';
import { HardhatUserConfig } from 'hardhat/config';
import * as dotenv from 'dotenv';

dotenv.config();

const INFURA_ID = process.env.INFURA_ID!;
const PRIVATE_KEY = process.env.PRIVATE_KEY!;
const config: HardhatUserConfig = {
  solidity: {
    version: '0.8.23',
    settings: {
      optimizer: { enabled: true, runs: 1000000 },
    },
  },
  networks: {
    sepolia: {
      url: `https://sepolia.infura.io/v3/${INFURA_ID}`,
      accounts: [PRIVATE_KEY],
    },
  },
  namedAccounts: {
    deployer: {
      default: 0,
    },
  },
};
export default config;
```

---

Now, move to the `deploy` directory. It contains two files: `1_deploy_entrypoint.ts` and `2_deploy_SimpleAccountFactory.ts`. We will deploy the `EntryPoint` contract and use it address to deploy the `SimpleAccountFactory`.

First, the `EntryPoint` contract is deployed using the `1_deploy_entrypoint.ts` script. Remember the`EntryPoint` handles both **Validation** and **Execution** of `UserOperation` objects.

Next, the `SimpleAccountFactory` contract is deployed using the `2_deploy_SimpleAccountFactory.ts` script. This contract acts as a "factory" that creates individual Simple Account contracts. The factory uses a deterministic process (such as the `CREATE2` opcode) to deploy these accounts, meaning the address of each account can be predicted before deployment.

Replace all contents with the following code snippets.

---

**`*1_deploy_entrypoint.ts*`**

```
import { HardhatRuntimeEnvironment } from 'hardhat/types';
import { DeployFunction } from 'hardhat-deploy/types';
import { ethers } from 'hardhat';

const deployEntryPoint: DeployFunction = async function (hre: HardhatRuntimeEnvironment) {
  const { deployments, getNamedAccounts } = hre;
  const { deploy } = deployments;
  const { deployer } = await getNamedAccounts();
  const ret = await deploy('EntryPoint', {
    from: deployer,
    args: [],
    gasLimit: 6e6,
    deterministicDeployment: process.env.SALT ?? true,
    log: true,
  });
  console.log('== EntryPoint deployed at:', ret.address);
};
export default deployEntryPoint;
deployEntryPoint.tags = ['EntryPoint'];
```
---

**`*2_deploy_SimpleAccountFactory.ts*`**

```
import { HardhatRuntimeEnvironment } from 'hardhat/types';
import { DeployFunction } from 'hardhat-deploy/types';

const deploySimpleAccountFactory: DeployFunction = async function (hre: HardhatRuntimeEnvironment) {
  const { deployments, getNamedAccounts } = hre;
  const { deploy, get } = deployments;
  const { deployer } = await getNamedAccounts();
  // Get the deployed EntryPoint address
  const entryPoint = await get('EntryPoint');
  // Deploy SimpleAccountFactory with EntryPoint address
  const factoryDeployment = await deploy('SimpleAccountFactory', {
    from: deployer,
    args: [entryPoint.address],
    gasLimit: 6e6,
    log: true,
    deterministicDeployment: true,
  });
  console.log('== SimpleAccountFactory deployed at:', factoryDeployment.address);
};
export default deploySimpleAccountFactory;
deploySimpleAccountFactory.tags = ['SimpleAccountFactory'];
deploySimpleAccountFactory.dependencies = ['EntryPoint'];
```

***Now, let’s deploy the contracts:***

For the `EntryPoint` contract:

```
npx hardhat deploy --network sepolia --tags EntryPoint
```

For the `SimpleAccountFactory` contract:

```
npx hardhat deploy --network sepolia --tags SimpleAccountFactory
```

Now that we have deployed the contracts, you’re all set!

---

### Interact with the Contract

To interact with the deployed contract, create a new script named `scripts/interact.ts`. Write the following code snippet to your interact.ts file

```
import { ethers } from "hardhat";
import fs from "fs";

async function main() {
  const [owner] = await ethers.getSigners();

  // Load ABIs
  const SimpleAccountContractJson = fs.readFileSync("./artifacts/contracts/samples/SimpleAccount.sol/SimpleAccount.json", "utf8");
  const SimpleAccountFactoryContractJson = fs.readFileSync("./artifacts/contracts/samples/SimpleAccountFactory.sol/SimpleAccountFactory.json", "utf8");

  const SimpleAccountABI = JSON.parse(SimpleAccountContractJson).abi;
  const SimpleAccountFactoryABI = JSON.parse(SimpleAccountFactoryContractJson).abi;

  const SimpleAccountFactoryAddress = "0x4104828bEdAa69eEB3c3e992e1eaF98E0Ce52228"; // Replace with actual address

  // Create factory contract instance
  const simpleAccountFactory = new ethers.Contract(SimpleAccountFactoryAddress, SimpleAccountFactoryABI, owner);

  // Generate deterministic account address
  const ownerAddress = await owner.getAddress();
  const salt = ethers.utils.formatBytes32String("your_custom_salt"); // Unique identifier, you can replace it with your string
  const expectedSimpleAccountAddress = await simpleAccountFactory.getAddress(ownerAddress, salt);
  console.log("Expected SimpleAccount Address:", expectedSimpleAccountAddress);

  // Deploy account if not deployed
  const tx = await simpleAccountFactory.createAccount(ownerAddress, salt);
  const receipt = await tx.wait();
  const simpleAccountAddress = receipt.events?.find((e: any) => e.event === "AccountCreated")?.args?.account || expectedSimpleAccountAddress;

  console.log("SimpleAccount Address:", simpleAccountAddress);

  // Interact with the deployed account
  const simpleAccount = new ethers.Contract(simpleAccountAddress, SimpleAccountABI, owner);

  // Deposit Ether
  const depositAmount = ethers.utils.parseEther("0.001");
  const depositTx = await simpleAccount.addDeposit({ value: depositAmount });
  await depositTx.wait();

  console.log(`Deposited: ${ethers.utils.formatEther(depositAmount)} ETH`);

  // Check balance
  const currentDeposit = await simpleAccount.getDeposit();
  console.log("Current Deposit:", ethers.utils.formatEther(currentDeposit));

  console.log("Check on Sepolia Etherscan:", `https://sepolia.etherscan.io/address/${simpleAccountAddress}`);
}

main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error(error);
    process.exit(1);
  });
```

Here is what the code does;

**Import Libraries and Setup:** Start by importing the necessary libraries like `fs` and `ethers`. These will be used for blockchain interactions and reading ABI files, respectively. Connect to your Ethereum wallet using `ethers.getSigners()`.

**Load Contract ABIs:** ABI (Application Binary Interface) files describe the structure of the contracts. The script reads the `SimpleAccount` and `SimpleAccountFactory` ABIs from the build artifacts to enable interaction with these contracts.

**Create Contract Instance for Factory:** Using the factory contract address (replace `SimpleAccountFactoryAddress` with your deployed factory's address), you create a contract instance that allows you to call its functions.

**Generate an Account Address and Deploy:** With the provided wallet address and a unique identifier (`salt`) to the factory calculates the deterministic address for your account. ****Call the factory’s `createAccount` function with the inputs to deploy the account.

**Interact with the Account:** Use its address to create an instance of the `SimpleAccount` contract. Deposit Ether into the account by sending a transaction. Check the current deposit balance stored in the contract.

### Run the Script

Run this script to interact with the contract:

```
npx hardhat run scripts/interact.ts --network sepolia
```
---

### Conclusion

Well done! You’ve just deployed and interacted with a contract implementing ERC-4337 account abstraction. This feature transforms how user accounts function on Ethereum, offering much-needed flexibility for both developers and users. But this is just the beginning — there’s so much more you can do with it.

### What’s Next? Try Out Advanced Features

Ready to take things up a notch? Here are some ideas:

- **Gasless Transactions**: Let users interact with your dApp without needing ETH for gas.
- **Multi-Sig Wallets**: Implement wallets that require multiple signatures for added security. These are perfect for groups or shared accounts.
- **Custom Authentication**: Think outside the box! Explore social logins, biometric data, or hardware wallets to give users new ways to connect.

---

### Read On

There’s always more to learn. Here are some must-read resources:

- [**Ethereum Developer Docs**](https://ethereum.org/en/developers/docs/accounts): The go-to place for learning Ethereum basics and beyond.
- [**EIP-4337**:](https://eips.ethereum.org/EIPS/eip-4337) Dive deep into how account abstraction works under the hood.

### What I Think About Account Abstraction

Honestly, account abstraction is a huge leap forward. It makes blockchain easier to use, which is exactly what we need for wider adoption. Gasless transactions, for example, can eliminate one of the biggest hurdles for new users.

That said, it’s not all sunshine and rainbows. Security is always a concern, especially when implementing custom smart contract logic or relying on third-party services. However, with careful design and testing, these challenges can be overcome.

### The Road Ahead

Looking ahead, I see account abstraction becoming a standard feature in most decentralized apps. It’s especially exciting when combined with Layer 2 scaling solutions. We’re entering a phase where blockchain apps will be faster, cheaper, and much easier to use.

So keep experimenting. Test new ideas. Stay curious. The potential of account abstraction — and blockchain technology in general — is just beginning to unfold.
