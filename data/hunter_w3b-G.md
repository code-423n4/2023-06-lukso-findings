# Gas Optimization

# Summary

| Number | Optimization Details                                                         | Context |
| :----: | :--------------------------------------------------------------------------- | :-----: |
| [G-01] |  Use constants instead of `type(uintx).max`              |   8    |
| [G-02] |  Add `unchecked{}` for subtractions where the operands can't underflow because of previous require() or if-statement                                                                             |    3    |
| [G-03] | Can make the variable `outside` the loop to save gas                         |    2    |
| [G-04] | Using `calldata` instead of `memory` for read-only                           |   32    |
| [G-05] | Use assembly to check for `address(0)`                                       |   19    |
| [G-06] | State variables `can be packed` into fewer storage slots                     |    5    |
| [G-07] | Empty blocks should be `removed` or `emit` something                         |    2    |
| [G-08] | State variables only set in the `constructor` should be declared `immutable` |    1    |
| [G-09] | Duplicated `if()` checks should be refactored to a modifier or function      |   36    |
| [G-10] |  Internal functions `only called` once can be inlined to save gas                                     |    34   |
| [G-11] | Use nested if and, avoid multiple check combinations `&&`                    |   13    |
| [G-12] | Use assembly to write `address storage` values                               |    1    |
| [G-13] | Use `selfbalance()` instead of address(this).balance                         |    4    |
| [G-14] | `abi.encode()` is less efficient than abi.encodePacked()                     |    3    |
| [G-15] | Using `> 0` costs more gas than `!= 0`                                       |    3    |
| [G-16] | `Ternary` operation is cheaper than if-else statement                        |    3    |
| [G-17] | Pre-increment and pre-decrement are cheaper than `±1`                        |    4    |
| [G-18] | When possible, use assembly instead of `unchecked{++i}`                      |   13    |


## [G-01] Use constants instead of `type(uintx).max`

type(uint224).max or type(uint160).max, etc. it uses more gas in the distribution process and also for each transaction than constant usage.

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L296

```solidity
File: LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol

296                bytes28(type(uint224).max)

378            allowedAddress == address(bytes20(type(uint160).max)) ||

395            allowedStandard == bytes4(type(uint32).max) ||

414            allowedFunction == bytes4(type(uint32).max) ||

```

```diff
diff --git a/LSP6ExecuteModule.org.sol b/LSP6ExecuteModule.sol
index 529790d..c1aec6e 100644
--- a/LSP6ExecuteModule.org.sol
+++ b/LSP6ExecuteModule.sol
@@ -1,3 +1,8 @@
+     uint224 constant MAX_UINT224 = type(uint224).max;
+     uint160 constant MAX_UINT160 = type(uint160).max;
+     uint32 constant MAX_UINT32 = type(uint32).max;


            if (
                bytes28(bytes32(allowedCall) << 32) ==
-                bytes28(type(uint224).max)
+                bytes28(MAX_UINT224)
            ) {



-            allowedAddress == address(bytes20(type(uint160).max)) ||
+            allowedAddress == address(bytes20(MAX_UINT160) ||


-            allowedStandard == bytes4(type(uint32).max) ||
+            allowedStandard == bytes4(MAX_UINT32) ||


-            allowedFunction == bytes4(type(uint32).max) ||
+            allowedFunction == bytes4(MAX_UINT32) ||
```

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP5ReceivedAssets/LSP5Utils.sol

```solidity
File: LSP5ReceivedAssets/LSP5Utils.sol

87        if (oldArrayLength == type(uint128).max) {

190            if (newArrayLength >= type(uint128).max) {
```

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP10ReceivedVaults/LSP10Utils.sol

```solidity
File: LSP10ReceivedVaults/LSP10Utils.sol


85        if (oldArrayLength == type(uint128).max) {

135        if (oldArrayLength > type(uint128).max) {

```


## [G-02] Add `unchecked{}` for subtractions where the operands can't underflow because of previous require() or if-statement

This will stop the check for overflow and underflow so it will save gas.

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L585

```solidity
File: LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol

585                (8 * (32 - length));

705                (8 * (32 - length));
```

add unchecked{} operands can't underflow because of if-statement

```solidity

559            if (length > 32)

679            if (length > 32)
```

```diff
diff --git a/LSP6SetDataModule.org.sol b/LSP6SetDataModule.sol
index 7ea6a1a..f48fbab 100644
--- a/LSP6SetDataModule.org.sol
+++ b/LSP6SetDataModule.sol
@@ -20,8 +20,11 @@
              *                       vvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvv
              *           mask = 0xffffff0000000000000000000000000000000000000000000000000000000000
              */
+           unchecked {
             mask =
                 bytes32(
                     0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
                 ) <<
                 (8 * (32 - length));
+                }
```

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L145

```solidity
File:  LSP7DigitalAsset/LSP7DigitalAssetCore.sol

145            _updateOperator(from, operator, operatorAmount - amount);

```

