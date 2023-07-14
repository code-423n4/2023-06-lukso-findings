## Summary

### Gas Optimization

no | Issue |Instances||
|-|:-|:-:|:-:|
| [G-01] | Use nested if and, avoid multiple check combinations | 14 | - |
| [G-02] | abi.encode() is less efficient than  abi.encodepacked() | 3 | - |
| [G-03] | Access mappings directly rather than using accessor functions | 4 | - |
| [G-04] | Use != 0 instead of > 0 for unsigned integer comparison | 8 | - |
| [G-05] | Use assembly to write address storage values | 3 | - |
| [G-06] | Use hardcode address instead of address(this) | 4 | - |
| [G-07] | Can Make The Variable Outside The Loop To Save Gas  | 4 | - |
| [G-08] | >= costs less gas than > | 11 | - |
| [G-09] | With assembly, .call (bool success)  transfer can be done gas-optimized | 2 | - |
| [G-10] | Internal functions only called once can be inlined to save gas | 36 | - |
| [G-11] | Empty blocks should be removed or emit something | 2 | - |
| [G-12] | Duplicated require()/if() checks should be refactored to a modifier or function | 15 | - |
| [G-13] | Use assembly to check for address(0) | 29 | - |
| [G-14] | Multiple accesses of a mapping/array should use a local variable cache | 8 | - |
| [G-15] | Use constants instead of type(uintx).max | 8 | - |
| [G-16] | Non efficient zero initialization | 3 | - |
| [G-17] | Pre-increments and pre-decrements are cheaper than post-increments and post-decrements | 1 | - |


## Gas Optimizations  

## [G-1] Use nested if and, avoid multiple check combinations

Using nested if is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.


```solidity
file: /contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol

801   if (msg.sig == bytes4(0) && extension == address(0)) return;

```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L801

```solidity
file: /contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol

227   if (dataKeys.length == 0 && dataValues.length == 0)

245   if (dataKeys.length == 0 && dataValues.length == 0)

```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol#L227

```solidity
file: /contracts/LSP5ReceivedAssets/LSP5Utils.sol

78     if (encodedArrayLength.length != 0 && encodedArrayLength.length != 16) {

```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP5ReceivedAssets/LSP5Utils.sol#L78

```solidity
file: /contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol

165   if (isFundingContract && !hasSuperTransferValue) {

214   if (isTransferringValue && !hasSuperTransferValue) {

224   if (!hasSuperCall && !isCallDataPresent && !isTransferringValue) {

228   if (isCallDataPresent && !hasSuperCall) {

236   if (hasSuperTransferValue && !isCallDataPresent && isTransferringValue)

240   if (hasSuperCall && hasSuperTransferValue) return;

```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L165

```solidity
file: /contracts/LSP6KeyManager/LSP6KeyManagerCore.sol

338   if (!isReentrantCall && !isSetData) {

404   if (!isReentrantCall && !isSetData) {    

```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L338

```solidity
file: /contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol

362    if (inputDataValue.length != 0 && inputDataValue.length != 20) {

```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L362

```solidity
file: /contracts/LSP10ReceivedVaults/LSP10Utils.sol

76  if (encodedArrayLength.length != 0 && encodedArrayLength.length != 16) {

```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP10ReceivedVaults/LSP10Utils.sol#L76


## [G-2] abi.encode() is less efficient than  abi.encodepacked()

In terms of efficiency, abi.encodePacked() is generally considered to be more gas-efficient than abi.encode(), because it skips the step of adding function signatures and other metadata to the encoded data. However, this comes at the cost of reduced safety, as abi.encodePacked() does not perform any type checking or padding of data.

Refference: https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison

```solidity
file: /contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol

253    LSP20CallVerification._verifyCallResult(_owner, abi.encode(result));

319    abi.encode(results)

528    returnedValues = abi.encode(

```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L253 


## [G-3] Access mappings directly rather than using accessor functions

```solidity
file: /ERC725/blob/v5.1.0/implementations/contracts/ERC725YCore.sol

99   return _store[dataKey];

```
https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725YCore.sol#L99


```solidity
file: /contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8Enumerable.sol

28   return _indexToken[index];

```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8Enumerable.sol#L28

```solidity
file: /contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8EnumerableInitAbstract.sol

30   return _indexToken[index];

```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8EnumerableInitAbstract.sol#L30

```solidity
file: /contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol

73   return _tokenOwnerBalances[tokenOwner];

```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L73


## [G-4] Use != 0 instead of > 0 for unsigned integer comparison

it's generally more gas-efficient to use != 0 instead of > 0 when
comparing unsigned integer types.

