`Dear Lukso team, as we have gone through each contract within the scope, we have noticed very good practices that have been implemented. However, we have identified some inconsistencies that we recommend addressing.We believe that at least 90% of the recommendations in the following report should be applied for better readability, and most importantly, safety.`

`Note: We have provided a description of the situation and recommendations to follow, including articles and resources we have created to help identify the problem and address it quickly, and to implement them in future standasrs.`

# QA Report for Lukso LSP contest

## [01] Improve the documentation on the LSP7DigitalAssetCore.decimals function
Reading the documentation of `LSP7DigitalAssetCore.sol`, we noticed that it specifies that if a token is divisible, it can have decimals (up to 18). However, it does not specify what should be done if a protocol or user wants to implement this standard with decimals smaller than 18. We strongly recommend following the best practices implemented by OpenZeppelin, which provides good documentation for different use cases of a function.

### PROOF OF CONCEPT
[Lukso docs link :](https://docs.lukso.tech/standards/nft-2.0/LSP7-Digital-Asset/#divisible-vs-non-divisible)
```text
Divisible asset can have decimals (up to 18) and token amounts can be fractional. For instance, it is possible to mint or transfer less than one token (e.g., 0.3 tokens). 
```

### Mittigation 
We recommend expanding the documentation on this function to make the step to follow more specific in case a protocol or the user wants to deploy this standard with tokens that are divisible by less than 18 decimal, as OpenZeppelin does, for example:

```solidity
    // ⬇ OpenZeppelin docs for ERC20.decimals

     * Tokens usually opt for a value of 18, imitating the relationship between
     * Ether and Wei. This is the default value returned by this function, unless
     * it's overridden. 
```

## [02] Inconsistent solidity pragma in LSP's contracts
The source files have different solidity compiler ranges referenced. This leads to potential security flaws between deployed contracts depending on the compiler version chosen for any particular file. It also greatly increases the cost of maintenance as different compiler versions have different semantics and behavior.

### PROOF OF CONCEPT
```solidity
// @audit In some contracts we found this issue.

contracts/ERC725YCore.sol
2: pragma solidity ^0.8.0;

contracts/UniversalProfile.sol
2: pragma solidity ^0.8.4;

LSP7CompatibleERC20.sol
2: pragma solidity ^0.8.12;
```
### Mittigation
We recommend to fix a definite compiler range that is consistent between contracts and upgrade any affected contracts to conform to the specified compiler.

## [03] In the contract LSP0ERC725Account Lack of address(0) checks in the constructor 
To prevent unintended behaviors, critical constructor inputs should be checked against address(0), missing checks for zero-addresses may lead to infunctional protocol, if the variable addresses are updated incorrectly. This inconsistency is also present in the contracts.

```solidity
// @audit Missing checks for address(0) also in this instance.
- LSP0ERC725Account.constructor 
- LSP14Ownable2Step._transferOwnership  
- LSP4DigitalAssetMetadata.constructor 
- LSP7DigitalAssetInitAbstract.constructor
- LSP0ERC725Account.constructor
- ERC725.constructor
- LSP0ERC725Account.constructor
```
### Mittigation
Consider adding zero-address checks in the discussed constructors, follow your best practices like the LSP6KeyManager contract in the line 19:  

```solidity
LSP6KeyManager.sol
18: constructor(address target_) {
        if (target_ == address(0)) revert InvalidLSP6Target();
        _target = target_;
        _setupLSP6ReentrancyGuard();
    };
```

## [04] Some Lukso contracts utilize Assembly - should have comments
The Lukso team makes use of Assembly, since this is a low level language that is more difficult to parse by readers, include extensive documentation, comments on the rationale behind its use, clearly explaining what each assembly instruction does

This will make it easier for users to trust the code, for reviewers to validate the code, and for developers to build on or update the code.

Note that using Assembly removes several important security features of Solidity, which can make the code more insecure and more error-prone.

### PROOF OF CONCEPT
```solidity
// @audit Document the Assembly code.
ERC725XCore.sol
265: assembly {
        contractAddress := create(
            value,
            add(creationCode, 0x20),
            mload(creationCode)
        )
    }
```

### Recommendation
It is essential to clearly and comprehensively document all activities related to critical function `assembly` within the project. By doing so, a more complete and accurate understanding of what is being done is provided, which is important both for auditors and for long-term project maintenance. By following the practices of monitoring and controlling project work, it can be ensured that all assemblies are properly documented in accordance with the project's objectives and goals.

## [05] Use a more recent version of solidity
Old version of solidity is used, consider using the new one `0.8.19` as OpenZeppelin does for all the developers who inhered from they contracts. You can see what new versions offer regarding bug fixed [here](https://github.com/ethereum/solidity/blob/develop/Changelog.md)


## [06] Also, values in assembly can be stored in constant variables.
Values in assembly can also benefit from constant variables, which would also increase the readability of these functions.

### PROOF OF CONCEPT
```solidity
ERC725XCore.sol
       assembly {
            contractAddress := create(
                value,
                add(creationCode, 0x20),
                mload(creationCode)
            )
        }

LSP17Extendable.sol
       assembly {
           ...................
            let success := call(
                gas(),
                extension,
                0,
                0,
                add(calldatasize(), 52),
                0,
                0
            )
            ...................
            }

LSP20CallVerification.sol
       assembly {
                let returndata_size := mload(returnedData)
                revert(add(32, returnedData), returndata_size)
            }
```

## [07] Mandatory checks for extra safety in the setters
In the folowings functions below, there is a check that can be made in order to achieve more safe and efficient code.

### PROOF OF CONCEPT
```solidity
ERC725InitAbstract.sol 
26: function _initialize(address newOwner) internal virtual onlyInitializing{
        // @audit - check the msg.value != 0
        require(
            newOwner != address(0),
            "Ownable: new owner is the zero address"
        ); /*....*/     
ERC725X.sol
20: constructor(address newOwner) payable {
        // @audit - check the msg.value != 0
        require(
            newOwner != address(0),
            "Ownable: new owner is the zero address"
        ); /*....*/

ERC725Y.sol
21: constructor(address newOwner) payable {
        // @audit - check the msg.value != 0

        require(
            newOwner != address(0),
            "Ownable: new owner is the zero address"
        );

LSP14Ownable2Step.sol
128: function _transferOwnership(address newOwner) internal virtual {
    // @audit - check the newOwner != address(0)  

```

## [08] We suggest to use named parameters for mapping type
Consider using named parameters in mappings (e.g. mapping(address account => uint256 balance)) to improve readability. This feature is present since Solidity 0.8.18.

```solidity
mapping(address account => uint256 balance); 
```

## [09] Use a single file for all system-wide constants
In all LPS implementations, there are many "constants" used in the system. It is recommended to store all constants in a single file (for example, Constants.sol) and use inheritance to access these values.

This helps improve readability and facilitates future maintenance. It also helps with any issues, as some of these hard-coded contracts are administrative contracts.

Constants.sol is used and imported in contracts that require access to these values. This is just a suggestion, as the project currently has more than 11 files that store constants.

We have created an example of how the Constants file should look like:
- [Constants Template](https://gist.github.com/catellaTech/e802b9f34f184d982d536bfc253ccf0a)