add unchecked{} operands can't underflow because of if-statement

```solidity

136     if (amount > operatorAmount) {
137                    revert LSP7AmountExceedsAuthorizedAmount(
```

```diff
diff --git a/LSP7DigitalAssetCore.org.sol b/LSP7DigitalAssetCore.sol
index 3e8daed..60d0cc5 100644
--- a/LSP7DigitalAssetCore.org.sol
+++ b/LSP7DigitalAssetCore.sol
@@ -4,5 +4,7 @@
                 );
             }

+            unchecked {
             _updateOperator(from, operator, operatorAmount - amount);
+            }
         }
```

## [G-03] Can make the variable `outside` the loop to save gas

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol#L262

```solidity
File: LSP2ERC725YJSONSchema/LSP2Utils.sol

261      for (uint256 ii = 0; ii < arrayLength; ) {
262            bytes32 key = data.toBytes32(pointer);



292       for (uint256 ii = 0; ii < arrayLength; ) {
293            bytes32 key = data.toBytes32(pointer);
```

```diff
diff --git a/LSP2Utils.org.sol b/LSP2Utils.sol
index 4e927ac..738c4a5 100644
--- a/LSP2Utils.org.sol
+++ b/LSP2Utils.sol
@@ -1,4 +1,5 @@

+        bytes32 key
         for (uint256 ii = 0; ii < arrayLength; ) {
-            bytes32 key = data.toBytes32(pointer);
+             key = data.toBytes32(pointer);

             // check that the leading bytes are zero bytes "00
```

## [G-04] Using `calldata` instead of `memory` for read-only

It is recommended to use calldata instead of memory for read-only arguments whenever possible. By accessing the arguments directly from calldata, you avoid unnecessary data copying and reduce gas costs associated with memory operations.

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol

```solidity
File: LSP0ERC725Account/LSP0ERC725AccountCore.sol

280    function executeBatch(
281        uint256[] memory operationsType,
282        address[] memory targets,
283        uint256[] memory values,
284        bytes[] memory datas
285    ) public


380    function setDataBatch(
381        bytes32[] memory dataKeys,
382        bytes[] memory dataValues
```

```diff

diff --git a/LSP0ERC725AccountCore.org.sol b/LSP0ERC725AccountCore.sol
index 5e63ef4..1834bfa 100644
--- a/LSP0ERC725AccountCore.org.sol
+++ b/LSP0ERC725AccountCore.sol
@@ -1,6 +1,6 @@
     function executeBatch(
-        uint256[] memory operationsType,
-        address[] memory targets,
-        uint256[] memory values,
-        bytes[] memory datas
+        uint256[] calldata operationsType,
+        address[] calldata targets,
+        uint256[] calldata values,
+        bytes[] calldata datas
     ) public payable virtual override returns (bytes[] memory) {

```

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/ILSP8IdentifiableDigitalAsset.sol#L216-L222

```solidity
File: LSP8IdentifiableDigitalAsset/ILSP8IdentifiableDigitalAsset.sol

216   function transferBatch(
217        address[] memory from,
218        address[] memory to,
219        bytes32[] memory tokenId,
220        bool[] memory allowNonLSP1Recipient,
220        bytes[] memory data
220    ) external;
```

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L210-L216

```solidity
File: LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol

210    function transferBatch(
211        address[] memory from,
212        address[] memory to,
213        bytes32[] memory tokenId,
214        bool[] memory allowNonLSP1Recipient,
215        bytes[] memory data
216    ) public virtual {
```

https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/interfaces/IERC725Y.sol#L33-L35

```solidity
File:  interfaces/IERC725Y.sol

33   function getDataBatch(
34         bytes32[] memory dataKeys
35     ) external view returns (bytes[] memory dataValues);
```

https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725YCore.sol

```solidity
File:  contracts/ERC725YCore.sol

73     function setDataBatch(
74         bytes32[] memory dataKeys,
75         bytes[] memory dataValues
76     ) public payable virtual override onlyOwner {


43        bytes32[] memory dataKeys
```

https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725XCore.sol#L52-L57

```solidity
File: ERC725XCore.sol

52    function executeBatch(
53         uint256[] memory operationsType,
54         address[] memory targets,
55         uint256[] memory values,
56         bytes[] memory datas



127    function _executeBatch(
128        uint256[] memory operationsType,
129        address[] memory targets,
130        uint256[] memory values,
131        bytes[] memory datas
132    ) internal virtual returns (bytes[] memory) {
```

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L154-L160

```solidity
File: LSP7DigitalAsset/LSP7DigitalAssetCore.sol

154    function transferBatch(
154         address[] memory from,
154         address[] memory to,
154         uint256[] memory amount,
154         bool[] memory allowNonLSP1Recipient,
154         bytes[] memory data
154    ) public virtual {
```

