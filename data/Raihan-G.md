# Gas optimization

# summry
|      |   ISSUE   |  INSTANCE  |
|------|-----------|------------|
|[G-01]|Using calldata instead of memory for read-only arguments in external functions saves gas|6|
|[G-02]|State variables only set in the constructor should be declared immutable|2|
|[G-03]|Caching global variables is more expensive than using the actual variable (use msg.sender instead of caching it)|10|
|[G-04]|Use Modifiers Instead of Functions To Save Gas|2|
|[G-05]| Use != 0 instead of > 0 for unsigned integer comparison|8|
|[G-06]| Use nested if statements instead of &&|14|
|[G-07]| Use Assembly To Check For address(0)|29|
|[G-08]| Can Make The Variable Outside The Loop To Save Gas|4|
|[G-09]| internal functions only called once can be inlined to save gas|37|
|[G-10]| Gas saving is achieved by removing the delete keyword (~60k)|4|
|[G-11]| With assembly, .call (bool success) transfer can be done gas-optimized|2|
|[G-12]|Unnecessary computation|1|
|[G-13]|Use assembly to hash instead of Solidity|11|
|[G-14]|Duplicated require()/if() checks should be refactored to a modifier or function|15|
|[G-15]|Use hardcode address instead address(this)|4|
|[G-16]|abi.encode() is less efficient than abi.encodePacked()|3|
|[G-17]|>= costs less gas than >|11|
|[G-18]|Multiple accesses of a mapping/array should use a local variable cache|8|
|[G-19]|Empty blocks should be removed or emit something|2|
|[G-20]|Uncheck arithmetics operations that can’t underflow/overflow|22|
|[G-21]|Use constants instead of type(uintx).max|8|
|[G-22]|Access mappings directly rather than using accessor functions|4|
|[G-23]|Use assembly to emit events|1|
|[G-24]|Use uint256(1)/uint256(2) instead for true and false boolean states|1|

## [G-01] Using calldata instead of memory for read-only arguments in external functions saves gas
When a function with a memory array is called externally, the abi.decode() step has to use a for-loop to copy each index of the calldata to the memory index. Each iteration of this for-loop costs at least 60 gas (i.e. 60 * <mem_array>.length). Using calldata directly, obliviates the need for such a loop in the contract code and runtime execution. Structs have the same overhead as an array of length one

```solidity
File: /contracts/interfaces/IERC725X.sol
93   function executeBatch(
        uint256[] memory operationsType,
        address[] memory targets,
        uint256[] memory values,
        bytes[] memory datas
    ) external payable returns (bytes[] memory);
```
https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/interfaces/IERC725X.sol#L93

```solidity
File: /contracts/interfaces/IERC725Y.sol
35  function getDataBatch(
        bytes32[] memory dataKeys
    ) external view returns (bytes[] memory dataValues);

52   function setData(bytes32 dataKey, bytes memory dataValue) external payable;

67   function setDataBatch(
        bytes32[] memory dataKeys,
        bytes[] memory dataValues
    ) external payable;    
```
https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/interfaces/IERC725Y.sol#L35

```solidity
File: /contracts/LSP20CallVerification/ILSP20CallVerifier.sol
19  function lsp20VerifyCall(
        address caller,
        uint256 value,
        bytes memory receivedCalldata
    ) external returns (bytes4 magicValue);

31    function lsp20VerifyCallResult(
        bytes32 callHash,
        bytes memory result
    ) external returns (bytes4);    
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP20CallVerification/ILSP20CallVerifier.sol#L19

## [G‑02] State variables only set in the constructor should be declared immutable
Avoids a Gsset (20000 gas) in the constructor, and replaces the first access in each transaction (Gcoldsload - 2100 gas) and each access thereafter (Gwarmacces - 100 gas) with a PUSH32 (3 gas).


INHERITED (address) StateVar LSP6KeyManagerCore._target (Declaration: LSP6KeyManagerCore#79)
```solidity
File: /contracts/LSP6KeyManager/ILSP6KeyManager.sol
20   _target = target_;
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/ILSP6KeyManager.sol#L20


