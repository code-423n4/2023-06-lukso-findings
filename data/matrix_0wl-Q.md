## Non Critical Issues

|       | Issue                                                                   |
| ----- | :---------------------------------------------------------------------- |
| NC-1  | ADD A TIMELOCK TO CRITICAL FUNCTIONS                                    |
| NC-2  | ADD TO BLACKLIST FUNCTION                                               |
| NC-3  | GENERATE PERFECT CODE HEADERS EVERY TIME                                |
| NC-4  | COMMENTED CODE                                                          |
| NC-5  | INCONSISTENT SOLIDITY VERSIONS                                          |
| NC-6  | IMPLEMENTATION CONTRACT MAY NOT BE INITIALIZED                          |
| NC-7  | MARK VISIBILITY OF INITIALIZE(…) FUNCTIONS AS EXTERNAL                  |
| NC-8  | DON’T WORRY ABOUT KECCAK256 COLLISIONS                                  |
| NC-9  | LACK OF EVENT EMISSION AFTER CRITICAL INITIALIZE() FUNCTIONS            |
| NC-10 | FOR EXTENDED “USING-FOR” USAGE, USE THE LATEST PRAGMA VERSION           |
| NC-11 | Omissions in events                                                     |
| NC-12 | FOR FUNCTIONS AND VARIABLES FOLLOW SOLIDITY STANDARD NAMING CONVENTIONS |
| NC-13 | Functions not used internally could be marked external                  |

### [NC-1] ADD A TIMELOCK TO CRITICAL FUNCTIONS

#### Description:

It is a good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate (less risk of a malicious owner making a sandwich attack on a user).

#### **Proof Of Concept**

```solidity
File: /contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol

339:     function setData(

380:     function setDataBatch(

875:     function _setData(

```

```solidity
File: /contracts/LSP4DigitalAssetMetadata/LSP4DigitalAssetMetadata.sol

52:     function _setData(

```

```solidity
File: /contracts/LSP4DigitalAssetMetadata/LSP4DigitalAssetMetadataInitAbstract.sol

50:     function _setData(

```

```solidity
File: /submodules/ERC725/implementations/contracts/ERC725YCore.sol

62:     function setData(

73:     function setDataBatch(

104:     function _setData(

```

### [NC-2] ADD TO BLACKLIST FUNCTION

#### Description:

NFT thefts have increased recently, so with the addition of hacked NFTs to the platform, NFTs can be converted into liquidity. To prevent this, I recommend adding the blacklist function.

#### Recommended Mitigation Steps:

Add to Blacklist function and modifier.

### [NC-3] GENERATE PERFECT CODE HEADERS EVERY TIME

#### Description:

I recommend using header for Solidity code layout and readability:
https://github.com/transmissions11/headers

```solidity
/*//////////////////////////////////////////////////////////////
                           TESTING 123
//////////////////////////////////////////////////////////////*/
```

### [NC-4] COMMENTED CODE

#### **Proof Of Concept**

```solidity
File: /contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol

38:             dataKey[dataKey.length - 2] == 0x5b && // "[" in utf8 encoded

39:                 dataKey[dataKey.length - 1] == 0x5d, // "]" in utf8

```

### [NC-5] INCONSISTENT SOLIDITY VERSIONS

#### Description:

The project is compiled with different versions of Solidity, which is not recommended because it can lead to undefined behaviors.

It is better to use one Solidity compiler version across all contracts instead of different versions with different bugs and security checks.

#### **Proof Of Concept**