```diff

diff --git a/LSP7DigitalAssetCore.org.sol b/LSP7DigitalAssetCore.sol
index 35c073d..7021bb7 100644
--- a/LSP7DigitalAssetCore.org.sol
+++ b/LSP7DigitalAssetCore.sol
@@ -1,8 +1,8 @@
     function transferBatch(
-        address[] memory from,
-        address[] memory to,
-        uint256[] memory amount,
-        bool[] memory allowNonLSP1Recipient,
-        bytes[] memory data
+        address[] calldata from,
+        address[] calldata to,
+        uint256[] calldata amount,
+        bool[] calldata allowNonLSP1Recipient,
+        bytes[] calldata data
     ) public virtual {
         uint256 fromLength = from.length;
```

## [G-05] Use assembly to check for `address(0)`

for example

```solidity
function ownerNotZero(address _addr) public pure {
    require(_addr != address(0), "zero address)");
}
```

```solidity
function assemblyOwnerNotZero(address _addr) public pure {
    assembly {
        if iszero(_addr) {
            mstore(0x00, "zero address")
            revert(0x00, 0x20)
        }
    }
}
```

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol

```solidity
File: LSP0ERC725Account/LSP0ERC725AccountCore.sol

804        if (extension == address(0))

```

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8Enumerable.sol

```solidity
File: LSP8IdentifiableDigitalAsset/extensions/LSP8Enumerable.sol

36        if (from == address(0)) {

40        } else if (to == address(0)) {
```

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8EnumerableInitAbstract.sol

```solidity
File: LSP8IdentifiableDigitalAsset/extensions/LSP8EnumerableInitAbstract.sol

38        if (from == address(0)) {

42        } else if (to == address(0)) {

```

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol

```solidity
File: LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol

84        if (tokenOwner == address(0)) {

115        if (operator == address(0)) {

139        if (operator == address(0)) {

319        if (to == address(0)) {

410        if (to == address(0)) {
```

https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725XCore.sol

```solidity
File: ERC725XCore.sol

87            if (target != address(0)) revert ERC725X_CreateOperationsRequireEmptyRecipientAddress();

93            if (target != address(0)) revert ERC725X_CreateOperationsRequireEmptyRecipientAddress();

239        if (contractAddress == address(0)) {
```

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol

```solidity
File: LSP7DigitalAsset/LSP7DigitalAssetCore.sol


268        if (operator == address(0)) {

300        if (to == address(0)) {

343        if (from == address(0)) {

406        if (from == address(0) || to == address(0)) {

```

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP17ContractExtension/LSP17Extendable.sol

```solidity
File: LSP17ContractExtension/LSP17Extendable.sol

50        if (erc165Extension == address(0)) return false;

91        if (extension == address(0))

```

## [G-06] State variables `can be packed` into fewer storage slots

Here, the storage variables can be tightly packed by putting data type that can fit together next to each other.

To save one storage slot by reordering the variables, you can declare the `_INTERFACEID_LSP6` constant after the `_LSP6KEY_ADDRESSPERMISSIONS_ALLOWEDCALLS_PREFIX` constant.

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Constants.sol

```solidity
File: LSP6KeyManager/LSP6Constants.sol

7   bytes4 constant _INTERFACEID_LSP6 = 0x38bb3cdb;
```

```diff
diff --git a/LSP6Constants.org.sol b/LSP6Constants.sol
index a891c24..b13f5d1 100644
--- a/LSP6Constants.org.sol
+++ b/LSP6Constants.sol
@@ -1,5 +1,7 @@
 // bytes6(keccak256('AddressPermissions')) + bytes4(keccak256('AllowedCalls'))
 bytes10 constant _LSP6KEY_ADDRESSPERMISSIONS_ALLOWEDCALLS_PREFIX = 0x4b80742de2bf393a64c7; // AddressPermissions:AllowedCalls:<address>
+// --- ERC165 interface ids
+bytes4 constant _INTERFACEID_LSP6 = 0x38bb3cdb;

 // DEFAULT PERMISSIONS VALUES
 // NB: the SUPER PERMISSIONS allo
```

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol#L39-L45

```solidity
File: LSP14Ownable2Step/LSP14Ownable2Step.sol

39    uint256 public constant RENOUNCE_OWNERSHIP_CONFIRMATION_DELAY = 200;

44    uint256 public constant RENOUNCE_OWNERSHIP_CONFIRMATION_PERIOD = 200;
```

To optimize storage usage and save one slot, we can combine the constants `RENOUNCE_OWNERSHIP_CONFIRMATION_DELAY` and `RENOUNCE_OWNERSHIP_CONFIRMATION_PERIOD` into a single storage slot of type uint128.

```solidity

39   uint128  public constant RENOUNCE_OWNERSHIP_CONFIRMATION_DELAY = 200;

44   uint128  public constant RENOUNCE_OWNERSHIP_CONFIRMATION_PERIOD = 200;
```

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP4DigitalAssetMetadata/LSP4Constants.sol