This is because the Solidity compiler can optimize the != 0 comparison to a simple bitwise operation,
 while the > 0 comparison requires an additional subtraction operation.
  As a result, using != 0 can be more gas-efficient and can help to reduce the overall cost of your contract.

Here's an example of how you can use != 0 instead of > 0:
``` solidity
contract MyContract {
    uint256 public myUnsignedInteger;
    
    function myFunction() public view returns (bool) {
        // Use != 0 instead of > 0
        return myUnsignedInteger != 0;
    }
}
```


```solidity
file: /contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol

182   if (result.length > 0) {

741   if (_owner.code.length > 0) {    

```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L182


```solidity
file: /contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol

165   if (notifier.code.length > 0) {

```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol#L165


```solidity
file: /contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol

491   if (to.code.length > 0) {

```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L491

```solidity
file: /contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol

313    if (to.code.length > 0) {

```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol#L313

```solidity
file: /contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol

313   if (to.code.length > 0) {

```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol#L313

```solidity
file: /contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol

493   if (to.code.length > 0) {

```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L493

```solidity
file: /contracts/LSP20CallVerification/LSP20CallVerification.sol

86    if (returnedData.length > 0) {

```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP20CallVerification/LSP20CallVerification.sol#L86


## [G-5] Use assembly to write address storage values

By using assembly to write to address storage values, you can bypass some of these operations and lower the gas cost of writing to storage. Assembly code allows you to directly access the Ethereum Virtual Machine (EVM) and perform low-level operations that are not possible in Solidity.

example of using assembly to write to address storage values:

contract MyContract {
    address private myAddress;
    
    function setAddressUsingAssembly(address newAddress) public {
        assembly {
            sstore(0, newAddress)
        }
    }
}

```solidity
file: /contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol

129    _pendingOwner = newOwner;

```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol#L129

```solidity
file: /ERC725/blob/v5.1.0/implementations/contracts/custom/OwnableUnset.sol

71   _owner = newOwner;

```
https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/custom/OwnableUnset.sol#L71