```solidity
File: /contracts/LSP0ERC725Account/ILSP0ERC725Account.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP0ERC725Account/LSP0Constants.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP0ERC725Account/LSP0ERC725Account.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP0ERC725Account/LSP0ERC725AccountInit.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP0ERC725Account/LSP0ERC725AccountInitAbstract.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP0ERC725Account/LSP0Utils.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP14Ownable2Step/ILSP14Ownable2Step.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP14Ownable2Step/LSP14Constants.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP14Ownable2Step/LSP14Errors.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP17ContractExtension/LSP17Constants.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP17ContractExtension/LSP17Errors.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP17ContractExtension/LSP17Extendable.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP17ContractExtension/LSP17Extension.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP17ContractExtension/LSP17Utils.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP20CallVerification/ILSP20CallVerifier.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP20CallVerification/LSP20CallVerification.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP20CallVerification/LSP20Constants.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP20CallVerification/LSP20Errors.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP4DigitalAssetMetadata/ILSP4Compatibility.sol

3: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP4DigitalAssetMetadata/LSP4Compatibility.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP4DigitalAssetMetadata/LSP4Constants.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP4DigitalAssetMetadata/LSP4DigitalAssetMetadata.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP4DigitalAssetMetadata/LSP4DigitalAssetMetadataInitAbstract.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP4DigitalAssetMetadata/LSP4Errors.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol

2: pragma solidity ^0.8.5;

```

```solidity
File: /contracts/LSP6KeyManager/LSP6Modules/LSP6OwnershipModule.sol

2: pragma solidity ^0.8.5;

```

```solidity
File: /contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol

2: pragma solidity ^0.8.5;

```

```solidity
File: /contracts/LSP7DigitalAsset/ILSP7DigitalAsset.sol

3: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP7DigitalAsset/LSP7Constants.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP7DigitalAsset/LSP7DigitalAsset.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP7DigitalAsset/LSP7DigitalAssetInitAbstract.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP7DigitalAsset/LSP7Errors.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP8IdentifiableDigitalAsset/ILSP8IdentifiableDigitalAsset.sol

3: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP8IdentifiableDigitalAsset/LSP8Constants.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP8IdentifiableDigitalAsset/LSP8Errors.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAsset.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetInitAbstract.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/UniversalProfile.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/UniversalProfileInit.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/UniversalProfileInitAbstract.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /submodules/ERC725/implementations/contracts/ERC725.sol

2: pragma solidity ^0.8.0;

```

```solidity
File: /submodules/ERC725/implementations/contracts/ERC725Init.sol

2: pragma solidity ^0.8.0;

```

```solidity
File: /submodules/ERC725/implementations/contracts/ERC725InitAbstract.sol

2: pragma solidity ^0.8.0;

```

```solidity
File: /submodules/ERC725/implementations/contracts/ERC725X.sol

2: pragma solidity ^0.8.0;

```

```solidity
File: /submodules/ERC725/implementations/contracts/ERC725XCore.sol

2: pragma solidity ^0.8.0;

```

```solidity
File: /submodules/ERC725/implementations/contracts/ERC725XInit.sol

2: pragma solidity ^0.8.0;

```

```solidity
File: /submodules/ERC725/implementations/contracts/ERC725XInitAbstract.sol

2: pragma solidity ^0.8.0;

```

```solidity
File: /submodules/ERC725/implementations/contracts/ERC725Y.sol

2: pragma solidity ^0.8.0;

```

```solidity
File: /submodules/ERC725/implementations/contracts/ERC725YCore.sol

2: pragma solidity ^0.8.0;

```

```solidity
File: /submodules/ERC725/implementations/contracts/ERC725YInit.sol

2: pragma solidity ^0.8.0;

```

```solidity
File: /submodules/ERC725/implementations/contracts/ERC725YInitAbstract.sol

2: pragma solidity ^0.8.0;

```

```solidity
File: /submodules/ERC725/implementations/contracts/constants.sol

2: pragma solidity ^0.8.0;

```

```solidity
File: /submodules/ERC725/implementations/contracts/errors.sol

2: pragma solidity ^0.8.0;

```

### [NC-6] IMPLEMENTATION CONTRACT MAY NOT BE INITIALIZED

#### Description:

OpenZeppelin recommends that the initializer modifier be applied to constructors.