```solidity
File:  LSP4DigitalAssetMetadata/LSP4Constants.sol

10      bytes constant _LSP4_SUPPORTED_STANDARDS_VALUE = hex"a4d96624";


24      bytes12 constant _LSP4_CREATORS_MAP_KEY_PREFIX = 0x6de85eaf5d982b4e5da00000;
```

To optimize storage usage and save two storage slots, we can pack the variables `_LSP4_SUPPORTED_STANDARDS_VALUE` and `_LSP4_CREATORS_MAP_KEY_PREFIX` into the same storage slot

```diff
diff --git a/LSP4Constants.org.sol b/LSP4Constants.sol
index b36ca4d..d7ea290 100644
--- a/LSP4Constants.org.sol
+++ b/LSP4Constants.sol
@@ -1,9 +1,6 @@
 // bytes10(keccak256('SupportedStandards')) + bytes2(0) + bytes20(keccak256('LSP4DigitalAsset'))
 bytes32 constant _LSP4_SUPPORTED_STANDARDS_KEY = 0xeafec4d89fa9619884b60000a4d96624a38f7ac2d8d9a604ecf07c12c77e480c;

-// bytes4(keccak256('LSP4DigitalAsset'))
-bytes constant _LSP4_SUPPORTED_STANDARDS_VALUE = hex"a4d96624";

 // keccak256('LSP4TokenName')
 bytes32 constant _LSP4_TOKEN_NAME_KEY = 0xdeba1e292f8ba88238e10ab3c7f88bd4be4fac56cad5194b6ecceaf653468af1;

// keccak256('LSP4TokenSymbol')
 bytes32 constant _LSP4_TOKEN_SYMBOL_KEY = 0x2f0a68ab07768e01943a599e73362a0e17a6

 // keccak256('LSP4Creators[]')
 bytes32 constant _LSP4_CREATORS_ARRAY_KEY = 0x114bd03b3a46d48759680d81ebb2b414fda7d030a7105a851867accf1c2352e7;

- // bytes10(keccak256('LSP4CreatorsMap')) + bytes2(0)
- bytes12 constant _LSP4_CREATORS_MAP_KEY_PREFIX = 0x6de85eaf5d982b4e5da00000;

// keccak256('LSP4Metadata')
bytes32 constant _LSP4_METADATA_KEY = 0x9afb95cacc9f95858ec44aa8c3b685511002e30ae54415823f406128b85b238e;

+ // bytes10(keccak256('LSP4CreatorsMap')) + bytes2(0)
+ bytes12 constant _LSP4_CREATORS_MAP_KEY_PREFIX = 0x6de85eaf5d982b4e5da00000;

+// bytes4(keccak256('LSP4DigitalAsset'))
+bytes constant _LSP4_SUPPORTED_STANDARDS_VALUE = hex"a4d96624";

```

## [G-07] Empty blocks should be `removed` or `emit` something

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting.
If the contract is meant to be extended, the contract should be abstract and the function signatures be added without any default implementation.

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L444-L449

```solidity
File: LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol

444   function _beforeTokenTransfer(
445        address from,
446        address to,
447        bytes32 tokenId
448    ) internal virtual {}
```

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L442-L446

```solidity
File: LSP7DigitalAsset/LSP7DigitalAssetCore.sol

442    function _beforeTokenTransfer(
443        address from,
444        address to,
445        uint256 amount
446    ) internal virtual {}
```

## [G-08] State variables only set in the `constructor` should be declared `immutable`

Avoids a Gsset (20000 gas) in the constructor, and replaces the first access in each transaction (Gcoldsload - 2100 gas) and each access thereafter (Gwarmacces - 100 gas) with a PUSH32 (3 gas).

While strings are not value types, and therefore cannot be immutable/constant if not hard-coded outside of the constructor, the same behavior can be achieved by making the current contract abstract with virtual functions for the string accessors, and having a child contract override the functions with the hard-coded implementation-specific values.

https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/custom/OwnableUnset.sol#L12

```solidity
File: custom/OwnableUnset.sol

12     address private _owner;
```

This variable should be immutable because it's set in the constructor

https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725.sol#L23-L27

```solidity
File: ERC725.sol

23    constructor(address newOwner) {
24          require(newOwner != address(0), "Ownable: new owner is the zero address");
25         OwnableUnset._setOwner(newOwner);
26     }
```

## [G-09] Duplicated `if()` checks should be refactored to a modifier or function

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol

```solidity
File: LSP0ERC725Account/LSP0ERC725AccountCore.sol

118        if (msg.value != 0) {
152        if (msg.value != 0) {
228        if (msg.value != 0) {
286        if (msg.value != 0) {
343        if (msg.value != 0) {
384        if (msg.value != 0) {
464        if (msg.value != 0) {


235        if (msg.sender == _owner) {
293        if (msg.sender == _owner) {
350        if (msg.sender == _owner) {
395        if (msg.sender == _owner) {
663        if (msg.sender == _owner) {

252        if (verifyAfter) {
316        if (verifyAfter) {
362        if (verifyAfter) {
421        if (verifyAfter) {
604        if (verifyAfter) {
676        if (verifyAfter) {
```

