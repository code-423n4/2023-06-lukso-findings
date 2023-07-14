# GAS OPTIMIZATIONS

##

## [G-1] State variables can be packed into fewer storage slots

If variables occupying the same slot are both written the same function or by the constructor, avoids a separate Gsset (20000 gas). Reads of the variables can also be cheaper.

### ``_existingTokens,_isNonDivisible`` can be packed with same slot . Saves ``2000 GAS, 1 SLOT``

_existingTokens downcast this to uint248 to avoid extra SLOT. This will not affect the protocols flow 

```solidity
FILE: 2023-06-lukso/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol

42:    mapping(address => mapping(address => uint256))
43:        private _operatorAuthorizedAmount;
44:
- 45:    uint256 private _existingTokens;
+ 45:    uint248 private _existingTokens;
46:
47:    bool internal _isNonDivisible;

```

##

## [G-2] State variables should be cached in stack variables rather than re-reading them from storage

### Saves ``600 GAS, 6 SLOT`` 

### NOTE: This instances are missed in BOT RACE

The instances below point to the second+ access of a state variable within a function. Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

### ``LSP6KeyManagerCore.sol``: ``_target`` should be cached with local address variable : Saves ``300 GAS, 3 SLOD``

https://github.com/code-423n4/2023-06-lukso/blob/9dbc96410b3052fc0fd9d423249d1fa42958cae8/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L462

```diff
FILE: Breadcrumbs2023-06-lukso/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol

+ address _targetAdd=_target;
- 462: bytes32 permissions = ERC725Y(_target).getPermissionsFor(from);
+ 462: bytes32 permissions = ERC725Y(_targetAdd).getPermissionsFor(from);
- 476     _target,
+ 476     _targetAdd,
- 490:   _target,
+ 490:   _targetAdd,
- 500:  _target,
+ 500:  _targetAdd,
```

### ``LSP7DigitalAssetCore.sol``: ``balance, authorizedAmount`` values should be used instead of state variables : Saves ``300 GAS, 3 SLOD``

https://github.com/code-423n4/2023-06-lukso/blob/9dbc96410b3052fc0fd9d423249d1fa42958cae8/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L347


```solidity
FILE: 2023-06-lukso/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol

  uint256 balance = _tokenOwnerBalances[from];
        if (amount > balance) {
            revert LSP7AmountExceedsBalance(balance, from, amount);
        }

        address operator = msg.sender;
        if (operator != from) {
            uint256 authorizedAmount = _operatorAuthorizedAmount[from][
                operator
            ];
            if (amount > authorizedAmount) {
                revert LSP7AmountExceedsAuthorizedAmount(
                    from,
                    authorizedAmount,
                    operator,
                    amount
                );
            }
-            _operatorAuthorizedAmount[from][operator] -= amount;
+            _operatorAuthorizedAmount[from][operator] = authorizedAmount - amount;
        }

        _beforeTokenTransfer(from, address(0), amount);

        // tokens being burned
        _existingTokens -= amount;

-         _tokenOwnerBalances[from] -= amount;
+         _tokenOwnerBalances[from] =balance - amount;

```

```diff

     uint256 balance = _tokenOwnerBalances[from];
        if (amount > balance) {
            revert LSP7AmountExceedsBalance(balance, from, amount);
        }

        address operator = msg.sender;

        _beforeTokenTransfer(from, to, amount);

-        _tokenOwnerBalances[from] -= amount;
+        _tokenOwnerBalances[from] =balance - amount;
```
##

## [G-3] Gas-Saving Benefits of Same Value Checks for Boolean State Variables 

Saves ``60000 GAS, 3 SSTORE``

This ensures that the value of the ``bool`` state variable is never overwritten with the same value. This will saves ``60000 GAS and 3 SSTORE``. Add same value check before assigning values 

```solidiity
FILE: 2023-06-lukso/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol

540: if (!isSetData) {
541:                _reentrancyStatus = true;
542:            }

```
https://github.com/code-423n4/2023-06-lukso/blob/9dbc96410b3052fc0fd9d423249d1fa42958cae8/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L540

```solidity
FILE: 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions
/LSP8CompatibleERC721InitAbstract.sol

293:  _operatorApprovals[tokensOwner][operator] = approved;

```
https://github.com/code-423n4/2023-06-lukso/blob/9dbc96410b3052fc0fd9d423249d1fa42958cae8/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol#L293