Per OZs Post implementation contract should be initialized to avoid potential griefs or exploits.

https://forum.openzeppelin.com/t/uupsupgradeable-vulnerability-post-mortem/15680/5

#### **Proof Of Concept**

```solidity
File: /contracts/LSP0ERC725Account/LSP0ERC725AccountInitAbstract.sol

7:     Initializable

8: } from "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";

45:     Initializable,

```

```solidity
File: /submodules/ERC725/implementations/contracts/ERC725InitAbstract.sol

7:     Initializable

22:     Initializable,

```

```solidity
File: /submodules/ERC725/implementations/contracts/ERC725XInitAbstract.sol

6:     Initializable

```

```solidity
File: /submodules/ERC725/implementations/contracts/ERC725YInitAbstract.sol

6:     Initializable

```

### [NC-7] MARK VISIBILITY OF INITIALIZE(…) FUNCTIONS AS EXTERNAL

#### Description:

If someone wants to extend via inheritance, it might make more sense that the overridden initialize(...) function calls the internal {...}\_init function, not the parent public initialize(...) function.

External instead of public would give more sense of the initialize(...) functions to behave like a constructor (only called on deployment, so should only be called externally).

From a security point of view, it might be safer so that it cannot be called internally by accident in the child contract.

#### **Proof Of Concept**

```solidity
File: /submodules/ERC725/implementations/contracts/ERC725Init.sol

27:     function initialize(address newOwner) public payable virtual initializer {

```

```solidity
File: /submodules/ERC725/implementations/contracts/ERC725XInit.sol

26:     function initialize(address newOwner) public payable virtual initializer {

```

```solidity
File: /submodules/ERC725/implementations/contracts/ERC725YInit.sol

26:     function initialize(address newOwner) public payable virtual initializer {

```

### [NC-8] DON’T WORRY ABOUT KECCAK256 COLLISIONS

#### **Proof Of Concept**

```solidity
File: /contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol

733:                     continue;

```

### [NC-9] LACK OF EVENT EMISSION AFTER CRITICAL INITIALIZE() FUNCTIONS

#### Description:

To record the init parameters for off-chain monitoring and transparency reasons, please consider emitting an event after the initialize() functions.

#### **Proof Of Concept**

```solidity
File: /contracts/LSP0ERC725Account/LSP0ERC725AccountInit.sol

62:     function initialize(

```

```solidity
File: /contracts/UniversalProfileInit.sol

34:     function initialize(

```

```solidity
File: /submodules/ERC725/implementations/contracts/ERC725Init.sol

27:     function initialize(address newOwner) public payable virtual initializer {

```

```solidity
File: /submodules/ERC725/implementations/contracts/ERC725XInit.sol

26:     function initialize(address newOwner) public payable virtual initializer {

```

```solidity
File: /submodules/ERC725/implementations/contracts/ERC725YInit.sol

26:     function initialize(address newOwner) public payable virtual initializer {

```

### [NC-10] FOR EXTENDED “USING-FOR” USAGE, USE THE LATEST PRAGMA VERSION

#### **Proof Of Concept**

```solidity
File: /contracts/LSP0ERC725Account/ILSP0ERC725Account.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP0ERC725Account/LSP0Constants.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP0ERC725Account/LSP0ERC725Account.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP0ERC725Account/LSP0ERC725AccountInit.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP0ERC725Account/LSP0ERC725AccountInitAbstract.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP0ERC725Account/LSP0Utils.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP14Ownable2Step/ILSP14Ownable2Step.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP14Ownable2Step/LSP14Constants.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP14Ownable2Step/LSP14Errors.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP17ContractExtension/LSP17Constants.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP17ContractExtension/LSP17Errors.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP17ContractExtension/LSP17Extendable.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP17ContractExtension/LSP17Extension.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP17ContractExtension/LSP17Utils.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP20CallVerification/ILSP20CallVerifier.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP20CallVerification/LSP20CallVerification.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP20CallVerification/LSP20Constants.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP20CallVerification/LSP20Errors.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP4DigitalAssetMetadata/ILSP4Compatibility.sol

3: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP4DigitalAssetMetadata/LSP4Compatibility.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP4DigitalAssetMetadata/LSP4Constants.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP4DigitalAssetMetadata/LSP4DigitalAssetMetadata.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP4DigitalAssetMetadata/LSP4DigitalAssetMetadataInitAbstract.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP4DigitalAssetMetadata/LSP4Errors.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol

2: pragma solidity ^0.8.5;

```

