
# Summary for GAS findings


|  NO  |   issue   |  instence  |
|------|-----------|------------|
|[G-01]| With assembly, .call (bool success) transfer can be done gas-optimized    | 2 |
|[G-02]| internal functions only called once can be inlined to save gas    | 39 |
|[G-03]| Empty blocks should be removed or emit something   | 2 |
|[G-04]| Multiple accesses of a mapping/array should use a local variable cache  | 8 |
|[G-05]| Duplicated require()/if() checks should be refactored to a modifier or function    | 15 |
|[G-06]| Use constants instead of type(uintx).max  | 8 |
|[G-07]| Use Assembly To Check For address(0)    | 26 |
|[G-08]| Use hardcode address instead address(this)  | 4 |
|[G-09]| Use != 0 instead of > 0 for unsigned integer comparison  | 8 |
|[G-10]| abi.encode() is less efficient than abi.encodePacked()   | 3 |
|[G-11]| >= costs less gas than >  | 11 |
|[G-12]| Use nested if statements instead of &&  | 14 |
|[G-13]| Use assembly to write address storage values | 3 |
|[G-14]| Access mappings directly rather than using accessor functions | 4 |
|[G-15]| Can Make The Variable Outside The Loop To Save Gas  | 4 |
|[G-16]| Use calldata instead of memory for function arguments that do not get mutated  | 4 |
|[G-17]| Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead  | 6 |
|[G-18]| Use assembly to hash instead of Solidity  | 11 |
|[G-19]| State variables only set in the constructor should be declared immutab  | 2 |
|[G-20]| Uncheck arithmetics operations that can’t underflow/overflow  | 20 |
|[G-21]| Using calldata instead of memory for read-only arguments in external functions saves gas  | 6 |




## [G-01] With assembly, .call (bool success) transfer can be done gas-optimized


In Solidity, the transfer function is commonly used to transfer Ether between addresses. However, transfer has a fixed gas stipend of 2,300 gas and limited error handling capabilities. This can lead to potential issues, such as failure in transferring funds if the recipient address's fallback function consumes more gas than the stipend allows.

By utilizing assembly and the .call function, you have more control over gas usage and can optimize transfers accordingly. Here's an example that demonstrates how to use assembly's .call for gas-optimized transfers:

```solidity

function transfer(address payable recipient, uint256 amount) public {
    (bool success, ) = recipient.call{value: amount}("");
    require(success, "Transfer failed");
}

```

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
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L338

## [G‑02] internal functions only called once can be inlined to save gas

In Solidity, functions marked as internal are only callable from within the same contract or contracts that inherit from it. If an internal function is only called once, inlining the function's code at the calling location can eliminate the overhead of the function call itself, resulting in gas savings.



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

## [G-03] Empty blocks should be removed or emit something

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


## [G‑04] Multiple accesses of a mapping/array should use a local variable cache

 When making multiple accesses to a mapping or an array in Solidity, using a local variable cache can significantly save gas and optimize contract execution. Caching the values locally reduces the number of redundant storage or memory accesses, resulting in improved efficiency and reduced gas costs.

Here's an example to demonstrate the concept:

```solidity

contract MyContract {
    mapping(address => uint) public balances;

    function getTotalBalance(address[] memory accounts) public view returns (uint) {
        uint totalBalance = 0;

        for (uint i = 0; i < accounts.length; i++) {
            uint balance = balances[accounts[i]];  // Cache the balance value
            totalBalance += balance;
        }

        return totalBalance;
    }
}



```

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

## [G-05] Duplicated require()/if() checks should be refactored to a modifier or function

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

## [G-06] Use constants instead of type(uintx).max

Certainly! When working with maximum values in Solidity, it's more gas-efficient to use constants instead of the expression type(uintX).max, where X represents the number of bits in the unsigned integer type. This optimization can help reduce the gas costs associated with storing and manipulating large numbers.

Here's an example that demonstrates the use of constants instead of type(uintX).max:

```solidity
pragma solidity ^0.8.18;

contract GasOptimizationExample {
    uint256 constant MAX_VALUE = type(uint256).max;
    
    function process(uint256 value) public {
        require(value < MAX_VALUE, "Value exceeds maximum limit");
        
        // Perform some operations with the value
        // ...
    }
}
```

In the above code, we define a constant MAX_VALUE and assign it the value type(uint256).max. This constant represents the maximum value that can be stored in a uint256 variable.


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


## [G-07] Use Assembly To Check For address(0)

In Solidity, checking for the zero address typically involves a function call to the Contract function, which incurs gas costs. However, by leveraging assembly, you can perform a direct comparison with the zero address more efficiently.