```diff
diff --git a/LSP0ERC725AccountCore.org.sol b/LSP0ERC725AccountCore.sol
index 60d5c82..ad0289d 100644
--- a/LSP0ERC725AccountCore.org.sol
+++ b/LSP0ERC725AccountCore.sol
@@ -3,7 +3,13 @@
         address target,
         uint256 value,
         bytes memory data
-    ) public payable virtual override returns (bytes memory) {
-        if (msg.value != 0) {
+    ) public payable virtual override NonZeroValue returns (bytes memory) {
             emit ValueReceived(msg.sender, msg.value);
        }
       }

+
+modifier NonZeroValue() {
+    require(msg.value != 0, "Value must be non-zero");
+    _;
+}


```

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol

```solidity
File: LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol

115        if (operator == address(0)) {
139        if (operator == address(0)) {

119        if (tokenOwner == operator) {
143        if (tokenOwner == operator) {

319        if (to == address(0)) {
410        if (to == address(0)) {

```

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol

```solidity
File: LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol

77        if (requiredPermission == _PERMISSION_SETDATA) {
142        if (requiredPermission == _PERMISSION_SETDATA) {

```

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol

```solidity
File: LSP6KeyManager/LSP6KeyManagerCore.sol

282            if (isReentrantCall) {
532            if (isReentrantCall) {

265        if (msg.sender == _target) {
309        if (msg.sender == _target) {

338        if (!isReentrantCall && !isSetData) {
404        if (!isReentrantCall && !isSetData) {

```

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol

```solidity
File: LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol

107        if (operationType == OPERATION_0_CALL) {
329        if (operationType == OPERATION_0_CALL) {

```

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol

```solidity
File: LSP2ERC725YJSONSchema/LSP2Utils.sol

254        if (!isEncodedArray(data)) return false;
286        if (!isEncodedArray(data)) return false;
```

## [G-10] Internal functions `only called` once can be inlined to save gas

Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L852

```solidity
File: contracts/LSP0ERC725Account/LSP0ERC725AccountCore.so

851    function _getExtension(
852        bytes4 functionSelector
853    ) internal view virtual override returns (address) {
```

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol

```solidity
File: LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol

152     function _whenReceiving(
153         bytes32 typeId,
154         address notifier,
155         bytes32 notifierMapKey,
156         bytes4 interfaceID
157:    ) internal virtual returns (bytes memory) {


201     function _whenSending(
202         bytes32 typeId,
203         address notifier,
204         bytes32 notifierMapKey,
205         uint128 arrayIndex
206:    ) internal virtual returns (bytes memory) {
```

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol#L284-L288

```solidity
File: LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol

284   function _setApprovalForAll(
285         address tokensOwner,
286         address operator,
287         bool approved
288:     ) internal virtual {
```

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol#L226-L232

```solidity
File: LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol

226    function _transfer(
227        address from,
228        address to,
229        bytes32 tokenId,
230        bool allowNonLSP1Recipient,
231        bytes memory data
232    ) internal virtual override {



284    function _setApprovalForAll(
285        address tokensOwner,
286        address operator,
287        bool approved
288    ) internal virtual {
```

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L394-L400

```solidity
File: LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol

394    function _transfer(
395        address from,
396        address to,
397        bytes32 tokenId,
398        bool allowNonLSP1Recipient,
399        bytes memory data
400:    ) internal virtual {
```

https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/custom/OwnableUnset.sol#L54

```solidity
File: custom/OwnableUnset.sol

54    function _checkOwner() internal view virtual {
```

https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725XCore.sol

```solidity
File:  ERC725XCore.sol

127     function _executeBatch(
128         uint256[] memory operationsType,
129         address[] memory targets,
130         uint256[] memory values,
131         bytes[] memory datas
132:    ) internal virtual returns (bytes[] memory) {



165     function _executeCall(
166         address target,
167         uint256 value,
168         bytes memory data
169:    ) internal virtual returns (bytes memory result) {



187     function _executeStaticCall(
188          address target,
189          bytes memory data
200:     ) internal virtual returns (bytes memory result) {



204     function _executeDelegateCall(
205         address target,
206         bytes memory data
207:    ) internal virtual returns (bytes memory result) {



253     function _deployCreate2(
254         uint256 value,
255         bytes memory creationCode
256:    ) internal virtual returns (bytes memory newContract) {
```

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L385-L388