```solidity
File: /contracts/LSP6KeyManager/LSP6Modules/LSP6OwnershipModule.sol

2: pragma solidity ^0.8.5;

```

```solidity
File: /contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol

2: pragma solidity ^0.8.5;

```

```solidity
File: /contracts/LSP7DigitalAsset/ILSP7DigitalAsset.sol

3: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP7DigitalAsset/LSP7Constants.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP7DigitalAsset/LSP7DigitalAsset.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP7DigitalAsset/LSP7DigitalAssetInitAbstract.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP7DigitalAsset/LSP7Errors.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP8IdentifiableDigitalAsset/ILSP8IdentifiableDigitalAsset.sol

3: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP8IdentifiableDigitalAsset/LSP8Constants.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP8IdentifiableDigitalAsset/LSP8Errors.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAsset.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetInitAbstract.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/UniversalProfile.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/UniversalProfileInit.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /contracts/UniversalProfileInitAbstract.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: /submodules/ERC725/implementations/contracts/ERC725.sol

2: pragma solidity ^0.8.0;

```

```solidity
File: /submodules/ERC725/implementations/contracts/ERC725Init.sol

2: pragma solidity ^0.8.0;

```

```solidity
File: /submodules/ERC725/implementations/contracts/ERC725InitAbstract.sol

2: pragma solidity ^0.8.0;

```

```solidity
File: /submodules/ERC725/implementations/contracts/ERC725X.sol

2: pragma solidity ^0.8.0;

```

```solidity
File: /submodules/ERC725/implementations/contracts/ERC725XCore.sol

2: pragma solidity ^0.8.0;

```

```solidity
File: /submodules/ERC725/implementations/contracts/ERC725XInit.sol

2: pragma solidity ^0.8.0;

```

```solidity
File: /submodules/ERC725/implementations/contracts/ERC725XInitAbstract.sol

2: pragma solidity ^0.8.0;

```

```solidity
File: /submodules/ERC725/implementations/contracts/ERC725Y.sol

2: pragma solidity ^0.8.0;

```

```solidity
File: /submodules/ERC725/implementations/contracts/ERC725YCore.sol

2: pragma solidity ^0.8.0;

```

```solidity
File: /submodules/ERC725/implementations/contracts/ERC725YInit.sol

2: pragma solidity ^0.8.0;

```

```solidity
File: /submodules/ERC725/implementations/contracts/ERC725YInitAbstract.sol

2: pragma solidity ^0.8.0;

```

```solidity
File: /submodules/ERC725/implementations/contracts/constants.sol

2: pragma solidity ^0.8.0;

```

```solidity
File: /submodules/ERC725/implementations/contracts/errors.sol

2: pragma solidity ^0.8.0;

```

#### Recommended Mitigation Steps:

Use solidity pragma version min. 0.8.13

### [NC-11] Omissions in events

#### Description:

Throughout the codebase, events are generally emitted when sensitive changes are made to the contracts. However, some events are missing important parameters.

#### **Proof Of Concept**

```solidity
File: /contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol

165:             emit RenounceOwnershipStarted();

178:         emit OwnershipRenounced();

```

### [NC-12] FOR FUNCTIONS AND VARIABLES FOLLOW SOLIDITY STANDARD NAMING CONVENTIONS