Here's an example that demonstrates how to use assembly to check for the zero address:

```solidity

function isZeroAddress(address _addr) public pure returns (bool) {
    uint256 addrInt;
    assembly {
        addrInt := _addr
    }
    return addrInt == 0;
}

```


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

## [G-08] Use hardcode address instead address(this)

To save gas, you can hardcode the contract's address as a constant variable within the contract. By doing so, you eliminate the need for a function call to retrieve the contract's address, resulting in gas savings

Here's an example to illustrate the concept:

```solidity

contract MyContract {
    address public constant CONTRACT_ADDRESS = 0x1234567890ABCDEF;  // Hardcoded contract address

    function doSomething() public {
        // Use CONTRACT_ADDRESS directly instead of address(this)
        // ...
    }
}

```

```solidity
File: /ERC725/blob/v5.1.0/implementations/contracts/ERC725XCore.sol
179   if (address(this).balance < value) {

180   revert ERC725X_InsufficientBalance(address(this).balance, value);

251   if (address(this).balance < value) {

252   revert ERC725X_InsufficientBalance(address(this).balance, value);    
```
https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725XCore.sol#L179


## [G-09] Use != 0 instead of > 0 for unsigned integer comparison

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

## [G-10] abi.encode() is less efficient than abi.encodePacked()

The abi.encode() function automatically pads each encoded argument to a multiple of 32 bytes, aligning the data according to the EVM's word size. This padding ensures that the encoded data fits neatly into the storage or memory slots, but it can lead to unnecessary gas consumption. This is because extra gas is required to perform the padding and alignment operations.

On the other hand, the abi.encodePacked() function packs the encoded arguments tightly, without any padding or alignment. It concatenates the encoded values directly, resulting in a more compact representation. By avoiding padding and alignment, abi.encodePacked() can save gas compared to abi.encode().


```solidity
File: /contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol
253    LSP20CallVerification._verifyCallResult(_owner, abi.encode(result));

319    abi.encode(results)

528    returnedValues = abi.encode(
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L253


## [G-11] >= costs less gas than >


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

## [G-12] Use nested if statements instead of &&


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

## [G-13] Use assembly to write address storage values

To save gas, you can utilize assembly to perform a single write operation that assigns an entire address value to a storage slot. By avoiding byte-level operations and directly writing the complete address, you can reduce gas consumption.

```solidity
File: /contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol
129    _pendingOwner = newOwner;
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol#L129

```solidity
File: /ERC725/blob/v5.1.0/implementations/contracts/custom/OwnableUnset.sol
71   _owner = newOwner;
```
https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/custom/OwnableUnset.sol#L71

