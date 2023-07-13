## Summary
|        | Issue | Instances | Gas Saved |
|--------|-------|-----------|-----------|
| [G-01] |  Use  `do while`  loops instead of  `for`  loops     |     5      |      -     |
|[G-02]|Make variable outside `for`/`while` loop|3|-|
|[G-03]|Ternary over  `if ... else`|4|52|
|[G-04]|Use named return|2|10-26|


## [G-01] Use  `do while`  loops instead of  `for`  loops
Utilizing a do-while loop would result in a lower gas cost as the condition is not evaluated during the initial iteration. This is only valid when it is sure the first iteration validate the condition.

*5 instances*

-   [LSP6SetDataModule.sol#L726-L753](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L726-L753)
```diff
-         for (uint256 ii; ii < inputKeysLength; ) {
+         uint256 ii;
+         do{
              // if the input data key has been marked as allowed previously,
              // SKIP it and move to the next input data key.
              if (validatedInputKeysList[ii]) {
                  unchecked {
                      ++ii;
                  }
                  continue;
              }
  
              // CHECK if the input data key is allowed.
              if ((inputDataKeys[ii] & mask) == allowedKey) {
                  // if the input data key is allowed, mark it as allowed
                  // and increment the number of allowed keys found.
                  validatedInputKeysList[ii] = true;
  
                  unchecked {
                      allowedDataKeysFound++;
                  }
  
                  // Continue checking until all the inputKeys` have been found.
                  if (allowedDataKeysFound == inputKeysLength) return;
              }
  
              unchecked {
                  ++ii;
              }
-         }
+         }while (ii < inputKeysLength);
```
-   [LSP6SetDataModule.sol#L762-L773](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L762-L773)
```diff
-     for (uint256 jj; jj < inputKeysLength; ) {
+     uint256 jj;
+     do{
          if (!validatedInputKeysList[jj]) {
              revert NotAllowedERC725YDataKey(
                  controllerAddress,
                  inputDataKeys[jj]
              );
          }
  
          unchecked {
              jj++;
          }
-     }
+     }while(jj < inputKeysLength);
```
-   [LSP6KeyManagerCore.sol#L163-L173](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L163-L173)
```diff
-        for (uint256 ii = 0; ii < payloads.length; ) {
+        uint256 ii = 0;    
+        do{
            if ((totalValues += values[ii]) > msg.value) {
                revert LSP6BatchInsufficientValueSent(totalValues, msg.value);
            }

            results[ii] = _execute(values[ii], payloads[ii]);

            unchecked {
                ++ii;
             }
-         }
+         }while(ii < payloads.length);
```
-   [LSP6KeyManagerCore.sol#L223-L236](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L223-L239)
```diff
-        for (uint256 ii = 0; ii < payloads.length; ) {
+        uint256 ii = 0;
+        do{
            if ((totalValues += values[ii]) > msg.value) {
                revert LSP6BatchInsufficientValueSent(totalValues, msg.value);
            }

            results[ii] = _executeRelayCall(
                signatures[ii],
                nonces[ii],
                validityTimestamps[ii],
                values[ii],
                payloads[ii]
            );

            unchecked {
                ++ii;
             }
-         }
+         }while(ii < payloads.length);
```
-   [LSP7DigitalAssetCore.sol#L171-L184](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L171-L184)
```diff
-        for (uint256 i = 0; i < fromLength; ) {
+        uint256 i = 0;
+        do{
            // using the public transfer function to handle updates to operator authorized amounts
            transfer(
                from[i],
                to[i],
                amount[i],
                allowNonLSP1Recipient[i],
                data[i]
            );

            unchecked {
                ++i;
            }
-        }
+        }while(i < fromLength);
```

### [G-02] Make variable outside `for`/`while` loop
In some specific case, it can save gases.

*3 instances*
- [LSP0ERC725AccountCore.sol#L175-L200)](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L175-L200)
```diff
+       bool success;
+       bytes  memory result;
        for (uint256 i; i < data.length; ) {
-            (bool success, bytes memory result) = address(this).delegatecall(
+            (success, result)  =  address(this).delegatecall(
                data[i]
            );

            if (!success) {
                // Look for revert reason and bubble it up if present
                if (result.length > 0) {
                    // The easiest way to bubble the revert reason is using memory via assembly
                    // solhint-disable no-inline-assembly
                    /// @solidity memory-safe-assembly
                    assembly {
                        let returndata_size := mload(result)
                        revert(add(32, result), returndata_size)
                    }
                } else {
                    revert("LSP0: batchCalls reverted");
                }
            }

            results[i] = result;

            unchecked {
                ++i;
            }
        }
```
-[LSP6ExecuteModule.sol#L272-L310](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L272-L310)
```diff
+       bytes memory allowedCall
        for (uint256 ii = 0; ii < allowedCalls.length; ii += 34) {
            /// @dev structure of an AllowedCall
            //
            /// AllowedCall = 0x00200000000ncafecafecafecafecafecafecafecafecafecafe5a5a5a5af1f1f1f1
            ///
            ///                                     0020 = hex for '32' bytes long
            ///                                 0000000n = call type(s)
            /// cafecafecafecafecafecafecafecafecafecafe = address
            ///                                 5a5a5a5a = standard
            ///                                 f1f1f1f1 = function

            // CHECK that we can extract an AllowedCall
            if (ii + 34 > allowedCalls.length) {
                revert InvalidEncodedAllowedCalls(allowedCalls);
            }

            // extract one AllowedCall at a time
-            bytes memory allowedCall = BytesLib.slice(allowedCalls, ii + 2, 32);
+            allowedCall = BytesLib.slice(allowedCalls, ii + 2, 32);

            // 0xxxxxxxxxffffffffffffffffffffffffffffffffffffffffffffffffffffffff
            // (excluding the callTypes) not allowed
            // as equivalent to whitelisting any call (= SUPER permission)
            if (
                bytes28(bytes32(allowedCall) << 32) ==
                bytes28(type(uint224).max)
            ) {
                revert InvalidWhitelistedCall(controllerAddress);
            }

            if (
                _isAllowedCallType(allowedCall, requiredCallTypes) &&
                _isAllowedAddress(allowedCall, to) &&
                _isAllowedStandard(allowedCall, to) &&
                _isAllowedFunction(allowedCall, selector)
            ) return;
        }

        revert NotAllowedCall(controllerAddress, to, selector);
    }