#### Description:

Solidity’s standard naming convention for internal and private functions and variables (apart from constants): the mixedCase format starting with an underscore (\_mixedCase starting with an underscore)

Solidity’s standard naming convention for internal and private constants variables: the snake_case format starting with an underscore (\_mixedCase starting with an underscore) and use ALL_CAPS for naming them.

#### **Proof Of Concept**

```solidity
File: /contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol

231:     function isEncodedArray(bytes memory data) internal pure returns (bool) {

```

### [NC-13] Functions not used internally could be marked external

#### **Proof Of Concept**

```solidity
File: /contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol

171:     function batchCalls(

```

## Low Issues

|     | Issue                                                             |
| --- | :---------------------------------------------------------------- |
| L-1 | Initializing state-variables in proxy-based upgradeable contracts |
| L-2 | CRITICAL CHANGES SHOULD USE TWO-STEP PROCEDURE                    |
| L-3 | UNIFY RETURN CRITERIA                                             |
| L-4 | TOKEN SHOULD THROW AN ERROR IF \_TOKENID IS NOT A VALID NFT       |

### [L-1] Initializing state-variables in proxy-based upgradeable contracts

#### Description:

This should be done in initializer functions and not as part of the state variable declarations in which case they won’t be set.

https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable#avoid-initial-values-in-field-declarations

#### **Proof Of Concept**

```solidity
File: /contracts/LSP0ERC725Account/LSP0ERC725AccountInit.sol

62:     function initialize(

```

```solidity
File: /contracts/UniversalProfileInit.sol

34:     function initialize(

```

```solidity
File: /submodules/ERC725/implementations/contracts/ERC725Init.sol

27:     function initialize(address newOwner) public payable virtual initializer {

```

```solidity
File: /submodules/ERC725/implementations/contracts/ERC725XInit.sol

26:     function initialize(address newOwner) public payable virtual initializer {

```

```solidity
File: /submodules/ERC725/implementations/contracts/ERC725YInit.sol

26:     function initialize(address newOwner) public payable virtual initializer {

```

### [L-2] CRITICAL CHANGES SHOULD USE TWO-STEP PROCEDURE

#### Description:

The critical procedures should be two step process.

#### **Proof Of Concept**

```solidity
File: /contracts/LSP4DigitalAssetMetadata/LSP4DigitalAssetMetadata.sol

52:     function _setData(

```

```solidity
File: /contracts/LSP4DigitalAssetMetadata/LSP4DigitalAssetMetadataInitAbstract.sol

50:     function _setData(

```

```solidity
File: /contracts/LSP6KeyManager/LSP6Modules/LSP6OwnershipModule.sol

14:     function _verifyOwnershipPermissions(

```

```solidity
File: /submodules/ERC725/implementations/contracts/ERC725YCore.sol

62:     function setData(

73:     function setDataBatch(

104:     function _setData(

```

#### Recommended Mitigation Steps:

Lack of two-step procedure for critical operations leaves them error-prone. Consider adding two step procedure on the critical functions.

### [L-3] UNIFY RETURN CRITERIA

#### Description:

In contracts, sometimes the name of the return variable is not defined and sometimes is, unifying the way of writing the code makes the code more uniform and readable.

#### **Proof Of Concept**

```solidity
File: /contracts/LSP0ERC725Account/ILSP0ERC725Account.sol

69:     ) external returns (bytes[] memory results);

```

```solidity
File: /contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol

173:     ) public returns (bytes[] memory results) {

227:     ) public payable virtual override returns (bytes memory) {

285:     ) public payable virtual override returns (bytes[] memory) {

463:     ) public payable virtual returns (bytes memory returnedValues) {

699:         returns (bool)

737:     ) public view virtual returns (bytes4 magicValue) {

853:     ) internal view virtual override returns (address) {

```