```solidity
File: LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol

385     function _getPermissionToSetControllerPermissions(
386         address controlledContract,
387         bytes32 inputPermissionDataKey
388:    ) internal view virtual returns (bytes32) {



406    function _getPermissionToSetAllowedCalls(
407         address controlledContract,
408         bytes32 dataKey,
409         bytes memory dataValue,
410         bool hasBothAddControllerAndEditPermissions
411:    ) internal view virtual returns (bytes32) {



438     function _getPermissionToSetAllowedERC725YDataKeys(
439         address controlledContract,
440         bytes32 dataKey,
441         bytes memory dataValue,
442         bool hasBothAddControllerAndEditPermissions
443:    ) internal view returns (bytes32) {


474     function _getPermissionToSetLSP1Delegate(
475         address controlledContract,
476         bytes32 lsp1DelegateDataKey
477:    ) internal view virtual returns (bytes32) {



490    function _getPermissionToSetLSP17Extension(
491        address controlledContract,
492        bytes32 lsp17ExtensionDataKey
493:    ) internal view virtual returns (bytes32) {



507    function _verifyAllowedERC725YSingleKey(
508        address controllerAddress,
509        bytes32 inputDataKey,
510        bytes memory allowedERC725YDataKeysCompacted
511:    ) internal pure virtual {



622     function _verifyAllowedERC725YDataKeys(
623         address controllerAddress,
624         bytes32[] memory inputDataKeys,
625         bytes memory allowedERC725YDataKeysCompacted,
626         bool[] memory validatedInputKeysList,
627         uint256 allowedDataKeysFound
628:    ) internal pure virtual {
```

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L441-L444

```solidity
File: LSP6KeyManager/LSP6KeyManagerCore.sol

441    function _isValidNonce(
442        address from,
443        uint256 idx
444    ) internal view virtual returns (bool) {
```

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L153-L157

```solidity
File: LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol


153     function _verifyCanDeployContract(
154         address controller,
155         bytes32 permissions,
156         bool isFundingContract
157:    ) internal view virtual {


170     function _verifyCanStaticCall(
171        address controlledContract,
172        address controller,
173        bytes32 permissions,
174        bytes calldata payload
175:    ) internal view virtual {



188     function _verifyCanCall(
188         address controlledContract,
190         address controller,
191         bytes32 permissions,
192         bytes calldata payload
193:    ) internal view virtual {



317     function _extractCallType(
318         uint256 operationType,
319         uint256 value,
320         bool isEmptyCall
320     ) internal pure returns (bytes4 requiredCallTypes) {



339     function _extractExecuteParameters(
340         bytes calldata executeCalldata
340:    ) internal pure returns (uint256, address, uint256, bytes4, bool) {



366     function _isAllowedAddress(
367         bytes memory allowedCall,
368         address to
369:     ) internal pure returns (bool) {


382    function _isAllowedStandard(
383        bytes memory allowedCall,
384        address to
385:    ) internal view returns (bool) {



399    function _isAllowedFunction(
400        bytes memory allowedCall,
401        bytes4 requiredFunction
402:    ) internal pure returns (bool) {



418     function _isAllowedCallType(
419         bytes memory allowedCall,
420         bytes4 requiredCallTypes
421:     ) internal pure returns (bool) {
```

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L399-L405

```solidity
File: LSP7DigitalAsset/LSP7DigitalAssetCore.sol

399     function _transfer(
400         address from,
401         address to,
402         uint256 amount,
403         bool allowNonLSP1Recipient,
404         bytes memory data
405:    ) internal virtual {
```

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP20CallVerification/LSP20CallVerification.sol#L84

```solidity
File: LSP20CallVerification/LSP20CallVerification.sol

84    function _revert(bool postCall, bytes memory returnedData) internal pure {
```

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol#L126

```solidity
File: LSP14Ownable2Step/LSP14Ownable2Step.sol

126    function _transferOwnership(address newOwner) internal virtual {

136    function _acceptOwnership() internal virtual {

```

## [G-11] Use nested if and, avoid multiple check combinations `&&`

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L362

```solidity
File: LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol

362        if (inputDataValue.length != 0 && inputDataValue.length != 20) {

```

```diff
diff --git a/LSP6SetDataModule.org.sol b/LSP6SetDataModule.sol
index cd052a5..8f6e424 100644
--- a/LSP6SetDataModule.org.sol
+++ b/LSP6SetDataModule.sol
@@ -1,6 +1,10 @@
-      if (inputDataValue.length != 0 && inputDataValue.length != 20) {
            revert AddressPermissionArrayIndexValueNotAnAddress(
                inputDataKey,
                inputDataValue
+      if (inputDataValue.length != 0) {
+            if (inputDataValue.length != 20) {
                revert AddressPermissionArrayIndexValueNotAnAddress(
                    inputDataKey,
                    inputDataValue
             );
        }

```

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol

```solidity
File: LSP6KeyManager/LSP6KeyManagerCore.sol

338        if (!isReentrantCall && !isSetData) {

404        if (!isReentrantCall && !isSetData) {
```

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol

```solidity
File: LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol

165        if (isFundingContract && !hasSuperTransferValue) {

214        if (isTransferringValue && !hasSuperTransferValue) {

224        if (!hasSuperCall && !isCallDataPresent && !isTransferringValue) {

228        if (isCallDataPresent && !hasSuperCall) {

233        if (hasSuperCall && !isTransferringValue) return;

236        if (hasSuperTransferValue && !isCallDataPresent && isTransferringValue)

240        if (hasSuperCall && hasSuperTransferValue) return;

301            if (
302                _isAllowedCallType(allowedCall, requiredCallTypes) &&
303                _isAllowedAddress(allowedCall, to) &&
304                _isAllowedStandard(allowedCall, to) &&
305                _isAllowedFunction(allowedCall, selector)
306            ) return;

```

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol

```solidity
File: LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol


227            if (dataKeys.length == 0 && dataValues.length == 0)

245            if (dataKeys.length == 0 && dataValues.length == 0)

```

## [G-12] Use assembly to write `address storage` values

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol#L129

```solidity
File: LSP14Ownable2Step/LSP14Ownable2Step.sol


126    function _transferOwnership(address newOwner) internal virtual {
127        if (newOwner == address(this)) revert CannotTransferOwnershipToSelf();
128
129        _pendingOwner = newOwner;
130        delete _renounceOwnershipStartedAt;
131    }
```

```diff
diff --git a/LSP14Ownable2Step.org.sol b/LSP14Ownable2Step.sol
index fdf1292..3215f5c 100644
--- a/LSP14Ownable2Step.org.sol
+++ b/LSP14Ownable2Step.sol
@@ -1,6 +1,8 @@
     function _transferOwnership(address newOwner) internal virtual {
         if (newOwner == address(this)) revert CannotTransferOwnershipToSelf();

-        _pendingOwner = newOwner;
+        assembly {
+            sstore(_pendingOwner.slot, newOwner)
+        }
         delete _renounceOwnershipStartedAt;
   }


```

## [G-13] Use `selfbalance()` instead of address(this).balance

https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725XCore.sol

You can use selfbalance() instead of address(this).balance when getting your contract’s balance of ETH to save gas.

for example:

```solidity
function addressInternalBalance() public returns (uint256) {
    return address(this).balance;
}


function assemblyInternalBalance() public returns (uint256) {
    assembly {
        let c := selfbalance()
        mstore(0x00, c)
        return(0x00, 0x20)
    }
}

```

```solidity
File: ERC725XCore.sol

170        if (address(this).balance < value) {
171            revert ERC725X_InsufficientBalance(address(this).balance, value);
172        }


225        if (address(this).balance < value) {
226            revert ERC725X_InsufficientBalance(address(this).balance, value);
227        }

```

## [G-14] `abi.encode()` is less efficient than abi.encodePacked()

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol

```solidity
File: LSP0ERC725Account/LSP0ERC725AccountCore.sol

253            LSP20CallVerification._verifyCallResult(_owner, abi.encode(result));

319                abi.encode(results)

528        returnedValues = abi.encode(
```

## [G-15] Using `> 0` costs more gas than `!= 0`

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol#L165

```solidity
File: LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol

165            if (notifier.code.length > 0) {

```

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol#L313

```solidity
File: LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol

313        if (to.code.length > 0) {

```

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L493

```solidity
File: LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol

493            if (to.code.length > 0) {

```

## [G-16] `Ternary` operation is cheaper than if-else statement

There are instances where a ternary operation can be used instead of if-else statement. In these cases, using ternary operation will save modest amounts of gas.

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L278-L282

```solidity
File: LSP7DigitalAsset/LSP7DigitalAssetCore.sol

112        if (tokenOwner == operator) {
113            return _tokenOwnerBalances[tokenOwner];
114        } else {
115            return _operatorAuthorizedAmount[tokenOwner][operator];
116        }




278      if (amount != 0) {
279            emit AuthorizedOperator(operator, tokenOwner, amount);
280        } else {
281            emit RevokedOperator(operator, tokenOwner);
282        }
```

```diff
diff --git a/LSP7DigitalAssetCore.org.sol b/LSP7DigitalAssetCore.sol
index 2384ebf..7283645 100644
--- a/LSP7DigitalAssetCore.org.sol
+++ b/LSP7DigitalAssetCore.sol
@@ -2,11 +2,8 @@


    function authorizedAmountFor(
         address operator,
         address tokenOwner
     ) public view virtual returns (uint256) {
-        if (tokenOwner == operator) {
-            return _tokenOwnerBalances[tokenOwner];
-        } else {
-            return _operatorAuthorizedAmount[tokenOwner][operator];
-        }
+   return tokenOwner == operator ? _tokenOwnerBalances[tokenOwner] : _operatorAuthorizedAmount[tokenOwner][operator];
     }





         _operatorAuthorizedAmount[tokenOwner][operator] = amount;

-        if (amount != 0) {
-            emit AuthorizedOperator(operator, tokenOwner, amount);
-        } else {
-            emit RevokedOperator(operator, tokenOwner);
-        }
+        amount != 0 ? emit AuthorizedOperator(operator, tokenOwner, amount) : emit RevokedOperator(operator, tokenOwner);

     }
```

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol#L129-L138