INHERITED (address) StateVar LSP6KeyManagerCore._target (Declaration: LSP6KeyManagerCore#79)
```solidity
22   _target = target_;
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerInitAbstract.sol#L22

## [G-14] Access mappings directly rather than using accessor functions

When you declare a mapping in Solidity, you can access its values using the key directly within your code. This direct access avoids the overhead of calling a separate accessor function, resulting in gas savings.

Here's an example to illustrate the concept:

```solidity
contract MyContract {
    mapping(address => uint) public balances;

    function updateBalance(address account, uint newBalance) public {
        balances[account] = newBalance;
    }

    function getBalance(address account) public view returns (uint) {
        return balances[account];
    }
}



```


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

## [G-15] Can Make The Variable Outside The Loop To Save Gas 

Declaring a variable outside of a loop means that the variable is instantiated only once, outside the loop's scope, rather than being instantiated multiple times within the loop. This optimization technique can help save gas by avoiding unnecessary variable instantiation and destruction during each iteration of the loop

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



## [G-16] Use calldata instead of memory for function arguments that do not get mutated

When you specify a data location as memory, that value will be copied into memory. When you specify the location as calldata, the value will stay static within calldata. If the value is a large, complex type, using memory may result in extra memory expansion costs.

```solidity

file:  interfaces/IERC725X.sol


93        function executeBatch(
        uint256[] memory operationsType,
        address[] memory targets,
        uint256[] memory values,
        bytes[] memory datas
    ) external payable returns (bytes[] memory);


```

https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/interfaces/IERC725X.sol#L93-L98

```solidity
file:   IERC725Y.sol

35       function getDataBatch(
        bytes32[] memory dataKeys
    ) external view returns (bytes[] memory dataValues);

67        function setDataBatch(
68        bytes32[] memory dataKeys,
69        bytes[] memory dataValues
70    ) external payable;

```
https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/interfaces/IERC725Y.sol#L35-L37


```solidity
file:    ILSP7DigitalAsset.sol

189        function transferBatch(
        address[] memory from,
        address[] memory to,
        uint256[] memory amount,
        bool[] memory allowNonLSP1Recipient,
        bytes[] memory data
    ) external;

```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/ILSP7DigitalAsset.sol#L189-L195


## [G-17] Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead
The EVM operates with 32 byte words. Therefore, if you declare state variables less than 32 bytes the EVM will need to perform extra operations to cast your value to the specified size.


```solidity
file:

65    function decimals() external view returns (uint8);


```

```solidity
file:   contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol

336    uint256 elementLength = uint16(

```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol#L336

```solidity
file:   contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol

544     length = uint16(

664      length = uint16(

```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L544

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L664


```solidity
file:  contracts/LSP6KeyManager/LSP6Utils.sol

98     uint256 elementLength = uint16(

128    uint256 elementLength = uint16(

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Utils.sol#L98


## [G-18] Use assembly to hash instead of Solidity

When we perform cryptographic hashing operations in Solidity, we typically use the built-in Solidity hash functions such as keccak256(). However, these functions can be relatively expensive in terms of gas cost, especially if we need to perform multiple hashing operations.

```solidity
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


## [G‑19] State variables only set in the constructor should be declared immutab


When a state variable is modified in Solidity, it requires a storage write operation, which can consume a significant amount of gas. However, immutable variables are stored differently in storage, and their value is directly embedded in the bytecode of the contract, which means that they do not require storage write operations when they are accessed.

```solidity
20   _target = target_;

```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/ILSP6KeyManager.sol#L20


```solidity
45   _isNonDivisible = isNonDivisible_;
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/LSP7DigitalAsset.sol#L45

## [G-20] Uncheck arithmetics operations that can’t underflow/overflow




In Solidity, performing arithmetic operations that cannot underflow or overflow can be more gas-efficient if we can guarantee that the operations will not result in an overflow or underflow condition. This is because Solidity automatically checks for overflow and underflow conditions during arithmetic operations, which can result in additional gas costs.

By using unchecked arithmetic operations, we can avoid the need for these checks and reduce the gas cost of the contract. However, it's important to note that unchecked arithmetic operations can be dangerous if we can't guarantee that the operations will not result in an overflow or underflow condition. If an overflow or underflow condition occurs, it can result in unexpected behavior and potentially compromise the security of the contract.

```solidity
269  pointer += 32;

299  pointer += 32;

344  pointer += elementLength + 2;
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol#L269


```solidity
137  uint128 newArrayLength = oldArrayLength - 1; 
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP5ReceivedAssets/LSP5Utils.sol#L137

```solidity
108   pointer += elementLength + 2;

138   pointer += elementLength + 2;

174   result += uint256(permissions[i]); 
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Utils.sol#L137


```solidity
145   _updateOperator(from, operator, operatorAmount - amount);  

365    _operatorAuthorizedAmount[from][operator] -= amount;

371    _existingTokens -= amount;

373    _tokenOwnerBalances[from] -= amount;

419     _tokenOwnerBalances[from] -= amount;

420    _tokenOwnerBalances[to] += amount;
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L145

```solidity
102  bytes memory uriBytes = BytesLib.slice(
            data,
            offset,
            data.length - offset
        );
138    return operatorsForTokenId[operatorListLength - 1];        
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol#L102

```solidity
102    bytes memory uriBytes = BytesLib.slice(
            data,
            offset,
            data.length - offset
        );

138   return operatorsForTokenId[operatorListLength - 1];
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol#L102

```solidity
41   uint256 lastIndex = totalSupply() - 1;      
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8Enumerable.sol#L41

```solidity
43   uint256 lastIndex = totalSupply() - 1;

```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8EnumerableInitAbstract.sol#L43

```solidity
140   uint128 newArrayLength = oldArrayLength - 1;
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP10ReceivedVaults/LSP10Utils.sol#L140


## [G-21] Using calldata instead of memory for read-only arguments in external functions saves gas


When a function is called externally in Solidity, the arguments are passed in the calldata, which is a special area of memory that is used to store the function arguments and any other data that is required for the function call. By using calldata instead of memory for read-only arguments, we can avoid the need for memory allocation and copy operations, which can reduce the gas cost of the contract.

```solidity
93   function executeBatch(
        uint256[] memory operationsType,
        address[] memory targets,
        uint256[] memory values,
        bytes[] memory datas
    ) external payable returns (bytes[] memory);
```
https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/interfaces/IERC725X.sol#L93


```solidity
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