```solidity
File: /contracts/LSP0ERC725Account/LSP0Utils.sol

26:     ) internal view returns (bytes memory) {

40:     ) internal view returns (bytes memory) {

```

```solidity
File: /contracts/LSP14Ownable2Step/ILSP14Ownable2Step.sol

42:     function pendingOwner() external view returns (address);

```

```solidity
File: /contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol

59:     function pendingOwner() public view virtual returns (address) {

```

```solidity
File: /contracts/LSP17ContractExtension/LSP17Extendable.sol

30:     ) public view virtual override returns (bool) {

46:     ) internal view virtual returns (bool) {

67:     ) internal view virtual returns (address);

```

```solidity
File: /contracts/LSP17ContractExtension/LSP17Extension.sol

22:     ) public view virtual override returns (bool) {

36:         returns (bytes calldata)

44:     function _extendableMsgSender() internal view virtual returns (address) {

54:     function _extendableMsgValue() internal view virtual returns (uint256) {

```

```solidity
File: /contracts/LSP17ContractExtension/LSP17Utils.sol

16:     ) internal pure returns (bool) {

```

```solidity
File: /contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol

81:     ) public payable virtual returns (bytes memory result) {

157:     ) internal virtual returns (bytes memory) {

206:     ) internal virtual returns (bytes memory) {

261:     ) public view virtual override returns (bool) {

```

```solidity
File: /contracts/LSP20CallVerification/ILSP20CallVerifier.sol

23:     ) external returns (bytes4 magicValue);

34:     ) external returns (bytes4);

```

```solidity
File: /contracts/LSP20CallVerification/LSP20CallVerification.sol

25:     ) internal virtual returns (bool verifyAfter) {

```

```solidity
File: /contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol

24:     ) internal pure returns (bytes32) {

34:     ) internal pure returns (bytes32) {

55:     ) internal pure returns (bytes32) {

74:     ) internal pure returns (bytes32) {

97:     ) internal pure returns (bytes32) {

118:     ) internal pure returns (bytes32) {

140:     ) internal pure returns (bytes32) {

165:     ) internal pure returns (bytes32) {

184:     ) internal pure returns (bytes32) {

203:     ) internal pure returns (bytes memory) {

220:     ) internal pure returns (bytes memory) {

231:     function isEncodedArray(bytes memory data) internal pure returns (bool) {

253:     ) internal pure returns (bool) {

285:     ) internal pure returns (bool) {

314:     ) internal pure returns (bool) {

```

```solidity
File: /contracts/LSP4DigitalAssetMetadata/ILSP4Compatibility.sol

18:     function name() external view returns (string memory);

24:     function symbol() external view returns (string memory);

```

```solidity
File: /contracts/LSP4DigitalAssetMetadata/LSP4Compatibility.sol

25:     function name() public view virtual override returns (string memory) {

34:     function symbol() public view virtual override returns (string memory) {

```

```solidity
File: /contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol

321:     ) internal pure returns (bytes4 requiredCallTypes) {

341:     ) internal pure returns (uint256, address, uint256, bytes4, bool) {

369:     ) internal pure returns (bool) {

385:     ) internal view returns (bool) {

402:     ) internal pure returns (bool) {

421:     ) internal pure returns (bool) {

```

```solidity
File: /contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol

195:     ) internal view virtual returns (bytes32) {

339:     ) internal view virtual returns (bytes32) {

388:     ) internal view virtual returns (bytes32) {

411:     ) internal view virtual returns (bytes32) {

443:     ) internal view returns (bytes32) {

477:     ) internal view virtual returns (bytes32) {

493:     ) internal view virtual returns (bytes32) {

```

```solidity
File: /contracts/LSP7DigitalAsset/ILSP7DigitalAsset.sol

65:     function decimals() external view returns (uint8);

71:     function totalSupply() external view returns (uint256);

80:     function balanceOf(address tokenOwner) external view returns (uint256);

134:     ) external view returns (uint256);

```