```solidity
File: LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol

129   if (operatorListLength == 0) {
130            return address(0);
131        } else {
132            // Read the last added operator authorized to provide "best" compatibility.
133            // In ERC721 there is one operator address at a time for a tokenId, so multiple calls to
134            // `approve` would cause `getApproved` to return the last added operator. In this
135            // compatibility version the same is true, when the authorized operators were not previously
136            // authorized. If addresses are removed, then `getApproved` returned address can change due
137            // to implementation of `EnumberableSet._remove`.
138            return operatorsForTokenId[operatorListLength - 1];
139          }
```

```diff

diff --git a/LSP8CompatibleERC721.org.sol b/LSP8CompatibleERC721.sol
index d76c438..775e599 100644
--- a/LSP8CompatibleERC721.org.sol
+++ b/LSP8CompatibleERC721.sol
@@ -7,16 +7,7 @@
         address[] memory operatorsForTokenId = getOperatorsOf(tokenIdAsBytes32);
         uint256 operatorListLength = operatorsForTokenId.length;

-        if (operatorListLength == 0) {
-            return address(0);
-        } else {
-            // Read the last added operator authorized to provide "best" compatibility.
-            // In ERC721 there is one operator address at a time for a tokenId, so multiple calls to
-            // `approve` would cause `getApproved` to return the last added operator. In this
-            // compatibility version the same is true, when the authorized operators were not previously
-            // authorized. If addresses are removed, then `getApproved` returned address can change due
-            // to implementation of `EnumberableSet._remove`.
-            return operatorsForTokenId[operatorListLength - 1];
-        }
+        return operatorListLength == 0 ? address(0) : operatorsForTokenId[operatorListLength - 1];

```

## [G-17] Pre-increment and pre-decrement are cheaper than `±1`

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8Enumerable.sol#L41

```diff
-            uint256 lastIndex = totalSupply() - 1;
+            uint256 lastIndex = --totalSupply() ;
```

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8EnumerableInitAbstract.sol#L43

```diff
-            uint256 lastIndex = totalSupply() - 1;
+            uint256 lastIndex = --totalSupply() ;
```

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Utils.sol#L126

```diff

-            if (pointer + 1 >= allowedERC725YDataKeysCompacted.length)
+            if (++pointer  >= allowedERC725YDataKeysCompacted.length)

```

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP10ReceivedVaults/LSP10Utils.sol#L89

```diff

-        uint128 newArrayLength = oldArrayLength + 1;
+        uint128 newArrayLength = ++oldArrayLength;

```

## [G-18] When possible, use assembly instead of `unchecked{++i}`

You can also use unchecked{++i;} for even more gas savings but this will not check to see if i overflows. For best gas savings, use inline assembly, however, this limits the functionality you can achieve.

for example:

```solidity
//loop with unchecked{++i}
function uncheckedPlusPlusI() public pure {
    uint256 j = 0;
    for (uint256 i; i < 10; ) {
        j++;
        unchecked {
            ++i;
        }
    }
}
Gas: 1329


//loop with inline assembly
function inlineAssemblyLoop() public pure {
    assembly {
        let j := 0
        for {
            let i := 0
        } lt(i, 10) {
            i := add(i, 0x01)
        } {
            j := add(j, 0x01)
        }
    }
}

Gas: 709

```

This is significant gas saving

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol

```solidity
File: LSP6KeyManager/LSP6KeyManagerCore.sol

170           unchecked {
171                ++ii;
172            }



236            unchecked {
237                ++ii;
238            }
```

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol

```solidity
File: LSP7DigitalAsset/LSP7DigitalAssetCore.sol

181            unchecked {
182                ++i;
183            }
```

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol

```solidity
File: LSP2ERC725YJSONSchema/LSP2Utils.sol

271            unchecked {
272                ++ii;
273            }


301            unchecked {
302                ++ii;
303            }
```

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol

```solidity
File: LSP0ERC725Account/LSP0ERC725AccountCore.sol

197            unchecked {
198                ++i;
199            }


399                unchecked {
400                    ++i;
401                }


414            unchecked {
415                ++i;
416            }
```

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol

```solidity
File: LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol

236            unchecked {
237                ++i;
238            }


278            unchecked {
279                ++i;
280            }
```

https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725YCore.sol

```solidity
File: ERC725YCore.sol

51            unchecked {
52                ++i;
53            }



92            unchecked {
93                ++i;
94            }
```

https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725XCore.sol

```solidity
File: ERC725XCore.sol

150            unchecked {
151                ++i;
152            }
```