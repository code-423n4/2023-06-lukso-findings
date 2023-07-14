# Gas Optimization

# Summary 

|   no   |  Issue   |  Instance  |
|------|----------|------------|
|[G-01]|Use constants instead of type(uintx).max|8|
|[G-02]|Empty blocks should be removed or emit something|2|
|[G-03]|With assembly, .call (bool success) transfer can be done gas-optimized|2|
|[G-04]|internal functions only called once can be inlined to save gas|37|
|[G-05]|Multiple accesses of a mapping/array should use a local variable cache|8|
|[G-06]|Use Assembly To Check For address(0)|28|
|[G-07]|>= costs less gas than >|11|
|[G-08]|Duplicated require()/if() checks should be refactored to a modifier or function|15|
|[G-09]|Use != 0 instead of > 0 for unsigned integer comparison|8|
|[G-10]|Use hardcode address instead address(this)|4|
|[G-11]|Access mappings directly rather than using accessor functions|4|
|[G-12]|Use nested if statements instead of &&|15|
|[G-13]|Can Make The Variable Outside The Loop To Save Gas |4|
|[G-14]|abi.encode() is less efficient than abi.encodePacked()|3|
|[G-15]|Use assembly to write address storage values|3|
|[G-16]|Using calldata instead of memory for read-only arguments in external functions saves gas|6|
|[G-17]|Uncheck arithmetics operations that can’t underflow/overflow|20|
|[G-18]|State variables only set in the constructor should be declared immutab|2|
|[G-19]|Use assembly to hash instead of Solidity|11|




## [G-01] Use constants instead of type(uintx).max

 it's generally more gas-efficient to use constants instead of type(uintX).max when you need to set the maximum value of an unsigned integer type.

The reason for this is that the type(uintX).max expression involves a computation at runtime, whereas a constant is evaluated at compile-time. This means that using type(uintX).max can result in additional gas costs for each transaction that involves the expression.

By using a constant instead of type(uintX).max, you can avoid these additional gas costs and make your code more efficient.

Here's an example of how you can use a constant instead of type(uintX).max:

```solidity
87   if (oldArrayLength == type(uint128).max) {
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP5ReceivedAssets/LSP5Utils.sol#L87

```solidity
190  if (newArrayLength >= type(uint128).max) {    
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP5ReceivedAssets/LSP5Utils.sol#L190

```solidity
296   bytes28(type(uint224).max)

378   allowedAddress == address(bytes20(type(uint160).max)) ||

395   allowedStandard == bytes4(type(uint32).max) ||

414   allowedFunction == bytes4(type(uint32).max) ||
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L296

```solidity
85   if (oldArrayLength == type(uint128).max) {

135  if (oldArrayLength > type(uint128).max) {    
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP10ReceivedVaults/LSP10Utils.sol#L85


## [G-02] Empty blocks should be removed or emit something

When an empty block of code is included in a smart contract, it may not perform any useful operations but still requires gas to be executed. This means that including empty blocks of code in a smart contract can increase the cost of executing the contract, as more gas needs to be paid to include the empty block.

```solidity
442    function _beforeTokenTransfer(
        address from,
        address to,
        uint256 amount
    ) internal virtual {}
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L442

```solidity
444  function _beforeTokenTransfer(
        address from,
        address to,
        bytes32 tokenId
    ) internal virtual {}
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L444


## [G-03] With assembly, .call (bool success) transfer can be done gas-optimized

When using assembly language, it is possible to call the transfer function of an Ethereum contract in a gas-optimized way by using the .call function with specific input parameters. The .call function takes a number of input parameters, including the address of the contract to call, the amount of Ether to transfer, and a specification of the gas limit for the call. By specifying a lower gas limit than the default, it is possible to reduce the gas cost of the transfer.

 ```solidity
 186     (bool success, bytes memory returnData) = target.call{value: value}(
            data
        );
 ```
 https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725XCore.sol#L186
 
 
 ```solidity
420   (bool success, bytes memory returnData) = _target.call{
            value: msgValue,
            gas: gasleft()
        }(payload);
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L420


## [G‑04] internal functions only called once can be inlined to save gas

When a function is inlined, its code is inserted directly into the calling function, eliminating the need to create a separate function call and reducing the gas cost of executing the function. This can be particularly beneficial for internal functions that are only called once, as the gas cost of creating and executing a separate function call may be greater than the gas cost of simply inlining the function's code.

```solidity
131    function _executeBatch(
        uint256[] memory operationsType,
        address[] memory targets,
        uint256[] memory values,
        bytes[] memory datas
    ) internal virtual returns (bytes[] memory) {  

```
https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725XCore.sol#L131

```solidity
174   function _executeCall(
        address target,
        uint256 value,
        bytes memory data
    ) internal virtual returns (bytes memory result) {
```
https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725XCore.sol#L174

```solidity
203   function _executeStaticCall(
        address target,
        bytes memory data
    ) internal virtual returns (bytes memory result) {
```
https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725XCore.sol#L203

```solidity
225     function _executeDelegateCall(
        address target,
        bytes memory data
    ) internal virtual returns (bytes memory result) {
```
https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725XCore.sol#L225

```solidity
247  function _deployCreate(
        uint256 value,
        bytes memory creationCode
    ) internal virtual returns (bytes memory newContract) {
```
https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725XCore.sol#L247

```solidity
288   function _deployCreate2(
        uint256 value,
        bytes memory creationCode
    ) internal virtual returns (bytes memory newContract) {                
```
https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725XCore.sol#L288

```solidity
796   function _fallbackLSP17Extendable() internal virtual override {
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L796


```solidity
851   function _getExtension(
        bytes4 functionSelector
    ) internal view virtual override returns (address) {

```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L851

```solidity
152    function _whenReceiving(
        bytes32 typeId,
        address notifier,
        bytes32 notifierMapKey,
        bytes4 interfaceID
    ) internal virtual returns (bytes memory) {
```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol#L152

```solidiyt
201   function _whenSending(
        bytes32 typeId,
        address notifier,
        bytes32 notifierMapKey,
        uint128 arrayIndex
    ) internal virtual returns (bytes memory) {        
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol#L201



```solidity
27   function _initialize(
        string memory name_,
        string memory symbol_,
        address newOwner_
    ) internal virtual onlyInitializing {
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP4DigitalAssetMetadata/LSP4DigitalAssetMetadataInitAbstract.sol#L27


```solidity
153   function _verifyCanDeployContract(
        address controller,
        bytes32 permissions,
        bool isFundingContract
    ) internal view virtual {
```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L153


```solidity
170   function _verifyCanStaticCall(
        address controlledContract,
        address controller,
        bytes32 permissions,
        bytes calldata payload
    ) internal view virtual {
```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L170

```solidity
188   function _verifyCanCall(
        address controlledContract,
        address controller,
        bytes32 permissions,
        bytes calldata payload
    ) internal view virtual {
```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L188

```solidity
317    function _extractCallType(
        uint256 operationType,
        uint256 value,
        bool isEmptyCall
    ) internal pure returns (bytes4 requiredCallTypes) {
```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L317

```solidity
339   function _extractExecuteParameters(
        bytes calldata executeCalldata
    ) internal pure returns (uint256, address, uint256, bytes4, bool) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L339


```solidity
366   function _isAllowedAddress(
        bytes memory allowedCall,
        address to
    ) internal pure returns (bool) {
```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L366

```solidity
382     function _isAllowedStandard(
        bytes memory allowedCall,
        address to
    ) internal view returns (bool) {
```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L382

```solidity
399   function _isAllowedFunction(
        bytes memory allowedCall,
        bytes4 requiredFunction
    ) internal pure returns (bool) {
```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L399

```solidity
418  function _isAllowedCallType(
        bytes memory allowedCall,
        bytes4 requiredCallTypes
    ) internal pure returns (bool) {


```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L418


```solidity
441       function _isValidNonce(
        address from,
        uint256 idx
    ) internal view virtual returns (bool) {       
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L441

```solidity
334   function _getPermissionToSetPermissionsArray(
        address controlledContract,
        bytes32 inputDataKey,
        bytes memory inputDataValue,
        bool hasBothAddControllerAndEditPermissions
    ) internal view virtual returns (bytes32) {
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L334


```solidity
385   function _getPermissionToSetControllerPermissions(
        address controlledContract,
        bytes32 inputPermissionDataKey
    ) internal view virtual returns (bytes32) {
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L385


```solidity
406   function _getPermissionToSetAllowedCalls(
        address controlledContract,
        bytes32 dataKey,
        bytes memory dataValue,
        bool hasBothAddControllerAndEditPermissions
    ) internal view virtual returns (bytes32) {
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L406


```solidity
438  function _getPermissionToSetAllowedERC725YDataKeys(
        address controlledContract,
        bytes32 dataKey,
        bytes memory dataValue,
        bool hasBothAddControllerAndEditPermissions
    ) internal view returns (bytes32) {
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L438

```solidity
474   function _getPermissionToSetLSP1Delegate(
        address controlledContract,
        bytes32 lsp1DelegateDataKey
    ) internal view virtual returns (bytes32) {
```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L474


```solidity
490   function _getPermissionToSetLSP17Extension(
        address controlledContract,
        bytes32 lsp17ExtensionDataKey
    ) internal view virtual returns (bytes32) {
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L490

```solidity
507  function _verifyAllowedERC725YSingleKey(
        address controllerAddress,
        bytes32 inputDataKey,
        bytes memory allowedERC725YDataKeysCompacted
    ) internal pure virtual {
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L507

```solidity
622    function _verifyAllowedERC725YDataKeys(
        address controllerAddress,
        bytes32[] memory inputDataKeys,
        bytes memory allowedERC725YDataKeysCompacted,
        bool[] memory validatedInputKeysList,
        uint256 allowedDataKeysFound
    ) internal pure virtual {

```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L622



```solidity
53   function _mint(
        address to,
        uint256 amount,
        bool allowNonLSP1Recipient,
        bytes memory data
    ) internal virtual override {
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/extensions/LSP7CappedSupply.sol#L53

```solidity
107  function _updateOperator(
        address tokenOwner,
        address operator,
        uint256 amount
    ) internal virtual override {
```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/extensions/LSP7CompatibleERC20.sol#L107

```solidity
116   function _transfer(
        address from,
        address to,
        uint256 amount,
        bool allowNonLSP1Recipient,
        bytes memory data
    ) internal virtual override {
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/extensions/LSP7CompatibleERC20.sol#L116


```solidity
127       function _mint(
        address to,
        uint256 amount,
        bool allowNonLSP1Recipient,
        bytes memory data
    ) internal virtual override {
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/extensions/LSP7CompatibleERC20.sol#L127


```solidity
137   function _burn(
        address from,
        uint256 amount,
        bytes memory data
    ) internal virtual override {
```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/extensions/LSP7CompatibleERC20.sol#L137

```solidity
146  function _setData(
        bytes32 key,
        bytes memory value
    ) internal virtual override(LSP4DigitalAssetMetadata, ERC725YCore) {                                
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/extensions/LSP7CompatibleERC20.sol#L146

```solidity
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
28   function _initialize(
        string memory name_,
        string memory symbol_,
        address newOwner_,
        bool isNonDivisible_
    ) internal virtual onlyInitializing {
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetInitAbstract.sol#L28




## [G‑05] Multiple accesses of a mapping/array should use a local variable cache

One way to optimize the gas cost of accessing mappings or arrays multiple times is to use a local variable cache. This involves storing the value of the mapping or array in a local variable and then accessing the local variable multiple times instead of accessing the mapping or array directly.
```solidity
38   _indexToken[index] = tokenId;
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8Enumerable.sol#L38

```solidity
44   bytes32 lastTokenId = _indexToken[lastIndex];
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8Enumerable.sol#L44

```solidity
45   _indexToken[index] = lastTokenId;
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8Enumerable.sol#L45

```solidity
48   delete _indexToken[lastIndex];
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8Enumerable.sol#L48

```solidity

39   _tokenIndex[tokenId] = index;
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8Enumerable.sol#L39

```solidity
42   uint256 index = _tokenIndex[tokenId];
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8Enumerable.sol#L42

```solidity
46   _tokenIndex[lastTokenId] = index;
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8Enumerable.sol#L46

```solidity
49   delete _tokenIndex[tokenId];
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8Enumerable.sol#L49


## [G-06] Use Assembly To Check For address(0)

One way to check for the zero address using assembly language is to use the iszero opcode, which returns true if the input value is zero and false otherwise. We can use this opcode to check whether an address is equal to the zero address by converting the address to a 256-bit integer and then checking whether the integer value is zero.
```solidity
28   newOwner != address(0),
```
https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725.sol#L28

```solidity
28   newOwner != address(0),
```
https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725InitAbstract


```solidity
22    newOwner != address(0),
```
https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725X.sol#L22

```solidity
89   if (target != address(0))

96   if (target != address(0))

296  if (contractAddress == address(0)) {
```
https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725XCore.sol#L89

```solidity
21    newOwner != address(0),
```
https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725XInitAbstract.sol#L21

```solidity
23    newOwner != address(0),
```
https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725Y.sol#L23

```solidity
21    newOwner != address(0),
```
https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725YInitAbstract.sol#L21

```solidity
801   if (msg.sig == bytes4(0) && extension == address(0)) return;

804   if (extension == address(0))
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L801


```solidity
19   if (target_ == address(0)) revert InvalidLSP6Target();
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManager.sol#L19


```solidity
21    if (target_ == address(0)) revert InvalidLSP6Target();
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerInitAbstract.sol#L21



```solidity
268   if (operator == address(0)) {

300   if (to == address(0)) {

343   if (from == address(0)) {

406   if (from == address(0) || to == address(0)) {            
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L268

```solidity
36   if (from == address(0)) {

40   } else if (to == address(0)) {    
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8Enumerable.sol#L36

```solidity
38    if (from == address(0)) {

42    } else if (to == address(0)) {    
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8EnumerableInitAbstract.sol#L38

```solidity
84   if (tokenOwner == address(0)) {

115  if (operator == address(0)) {

139  if (operator == address(0)) {

319  if (to == address(0)) {

410  if (to == address(0)) {                
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L84



```solidity
50   if (erc165Extension == address(0)) return false;

91   if (extension == address(0))
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP17ContractExtension/LSP17Extendable.sol#L50

```solidity
51    newOwner != address(0),
```
https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/custom/OwnableUnset.sol#L51



## [G-07] >= costs less gas than >

In Solidity, the comparison operators > and >= are used to compare two values and return a boolean value indicating whether the comparison is true or false. However, it's important to note that using >= can often be more gas-efficient than using >.

The reason for this is that the EVM (Ethereum Virtual Machine) uses a 256-bit word size, which means that operations involving values smaller than 256 bits still require a full 256 bits of data to be loaded into memory. This means that when we use > to compare two values, we may end up loading more data into memory than necessary, resulting in higher gas costs.

```solidity
182   if (result.length > 0) {

741   if (_owner.code.length > 0) {    
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L182



```solidity
165   if (notifier.code.length > 0) {
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol#L165


```solidity
284   if (ii + 34 > allowedCalls.length) {
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L284

```solidity
559   if (length > 32)

679   if (length > 32)
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L559


```solidity
137   if (elementLength == 0 || elementLength > 32) return false;
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Utils.sol#L137

```solidity
136   if (amount > operatorAmount) {

348    if (amount > balance) {

357    if (amount > authorizedAmount) {

491   if (to.code.length > 0) {            
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L136



## [G-08] Duplicated require()/if() checks should be refactored to a modifier or function

To optimize the gas cost of a contract and reduce redundant code, it's often recommended to refactor duplicated require() or if() checks into a modifier or function. This can help to reduce the size of the contract and make it easier to read and maintain.

```solidity
89   if (target != address(0))

96   if (target != address(0))


179  if (address(this).balance < value) {

251  if (address(this).balance < value) {   
```
https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725XCore.sol#L89

```solidity
66  if (msg.value != 0) revert ERC725Y_MsgValueDisallowed();

78  if (msg.value != 0) revert ERC725Y_MsgValueDisallowed();
```
https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725YCore.sol#L66


```solidity
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


## [G-09]Use != 0 instead of > 0 for unsigned integer comparison

In Solidity, unsigned integer values can be compared using either the > or != operators. However, it's generally more gas-efficient to use the != 0 operator instead of the > 0 operator, especially when dealing with large integer values.

The reason for this is that the EVM (Ethereum Virtual Machine) uses a 256-bit word size, which means that operations involving values smaller than 256 bits still require a full 256 bits of data to be loaded into memory. This means that when we use the > 0 operator to compare an unsigned integer value, we may end up loading more data into memory than necessary, resulting in higher gas costs.

```solidity
182   if (result.length > 0) {
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L182


```solidity
741   if (_owner.code.length > 0) {    
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L741


```solidity
165   if (notifier.code.length > 0) {
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol#L165


```solidity
491   if (to.code.length > 0) {
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L491

```solidity
313    if (to.code.length > 0) {
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol#L313

```solidity
313   if (to.code.length > 0) {
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol#L313

```solidity
493   if (to.code.length > 0) {
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L493

```solidity
86    if (returnedData.length > 0) {
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP20CallVerification/LSP20CallVerification.sol#L86


## [G-10] Use hardcode address instead address(this)

The reason for this is that address(this) requires the EVM to perform a CALL operation to obtain the address of the current contract. This operation can be relatively expensive in terms of gas costs, especially if it's performed repeatedly within the same contract.

```solidity
179   if (address(this).balance < value) {
```
https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725XCore.sol#L179


```solidity
180   revert ERC725X_InsufficientBalance(address(this).balance, value);
```
https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725XCore.sol#L180

```solidity
251   if (address(this).balance < value) {
```
https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725XCore.sol#L251

```solidity
252   revert ERC725X_InsufficientBalance(address(this).balance, value);    
```
https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725XCore.sol#L252


## [G-11] Access mappings directly rather than using accessor functions

In Solidity, it's common to use accessor functions to retrieve values from mappings. However, in some cases, it can be more gas-efficient to access the mapping directly, especially if the mapping is not too large.
The reason for this is that accessor functions typically involve additional function call overhead, which can add to the gas cost of the contract. If we access the mapping directly, we can avoid this overhead and reduce the gas cost of the contract.

```solidity
99   return _store[dataKey];
```
https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725YCore.sol#L99

```solidity
28   return _indexToken[index];
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8Enumerable.sol#L28

```solidity
30   return _indexToken[index];
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8EnumerableInitAbstract.sol#L30

```solidity
73   return _tokenOwnerBalances[tokenOwner];
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L73


## [G-12]Use nested if statements instead of &&

The reason for this is that && requires the EVM to evaluate both conditions and perform a logical operation to combine the results. This can be relatively expensive in terms of gas costs, especially if the conditions involve complex expressions or function calls.
```solidity
801   if (msg.sig == bytes4(0) && extension == address(0)) return;
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L801

```solidity
227   if (dataKeys.length == 0 && dataValues.length == 0)

245   if (dataKeys.length == 0 && dataValues.length == 0)
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol#L227

```solidity
78     if (encodedArrayLength.length != 0 && encodedArrayLength.length != 16) {
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP5ReceivedAssets/LSP5Utils.sol#L78

```solidity
165   if (isFundingContract && !hasSuperTransferValue) {

214   if (isTransferringValue && !hasSuperTransferValue) {

224    if (!hasSuperCall && !isCallDataPresent && !isTransferringValue) {

228   if (isCallDataPresent && !hasSuperCall) {

236   if (hasSuperTransferValue && !isCallDataPresent && isTransferringValue)

240   if (hasSuperCall && hasSuperTransferValue) return;
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L165

```solidity
338   if (!isReentrantCall && !isSetData) {
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L338


```solidity
404   if (!isReentrantCall && !isSetData) {    
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L404

```solidity
362    if (inputDataValue.length != 0 && inputDataValue.length != 20) {
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L362

```solidity
76  if (encodedArrayLength.length != 0 && encodedArrayLength.length != 16) {
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP10ReceivedVaults/LSP10Utils.sol#L76

## [G-13] Can Make The Variable Outside The Loop To Save Gas 


The reason for this is that each time a variable is declared inside a loop, it consumes gas to allocate memory for the variable. If the loop is executed multiple times, this can result in redundant gas costs for memory allocation.

```solidity
176   (bool success, bytes memory result) = address(this).delegatecall(
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L176


```solidity
262   bytes32 key = data.toBytes32(pointer);
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol#L262

```solidity
293   bytes32 key = data.toBytes32(pointer);
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol#L262


```solidity
275   address operator = operatorsForTokenId.at(0);
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L275

## [G-14] abi.encode() is less efficient than abi.encodePacked()

The reason for this is that abi.encode() adds additional metadata to the encoded data, including the function signature and the length of the encoded data. This metadata can be useful in some cases, but it also adds to the gas cost of the encoding operation.

```solidity
253    LSP20CallVerification._verifyCallResult(_owner, abi.encode(result));
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L253

```solidity
319    abi.encode(results)
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L319

```solidity
528    returnedValues = abi.encode(
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L528



## [G-15] Use assembly to write address storage values

One use case where assembly can be more gas-efficient is when writing to storage, especially if we're writing to multiple storage slots in a single operation. Using Solidity code to write to storage can result in redundant gas costs for memory allocation, whereas assembly allows us to write to multiple storage slots in a single operation, which can reduce the gas cost of the contract.

```solidity
129    _pendingOwner = newOwner;
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol#L129

```solidity
71   _owner = newOwner;
```
https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/custom/OwnableUnset.sol#L71

```solidity
22   _target = target_;
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerInitAbstract.sol#L22


## [G-16] Using calldata instead of memory for read-only arguments in external functions saves gas


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


## [G-17] Uncheck arithmetics operations that can’t underflow/overflow




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


## [G‑18] State variables only set in the constructor should be declared immutab


When a state variable is modified in Solidity, it requires a storage write operation, which can consume a significant amount of gas. However, immutable variables are stored differently in storage, and their value is directly embedded in the bytecode of the contract, which means that they do not require storage write operations when they are accessed.

```solidity
20   _target = target_;

```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/ILSP6KeyManager.sol#L20


```solidity
45   _isNonDivisible = isNonDivisible_;
```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/LSP7DigitalAsset.sol#L45


## [G-19] Use assembly to hash instead of Solidity

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