```solidity
File: /contracts/LSP7DigitalAsset/LSP7DigitalAsset.sol

53:     ) public view virtual override(IERC165, ERC725YCore) returns (bool) {

```

```solidity
File: /contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol

54:     function decimals() public view virtual returns (uint8) {

61:     function totalSupply() public view virtual returns (uint256) {

72:     ) public view virtual returns (uint256) {

111:     ) public view virtual returns (uint256) {

```

```solidity
File: /contracts/LSP7DigitalAsset/LSP7DigitalAssetInitAbstract.sol

47:     ) public view virtual override(IERC165, ERC725YCore) returns (bool) {

```

```solidity
File: /contracts/LSP8IdentifiableDigitalAsset/ILSP8IdentifiableDigitalAsset.sol

66:     function totalSupply() external view returns (uint256);

77:     function balanceOf(address tokenOwner) external view returns (uint256);

88:     function tokenOwnerOf(bytes32 tokenId) external view returns (address);

97:     ) external view returns (bytes32[] memory);

150:     ) external view returns (bool);

163:     ) external view returns (address[] memory);

```

```solidity
File: /contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAsset.sol

47:     ) public view virtual override(IERC165, ERC725YCore) returns (bool) {

```

```solidity
File: /contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol

61:     function totalSupply() public view virtual returns (uint256) {

72:     ) public view virtual returns (uint256) {

81:     ) public view virtual returns (address) {

96:     ) public view virtual returns (bytes32[] memory) {

156:     ) public view virtual returns (bool) {

167:     ) public view virtual returns (address[] memory) {

180:     ) internal view virtual returns (bool) {

290:     function _exists(bytes32 tokenId) internal view virtual returns (bool) {

```

```solidity
File: /contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetInitAbstract.sol

47:     ) public view virtual override(IERC165, ERC725YCore) returns (bool) {

```

```solidity
File: /submodules/ERC725/implementations/contracts/ERC725.sol

39:     ) public view virtual override(ERC725XCore, ERC725YCore) returns (bool) {

```

```solidity
File: /submodules/ERC725/implementations/contracts/ERC725InitAbstract.sol

41:     ) public view virtual override(ERC725XCore, ERC725YCore) returns (bool) {

```

```solidity
File: /submodules/ERC725/implementations/contracts/ERC725XCore.sol

45:     ) public payable virtual override onlyOwner returns (bytes memory) {

57:     ) public payable virtual override onlyOwner returns (bytes[] memory) {

66:     ) public view virtual override(IERC165, ERC165) returns (bool) {

81:     ) internal virtual returns (bytes memory) {

136:     ) internal virtual returns (bytes[] memory) {

178:     ) internal virtual returns (bytes memory result) {

206:     ) internal virtual returns (bytes memory result) {

228:     ) internal virtual returns (bytes memory result) {

250:     ) internal virtual returns (bytes memory newContract) {

291:     ) internal virtual returns (bytes memory newContract) {

```

```solidity
File: /submodules/ERC725/implementations/contracts/ERC725YCore.sol

35:     ) public view virtual override returns (bytes memory dataValue) {

44:     ) public view virtual override returns (bytes[] memory dataValues) {

100:     ) internal view virtual returns (bytes memory dataValue) {

117:     ) public view virtual override(IERC165, ERC165) returns (bool) {

```

#### Recommended Mitigation Steps:

add `{return x}` if you want to return the updated value or else remove `returns(uint)` from the `function(){}` if no value you wanted to return

### [L-4] TOKEN SHOULD THROW AN ERROR IF \_TOKENID IS NOT A VALID NFT

```solidity
File: /contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol

54:     mapping(address => EnumerableSet.Bytes32Set) private _tokenIdsForOperator;

```

#### Recommended Mitigation Steps:

Consider throwing an error if \_tokenId is not a valid NFT.