```solidity
FILE: 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol

293: _operatorApprovals[tokensOwner][operator] = approved;

```
https://github.com/code-423n4/2023-06-lukso/blob/9dbc96410b3052fc0fd9d423249d1fa42958cae8/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol#L293C7-L293C62

##

## [G-4] ``Increment`` or ``decrement`` by 1 can use ``++X`` instead of ``X+=1`` saves gas in state variables 

### Saves ``64 GAS, 2 Instances``

using the increment or decrement operator (++) or (--) instead of the assignment operator (+=) or (-=) can save gas when incrementing or decrementing a state variable.

The reason for this is that the increment or decrement operator is a single instruction, while the assignment operator is two instructions. The first instruction assigns the value of the variable to itself, and the second instruction increments or decrements the value.

Saves ``32 GAS`` per Instance 

```solidity
FILE: 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol

- 332:  _existingTokens += 1;
+ 332:  ++_existingTokens;

- 336:  _existingTokens -= 1;
+ 336:  --_existingTokens;
```
https://github.com/code-423n4/2023-06-lukso/blob/9dbc96410b3052fc0fd9d423249d1fa42958cae8/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L332C8-L332C30

## 

## [G-5] Don't assign default values to variables to save gas 

### Saves ``200 GAS, 6 Instances``

 You should avoid assigning default values to variables to save gas. This is because the gas cost of assigning a value to a variable is ``32 GAS``, regardless of the value being assigned.

```solidity
FILE: 2023-06-lukso/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol

163: for (uint256 ii = 0; ii < payloads.length; ) {
223: for (uint256 ii = 0; ii < payloads.length; ) {

FILE: Breadcrumbs2023-06-lukso/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol

272: for (uint256 ii = 0; ii < allowedCalls.length; ii += 34) {

FILE: Breadcrumbs2023-06-lukso/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol

171: for (uint256 i = 0; i < fromLength; ) {

FILE: 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol

227: for (uint256 i = 0; i < fromLength; ) {

273: for (uint256 i = 0; i < operatorListLength; ) {

```

##

## [G-6] Splitting if() statements that use && saves gas

Saves 13 gas per instance 

```solidity
FILE: 2023-06-lukso/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol

165: if (isFundingContract && !hasSuperTransferValue) {
214: if (isTransferringValue && !hasSuperTransferValue) {
301: if (
                _isAllowedCallType(allowedCall, requiredCallTypes) &&
                _isAllowedAddress(allowedCall, to) &&
                _isAllowedStandard(allowedCall, to) &&
                _isAllowedFunction(allowedCall, selector)
            ) return;

224: if (!hasSuperCall && !isCallDataPresent && !isTransferringValue) {
228: if (isCallDataPresent && !hasSuperCall) {
233: if (hasSuperCall && !isTransferringValue) return;
236: if (hasSuperTransferValue && !isCallDataPresent && isTransferringValue)
240: if (hasSuperCall && hasSuperTransferValue) return;

```

## 

## [G-7] Avoid declaring new variable when only used once inside the function 

Saves ``32 GAS``

```diff
FILE: 2023-06-lukso/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol

- 111: uint256 nonceInChannel = _nonceStore[from][channelId];
- 112:    return (uint256(channelId) << 128) | nonceInChannel;
+ 112:    return (uint256(channelId) << 128) | _nonceStore[from][channelId];
```
https://github.com/code-423n4/2023-06-lukso/blob/9dbc96410b3052fc0fd9d423249d1fa42958cae8/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L111

```diff
FILE: Breadcrumbs2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol

- 123:  bool isAdded = _operators[tokenId].add(operator);
- 124:        if (!isAdded) revert LSP8OperatorAlreadyAuthorized(operator, tokenId);
+ 124:        if (!_operators[tokenId].add(operator)) revert LSP8OperatorAlreadyAuthorized(operator, tokenId);


- 181: address tokenOwner = tokenOwnerOf(tokenId);
182:
- 184:        return (caller == tokenOwner || _operators[tokenId].contains(caller));
+ 184:        return (caller == tokenOwnerOf(tokenId) || _operators[tokenId].contains(caller));


- 250: bool isRemoved = _operators[tokenId].remove(operator);
- 251:        if (!isRemoved) revert LSP8NonExistingOperator(operator, tokenId);
+ 251:        if (!_operators[tokenId].remove(operator)) revert LSP8NonExistingOperator(operator, tokenId);


```
https://github.com/code-423n4/2023-06-lukso/blob/9dbc96410b3052fc0fd9d423249d1fa42958cae8/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L123C7-L125C1

