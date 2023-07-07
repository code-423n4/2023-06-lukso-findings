## Gas Optimizations

|        | Issue                                                                                                                                     |
| ------ | :---------------------------------------------------------------------------------------------------------------------------------------- |
| GAS-1  | Use `selfbalance()` instead of `address(this).balance`                                                                                    |
| GAS-2  | `ABI.ENCODE()` IS LESS EFFICIENT THAN `ABI.ENCODEPACKED()`                                                                                |
| GAS-3  | Use assembly to check for `address(0)`                                                                                                    |
| GAS-4  | BEFORE SOME FUNCTIONS, WE SHOULD CHECK SOME VARIABLES FOR POSSIBLE GAS SAVE                                                               |
| GAS-5  | USING CALLDATA INSTEAD OF MEMORY FOR READ-ONLY ARGUMENTS IN EXTERNAL FUNCTIONS SAVES GAS                                                  |
| GAS-5  | Setting the constructor to payable                                                                                                        |
| GAS-7  | INTERNAL FUNCTIONS NOT CALLED BY THE CONTRACT SHOULD BE REMOVED TO SAVE DEPLOYMENT GAS                                                    |
| GAS-8  | MAKING CONSTANT VARIABLES PRIVATE WILL SAVE GAS DURING DEPLOYMENT                                                                         |
| GAS-9  | SAVE GAS BY NOT REQURING NON-ZERO INTERVAL IF NO LINEAR AMOUNT                                                                            |
| GAS-10 | The increment in for loop postcondition can be made unchecked                                                                             |
| GAS-11 | Use a more recent version of solidity                                                                                                     |
| GAS-12 | TERNARY OPERATION IS CHEAPER THAN IF-ELSE STATEMENT                                                                                       |
| GAS-13 | Usage of `uint`/`int` smaller than 32 bytes (256 bits) incurs overhead                                                                    |
| GAS-14 | Use != 0 instead of > 0 for unsigned integer comparison                                                                                   |
| GAS-15 | USE BYTES32 INSTEAD OF STRING                                                                                                             |
| GAS-16 | Public functions not called by the contract should be declared external instead OR Functions not used internally could be marked external |

### [GAS-1] Use `selfbalance()` instead of `address(this).balance`

#### Description:

You can use `selfbalance()` instead of `address(this).balance` when getting your contract's balance of ETH to save gas.

Additionally, you can use `balance(address)` instead of `address.balance()` when getting an external contract's balance of ETH.

#### **Proof Of Concept**

```solidity
File: /submodules/ERC725/implementations/contracts/ERC725XCore.sol

179:         if (address(this).balance < value) {

180:             revert ERC725X_InsufficientBalance(address(this).balance, value);

251:         if (address(this).balance < value) {

252:             revert ERC725X_InsufficientBalance(address(this).balance, value);

```

### [GAS-2] `ABI.ENCODE()` IS LESS EFFICIENT THAN `ABI.ENCODEPACKED()`

#### Description:

Use `abi.encodePacked()` where possible to save gas

#### **Proof Of Concept**

```solidity
File: /contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol

253:             LSP20CallVerification._verifyCallResult(_owner, abi.encode(result));

319:                 abi.encode(results)

```

### [GAS-3] Use assembly to check for `address(0)`

#### **Proof Of Concept**

```solidity
File: /contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol

801:         if (msg.sig == bytes4(0) && extension == address(0)) return;

801:         if (msg.sig == bytes4(0) && extension == address(0)) return;

804:         if (extension == address(0))

```

```solidity
File: /contracts/LSP17ContractExtension/LSP17Extendable.sol

50:         if (erc165Extension == address(0)) return false;

91:         if (extension == address(0))

```

```solidity
File: /contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol

116:         bool isMapValueSet = bytes20(notifierMapValue) != bytes20(0);

```

```solidity
File: /contracts/LSP20CallVerification/LSP20CallVerification.sol

80:             bytes28(bytes32(returnedData) << 32) != bytes28(0)

```

```solidity
File: /contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol

266:             if (bytes12(key) != bytes12(0)) return false;

```

```solidity
File: /contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol

345:         if (bytes12(executeCalldata[36:48]) != bytes12(0)) {

410:         bool isFunctionCall = requiredFunction != bytes4(0);

```

