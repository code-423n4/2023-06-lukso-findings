## [G-01] Use assembly for address(0) comparison

### saves 65 gas each time

_5 instances - 2 file:_

### Instance#1 :

```solidity
File : contracts/LSP17ContractExtension/LSP17Extendable.sol

50:  if (erc165Extension == address(0)) return false;
```

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP17ContractExtension/LSP17Extendable.sol#L50

Recommended Changes:

```diff
+       bool isAddressZero;
+        assembly {
+            isAddressZero := eq(address(erc165Extension), 0)
+        }
-   if (erc165Extension == address(0)) return false;
+   if (isAddressZero) return false;
```

### Instance#2 :

```solidity
File : contracts/LSP17ContractExtension/LSP17Extendable.sol

91:  if (extension == address(0))
92:            revert NoExtensionFoundForFunctionSelector(msg.sig);

```

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP17ContractExtension/LSP17Extendable.sol#L91C9-L92C65

Recommendation : Use assembly for address(0) like above instance#1 recommendation

### Instance#3-5 :

```soldity
File: contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol
84:    if (tokenOwner == address(0)) {
       ...

115:  if (operator == address(0)) {
            revert LSP8CannotUseAddressZeroAsOperator();
        }
       ...
139:   if (operator == address(0)) {
            revert LSP8CannotUseAddressZeroAsOperator();
        }
```

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L84
https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L115
https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L139

Recommendation : Use assembly for address(0) like above instance#1 recommendation

## [G-02] Use nested if and, avoid multiple check combinations

Using nested if is cheaper gas wise than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

_1 instances - 1 files:_

### Instance#1:

```solidity
File: /contracts/LSP10ReceivedVaults/LSP10Utils.sol

76: if (encodedArrayLength.length != 0 && encodedArrayLength.length != 16) {
            revert InvalidLSP10ReceivedVaultsArrayLength({
                invalidValueStored: encodedArrayLength,
                invalidValueLength: encodedArrayLength.length
            });
```

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP10ReceivedVaults/LSP10Utils.sol#L76C9-L80C16

## [G-03] Redundant if check.

_1 instance - 1 file:_

### Instance#1 : `oldArrayLength` can't go above max value of uint128.

```solidity
File:
133: uint128 oldArrayLength = uint128(bytes16(lsp10VaultsCountValue));
          //@audit this check can be removed
135:       if (oldArrayLength > type(uint128).max) {
136:           revert VaultIndexSuperiorToUint128(oldArrayLength);
       }
```

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP10ReceivedVaults/LSP10Utils.sol#L133C9-L137C10