##

## [G-8] Avoid contract existence checks by using low level calls

### Saves ``600 GAS, 6 Instances`` 

### NOTE: Missing instances from  BOT RACE

Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence

```solidity
FILE: 2023-06-lukso/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol

462: ILSP1UniversalReceiver(from).universalReceiver(
                _TYPEID_LSP7_TOKENSSENDER,
                lsp1Data
            );

486: ILSP1UniversalReceiver(to).universalReceiver(
                _TYPEID_LSP7_TOKENSRECIPIENT,
                lsp1Data
            );

```
https://github.com/code-423n4/2023-06-lukso/blob/9dbc96410b3052fc0fd9d423249d1fa42958cae8/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L462

```solidity
FILE: 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol

464:      ILSP1UniversalReceiver(from).universalReceiver(
                _TYPEID_LSP8_TOKENSSENDER,
                lsp1Data
            );
        }

488: ILSP1UniversalReceiver(to).universalReceiver(
                _TYPEID_LSP8_TOKENSRECIPIENT,
                lsp1Data
            );


```
https://github.com/code-423n4/2023-06-lukso/blob/9dbc96410b3052fc0fd9d423249d1fa42958cae8/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L464

```solidity
FILE: Breadcrumbs2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions
/LSP8CompatibleERC721InitAbstract.sol

315: IERC721Receiver(to).onERC721Received(
                    msg.sender,
                    from,
                    tokenId,
                    data
                )

FILE:2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions
/LSP8CompatibleERC721.sol

315: IERC721Receiver(to).onERC721Received(
                    msg.sender,
                    from,
                    tokenId,
                    data
                )



```
##

## [L-9] internal functions only called once can be inlined to save gas

Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

```solidity
FILE: 2023-06-lukso/contracts/LSP0ERC725Account
/LSP0ERC725AccountCore.sol

796: function _fallbackLSP17Extendable() internal virtual override {
851: function _getExtension(
        bytes4 functionSelector
    ) internal view virtual override returns (address) {

FILE: 2023-06-lukso/contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP
/LSP1UniversalReceiverDelegateUP.sol


201: function _whenSending(
        bytes32 typeId,
        address notifier,
        bytes32 notifierMapKey,
        uint128 arrayIndex
    ) internal virtual returns (bytes memory) {

152: function _whenReceiving(
        bytes32 typeId,
        address notifier,
        bytes32 notifierMapKey,
        bytes4 interfaceID
    ) internal virtual returns (bytes memory) {

FILE: 2023-06-lukso/contracts/LSP6KeyManager/LSP6Modules
/LSP6SetDataModule.sol

622: function _verifyAllowedERC725YDataKeys(
        address controllerAddress,
        bytes32[] memory inputDataKeys,
        bytes memory allowedERC725YDataKeysCompacted,
        bool[] memory validatedInputKeysList,
        uint256 allowedDataKeysFound
    ) internal pure virtual {


507:    function _verifyAllowedERC725YSingleKey(
        address controllerAddress,
        bytes32 inputDataKey,
        bytes memory allowedERC725YDataKeysCompacted
    ) internal pure virtual {

490: function _getPermissionToSetLSP17Extension(
        address controlledContract,
        bytes32 lsp17ExtensionDataKey
    ) internal view virtual returns (bytes32) {

474: function _getPermissionToSetLSP1Delegate(
        address controlledContract,
        bytes32 lsp1DelegateDataKey
    ) internal view virtual returns (bytes32) {

438: function _getPermissionToSetAllowedERC725YDataKeys(
        address controlledContract,
        bytes32 dataKey,
        bytes memory dataValue,
        bool hasBothAddControllerAndEditPermissions
    ) internal view returns (bytes32) {

406: function _getPermissionToSetAllowedCalls(
        address controlledContract,
        bytes32 dataKey,
        bytes memory dataValue,
        bool hasBothAddControllerAndEditPermissions
    ) internal view virtual returns (bytes32) {

385:    function _getPermissionToSetControllerPermissions(
        address controlledContract,
        bytes32 inputPermissionDataKey
    ) internal view virtual returns (bytes32) {

334: function _getPermissionToSetPermissionsArray(
        address controlledContract,
        bytes32 inputDataKey,
        bytes memory inputDataValue,
        bool hasBothAddControllerAndEditPermissions
    ) internal view virtual returns (bytes32) {



```