INHERITED (bool) StateVar LSP7DigitalAssetCore._isNonDivisible (Declaration: LSP7DigitalAssetCore#47)
```solidity
File: /contracts/LSP7DigitalAsset/LSP7DigitalAsset.sol
45   _isNonDivisible = isNonDivisible_;
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/LSP7DigitalAsset.sol#L45

## [G-03] Caching global variables is more expensive than using the actual variable (use msg.sender instead of caching it)
[Reffrence](https://code4rena.com/reports/2022-12-forgeries#g-07-caching-global-variables-is-more-expensive-than-using-the-actual-variable-use-msgsender-instead-of-caching-it)

```solidity
File: /contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol
133   address operator = msg.sender;

304   address operator = msg.sender;

352   address operator = msg.sender;

315   address operator = msg.sender;
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L133


```solidity
File: /contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol
233  address operator = msg.sender;
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol#L233

```solidity
File: /contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol
233  address operator = msg.sender;
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol#L233

```solidity
File: /contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol
198   address operator = msg.sender;

327   address operator = msg.sender;

361   address operator = msg.sender;

414   address operator = msg.sender;
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L198


## [G-04]Use Modifiers Instead of Functions To Save Gas
Example of two contracts with modifiers and internal view function:
```
// SPDX-License-Identifier: MIT
pragma solidity 0.8.9;
contract Inlined {
    function isNotExpired(bool _true) internal view {
        require(_true == true, "Exchange: EXPIRED");
    }
function foo(bool _test) public returns(uint){
            isNotExpired(_test);
            return 1;
    }
}
// SPDX-License-Identifier: MIT
pragma solidity 0.8.9;
contract Modifier {
modifier isNotExpired(bool _true) {
        require(_true == true, "Exchange: EXPIRED");
        _;
    }
function foo(bool _test) public isNotExpired(_test)returns(uint){
        return 1;
    }
}
```
Differences:

Deploy Modifier.sol
108727
Deploy Inlined.sol
110473
Modifier.foo
21532
Inlined.foo
21556


```solidity
File: /contracts/LSP6KeyManager/LSP6KeyManagerCore.sol
527 function _nonReentrantBefore(
        bool isSetData,
        address from
    ) internal virtual returns (bool isReentrantCall) {
        isReentrantCall = _reentrancyStatus;
        if (isReentrantCall) {
            // CHECK the caller has REENTRANCY permission
            _requirePermissions(
                from,
                ERC725Y(_target).getPermissionsFor(from),
                _PERMISSION_REENTRANCY
            );
        } else {
            if (!isSetData) {
                _reentrancyStatus = true;
            }
        }
    }

550 function _nonReentrantAfter() internal virtual {
        // By storing the original value once again, a refund is triggered (see
        // https://eips.ethereum.org/EIPS/eip-2200)
        _reentrancyStatus = false;
    }      
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L527-544

## [G-05] Use != 0 instead of > 0 for unsigned integer comparison
it's generally more gas-efficient to use != 0 instead of > 0 when
comparing unsigned integer types.

This is because the Solidity compiler can optimize the != 0 comparison to a simple bitwise operation,
 while the > 0 comparison requires an additional subtraction operation.
  As a result, using != 0 can be more gas-efficient and can help to reduce the overall cost of your contract.

Here's an example of how you can use != 0 instead of > 0:

```
contract MyContract {
    uint256 public myUnsignedInteger;
    
    function myFunction() public view returns (bool) {
        // Use != 0 instead of > 0
        return myUnsignedInteger != 0;
    }
}
```


```solidity
File: /contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol
182   if (result.length > 0) {

741   if (_owner.code.length > 0) {    
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L182


```solidity
File: /contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol
165   if (notifier.code.length > 0) {
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol#L165


```solidity
File: /contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol
491   if (to.code.length > 0) {
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L491

```solidity
File: /contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol
313    if (to.code.length > 0) {
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol#L313

```solidity
File: /contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol
313   if (to.code.length > 0) {
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol#L313

```solidity
File: /contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol
493   if (to.code.length > 0) {
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L493

```solidity
File: /contracts/LSP20CallVerification/LSP20CallVerification.sol
86    if (returnedData.length > 0) {
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP20CallVerification/LSP20CallVerification.sol#L86


## [G-06] Use nested if statements instead of &&
If the if statement has a logical AND and is not followed by an else statement, it can be replaced with 2 if statements.
```
contract NestedIfTest {

    //Execution cost: 22334 gas
    function funcBad(uint256 input) public pure returns (string memory) { 
       if (input<10 && input>0 && input!=6){ 
           return "If condition passed";
       } 

   }

    //Execution cost: 22294 gas
    function funcGood(uint256 input) public pure returns (string memory) { 
    if (input<10) { 
        if (input>0){
            if (input!=6){
                return "If condition passed";
            }
        }
    }
   }
}
   
```

```solidity
File: /contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol
801   if (msg.sig == bytes4(0) && extension == address(0)) return;
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L801

```solidity
File: /contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol
227   if (dataKeys.length == 0 && dataValues.length == 0)

245   if (dataKeys.length == 0 && dataValues.length == 0)
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol#L227

```solidity
File: /contracts/LSP5ReceivedAssets/LSP5Utils.sol
78     if (encodedArrayLength.length != 0 && encodedArrayLength.length != 16) {
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP5ReceivedAssets/LSP5Utils.sol#L78

```solidity
File: /contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol
165   if (isFundingContract && !hasSuperTransferValue) {

214   if (isTransferringValue && !hasSuperTransferValue) {

224    if (!hasSuperCall && !isCallDataPresent && !isTransferringValue) {

228   if (isCallDataPresent && !hasSuperCall) {

236   if (hasSuperTransferValue && !isCallDataPresent && isTransferringValue)

240   if (hasSuperCall && hasSuperTransferValue) return;
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L165

```solidity
File: /contracts/LSP6KeyManager/LSP6KeyManagerCore.sol
338   if (!isReentrantCall && !isSetData) {

404   if (!isReentrantCall && !isSetData) {    
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L338

```solidity
File: /contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol
362    if (inputDataValue.length != 0 && inputDataValue.length != 20) {
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L362

```solidity
File: /contracts/LSP10ReceivedVaults/LSP10Utils.sol
76  if (encodedArrayLength.length != 0 && encodedArrayLength.length != 16) {
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP10ReceivedVaults/LSP10Utils.sol#L76

## [G-07] Use Assembly To Check For address(0)
it's generally more gas-efficient to use assembly to check for a zero address (address(0)) than to use the == operator.

The reason for this is that the == operator generates additional instructions in the EVM bytecode, which can increase the gas cost of your contract. By using assembly, you can perform the zero address check more efficiently and reduce the overall gas cost of your contract.

Here's an example of how you can use assembly to check for a zero address:

```
contract MyContract {
    function isZeroAddress(address addr) public pure returns (bool) {
        uint256 addrInt = uint256(addr);
        
        assembly {
            // Load the zero address into memory
            let zero := mload(0x00)
            
            // Compare the address to the zero address
            let isZero := eq(addrInt, zero)
            
            // Return the result
            mstore(0x00, isZero)
            return(0, 0x20)
        }
    }
}
```
In the above example, we have a function isZeroAddress that takes an address as input and returns a boolean value indicating whether the address is equal to the zero address. Inside the function, we convert the address to an integer using uint256(addr), and then use assembly to compare the integer to the zero address.

By using assembly to perform the zero address check, we can make our code more gas-efficient and reduce the overall cost of our contract. It's important to note that assembly can be more difficult to read and maintain than Solidity code, so it should be used with caution and only when necessary


```solidity
File: /ERC725/blob/v5.1.0/implementations/contracts/ERC725.sol
28   newOwner != address(0),
```
https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725.sol#L28

```solidity
File: /ERC725/blob/v5.1.0/implementations/contracts/ERC725InitAbstract
28   newOwner != address(0),
```
https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725InitAbstract


```solidity
File: /ERC725/blob/v5.1.0/implementations/contracts/ERC725X.sol
22    newOwner != address(0),
```
https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725X.sol#L22

```solidity
File: /ERC725/blob/v5.1.0/implementations/contracts/ERC725XCore.sol
89   if (target != address(0))

96   if (target != address(0))

296  if (contractAddress == address(0)) {
```
https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725XCore.sol#L89

```solidity
File: /ERC725/blob/v5.1.0/implementations/contracts/ERC725XInitAbstract.sol
21    newOwner != address(0),
```
https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725XInitAbstract.sol#L21

```solidity
File: /ERC725/blob/v5.1.0/implementations/contracts/ERC725Y.sol
23    newOwner != address(0),
```
https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725Y.sol#L23

```solidity
File: /ERC725/blob/v5.1.0/implementations/contracts/ERC725YInitAbstract.sol
21    newOwner != address(0),
```
https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725YInitAbstract.sol#L21

```solidity
File: /contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol
801   if (msg.sig == bytes4(0) && extension == address(0)) return;

804   if (extension == address(0))
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L801


```solidity
19   if (target_ == address(0)) revert InvalidLSP6Target();
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManager.sol#L19


```solidity
File: /contracts/LSP6KeyManager/LSP6KeyManagerInitAbstract.sol
21    if (target_ == address(0)) revert InvalidLSP6Target();
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerInitAbstract.sol#L21



```solidity
File: /contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol
268   if (operator == address(0)) {

300   if (to == address(0)) {

343   if (from == address(0)) {

406   if (from == address(0) || to == address(0)) {            
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L268

```solidity
File: /contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8Enumerable.sol
36   if (from == address(0)) {

40   } else if (to == address(0)) {    
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8Enumerable.sol#L36

```solidity
File: /contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8EnumerableInitAbstract.sol
38    if (from == address(0)) {

42    } else if (to == address(0)) {    
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8EnumerableInitAbstract.sol#L38

```solidity
File: /contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol
84   if (tokenOwner == address(0)) {

115  if (operator == address(0)) {

139  if (operator == address(0)) {

319  if (to == address(0)) {

410  if (to == address(0)) {                
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L84



```solidity
File: 
50   if (erc165Extension == address(0)) return false;

91   if (extension == address(0))
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP17ContractExtension/LSP17Extendable.sol#L50

```solidity
File: /ERC725/blob/v5.1.0/implementations/contracts/custom/OwnableUnset.sol
51    newOwner != address(0),
```
https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/custom/OwnableUnset.sol#L51

## [G-08] Can Make The Variable Outside The Loop To Save Gas 
When you declare a variable inside a loop, Solidity creates a new instance of the variable for each iteration of the loop. This can lead to unnecessary gas costs, especially if the loop is executed frequently or iterates over a large number of elements.

By declaring the variable outside the loop, you can avoid the creation of multiple instances of the variable and reduce the gas cost of your contract. Here's an example:

```
contract MyContract {
    function sum(uint256[] memory values) public pure returns (uint256) {
        uint256 total = 0;
        
        for (uint256 i = 0; i < values.length; i++) {
            total += values[i];
        }
        
        return total;
    }
}
```

```solidity
File: /contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol
176   (bool success, bytes memory result) = address(this).delegatecall(
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L176


```solidity
File: /contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol
262   bytes32 key = data.toBytes32(pointer);

293   bytes32 key = data.toBytes32(pointer);
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol#L262


```solidity
File: /contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol
275   address operator = operatorsForTokenId.at(0);
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L275

## [G‑09] internal functions only called once can be inlined to save gas

```solidity
File: /ERC725/blob/v5.1.0/implementations/contracts/ERC725XCore.sol
131    function _executeBatch(
        uint256[] memory operationsType,
        address[] memory targets,
        uint256[] memory values,
        bytes[] memory datas
    ) internal virtual returns (bytes[] memory) {  


174   function _executeCall(
        address target,
        uint256 value,
        bytes memory data
    ) internal virtual returns (bytes memory result) {

203   function _executeStaticCall(
        address target,
        bytes memory data
    ) internal virtual returns (bytes memory result) {


225     function _executeDelegateCall(
        address target,
        bytes memory data
    ) internal virtual returns (bytes memory result) {

247  function _deployCreate(
        uint256 value,
        bytes memory creationCode
    ) internal virtual returns (bytes memory newContract) {

288   function _deployCreate2(
        uint256 value,
        bytes memory creationCode
    ) internal virtual returns (bytes memory newContract) {                
```
https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725XCore.sol#L131

```solidity
File: /contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol
796   function _fallbackLSP17Extendable() internal virtual override {

851   function _getExtension(
        bytes4 functionSelector
    ) internal view virtual override returns (address) {

```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L796

```solidity
File: /contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol
152    function _whenReceiving(
        bytes32 typeId,
        address notifier,
        bytes32 notifierMapKey,
        bytes4 interfaceID
    ) internal virtual returns (bytes memory) {

201   function _whenSending(
        bytes32 typeId,
        address notifier,
        bytes32 notifierMapKey,
        uint128 arrayIndex
    ) internal virtual returns (bytes memory) {        
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol#L152



```solidity
File: /contracts/LSP4DigitalAssetMetadata/LSP4DigitalAssetMetadataInitAbstract.sol
27   function _initialize(
        string memory name_,
        string memory symbol_,
        address newOwner_
    ) internal virtual onlyInitializing {
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP4DigitalAssetMetadata/LSP4DigitalAssetMetadataInitAbstract.sol#L27


```solidity
File: /contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol
153   function _verifyCanDeployContract(
        address controller,
        bytes32 permissions,
        bool isFundingContract
    ) internal view virtual {

170   function _verifyCanStaticCall(
        address controlledContract,
        address controller,
        bytes32 permissions,
        bytes calldata payload
    ) internal view virtual {

188   function _verifyCanCall(
        address controlledContract,
        address controller,
        bytes32 permissions,
        bytes calldata payload
    ) internal view virtual {

317    function _extractCallType(
        uint256 operationType,
        uint256 value,
        bool isEmptyCall
    ) internal pure returns (bytes4 requiredCallTypes) {

339   function _extractExecuteParameters(
        bytes calldata executeCalldata
    ) internal pure returns (uint256, address, uint256, bytes4, bool) {


366   function _isAllowedAddress(
        bytes memory allowedCall,
        address to
    ) internal pure returns (bool) {

382     function _isAllowedStandard(
        bytes memory allowedCall,
        address to
    ) internal view returns (bool) {

399   function _isAllowedFunction(
        bytes memory allowedCall,
        bytes4 requiredFunction
    ) internal pure returns (bool) {

418  function _isAllowedCallType(
        bytes memory allowedCall,
        bytes4 requiredCallTypes
    ) internal pure returns (bool) {


```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L153


```solidity
File: /contracts/LSP6KeyManager/LSP6KeyManagerCore.sol
441       function _isValidNonce(
        address from,
        uint256 idx
    ) internal view virtual returns (bool) {       
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L441

```solidity
File: /contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol
334   function _getPermissionToSetPermissionsArray(
        address controlledContract,
        bytes32 inputDataKey,
        bytes memory inputDataValue,
        bool hasBothAddControllerAndEditPermissions
    ) internal view virtual returns (bytes32) {

385   function _getPermissionToSetControllerPermissions(
        address controlledContract,
        bytes32 inputPermissionDataKey
    ) internal view virtual returns (bytes32) {

406   function _getPermissionToSetAllowedCalls(
        address controlledContract,
        bytes32 dataKey,
        bytes memory dataValue,
        bool hasBothAddControllerAndEditPermissions
    ) internal view virtual returns (bytes32) {

438  function _getPermissionToSetAllowedERC725YDataKeys(
        address controlledContract,
        bytes32 dataKey,
        bytes memory dataValue,
        bool hasBothAddControllerAndEditPermissions
    ) internal view returns (bytes32) {

474   function _getPermissionToSetLSP1Delegate(
        address controlledContract,
        bytes32 lsp1DelegateDataKey
    ) internal view virtual returns (bytes32) {

490   function _getPermissionToSetLSP17Extension(
        address controlledContract,
        bytes32 lsp17ExtensionDataKey
    ) internal view virtual returns (bytes32) {

507  function _verifyAllowedERC725YSingleKey(
        address controllerAddress,
        bytes32 inputDataKey,
        bytes memory allowedERC725YDataKeysCompacted
    ) internal pure virtual {

622    function _verifyAllowedERC725YDataKeys(
        address controllerAddress,
        bytes32[] memory inputDataKeys,
        bytes memory allowedERC725YDataKeysCompacted,
        bool[] memory validatedInputKeysList,
        uint256 allowedDataKeysFound
    ) internal pure virtual {


```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L334



```solidity
File: /contracts/LSP7DigitalAsset/extensions/LSP7CappedSupply.sol
53   function _mint(
        address to,
        uint256 amount,
        bool allowNonLSP1Recipient,
        bytes memory data
    ) internal virtual override {
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/extensions/LSP7CappedSupply.sol#L53

```solidity
File: 
107  function _updateOperator(
        address tokenOwner,
        address operator,
        uint256 amount
    ) internal virtual override {

116   function _transfer(
        address from,
        address to,
        uint256 amount,
        bool allowNonLSP1Recipient,
        bytes memory data
    ) internal virtual override {

127       function _mint(
        address to,
        uint256 amount,
        bool allowNonLSP1Recipient,
        bytes memory data
    ) internal virtual override {

137   function _burn(
        address from,
        uint256 amount,
        bytes memory data
    ) internal virtual override {

146  function _setData(
        bytes32 key,
        bytes memory value
    ) internal virtual override(LSP4DigitalAssetMetadata, ERC725YCore) {                                
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/extensions/LSP7CompatibleERC20.sol#L107

```solidity
File: /contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol
399   function _transfer(
        address from,
        address to,
        uint256 amount,
        bool allowNonLSP1Recipient,
        bytes memory data
    ) internal virtual {
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L399

```solidity
File: /contracts/LSP7DigitalAsset/LSP7DigitalAssetInitAbstract.sol
28   function _initialize(
        string memory name_,
        string memory symbol_,
        address newOwner_,
        bool isNonDivisible_
    ) internal virtual onlyInitializing {
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetInitAbstract.sol#L28

## [G-10] Gas saving is achieved by removing the delete keyword (~60k)
30k gas savings were made by removing the delete keyword. The reason for using the delete keyword here is to reset the struct values (set to default value) in every operation. However, the struct values do not need to be zero each time the function is run. Therefore, the delete” key word is unnecessary. If it is removed, around 30k gas savings will be achieved.
[Reffrence](https://code4rena.com/reports/2023-01-timeswap#g-01-gas-saving-is-achieved-by-removing-the-delete-keyword-60k)

```solidity
File: /contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol
130    delete _renounceOwnershipStartedAt;

143    delete _pendingOwner;

164    delete _pendingOwner;

177    delete _renounceOwnershipStartedAt;
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol#L130

## [G-11] With assembly, .call (bool success) transfer can be done gas-optimized
return data (bool success,) has to be stored due to EVM architecture, but in a usage like below, ‘out’ and ‘outsize’ values are given (0,0), this storage disappears and gas optimization is provided.
-   (bool success,) = dest.call{value:amount}("");
 bool success; 
 assembly {  
            success := call(gas(), dest, amount, 0, 0)
     }  
 [Reffrence](https://code4rena.com/reports/2023-01-biconomy#g-01-with-assembly-call-bool-success-transfer-can-be-done-gas-optimized)    
 ```solidity
 186     (bool success, bytes memory returnData) = target.call{value: value}(
            data
        );
 ```
 https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725XCore.sol#L186
 
 
 ```solidity
File: /contracts/LSP6KeyManager/LSP6KeyManagerCore.sol
420   (bool success, bytes memory returnData) = _target.call{
            value: msgValue,
            gas: gasleft()
        }(payload);
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L420

## [G-12] Unnecessary computation
When emitting an event that includes a new and an old value, it is cheaper in gas to avoid caching the old value in memory. Instead, emit the event, then save the new value in storage.

Proof of Concept
Instances include:

```
OwnableProxyDelegation.sol
function _setOwner
Recommended Mitigation
```
Replace
```
address oldOwner = _owner;
_owner = newOwner;
emit OwnershipTransferred(oldOwner, newOwner)
```
with
```
emit OwnershipTransferred(_owner_, newOwner)
_owner = newOwner;
```


```solidity
File: /ERC725/blob/v5.1.0/implementations/contracts/custom/OwnableUnset.sol
             address oldOwner = _owner;
            _owner = newOwner;
            emit OwnershipTransferred(oldOwner, newOwner);
```
https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/custom/OwnableUnset.sol#L70-#L72


## [G-13] Use assembly to hash instead of Solidity

```
contract GasTest is DSTest {
  Contract0 c0;
  Contract1 c1;
  function setUp() public {
      c0 = new Contract0();
      c1 = new Contract1();
  }
  function testGas() public view {
      c0.solidityHash(2309349, 2304923409);
      c1.assemblyHash(2309349, 2304923409);
  }
}
contract Contract0 {
  function solidityHash(uint256 a, uint256 b) public view {
      //unoptimized
      keccak256(abi.encodePacked(a, b));
  }
}
contract Contract1 {
  function assemblyHash(uint256 a, uint256 b) public view {
      //optimized
      assembly {
          mstore(0x00, a)
          mstore(0x20, b)
          let hashedVal := keccak256(0x00, 0x40)
      }
  }
}

```
```solidity
File: /contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol
25    return keccak256(bytes(keyName));

43    return keccak256(dataKey);

75   bytes32 firstWordHash = keccak256(bytes(firstWord));

76   bytes32 lastWordHash = keccak256(bytes(lastWord));

98   bytes32 firstWordHash = keccak256(bytes(firstWord));

141   bytes32 firstWordHash = keccak256(bytes(firstWord));

142  bytes32 secondWordHash = keccak256(bytes(secondWord));

204   bytes32 hashFunctionDigest = keccak256(bytes(hashFunction));

205   bytes32 jsonDigest = keccak256(bytes(json));

221   bytes32 hashFunctionDigest = keccak256(bytes(hashFunction));

222  bytes32 jsonDigest = keccak256(bytes(assetBytes));
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol#L25

## [G-14] Duplicated require()/if() checks should be refactored to a modifier or function
sing modifiers or functions can make your code more gas-efficient by reducing the overall number of operations that need to be executed. For example, if you have a complex validation check that involves multiple operations, and you refactor it into a function, then the function can be executed with a single opcode, rather than having to execute each operation separately in multiple locations.

Recommendation
You can consider adding a modifier like below
```
 modifer check (address checkToAddress) {    
     require(checkToAddress != address(0) && checkToAddress != SENTINEL_MODULES, "BSA101");  
      _; 
 }
```
```solidity
File: /ERC725/blob/v5.1.0/implementations/contracts/ERC725XCore.sol
89   if (target != address(0))

96   if (target != address(0))


179  if (address(this).balance < value) {

251  if (address(this).balance < value) {   
```
https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725XCore.sol#L89

```solidity
File: /ERC725/blob/v5.1.0/implementations/contracts/ERC725YCore.sol
66  if (msg.value != 0) revert ERC725Y_MsgValueDisallowed();

78  if (msg.value != 0) revert ERC725Y_MsgValueDisallowed();
```
https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725YCore.sol#L66


```solidity
File: /contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol
118  if (msg.value != 0) {

152  if (msg.value != 0) {

228  if (msg.value != 0) {

286  if (msg.value != 0) {

343  if (msg.value != 0) {

384  if (msg.value != 0) {

464  if (msg.value != 0) { 

235  if (msg.sender == _owner) {

293  if (msg.sender == _owner) {                               
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L118

## [G-15] Use hardcode address instead address(this)
it can be more gas-efficient to use a hardcoded address instead of the address(this) expression, especially if you need to use the same address multiple times in your contract.

The reason for this is that using address(this) requires an additional EXTCODESIZE operation to retrieve the contract's address from its bytecode, which can increase the gas cost of your contract. By pre-calculating and using a hardcoded address, you can avoid this additional operation and reduce the overall gas cost of your contract.

Here's an example of how you can use a hardcoded address instead of address(this):

```
contract MyContract {
    address public myAddress = 0x1234567890123456789012345678901234567890;
    
    function doSomething() public {
        // Use myAddress instead of address(this)
        require(msg.sender == myAddress, "Caller is not authorized");
        
        // Do something
    }
}
```
In the above example, we have a contract MyContract with a public address variable myAddress. Instead of using address(this) to retrieve the contract's address, we have pre-calculated and hardcoded the address in the variable. This can help to reduce the gas cost of our contract and make our code more efficient.

[References](https://book.getfoundry.sh/reference/forge-std/compute-create-address)

```solidity
File: /ERC725/blob/v5.1.0/implementations/contracts/ERC725XCore.sol
179   if (address(this).balance < value) {

180   revert ERC725X_InsufficientBalance(address(this).balance, value);

251   if (address(this).balance < value) {

252   revert ERC725X_InsufficientBalance(address(this).balance, value);    
```
https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725XCore.sol#L179

## [G-16] abi.encode() is less efficient than abi.encodePacked()
In terms of efficiency, abi.encodePacked() is generally considered to be more gas-efficient than abi.encode(), because it skips the step of adding function signatures and other metadata to the encoded data. However, this comes at the cost of reduced safety, as abi.encodePacked() does not perform any type checking or padding of data.

```solidity
File: /contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol
253    LSP20CallVerification._verifyCallResult(_owner, abi.encode(result));

319    abi.encode(results)

528    returnedValues = abi.encode(
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L253

## [G-17] >= costs less gas than >
The compiler uses opcodes GT and ISZERO for solidity code that uses >, but only requires LT for >=, which saves 3 gas

```solidity
File: /contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol
182   if (result.length > 0) {

741   if (_owner.code.length > 0) {    
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L182



```solidity
File: /contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol
165   if (notifier.code.length > 0) {
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol#L165


```solidity
File: /contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol
284   if (ii + 34 > allowedCalls.length) {
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L284

```solidity
File: /contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol
559   if (length > 32)

679   if (length > 32)
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L559


```solidity
File: /contracts/LSP6KeyManager/LSP6Utils.sol
137   if (elementLength == 0 || elementLength > 32) return false;
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Utils.sol#L137

```solidity
File: /contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol
136   if (amount > operatorAmount) {

348    if (amount > balance) {

357    if (amount > authorizedAmount) {

491   if (to.code.length > 0) {            
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L136

## [G‑18] Multiple accesses of a mapping/array should use a local variable cache
The instances below point to the second+ access of a value inside a mapping/array, within a function. Caching a mapping’s value in a local storage or calldata variable when the value is accessed multiple times, saves ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations. Caching an array’s struct avoids recalculating the array offsets into memory/calldata.

```solidity
File: /contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8Enumerable.sol
38   _indexToken[index] = tokenId;

44   bytes32 lastTokenId = _indexToken[lastIndex];

45   _indexToken[index] = lastTokenId;

48   delete _indexToken[lastIndex];



39   _tokenIndex[tokenId] = index;

42   uint256 index = _tokenIndex[tokenId];

46   _tokenIndex[lastTokenId] = index;

49   delete _tokenIndex[tokenId];
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8Enumerable.sol#L38


## [G-19] Empty blocks should be removed or emit something
The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting. If the contract is meant to be extended, the contract should be abstract and the function signatures be added without any default implementation. If the block is an empty if-statement block to avoid doing subsequent checks in the else-if/else conditions, the else-if/else conditions should be nested under the negation of the if-statement, because they involve different classes of checks, which may lead to the introduction of errors when the code is later modified (if(x){}else if(y){...}else{...} => if(!x){if(y){...}else{...}})

[Reffrence](https://code4rena.com/reports/2022-05-sturdy#g-15-empty-blocks-should-be-removed-or-emit-something)

```solidity
File: /contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol
442    function _beforeTokenTransfer(
        address from,
        address to,
        uint256 amount
    ) internal virtual {}
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L442

```solidity
File: /contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol
444  function _beforeTokenTransfer(
        address from,
        address to,
        bytes32 tokenId
    ) internal virtual {}
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L444

## [G-20] Uncheck arithmetics operations that can’t underflow/overflow

Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an [unchecked block](https://docs.soliditylang.org/en/v0.8.10/control-structures.html#checked-or-unchecked-arithmetic)

Replace this:
```
 uint256 value = yield(a, b, c - totalFee(c), address(this));
```
with this:
```
 unchecked {uint256 value = yield(a, b, c - totalFee(c), address(this));}
```

```solidity
File: /contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol
269  pointer += 32;

299  pointer += 32;

344  pointer += elementLength + 2;
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol#L269


```solidity
File:  /contracts/LSP5ReceivedAssets/LSP5Utils.sol
137  uint128 newArrayLength = oldArrayLength - 1; 
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP5ReceivedAssets/LSP5Utils.sol#L137

```solidity
File: /contracts/LSP6KeyManager/LSP6Utils.sol
108   pointer += elementLength + 2;

138   pointer += elementLength + 2;

174   result += uint256(permissions[i]); 
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Utils.sol#L137


```solidity
File: /contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol
145   _updateOperator(from, operator, operatorAmount - amount);  

309    _existingTokens += amount;

311    _tokenOwnerBalances[to] += amount;

365    _operatorAuthorizedAmount[from][operator] -= amount;

371    _existingTokens -= amount;

373    _tokenOwnerBalances[from] -= amount;

419     _tokenOwnerBalances[from] -= amount;

420    _tokenOwnerBalances[to] += amount;
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L145

```solidity
File: /contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol
102  bytes memory uriBytes = BytesLib.slice(
            data,
            offset,
            data.length - offset
        );
138    return operatorsForTokenId[operatorListLength - 1];        
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol#L102

```solidity
File: /contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol
102    bytes memory uriBytes = BytesLib.slice(
            data,
            offset,
            data.length - offset
        );

138   return operatorsForTokenId[operatorListLength - 1];
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol#L102

```solidity
File: /contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8Enumerable.sol
41   uint256 lastIndex = totalSupply() - 1;      
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8Enumerable.sol#L41

```solidity
File: /contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8EnumerableInitAbstract.sol
43   uint256 lastIndex = totalSupply() - 1;
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8EnumerableInitAbstract.sol#L43

```solidity
File: /contracts/LSP10ReceivedVaults/LSP10Utils.sol
140   uint128 newArrayLength = oldArrayLength - 1;
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP10ReceivedVaults/LSP10Utils.sol#L140


## [G-21] Use constants instead of type(uintx).max
 it's generally more gas-efficient to use constants instead of type(uintX).max when you need to set the maximum value of an unsigned integer type.

The reason for this is that the type(uintX).max expression involves a computation at runtime, whereas a constant is evaluated at compile-time. This means that using type(uintX).max can result in additional gas costs for each transaction that involves the expression.

By using a constant instead of type(uintX).max, you can avoid these additional gas costs and make your code more efficient.

Here's an example of how you can use a constant instead of type(uintX).max:
```
contract MyContract {
    uint120 constant MAX_VALUE = 2**120 - 1;
    
    function doSomething(uint120 value) public {
        require(value <= MAX_VALUE, "Value exceeds maximum");
        
        // Do something
    }
}
```
In the above example, we have a contract with a constant MAX_VALUE that represents the maximum value of a uint120. When the doSomething function is called with a value parameter, it checks whether the value is less than or equal to MAX_VALUE using the <= operator.

By using a constant instead of type(uint120).max, we can make our code more efficient and reduce the gas cost of our contract.

It's important to note that using constants can make your code more readable and maintainable, since the value is defined in one place and can be easily updated if necessary. However, constants should be used with caution and only when their value is known at compile-time.


```solidity
File: /contracts/LSP5ReceivedAssets/LSP5Utils.sol
87   if (oldArrayLength == type(uint128).max) {

190  if (newArrayLength >= type(uint128).max) {    
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP5ReceivedAssets/LSP5Utils.sol#L87

```solidity
File: /contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol
296   bytes28(type(uint224).max)

378   allowedAddress == address(bytes20(type(uint160).max)) ||

395   allowedStandard == bytes4(type(uint32).max) ||

414   allowedFunction == bytes4(type(uint32).max) ||
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L296

```solidity
File: /contracts/LSP10ReceivedVaults/LSP10Utils.sol
85   if (oldArrayLength == type(uint128).max) {

135  if (oldArrayLength > type(uint128).max) {    
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP10ReceivedVaults/LSP10Utils.sol#L85

## [G-22] Access mappings directly rather than using accessor functions
Saves having to do two JUMP instructions, along with stack setup

```solidity
File: /ERC725/blob/v5.1.0/implementations/contracts/ERC725YCore.sol
99   return _store[dataKey];
```
https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725YCore.sol#L99

```solidity
File: /contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8Enumerable.sol
28   return _indexToken[index];
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8Enumerable.sol#L28

```solidity
File: /contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8EnumerableInitAbstract.sol
30   return _indexToken[index];
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8EnumerableInitAbstract.sol#L30

```solidity
File: /contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol
73   return _tokenOwnerBalances[tokenOwner];
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L73


## [G‑23] Use assembly to emit events
We can use assembly to emit events efficiently by utilizing scratch space and the free memory pointer. This will allow us to potentially avoid memory expansion costs. Note: In order to do this optimization safely, we will need to cache and restore the free memory pointer.

For example, for a generic emit event for eventSentAmountExample:
```
 // uint256 id, uint256 value, uint256 amount
emit eventSentAmountExample(id, value, amount);
```

We can use the following assembly emit events:
```
   assembly {
            let memptr := mload(0x40)
            mstore(0x00, calldataload(0x44))
            mstore(0x20, calldataload(0xa4))
            mstore(0x40, amount)
            log1(
                0x00,
                0x60,
                // keccak256("eventSentAmountExample(uint256,uint256,uint256)")
                0xa622cf392588fbf2cd020ff96b2f4ebd9c76d7a4bc7f3e6b2f18012312e76bc3
            )
            mstore(0x40, memptr)
        }
```        
```solidity
File: /contracts/custom/OwnableUnset.sol
72   emit OwnershipTransferred(oldOwner, newOwner);
```
https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/custom/OwnableUnset.sol#L72


## [G‑24] Use uint256(1)/uint256(2) instead for true and false boolean states
If you don't use boolean for storage you will avoid Gwarmaccess 100 gas. In addition, state changes of boolean from true to false can cost up to ~20000 gas rather than uint256(2) to uint256(1) that would cost significantly less.

```solidity
File: /contracts/LSP6KeyManager/LSP6KeyManagerCore.sol
541   _reentrancyStatus = true;
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L541



