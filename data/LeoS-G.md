## Summary
|        | Issue | Instances | Gas Saved |
|--------|-------|-----------|-----------|
| [G-01] |  Enhance optimizer settings     |     -      |      109 676    |
| [G-02] |  Use `calldata` instead of `memory`     |     2      |      7382     |
| [G-03] |  Use  `do while`  loops instead of  `for`  loops     |     5      |      -     |
|[G-04]|Make variable outside `for`/`while` loop|3|-|
|[G-05]|Ternary over  `if ... else`|4|52|
|[G-06]|Use named return|2|10-26|


## [G-01] Enhance optimizer settings
The aim of this finding is to highlight the potential gains that can be achieved thanks to a good setting of the solidity optimizer. However, it includes an overlap with a known finding (*Reduce gas usage by moving to Solidity 0.8.19 or later*) in order to complete/correct it. To compare potential gains, the `forge test --gas-report` command is used with a fuzz seed of 0 and 10000 runs. The tests executed by this command are not exhaustive or fully representative of current contract usage. Nonetheless, it does highlight the potential gains offered by a suitable setup.

### Activating the `--via-ir`
This option allows YUL optimization. It is currently in heavy development and will allow the biggest gain. 
With this activated, this option produces those results:

| |`--via-ir`|
|-|-|
|Deployment Sum|-1357644|
|Deployment %|-11.928|
|Usage Sum|-63688|
|Usage %|-3.015|

Deployment sum should be understood as the difference between the sum of all deployment costs presented in the gas report before and after modification. Usage sum is the same thing, but for all the average usage costs. The % represents the percentage increase in relation to the initial value. 
As can be seen here, gains from activating this option are by no means negligible.

### Compiling with another version
To perform those tests, the `--via-ir` options is activated. The results are thus compared with the ones from the previous change. A recent version of the compiler achieves better results with the optimizer, most of the time. But it is not always the case, and as this mostly works like a black box, it is hard to predict it. Thus, versions must be tested. 
The results are presented below in the same format as before.

|Version |0.8.14|0.8.15|0.8.16|0.8.17|0.8.18|0.8.19|0.8.20|
|--|--|--|--|--|--|--|--|
|Deployment Sum|156|0|-24016|-38446|-241086|-241086|-241086|
|Deployment %|0.002|0.0|-0.24|-0.384|-2.405|-2.405|-2.405|
|Usage Sum|-485|0|-13232|-13312|-16189|-16189|-16189|
|Usage %|-0.024|0.0|-0.646|-0.65|-0.79|-0.79|-0.79|

The actual setup version (`0.8.15`) seems to be the worst choice. Barring any special security constraints, it would be worthwhile changing versions. `0.8.18` is suggested, as it is both better known and more efficient. Further increases are pointless (this is the discrepancy with the optimization presented in the automatic findings, which suggest `0.8.19` without any additional value).

### Change the number of runs
To perform those tests, the `--via-ir` options is activated and the version `0.8.18` is used. The results are thus compared with the ones from the previous change. The optimizer must be set to a number of runs (200 by default), meaning how many times the contracts are meant to be used. A huge number of runs improves the running cost, but penalizes the deployment cost, and inversely. 
The results are presented below in the same format as before.

| Runs|1|10|50|100|500|1000|10000|20000|50000|100000|500000|
|-|-|-|-|-|-|-|-|-|-|-|-|
|Deployment Sum|-68874|-68874|-75290|-35230|175410|1110118|3408928|3841472|3841472|3841472|3841472|
|Deployment %|-0.704|-0.704|-0.77|-0.36|1.793|11.347|34.845|39.266|39.266|39.266|39.266|
|Usage Sum|10747|10603|7012|4767|-16327|-18692|-29933|-31754|-33636|-33636|-33636|
|Usage %|0.529|0.522|0.345|0.235|-0.803|-0.92|-1.473|-1.562|-1.655|-1.655|-1.655|

It's important to note that the number of runs has no impact on compilation time. The final choice will therefore depend on how much the deployer is willing to pay for this part.
A good compromise seems to be 1000 runs, however, the best results are achieved at 10000.

### Summary
By comparing the default setup to the one with `via-ir`, version `0.8.18` and `10000` runs (of the optimizer). Those change in the usage gas of the gas report are observed.