INHERITED (address) StateVar LSP6KeyManagerCore._target (Declaration: LSP6KeyManagerCore#79)
```solidity

22   _target = target_;

```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerInitAbstract.sol#L22


## [G-6] Use hardcode address instead of address(this)
 
 Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this.
References: https://book.getfoundry.sh/reference/forge-std/compute-create-address

```solidity
file: /ERC725/blob/v5.1.0/implementations/contracts/ERC725XCore.sol

179     if (address(this).balance < value) {

180     revert ERC725X_InsufficientBalance(address(this).balance, value);

251     if (address(this).balance < value) {

252     revert ERC725X_InsufficientBalance(address(this).balance, value);    

```
https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725XCore.sol#L179


## [G-7] Can Make The Variable Outside The Loop To Save Gas 

Consider making the stack variables before the loop which gonna save gas

```solidity
file: /contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol

176   (bool success, bytes memory result) = address(this).delegatecall(

```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L176


```solidity
file: /contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol

262   bytes32 key = data.toBytes32(pointer);

293   bytes32 key = data.toBytes32(pointer);

```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol#L262


```solidity
file: /contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol

275   address operator = operatorsForTokenId.at(0);

```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L275


## [G-8] >= costs less gas than >
 
The compiler uses opcodes GT and ISZERO for solidity code that uses >, but only requires LT for >=, which saves 3 gas 
Reference:  https://gist.github.com/IllIllI000/3dc79d25acccfa16dee4e83ffdc6ffde


```solidity
file: /contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol

182   if (result.length > 0) {

741   if (_owner.code.length > 0) {    

```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L182


```solidity
file: /contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol

165   if (notifier.code.length > 0) {

```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol#L165


```solidity
file: /contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol

284   if (ii + 34 > allowedCalls.length) {

```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L284

```solidity
file: /contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol

559   if (length > 32)

679   if (length > 32)

```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L559


```solidity
file: /contracts/LSP6KeyManager/LSP6Utils.sol

137   if (elementLength == 0 || elementLength > 32) return false;

```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Utils.sol#L137

```solidity
file: /contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol

136    if (amount > operatorAmount) {

348    if (amount > balance) {

357    if (amount > authorizedAmount) {

491    if (to.code.length > 0) {            

```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L136

## [G-9] With assembly, .call (bool success)  transfer can be done gas-optimized

return data (bool success,) has to be stored due to EVM architecture, but in a usage like below, ‘out’ and ‘outsize’ values are given (0,0), 
this storage disappears and gas optimization is provided.

 ```solidity
 file: /contracts/ERC725XCore.sol

186     (bool success, bytes memory returnData) = target.call{value: value}(
            data
        );

 ```
 https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725XCore.sol#L186
 
 
 ```solidity
file: /contracts/LSP6KeyManager/LSP6KeyManagerCore.sol

420   (bool success, bytes memory returnData) = _target.call{
            value: msgValue,
            gas: gasleft()
        }(payload);

```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L420

## [G‑10] Internal functions only called once can be inlined to save gas

Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

```solidity
file: /ERC725/blob/v5.1.0/implementations/contracts/ERC725XCore.sol

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
file: /contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol

796   function _fallbackLSP17Extendable() internal virtual override {

851   function _getExtension(
        bytes4 functionSelector
    ) internal view virtual override returns (address) {

```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L796

```solidity
file: /contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol

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
file: /contracts/LSP4DigitalAssetMetadata/LSP4DigitalAssetMetadataInitAbstract.sol

27   function _initialize(
        string memory name_,
        string memory symbol_,
        address newOwner_
    ) internal virtual onlyInitializing {

```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP4DigitalAssetMetadata/LSP4DigitalAssetMetadataInitAbstract.sol#L27


```solidity
file: /contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol

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
file: /contracts/LSP6KeyManager/LSP6KeyManagerCore.sol

441       function _isValidNonce(
        address from,
        uint256 idx
    ) internal view virtual returns (bool) {       

```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L441

```solidity
file: /contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol

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
file: /contracts/LSP7DigitalAsset/extensions/LSP7CappedSupply.sol

53   function _mint(
        address to,
        uint256 amount,
        bool allowNonLSP1Recipient,
        bytes memory data
    ) internal virtual override {

```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/extensions/LSP7CappedSupply.sol#L53

```solidity
file: /contracts/LSP7DigitalAsset/extensions/LSP7CompatibleERC20.sol

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
file: /contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol

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
file: /contracts/LSP7DigitalAsset/LSP7DigitalAssetInitAbstract.sol

28   function _initialize(
        string memory name_,
        string memory symbol_,
        address newOwner_,
        bool isNonDivisible_
    ) internal virtual onlyInitializing {

```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetInitAbstract.sol#L28


## [G-11] Empty blocks should be removed or emit something

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting.
If the contract is meant to be extended, the contract should be abstract and the function signatures be added without any default implementation.

If the block is an empty if-statement block to avoid doing subsequent checks in the else-if/else conditions, the else-if/else conditions should be 
nested under the negation of the if-statement, because they involve different classes of checks, which may lead to the introduction of errors when 
the code is later modified (if( x ) {}else if(y){ . . . }else{ . . . } => if ( !x) {if(y) { . . . }else{ . . .}}) . 
Empty receive()/fallback() payable functions that are not used, can be removed to save deployment gas.

```solidity
file: /contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol

442    function _beforeTokenTransfer(
        address from,
        address to,
        uint256 amount
    ) internal virtual {}

```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L442

```solidity
file: /contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol

444  function _beforeTokenTransfer(
        address from,
        address to,
        bytes32 tokenId
    ) internal virtual {}

```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L444

## [G-12] Duplicated require()/if() checks should be refactored to a modifier or function

 to reduce code duplication and improve readability.
•  Modifiers can be used to perform additional checks on the function inputs or state before it is executed. By defining a modifier to perform a specific check, we can reuse it across multiple functions that require the same check.
• A function can also be used to perform a specific check and return a boolean value indicating whether the check has passed or failed. This can be useful when the check is more complex and cannot be performed easily in a modifier.

```solidity
file: /ERC725/blob/v5.1.0/implementations/contracts/ERC725XCore.sol

89   if (target != address(0))

96   if (target != address(0))


179  if (address(this).balance < value) {

251  if (address(this).balance < value) {   

```
https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725XCore.sol#L89

```solidity
file: /ERC725/blob/v5.1.0/implementations/contracts/ERC725YCore.sol

66  if (msg.value != 0) revert ERC725Y_MsgValueDisallowed();

78  if (msg.value != 0) revert ERC725Y_MsgValueDisallowed();

```
https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725YCore.sol#L66


```solidity
file: /contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol

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


## [G-13] Use assembly to check for address(0)

  Save 6 gas per instance.

```solidity
file: /ERC725/blob/v5.1.0/implementations/contracts/ERC725.sol

28   newOwner != address(0),

```
https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725.sol#L28

```solidity
file: /ERC725/blob/v5.1.0/implementations/contracts/ERC725InitAbstract

25   newOwner != address(0),

```
https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725InitAbstract.sol#L25


```solidity
file: /ERC725/blob/v5.1.0/implementations/contracts/ERC725X.sol

22    newOwner != address(0),

```
https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725X.sol#L22

```solidity
file: /ERC725/blob/v5.1.0/implementations/contracts/ERC725XCore.sol

89   if (target != address(0))

96   if (target != address(0))

296  if (contractAddress == address(0)) {

```
https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725XCore.sol#L89

```solidity
file: /ERC725/blob/v5.1.0/implementations/contracts/ERC725XInitAbstract.sol

21    newOwner != address(0),

```
https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725XInitAbstract.sol#L21

```solidity
file: /ERC725/blob/v5.1.0/implementations/contracts/ERC725Y.sol

23    newOwner != address(0),

```
https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725Y.sol#L23

```solidity
file: /ERC725/blob/v5.1.0/implementations/contracts/ERC725YInitAbstract.sol

21    newOwner != address(0),

```
https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725YInitAbstract.sol#L21

```solidity
file: /contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol

801   if (msg.sig == bytes4(0) && extension == address(0)) return;

804   if (extension == address(0))

```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L801


```solidity
file: /contracts/LSP6KeyManager/LSP6KeyManager.sol

19   if (target_ == address(0)) revert InvalidLSP6Target();

```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManager.sol#L19


```solidity
file: /contracts/LSP6KeyManager/LSP6KeyManagerInitAbstract.sol

21    if (target_ == address(0)) revert InvalidLSP6Target();

```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerInitAbstract.sol#L21


```solidity
file: /contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol

268   if (operator == address(0)) {

300   if (to == address(0)) {

343   if (from == address(0)) {

406   if (from == address(0) || to == address(0)) {            

```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L268

```solidity
file: /contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8Enumerable.sol

36   if (from == address(0)) {

40   } else if (to == address(0)) {    

```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8Enumerable.sol#L36

```solidity
file: /contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8EnumerableInitAbstract.sol

38    if (from == address(0)) {

42    } else if (to == address(0)) {    

```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8EnumerableInitAbstract.sol#L38

```solidity
file: /contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol

84   if (tokenOwner == address(0)) {

115  if (operator == address(0)) {

139  if (operator == address(0)) {

319  if (to == address(0)) {

410  if (to == address(0)) {                

```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L84


```solidity
file: /contracts/LSP17ContractExtension/LSP17Extendable.sol

50   if (erc165Extension == address(0)) return false;

91   if (extension == address(0))

```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP17ContractExtension/LSP17Extendable.sol#L50

```solidity
file: /ERC725/blob/v5.1.0/implementations/contracts/custom/OwnableUnset.sol

51    newOwner != address(0),

```
https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/custom/OwnableUnset.sol#L51


## [G‑14] Multiple accesses of a mapping/array should use a local variable cache

The instances below point to the second+ access of a value inside a mapping/array, within a function. Caching a mapping’s value in a local storage or calldata variable when the value is accessed multiple times, saves ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations. Caching an array’s struct avoids recalculating the array offsets into memory/calldata.

```solidity
file: /contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8Enumerable.sol

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


## [G-15] Use constants instead of type(uintx).max

type(uint120).max or type(uint128).max, etc. it uses more gas in the distribution process and also for each transaction than constant usage.


```solidity
file: /contracts/LSP5ReceivedAssets/LSP5Utils.sol

87   if (oldArrayLength == type(uint128).max) {

190  if (newArrayLength >= type(uint128).max) {    

```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP5ReceivedAssets/LSP5Utils.sol#L87

```solidity
file: /contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol

296   bytes28(type(uint224).max)

378   allowedAddress == address(bytes20(type(uint160).max)) ||

395   allowedStandard == bytes4(type(uint32).max) ||

414   allowedFunction == bytes4(type(uint32).max) ||

```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L296

```solidity
file: /contracts/LSP10ReceivedVaults/LSP10Utils.sol

85   if (oldArrayLength == type(uint128).max) {

135  if (oldArrayLength > type(uint128).max) {    

```
https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP10ReceivedVaults/LSP10Utils.sol#L85


## [G-16] Non efficient zero initialization
 
Solidity does not recognize null as a value, so uint variables are initialized to zero. Setting a uint variable to zero is redundant and can waste gas.


```solidity
file: contracts/ERC725XCore.sol

146        for (uint256 i = 0; i < operationsType.length; ) {

```
https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725XCore.sol#L146


```solidity
file: contracts/ERC725YCore.sol

47        for (uint256 i = 0; i < dataKeys.length; ) {

88        for (uint256 i = 0; i < dataKeys.length; ) {    

```
https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725YCore.sol#L47

## [G-17] Pre-increments and pre-decrements are cheaper than post-increments and post-decrements

 Saves 5 gas per iteration

 ```solidity
 file: contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol

 156                inputDataKeysAllowed++;

 ```
 https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L156