```solidity
File: /contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol

97:             if (requiredPermission == bytes32(0)) return;

146:                 if (requiredPermission != bytes32(0)) {

392:             bytes32(

```

```solidity
File: /contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol

268:         if (operator == address(0)) {

300:         if (to == address(0)) {

343:         if (from == address(0)) {

406:         if (from == address(0) || to == address(0)) {

406:         if (from == address(0) || to == address(0)) {

```

```solidity
File: /contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol

84:         if (tokenOwner == address(0)) {

115:         if (operator == address(0)) {

139:         if (operator == address(0)) {

291:         return _tokenOwners[tokenId] != address(0);

319:         if (to == address(0)) {

410:         if (to == address(0)) {

```

```solidity
File: /submodules/ERC725/implementations/contracts/ERC725.sol

28:             newOwner != address(0),

```

```solidity
File: /submodules/ERC725/implementations/contracts/ERC725InitAbstract.sol

28:             newOwner != address(0),

```

```solidity
File: /submodules/ERC725/implementations/contracts/ERC725X.sol

22:             newOwner != address(0),

```

```solidity
File: /submodules/ERC725/implementations/contracts/ERC725XCore.sol

89:             if (target != address(0))

96:             if (target != address(0))

269:         if (contractAddress == address(0)) {

```

```solidity
File: /submodules/ERC725/implementations/contracts/ERC725XInitAbstract.sol

21:             newOwner != address(0),

```

```solidity
File: /submodules/ERC725/implementations/contracts/ERC725Y.sol

23:             newOwner != address(0),

```

```solidity
File: /submodules/ERC725/implementations/contracts/ERC725YInitAbstract.sol

21:             newOwner != address(0),

```

### [GAS-4] BEFORE SOME FUNCTIONS, WE SHOULD CHECK SOME VARIABLES FOR POSSIBLE GAS SAVE

#### Description:

Before transfer, we should check for amount being 0 so the function doesnt run when its not gonna do anything:

#### **Proof Of Concept**

```solidity
File: /contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol

557:     function transferOwnership(

565:             LSP14Ownable2Step._transferOwnership(pendingNewOwner);

586:             LSP14Ownable2Step._transferOwnership(pendingNewOwner);

```

```solidity
File: /contracts/LSP14Ownable2Step/ILSP14Ownable2Step.sol

54:     function transferOwnership(address newOwner) external;

```

```solidity
File: /contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol

66:     function transferOwnership(

69:         _transferOwnership(newOwner);

126:     function _transferOwnership(address newOwner) internal virtual {

```

```solidity
File: /contracts/LSP7DigitalAsset/ILSP7DigitalAsset.sol

159:     function transfer(

189:     function transferBatch(

```

```solidity
File: /contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol

124:     function transfer(

148:         _transfer(from, to, amount, allowNonLSP1Recipient, data);

154:     function transferBatch(

173:             transfer(

399:     function _transfer(

```

```solidity
File: /contracts/LSP8IdentifiableDigitalAsset/ILSP8IdentifiableDigitalAsset.sol

186:     function transfer(

216:     function transferBatch(

```

```solidity
File: /contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol

191:     function transfer(

204:         _transfer(from, to, tokenId, allowNonLSP1Recipient, data);

210:     function transferBatch(

228:             transfer(

394:     function _transfer(

```

### [GAS-5] USING CALLDATA INSTEAD OF MEMORY FOR READ-ONLY ARGUMENTS IN EXTERNAL FUNCTIONS SAVES GAS

#### Description:

Mark data types as `calldata` instead of `memory` where possible. This makes it so that the data is not automatically loaded into memory. If the data passed into the function does not need to be changed (like updating values in an array), it can be passed in as `calldata`. The one exception to this is if the argument must later be passed into another function that takes an argument that specifies `memory` storage.
[Source](https://forum.openzeppelin.com/t/a-collection-of-gas-optimisation-tricks/19966/6)
[Source 2](https://dev.to/juanxavier/advanced-gas-optimizations-tips-for-solidity-1j2f)

#### **Proof Of Concept**

```solidity
File: /contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol

226:         bytes memory data

281:         uint256[] memory operationsType,

282:         address[] memory targets,

283:         uint256[] memory values,

284:         bytes[] memory datas

341:         bytes memory dataValue

381:         bytes32[] memory dataKeys,

382:         bytes[] memory dataValues

736:         bytes memory signature

```

```solidity
File: /contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol

80:         bytes memory /* data */

```

```solidity
File: /contracts/LSP20CallVerification/ILSP20CallVerifier.sol

22:         bytes memory receivedCalldata

33:         bytes memory result

```

```solidity
File: /contracts/LSP7DigitalAsset/ILSP7DigitalAsset.sol

164:         bytes memory data

190:         address[] memory from,

191:         address[] memory to,

192:         uint256[] memory amount,

193:         bool[] memory allowNonLSP1Recipient,

194:         bytes[] memory data

```

```solidity
File: /contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol

129:         bytes memory data

155:         address[] memory from,

156:         address[] memory to,

157:         uint256[] memory amount,

158:         bool[] memory allowNonLSP1Recipient,

159:         bytes[] memory data

```

```solidity
File: /contracts/LSP8IdentifiableDigitalAsset/ILSP8IdentifiableDigitalAsset.sol

191:         bytes memory data

217:         address[] memory from,

218:         address[] memory to,

219:         bytes32[] memory tokenId,

220:         bool[] memory allowNonLSP1Recipient,

221:         bytes[] memory data

```

```solidity
File: /contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol

196:         bytes memory data

211:         address[] memory from,

212:         address[] memory to,

213:         bytes32[] memory tokenId,

214:         bool[] memory allowNonLSP1Recipient,

215:         bytes[] memory data

```

```solidity
File: /submodules/ERC725/implementations/contracts/ERC725XCore.sol

44:         bytes memory data

53:         uint256[] memory operationsType,

54:         address[] memory targets,

55:         uint256[] memory values,

56:         bytes[] memory datas

```

```solidity
File: /submodules/ERC725/implementations/contracts/ERC725YCore.sol

43:         bytes32[] memory dataKeys

64:         bytes memory dataValue

74:         bytes32[] memory dataKeys,

75:         bytes[] memory dataValues

```

### [GAS-6] Setting the constructor to payable

#### Description:

Saves ~13 gas per instance

#### **Proof Of Concept**

```solidity
File: /contracts/LSP0ERC725Account/LSP0ERC725AccountInit.sol

47:     constructor() {

```

```solidity
File: /contracts/LSP4DigitalAssetMetadata/LSP4DigitalAssetMetadata.sol

31:     constructor(

```

```solidity
File: /contracts/LSP7DigitalAsset/LSP7DigitalAsset.sol

39:     constructor(

```

```solidity
File: /contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAsset.sol

36:     constructor(

```

```solidity
File: /contracts/UniversalProfileInit.sol

19:     constructor() {

```

```solidity
File: /submodules/ERC725/implementations/contracts/ERC725Init.sol

19:     constructor() {

```

```solidity
File: /submodules/ERC725/implementations/contracts/ERC725XInit.sol

18:     constructor() {

```

```solidity
File: /submodules/ERC725/implementations/contracts/ERC725YInit.sol

18:     constructor() {

```

### [GAS-7] INTERNAL FUNCTIONS NOT CALLED BY THE CONTRACT SHOULD BE REMOVED TO SAVE DEPLOYMENT GAS

#### Description:

If the functions are required by an interface, the contract should inherit from that interface and use the override keyword

#### **Proof Of Concept**

```solidity
File: /contracts/LSP17ContractExtension/LSP17Extendable.sol

86:     function _fallbackLSP17Extendable() internal virtual {

```

```solidity
File: /contracts/LSP17ContractExtension/LSP17Extension.sol

44:     function _extendableMsgSender() internal view virtual returns (address) {

54:     function _extendableMsgValue() internal view virtual returns (uint256) {

```

```solidity
File: /contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol

359:     function _burn(bytes32 tokenId, bytes memory data) internal virtual {

```

```solidity
File: /submodules/ERC725/implementations/contracts/ERC725InitAbstract.sol

26:     function _initialize(address newOwner) internal virtual onlyInitializing {

```

```solidity
File: /submodules/ERC725/implementations/contracts/ERC725XInitAbstract.sol

19:     function _initialize(address newOwner) internal virtual onlyInitializing {

```

```solidity
File: /submodules/ERC725/implementations/contracts/ERC725YInitAbstract.sol

19:     function _initialize(address newOwner) internal virtual onlyInitializing {

```

### [GAS-8] MAKING CONSTANT VARIABLES PRIVATE WILL SAVE GAS DURING DEPLOYMENT

#### Description:

When constants are marked public, extra getter functions are created, increasing the deployment cost. Marking these functions private will decrease gas cost. One can still read these variables through the source code. If they need to be accessed by an external contract, a separate single getter function can be used to return all constants as a tuple. There are four instances of public constants.

#### **Proof Of Concept**

```solidity
File: /contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol

39:     uint256 public constant RENOUNCE_OWNERSHIP_CONFIRMATION_DELAY = 200;

44:     uint256 public constant RENOUNCE_OWNERSHIP_CONFIRMATION_PERIOD = 200;

```

### [GAS-9] SAVE GAS BY NOT REQURING NON-ZERO INTERVAL IF NO LINEAR AMOUNT

#### Description:

If there is no linear amount, a Gsset for the claim’s interval can be converted to a Gsreset, saving 17100 gas.

#### **Proof Of Concept**

```solidity
File: /contracts/LSP14Ownable2Step/LSP14Errors.sol

7: error NotInRenounceOwnershipInterval(

```

```solidity
File: /contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol

170:             revert NotInRenounceOwnershipInterval(

```

### [GAS-10] The increment in for loop postcondition can be made unchecked

#### Description:

This is only relevant if you are using the default solidity checked arithmetic.
the for loop postcondition, i.e., `i++` involves checked arithmetic, which is not required. This is because the value of i is always strictly less than `length <= 2**256 - 1`. Therefore, the theoretical maximum value of i to enter the for-loop body is `2**256 - 2`. This means that the `i++` in the for loop can never overflow. Regardless, the overflow checks are performed by the compiler.

Unfortunately, the Solidity optimizer is not smart enough to detect this and remove the checks.One can manually do this.

[Source](https://forum.openzeppelin.com/t/a-collection-of-gas-optimisation-tricks/19966/6)

#### **Proof Of Concept**

```solidity
File: /contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol

156:                 inputDataKeysAllowed++;

743:                         allowedDataKeysFound++;

771:                 jj++;

```

### [GAS-11] Use a more recent version of solidity

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

### [GAS-12] TERNARY OPERATION IS CHEAPER THAN IF-ELSE STATEMENT

#### Description:

There are instances where a ternary operation can be used instead of if-else statement. In these cases, using ternary operation will save modest amounts of gas.
Note that this optimization seems to be dependent on usage of a more recent Solidity version. The following gas savings are on version 0.8.17.

#### **Proof Of Concept**

```solidity
File: /contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol

112:    if (tokenOwner == operator) {
            return _tokenOwnerBalances[tokenOwner];
        } else {
            return _operatorAuthorizedAmount[tokenOwner][operator];
        }

278:    if (amount != 0) {
            emit AuthorizedOperator(operator, tokenOwner, amount);
        } else {
            emit RevokedOperator(operator, tokenOwner);
        }

491:    if (to.code.length > 0) {
                revert LSP7NotifyTokenReceiverContractMissingLSP1Interface(to);
            } else {
                revert LSP7NotifyTokenReceiverIsEOA(to);
            }
```

```solidity
File: /contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol

493:       if (to.code.length > 0) {
                revert LSP8NotifyTokenReceiverContractMissingLSP1Interface(to);
            } else {
                revert LSP8NotifyTokenReceiverIsEOA(to);
            }

```

### [GAS-13] Usage of `uint`/`int` smaller than 32 bytes (256 bits) incurs overhead

#### Description:

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.
Each operation involving a `uint8` costs an extra 22-28 gas (depending on whether the other operand is also a variable of type `uint8`) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so. Use a larger size then downcast where needed.
https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html
Use a larger size then downcast where needed.

#### **Proof Of Concept**

```solidity
File: /contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol

336:             uint256 elementLength = uint16(

```

```solidity
File: /contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol

395:             allowedStandard == bytes4(type(uint32).max) ||

414:             allowedFunction == bytes4(type(uint32).max) ||

```

```solidity
File: /contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol

544:             length = uint16(

664:             length = uint16(

```

```solidity
File: /contracts/LSP7DigitalAsset/ILSP7DigitalAsset.sol

65:     function decimals() external view returns (uint8);

```

```solidity
File: /contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol

54:     function decimals() public view virtual returns (uint8) {

```

### [GAS-14] Use != 0 instead of > 0 for unsigned integer comparison

#### Description:

To update this with using at least 0.8.6 there is no difference in gas usage with!= 0 or > 0.
[Source](https://forum.openzeppelin.com/t/a-collection-of-gas-optimisation-tricks/19966/6)

#### **Proof Of Concept**

```solidity
File: /contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol

182:                 if (result.length > 0) {

741:         if (_owner.code.length > 0) {

```

```solidity
File: /contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol

165:             if (notifier.code.length > 0) {

```

```solidity
File: /contracts/LSP20CallVerification/LSP20CallVerification.sol

86:         if (returnedData.length > 0) {

```

```solidity
File: /contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol

491:             if (to.code.length > 0) {

```

```solidity
File: /contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol

493:             if (to.code.length > 0) {

```

### [GAS-15] USE BYTES32 INSTEAD OF STRING

#### Description:

Use bytes32 instead of string to save gas whenever possible. String is a dynamic data structure and therefore is more gas consuming then bytes32.

#### **Proof Of Concept**

```solidity
File: /contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol

23:         string memory keyName

33:         string memory keyName

72:         string memory firstWord,

73:         string memory lastWord

95:         string memory firstWord,

137:         string memory firstWord,

138:         string memory secondWord,

200:         string memory hashFunction,

201:         string memory json,

202:         string memory url

217:         string memory hashFunction,

218:         string memory assetBytes,

219:         string memory url

```

```solidity
File: /contracts/LSP4DigitalAssetMetadata/ILSP4Compatibility.sol

18:     function name() external view returns (string memory);

24:     function symbol() external view returns (string memory);

```

```solidity
File: /contracts/LSP4DigitalAssetMetadata/LSP4Compatibility.sol

25:     function name() public view virtual override returns (string memory) {

27:         return string(data);

34:     function symbol() public view virtual override returns (string memory) {

36:         return string(data);

```

```solidity
File: /contracts/LSP4DigitalAssetMetadata/LSP4DigitalAssetMetadata.sol

32:         string memory name_,

33:         string memory symbol_,

```

```solidity
File: /contracts/LSP4DigitalAssetMetadata/LSP4DigitalAssetMetadataInitAbstract.sol

28:         string memory name_,

29:         string memory symbol_,

```

```solidity
File: /contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol

445:             string memory permissionErrorString = LSP6Utils.getPermissionName(

```

```solidity
File: /contracts/LSP6KeyManager/LSP6Modules/LSP6OwnershipModule.sol

24:             string memory permissionErrorString = LSP6Utils.getPermissionName(

```

```solidity
File: /contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol

788:             string memory permissionErrorString = LSP6Utils.getPermissionName(

```

```solidity
File: /contracts/LSP7DigitalAsset/LSP7DigitalAsset.sol

40:         string memory name_,

41:         string memory symbol_,

```

```solidity
File: /contracts/LSP7DigitalAsset/LSP7DigitalAssetInitAbstract.sol

29:         string memory name_,

30:         string memory symbol_,

```

```solidity
File: /contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAsset.sol

37:         string memory name_,

38:         string memory symbol_,

```

```solidity
File: /contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetInitAbstract.sol

31:         string memory name_,

32:         string memory symbol_,

```

### [GAS-16] Public functions not called by the contract should be declared external instead OR Functions not used internally could be marked external

#### Description:

Contracts [are allowed](https://docs.soliditylang.org/en/latest/contracts.html#function-overriding) to override their parents’ functions and change the visibility from external to public and can save gas by doing so.

#### **Proof Of Concept**

```solidity
File: /contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol

171:     function batchCalls(

```