```markdown
| tests/foundry/GasTests/LSP6s/LSP6ExecuteRC.sol:LSP6ExecuteRestrictedController contract   |            |           |            |
|-------------------------------------------------------------------------------------------|------------|-----------|------------|
| Function Name                                                                             | avg before | avg after | difference |
| lsp20VerifyCall                                                                           | 17035      | 15705     | -1330      |
| supportsInterface                                                                         | 560        | 344       | -216       |
| transferLYXToEOA                                                                          | 57163      | 53654     | -3509      |
| transferLYXToUP                                                                           | 31465      | 29381     | -2084      |
| transferNFTToRandomEOA                                                                    | 130864     | 121749    | -8715      |
| transferNFTToRandomUP                                                                     | 238712     | 224699    | -14013     |
| transferTokensToRandomEOA                                                                 | 77091      | 69408     | -7683      |
| transferTokensToRandomUP                                                                  | 211679     | 198115    | -13564     |
|                                                                                           |            |           |            |
|                                                                                           |            |           |            |
| tests/foundry/GasTests/LSP6s/LSP6ExecuteUC.sol:LSP6ExecuteUnrestrictedController contract |            |           |            |
| Function Name                                                                             | avg        | avg after | difference |
| lsp20VerifyCall                                                                           | 17035      | 15705     | -1330      |
| supportsInterface                                                                         | 560        | 344       | -216       |
| transferLYXToEOA                                                                          | 56764      | 54612     | -2152      |
| transferLYXToUP                                                                           | 33465      | 31381     | -2084      |
| transferNFTToRandomEOA                                                                    | 128700     | 120825    | -7875      |
| transferNFTToRandomUP                                                                     | 236549     | 223775    | -12774     |
| transferTokensToRandomEOA                                                                 | 74387      | 68253     | -6134      |
| transferTokensToRandomUP                                                                  | 208975     | 196960    | -12015     |
|                                                                                           |            |           |            |
|                                                                                           |            |           |            |
| tests/foundry/GasTests/LSP6s/LSP6SetDataRC.sol:LSP6SetDataRestrictedController contract   |            |           |            |
| Function Name                                                                             | avg        | avg after | difference |
| execute                                                                                   | 28453      | 25853     | -2600      |
| givePermissionsToController                                                               | 120527     | 118229    | -2298      |
| restrictControllerToERC725YKeys                                                           | 138829     | 137062    | -1767      |
| supportsInterface                                                                         | 537        | 344       | -193       |
|                                                                                           |            |           |            |
|                                                                                           |            |           |            |
| tests/foundry/GasTests/LSP6s/LSP6SetDataUC.sol:LSP6SetDataUnrestrictedController contract |            |           |            |
| Function Name                                                                             | avg        | avg after | difference |
| execute                                                                                   | 28453      | 25853     | -2600      |
| givePermissionsToController                                                               | 126527     | 124229    | -2598      |
| restrictControllerToERC725YKeys                                                           | 147329     | 145562    | -1767      |
| supportsInterface                                                                         | 537        | 344       | -189       |
```
The main point of this optimization is not to present the ideal setup, as this obviously varies according to future changes. But to show that setting up the optimizer should be the priority when it comes to saving gas, given how significant gains are.

### [G-02] Use `calldata` instead of `memory`
Using `calldata` instead of `memory` for function parameters can save gas if the argument is only read in the function

*2 instances*

-   [LSP1UniversalReceiverDelegateUP.sol#L83](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol#L83)
-   [LSP6Utils.sol#L153](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Utils.sol#L153)

*It may be possible that other instances are present, but only those whose impact is visible thanks to `forge snapshot --diff` are considered valid. There are also numerous instances that look similar but worsen the results and are therefore invalid. The result of `forge snapshot -diff` on the optimized code is shown below.*
```
testTransferNFTToRandomEOA() (gas: -620 (-0.026%))
testTransferTokensToRandomEOA() (gas: -567 (-0.026%))
testTransferNFTToRandomEOA() (gas: -620 (-0.028%))
testTransferTokensToRandomEOA() (gas: -567 (-0.028%))
testTransferNFTToRandomUP() (gas: -932 (-0.037%))
testTransferTokensToRandomUP() (gas: -879 (-0.038%))
testTransferNFTToRandomUP() (gas: -932 (-0.039%))
testTransferTokensToRandomUP() (gas: -879 (-0.041%))
testHasPermissionShouldReturnTrueToAllRegularPermission(uint256) (gas: -1386 (-9.064%))
Overall gas change: -7382 (-0.037%)
```

### [G-03] Use  `do while`  loops instead of  `for`  loops
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

### [G-04] Make variable outside `for`/`while` loop
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


With these changes, these evolutions in gas benchmark report can be observed (only average values):


### [G-05] Ternary over  `if ... else`
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

### [G-06] Use named return
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