```

- [LSP8IdentifiableDigitalAssetCore.sol#L273-L281](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L273-L281)
```diff
+       address operator;
        for (uint256 i = 0; i < operatorListLength; ) {
            // we are emptying the list, always remove from index 0
-          address operator = operatorsForTokenId.at(0);
+          operator = operatorsForTokenId.at(0);
            _revokeOperator(operator, tokenOwner, tokenId);

            unchecked {
                ++i;
            }
        }
```

### [G-03] Ternary over  `if ... else`
Replacing an `if-else` statement with the ternary operator can save gas.

*4 instances*
*Save up to **13 gas per call***
-   [LSP7DigitalAssetCore.sol#L112-L116](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L112-L116)
```diff
- if (tokenOwner == operator) {
-     return _tokenOwnerBalances[tokenOwner];
- } else {
-     return _operatorAuthorizedAmount[tokenOwner][operator];
- }

+ return tokenOwner == operator ?
+     _tokenOwnerBalances[tokenOwner] :
+     _operatorAuthorizedAmount[tokenOwner][operator];

```
-   [LSP7DigitalAssetCore.sol#L278-L282](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L278-L282)
```diff
- if (amount != 0) {
-     emit AuthorizedOperator(operator, tokenOwner, amount);
- } else {
-     emit RevokedOperator(operator, tokenOwner);
- }

+ emit amount != 0 ?
+     AuthorizedOperator(operator, tokenOwner, amount) :
+     RevokedOperator(operator, tokenOwner);
```

-   [LSP8CompatibleERC721InitAbstract.sol#L129-L139](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol#L129-L139)
```diff
- if (operatorListLength == 0) {
-     return address(0);
- } else {
-     return operatorsForTokenId[operatorListLength - 1];
- }

+ return operatorListLength == 0 ?
+     address(0) :
+     operatorsForTokenId[operatorListLength - 1];
```

- [LSP8CompatibleERC721.sol#L129-L139](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol#L129-L139)
```diff
- if (operatorListLength == 0) {
-     return address(0);
- } else {
-     return operatorsForTokenId[operatorListLength - 1];
- }

+ return operatorListLength == 0 ?
+     address(0) :
+     operatorsForTokenId[operatorListLength - 1];
```

### [G-04] Use named return
Using the named return whenever possible saves gas by avoiding a return and the double declaration of a variable.

*2 instances*
*Save up to **5-13 gas per call*** (depending on the complexity of the function)

- [LSP0ERC725AccountCore.sol#L851-L866](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L851-L866)

```diff
    function _getExtension(
        bytes4 functionSelector
-    ) internal view virtual override returns (address) {
+    ) internal view virtual override returns (address extension) {
        // Generate the data key relevant for the functionSelector being called
        bytes32 mappedExtensionDataKey = LSP2Utils.generateMappingKey(
            _LSP17_EXTENSION_PREFIX,
            functionSelector
        );

        // Check if there is an extension stored under the generated data key
-        address extension = address(
+        extension = address(
            bytes20(ERC725YCore._getData(mappedExtensionDataKey))
        );

-        return extension;
    }
```

- [LSP8IdentifiableDigitalAssetCore.sol#L79-L89](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L79-L89)
```diff
    function tokenOwnerOf(
        bytes32 tokenId
-   ) public view virtual returns (address) {
+   ) public view virtual returns (address tokenOwner) {
-       address tokenOwner = _tokenOwners[tokenId];
+       tokenOwner = _tokenOwners[tokenId];

        if (tokenOwner == address(0)) {
            revert LSP8NonExistentTokenId(tokenId);
        }

-        return tokenOwner;
    }
```