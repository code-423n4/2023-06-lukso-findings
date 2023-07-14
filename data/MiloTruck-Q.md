
## Findings Summary

| Label | Description |
| - | - |
| [L-01](#l-01-misleading-comment-in-lsp7digitalassetcoresol) | Misleading comment in `LSP7DigitalAssetCore.sol` |
| [L-02](#l-02-_fallbacklsp17extendable-doesnt-forward-msgvalue) | `_fallbackLSP17Extendable()` doesn't forward `msg.value` |
| [L-03](#l-03-consider-validating-_beforetokentransfer-in-lsp8identifiabledigitalassetcoresol) | Consider validating `_beforeTokenTransfer()` in `LSP8IdentifiableDigitalAssetCore.sol` |
| [L-04](#l-04-incorrect-natspec-dev-in-lsp6utilssol) | Incorrect NatSpec `@dev` in `LSP6Utils.sol` |
| [L-05](#l-05-isencodedarray-could-revert-or-return-true-incorrectly) | `isEncodedArray()` could revert or return `true` incorrectly |
| [L-06](#l-06-extensions-should-ensure-that-msgvalue-is-0-to-prevent-user-mistakes) | Extensions should ensure that `msg.value` is 0 to prevent user mistakes |
| [L-07](#l-07-data-passed-to-lsp20verifycallresult-is-different-in-execute-and-executebatch) | Data passed to `lsp20VerifyCallResult()` is different in `execute()` and `executeBatch()` |
| [L-08](#l-08-extensions-with-a-0x00000000-function-selector-could-create-phantom-functions) | Extensions with a `0x00000000` function selector could create phantom functions |
| [L-09](#l-09-lsp-17-extensions-in-lsp0-accounts-are-vulnerable-to-metamoprhic-contracts) | LSP-17 extensions in LSP0 accounts are vulnerable to metamoprhic contracts |
| [L-10](#l-10-allowed-calls-in-lsp6executemodule-isnt-compatible-with-some-function-selectors) | Allowed calls in `LSP6ExecuteModule` isn't compatible with some function selectors |
| [L-11](#l-11-combinepermissions-handles-duplicate-permissions-incorrectly) | `combinePermissions()` handles duplicate permissions incorrectly |
| [L-12](#l-12-lsp7digitalassetcores-_burn-function-shouldnt-have-an-allowance-check) | `LSP7DigitalAssetCore`'s `_burn()` function shouldn't have an allowance check |
| [L-13](#l-13-_getpermissiontosetcontrollerpermissions-might-return-incorrect-permissions-for-malformed-data) | `_getPermissionToSetControllerPermissions()` might return incorrect permissions for malformed data |
| [N-01](#n-01-import-statement-in-lsp0erc725accountcoresol-can-be-more-succinct) | Import statement in `LSP0ERC725AccountCore.sol` can be more succinct |
| [N-02](#n-02-typos) | Typos |
| [N-03](#n-03-unreachable-if-statement-in-generatesentvaultkeys-can-be-removed) | Unreachable if-statement in `generateSentVaultKeys()` can be removed |
| [N-04](#n-04-code-in-_whenreceiving-and-_whensending-can-be-shorter) | Code in `_whenReceiving()` and `_whenSending()` can be shorter |
| [N-05](#n-05-unnecessary-use-of-ternary-operator) | Unnecessary use of ternary operator |
| [N-06](#n-06-code-in-lsp20verifycall-can-be-more-succinct) | Code in `lsp20VerifyCall()` can be more succinct |
| [N-07](#n-07-use-typeuint128max-instead-of-uint1280) | Use `type(uint128).max` instead of `~uint128(0)` |
| [N-08](#n-08-_setuplsp6reentrancyguard-is-redundant) | `_setupLSP6ReentrancyGuard()` is redundant |
| [N-09](#n-09-missing-check-in-_deploycreate2) | Missing check in `_deployCreate2()` |
| [N-10](#n-10-document-_interfaceid_lsp20_call_verification-in-the-lsp-20-specification) | Document `_INTERFACEID_LSP20_CALL_VERIFICATION` in the LSP-20 specification |
| [N-11](#n-11-lsp8compatibleerc721sol-doesnt-have-a-_safemint-function) | `LSP8CompatibleERC721.sol` doesn't have a `_safeMint()` function |

## [L-01] Misleading comment in `LSP7DigitalAssetCore.sol`

The following comment states that [ERC20's approval race condition issue](https://docs.google.com/document/d/1YLPtQxZu1UAvO9cZ1O2RPXBbT0mooh4DYKjA_jp-RLM/) can be avoided by calling LSP7's `revokeOperator()` before `authorizeOperator()` to update a spender's allowance:

[LSP7DigitalAssetCore.sol#L78-L90](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L78-L90)

```solidity
    /**
     * @inheritdoc ILSP7DigitalAsset
     *
     * @dev To avoid front-running and Allowance Double-Spend Exploit when
     * increasing or decreasing the authorized amount of an operator,
     * it is advised to:
     *     1. call {revokeOperator} first, and
     *     2. then re-call {authorizeOperator} with the new amount
     *
     * for more information, see:
     * https://docs.google.com/document/d/1YLPtQxZu1UAvO9cZ1O2RPXBbT0mooh4DYKjA_jp-RLM/
     *
     */
```

This is incorrect as `revokeOperator()` simply sets the spender's allowance back to 0:

[LSP7DigitalAssetCore.sol#L101-L103](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L101-L103)

```solidity
    function revokeOperator(address operator) public virtual {
        _updateOperator(msg.sender, operator, 0);
    }
```

Therefore, someone could still spend double of his allowance by front-running the call to `revokeOperator()`.

### Recommendation

Change the comment to recommend using [`decreaseAllowance()`](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L232-L248) instead.

## [L-02] `_fallbackLSP17Extendable()` doesn't forward `msg.value`

In [`LSP17Extendable.sol`](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP17ContractExtension/LSP17Extendable.sol), `_fallbackLSP17Extendable()` is used in `fallback` functions to forward calls to their respective extensions, similar to how proxies work. However, it uses a low-level call instead of `delegatecall`:

[LSP17Extendable.sol#L107-L116](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP17ContractExtension/LSP17Extendable.sol#L107-L116)

```solidity
            // Add 52 bytes for the msg.sender and msg.value appended at the end of the calldata
            let success := call(
                gas(),
                extension,
                0, // value is set to 0
                0,
                add(calldatasize(), 52),
                0,
                0
            )
```

As seen from above, none of `msg.value` will be fowarded to the extension contract in the low-level call. This might cause several problems:

1. The functionality of LSP-17 is limited; developers might want to receive native tokens in their extension contracts but will be unable to do so.

1. This behaviour isn't mentioned anywhere in the [documentation](https://docs.lukso.tech/standards/generic-standards/lsp17-contract-extension) or [LSP specification](https://github.com/lukso-network/LIPs/blob/main/LSPs/LSP-17-ContractExtension.md). Developers might expect LSP-17 to behave similarly to proxies and write extensions that require native tokens, causing their code to be incorrect.

### Recommendation

Consider mentioning that no native tokens will be forwarded to the extension using the LSP-17 standard. 

## [L-03] Consider validating `_beforeTokenTransfer()` in `LSP8IdentifiableDigitalAssetCore.sol`

In `LSP8IdentifiableDigitalAssetCore.sol`, consider adding the following checks after various `_beforeTokenTransfer` hooks:

* `_mint()` - Check that `tokenId` was not minted in `_beforeTokenTransfer`:

[LSP8IdentifiableDigitalAssetCore.sol#L329](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L329)

```diff
        _beforeTokenTransfer(address(0), to, tokenId);

+       // Check that `tokenId` was not minted by `_beforeTokenTransfer` hook
+       if (_exists(tokenId)) {
+           revert LSP8TokenIdAlreadyMinted(tokenId);
+       }
```

* `_burn()` - Update `tokenOwner` if `tokenId` was transferred in `_beforeTokenTransfer`:

[LSP8IdentifiableDigitalAssetCore.sol#L363](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L363)

```diff
        _beforeTokenTransfer(tokenOwner, address(0), tokenId);

+       // Update `tokenOwner` in case `tokenId` was transferred by `_beforeTokenTransfer` hook
+       tokenOwner = tokenOwnerOf(tokenId);
```

* `_transfer()` - Check that `tokenId` was not transferred to a new owner in `_beforeTokenTransfer`:

[LSP8IdentifiableDigitalAssetCore.sol#L416](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L416)

```diff
        _beforeTokenTransfer(from, to, tokenId);

+       // Check that `tokenId` was not transferred by `_beforeTokenTransfer` hook
+       if (tokenOwner != tokenOwnerOf(tokenId)) {
+           revert ERC721IncorrectOwner(from, tokenId, owner);
+       }
```

If a contract inheriting `LSP8IdentifiableDigitalAssetCore` has a bug in its `_beforeTokenTransfer` hook (eg. reentrancy), these checks will prevent tokens from being duplicated.

## [L-04] Incorrect NatSpec `@dev` in `LSP6Utils.sol`

The following comment states each element in the array passed to `isCompactBytesArrayOfAllowedCalls()` must be 28 bytes long:

[LSP6Utils.sol#L85-L86](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Utils.sol#L85-L86)

```solidity
    /*
     * @dev same as LSP2Utils.isCompactBytesArray with the additional requirement that each element must be 28 bytes long.
```

However, `isCompactBytesArrayOfAllowedCalls()` actually validates that each element is 32 bytes long:

[LSP6Utils.sol#L106-L107](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Utils.sol#L106-L107)

```solidity
            // each entries in the allowedCalls (compact) array must be 32 bytes long
            if (elementLength != 32) return false;
```

This might mislead developers who only read the NatSpec, causing them to utilize the `isCompactBytesArrayOfAllowedCalls()` function incorrectly.

## [L-05] `isEncodedArray()` could revert or return `true` incorrectly

In `LSP2Utils.sol`, the [`isEncodedArray()`](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol#L227-L245) function is used to check if `data` is an encoded array. However, its implementations has several issues:
1. If `offset` or `arrayLength` is too large, the function might revert due to an arithmetic overflow in the following lines:

[LSP2Utils.sol#L236](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol#L236)

```solidity
        if (nbOfBytes < offset + 32) return false;
```

[LSP2Utils.sol#L242](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol#L242)

```solidity
        if (nbOfBytes < (offset + 32 + (arrayLength * 32))) return false;
```

2. As there is no `offset >= 32` check, the function considers `bytes32(0)` is a valid encoded array.

3. The function only ensures that `data.length` is not smaller than the length calculated with `offset` and `arrayLength`. Therefore, even if `data` contains more bytes than the correct length, `isEncodedArray()` will still return `true`.

Should a developer inherit the `LSP2Utils` contract and use `isEncodedArray()` to validate user input, an attacker could intentionally craft malformed data to bypass `isEncodedArray()` or cause it to revert.

## [L-06] Extensions should ensure that `msg.value` is 0 to prevent user mistakes

The fallback function of the `LSP0ERC725AccountCore` contract is declared as `payable`: 

[LSP0ERC725AccountCore.sol#L151-L161](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L151-L161)

```solidity
    fallback() external payable virtual {
        if (msg.value != 0) {
            emit ValueReceived(msg.sender, msg.value);
        }

        if (msg.data.length < 4) {
            return;
        }

        _fallbackLSP17Extendable();
    }
```

This is meant to support LSP17 extensions that require transfers of LYX. However, this allows users to accidentally transfer LYX to the fallback function while calling an extension that doesn't use `msg.value`.  

### Recommendation

In the documentation, state that developers should ensure `msg.value == 0` if their extensions do not use `msg.value`. 

## [L-07] Data passed to `lsp20VerifyCallResult()` is different in `execute()` and `executeBatch()`

In `execute()`, the result from `ERC725XCore._execute()` is abi-encoded as `bytes` and passed to `_verifyCallResult()`, which calls the owner's `lsp20VerifyCallResult()` function:

[LSP0ERC725AccountCore.sol#L243-L254](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L243-L254)

```solidity
        // Perform the execution
        bytes memory result = ERC725XCore._execute(
            operationType,
            target,
            value,
            data
        );

        // if verifyAfter is true, Call {lsp20VerifyCallResult} on the owner
        if (verifyAfter) {
            LSP20CallVerification._verifyCallResult(_owner, abi.encode(result));
        }
```

However, in `executeBatch()`, the data is abi-encoded as a `bytes` array instead:

[LSP0ERC725AccountCore.sol#L307-L321](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L307-L321)

```solidity
        // Perform the execution
        bytes[] memory results = ERC725XCore._executeBatch(
            operationsType,
            targets,
            values,
            datas
        );

        // if verifyAfter is true, Call {lsp20VerifyCallResult} on the owner
        if (verifyAfter) {
            LSP20CallVerification._verifyCallResult(
                _owner,
                abi.encode(results)
            );
        }
```

This creates two issues:
* The owner's `lsp20VerifyCallResult()` has no way to differentiate between a `bytes` result from `execute()` and a `bytes` array from `executeBatch()`.
* When calling `execute()`, a malicious attacker could potentially manipulate `ERC725XCore._execute()` into returning `bytes` that resembles a `bytes` array in data, which would trick `lsp20VerifyCallResult()` into thinking the result came from `executeBatch()`.

### Recommendation

Consider calling `_verifyCallResult()` in a loop with each result in the `results` array individually:

[LSP0ERC725AccountCore.sol#L316-L321](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L315-L321)

```diff
        // if verifyAfter is true, Call {lsp20VerifyCallResult} on the owner
        if (verifyAfter) {
-           LSP20CallVerification._verifyCallResult(
-               _owner,
-               abi.encode(results)
-           );
+           for (uint256 i; i < results.length; ++i) {
+               LSP20CallVerification._verifyCallResult(
+                   _owner,
+                   abi.encode(results[i])
+               );
+           }
        }
```

## [L-08] Extensions with a `0x00000000` function selector could create phantom functions

In the `LSP0ERC725AccountCore` contract, calls to the fallback function should revert if no matching function selector is found. However, the `0x00000000` function selector simply returns instead:

[LSP0ERC725AccountCore.sol#L800-L801](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L800-L801)

```solidity
        // if no extension was found for bytes4(0) return don't revert
        if (msg.sig == bytes4(0) && extension == address(0)) return;
```

This could create [phantom functions](https://media.dedaub.com/phantom-functions-and-the-billion-dollar-no-op-c56f062ae49f) for functions in extensions with the `0x00000000` selector. 

For example, a contract might perform some kind of validation in an extension, such as checking for LSP6 permissions, and expect the function to revert if the user is not authorized. However, if the function's selector is `0x00000000` and the LSP0 account doesn't have this extension, the contract's call to the function will return instead of reverting, giving the contract the impression that the user is authorized.

### Recommendation

Consider treating reverting for the `0x00000000` function selector as well when no extension matches. Otherwise, warn developers that they should not rely on extensions to revert, but check its return value instead. 

## [L-09] LSP-17 extensions in LSP0 accounts are vulnerable to metamoprhic contracts

To add an extension to a LSP0 account, the owner has to add its address to the list of extensions, similar to a whitelist. However, this pattern is vulnerable to metamorphic contracts. For example:
* Attacker deploys an extension contract with `CREATE2` to a predetermined address.
  * This extension contract is not able to to anything malicious, but has the ability to `selfdestruct` itself.
* As the extension looks harmless, the owner of an LSP0 account adds it.
* The attacker selfdestructs the contract and deploys a new one with different runtime bytecode using `CREATE2`.
  * As long as the same `salt` and initialization code was provided to `CREATE2`, this new contract will have the same address as the destructed extension contract.
* The attacker can now use this contract to perform malicious actions on the LSP0 account.

This attack was famously used in the [Tornado Cash Governance Hack](https://twitter.com/samczsun/status/1660012956632104960).

### Recommendation

Warn users about the risk of adding extensions with the ability to `selfdestruct`.

## [L-10] Allowed calls in `LSP6ExecuteModule` isn't compatible with some function selectors

Whenever a controller attempts to call a LSP0 account's `execute()` function without the relevant [SUPER permissions](https://docs.lukso.tech/standards/universal-profile/lsp6-key-manager/#super-permissions), `LSP6ExecuteModule` will check that the call is one of the whitelisted [allowed calls](https://docs.lukso.tech/standards/universal-profile/lsp6-key-manager/#allowed-calls).

However, in `_isAllowedFunction()`, the function selectors `0x00000000` and `0xffffffff` have special meanings:

[LSP6ExecuteModule.sol#L410-L415](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L410-L415)

```solidity
        bool isFunctionCall = requiredFunction != bytes4(0);

        // ANY function = 0xffffffff
        return
            allowedFunction == bytes4(type(uint32).max) ||
            (isFunctionCall && (requiredFunction == allowedFunction));
```

`0x00000000` represents a call with empty calldata, while `0xffffffff` means that all function selectors are permitted. This causes `_isAllowedFunction()` unable to handle functions that have either of these selectors.

Therefore, functions with either of these selectors cannot be called by users with only `SETDATA` permissions.

## [L-11] `combinePermissions()` handles duplicate permissions incorrectly

In `LSP6Utils.sol`, `combinePermissions()` is a helper function for combining multiple permissions into a single `bytes32`:

[LSP6Utils.sol#L169-L177](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Utils.sol#L169-L177)

```solidity
    function combinePermissions(
        bytes32[] memory permissions
    ) internal pure returns (bytes32) {
        uint256 result = 0;
        for (uint256 i = 0; i < permissions.length; i++) {
            result += uint256(permissions[i]);
        }
        return bytes32(result);
    }
```

However, as it uses addition to combine the permissions, the result will be incorrect when the `permissions` array contains two or more of the same permissions. 

For example, if `combinePermissions()` is called with an array containing two `CHANGEOWNER` (`0x1`) permissions, it will return `0x2``, which is the `ADDCONTROLLER` permission. This could potentially be dangerous as contracts could end up assigning users with wrong permissions when using `combinePermissions()`.

### Recommendation

Consider using bitwise OR to combine permissions instead:

[LSP6Utils.sol#L169-L177](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Utils.sol#L169-L177)

```diff
    function combinePermissions(
        bytes32[] memory permissions
    ) internal pure returns (bytes32) {
        uint256 result = 0;
        for (uint256 i = 0; i < permissions.length; i++) {
-           result += uint256(permissions[i]);
+           result |= uint256(permissions[i]);
        }
        return bytes32(result);
    }
```

## [L-12] `LSP7DigitalAssetCore`'s `_burn()` function shouldn't have an allowance check 

In `LSP7DigitalAssetCore`, the `_burn()` function checks that the caller has a sufficient allowance from the `from` address:

[LSP7DigitalAssetCore.sol#L352-L366](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L352-L366)

```solidity
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
            _operatorAuthorizedAmount[from][operator] -= amount;
        }
```

However, this check could limit the functionality of contracts that inherit from `LSP7DigitalAsset`. Some protocols might want to allow other addresses, such a contract trusted by the protocol, to burn the LSP7 tokens of the user. 

Furthermore, this is inconsistent with LSP8's `_burn()` function, which does not contain any checks for the allowance of `msg.sender`.

### Recommendation

Consider removing the allowance check in `_burn()` and adding it to the [`LSP8Burnable`](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8Burnable.sol) extension instead.

## [L-13] `_getPermissionToSetControllerPermissions()` might return incorrect permissions for malformed data

The `_getPermissionToSetControllerPermissions()` function is used to check if the `ADDCONTROLLER` or `EDITPERMISSIONS` permission is required when adding a controller using `setData()` or `setDataBatch()`:

[LSP6SetDataModule.sol#L385-L397](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L385-L397)

```solidity
    function _getPermissionToSetControllerPermissions(
        address controlledContract,
        bytes32 inputPermissionDataKey
    ) internal view virtual returns (bytes32) {
        return
            // if there is nothing stored under the data key, we are trying to ADD a new controller.
            // if there are already some permissions set under the data key, we are trying to CHANGE the permissions of a controller.
            bytes32(
                ERC725Y(controlledContract).getData(inputPermissionDataKey)
            ) == bytes32(0)
                ? _PERMISSION_ADDCONTROLLER
                : _PERMISSION_EDITPERMISSIONS;
    }
```

However, instead of checking if the data's length is 0, it checks if the data under the `inputPermissionDataKey` key is empty by comparing it to `bytes32(0)`. This will return true as long as the first 32 bytes are `0x00`, regardless of whether there is data after the first 32 bytes. 

This could potentially cause problems if a contract sets permissions to `bytes32(0)` temporarily to ensure that users with `EDITPERMISSIONS` are still able to edit them.

### Recommendation

Check if the data is empty using by comparing its length to 0 instead:

[LSP6SetDataModule.sol#L385-L397](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L385-L397)

```diff
    function _getPermissionToSetControllerPermissions(
        address controlledContract,
        bytes32 inputPermissionDataKey
    ) internal view virtual returns (bytes32) {
        return
            // if there is nothing stored under the data key, we are trying to ADD a new controller.
            // if there are already some permissions set under the data key, we are trying to CHANGE the permissions of a controller.
-           bytes32(
-               ERC725Y(controlledContract).getData(inputPermissionDataKey)
-           ) == bytes32(0)
+           ERC725Y(controlledContract).getData(inputPermissionDataKey).length == 0
                ? _PERMISSION_ADDCONTROLLER
                : _PERMISSION_EDITPERMISSIONS;
    }
```

## [N-01] Import statement in `LSP0ERC725AccountCore.sol` can be more succinct

Consider modifying the following import statement as such:

[LSP0ERC725AccountCore.sol#L35-L43](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L35-L43)

```diff
    import {
        _INTERFACEID_LSP0,
        _INTERFACEID_ERC1271,
        _ERC1271_MAGICVALUE,
        _ERC1271_FAILVALUE,
        _TYPEID_LSP0_OwnershipTransferStarted,
        _TYPEID_LSP0_OwnershipTransferred_SenderNotification,
        _TYPEID_LSP0_OwnershipTransferred_RecipientNotification
-   } from "../LSP0ERC725Account/LSP0Constants.sol";
+   } from "./LSP0Constants.sol";
```

## [N-02] Typos

"loose" should be "lose":

[LSP0ERC725AccountCore.sol#L616](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L616)

```solidity
     * - the current {`owner()`} will loose access to the functions restricted to the {`owner()`} only.
```

Missing ")" in the comments below:

[LSP5Utils.sol#L136](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP5ReceivedAssets/LSP5Utils.sol#L136)

```solidity
        // Updating the number of the received assets (decrementing by 1
```

[LSP10Utils.sol#L139](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP10ReceivedVaults/LSP10Utils.sol#L139)

```solidity
        // Updating the number of the received vaults (decrementing by 1
```

"substractedAmount" should be "subtractedAmount":

[LSP7DigitalAssetCore.sol#L234](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L234)

```solidity
        uint256 substractedAmount
```

This occurs on [LSP7DigitalAssetCore.sol#L237](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L237) and [LSP7DigitalAssetCore.sol#L245](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L245) as well.

## [N-03] Unreachable if-statement in `generateSentVaultKeys()` can be removed

The if-statement below is unreachable as `oldArrayLength` is a `uint128` and will never be larger than `type(uint128).max`:

[LSP10Utils.sol#L132-L137](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP10ReceivedVaults/LSP10Utils.sol#L132-L137)

```solidity
        // Updating the number of the received vaults
        uint128 oldArrayLength = uint128(bytes16(lsp10VaultsCountValue));

        if (oldArrayLength > type(uint128).max) {
            revert VaultIndexSuperiorToUint128(oldArrayLength);
        }
```

## [N-04] Code in `_whenReceiving()` and `_whenSending()` can be shorter

The [`_whenReceiving()`](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol#L152-L194) function is implemented as such:

```solidity
    function _whenReceiving(
        ...
    ) internal virtual returns (bytes memory) {
        bytes32[] memory dataKeys;
        bytes[] memory dataValues;

        // if it's a token transfer (LSP7/LSP8)
        if (typeId != _TYPEID_LSP9_OwnershipTransferred_RecipientNotification) {
            // Some code here...
            return "";
        } else {
            // Some code here...
            return "";
        }
    }
```

The code can be shortened by using one return statement at the end of the function:

```diff
    function _whenReceiving(
        ...
    ) internal virtual returns (bytes memory) {
        bytes32[] memory dataKeys;
        bytes[] memory dataValues;

        // if it's a token transfer (LSP7/LSP8)
        if (typeId != _TYPEID_LSP9_OwnershipTransferred_RecipientNotification) {
            // Some code here...
-           return "";
        } else {
            // Some code here...
-           return "";
        }
+       return "";
    }
```

This also applies to [`_whenSending()`](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol#L201-L252), which follows the same pattern.

## [N-05] Unnecessary use of ternary operator

In `_verifyCall()`, return `bytes1(magicValue[3]) == 0x01` instead of using a ternary operator with `true` and `false`:

[LSP20CallVerification/LSP20CallVerification.sol#L42](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP20CallVerification/LSP20CallVerification.sol#L42)

```diff
-       return bytes1(magicValue[3]) == 0x01 ? true : false;
+       return bytes1(magicValue[3]) == 0x01;
```

In `getTransferDetails()`, set `isReceiving` to `typeId == CONSTANT` directly:

[LSP1Utils.sol#L84-L107](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP1UniversalReceiver/LSP1Utils.sol#L84-L107)

```diff
        if (
            typeId == _TYPEID_LSP7_TOKENSSENDER ||
            typeId == _TYPEID_LSP7_TOKENSRECIPIENT
        ) {
            mapPrefix = _LSP5_RECEIVED_ASSETS_MAP_KEY_PREFIX;
            interfaceId = _INTERFACEID_LSP7;
-           isReceiving = typeId == _TYPEID_LSP7_TOKENSRECIPIENT ? true : false;
+           isReceiving = typeId == _TYPEID_LSP7_TOKENSRECIPIENT;
        } else if (
            typeId == _TYPEID_LSP8_TOKENSSENDER ||
            typeId == _TYPEID_LSP8_TOKENSRECIPIENT
        ) {
            mapPrefix = _LSP5_RECEIVED_ASSETS_MAP_KEY_PREFIX;
            interfaceId = _INTERFACEID_LSP8;
-           isReceiving = typeId == _TYPEID_LSP8_TOKENSRECIPIENT ? true : false;
+           isReceiving = typeId == _TYPEID_LSP8_TOKENSRECIPIENT;
        } else if (
            typeId == _TYPEID_LSP9_OwnershipTransferred_SenderNotification ||
            typeId == _TYPEID_LSP9_OwnershipTransferred_RecipientNotification
        ) {
            mapPrefix = _LSP10_VAULTS_MAP_KEY_PREFIX;
            interfaceId = _INTERFACEID_LSP9;
-           isReceiving = (typeId ==
-               _TYPEID_LSP9_OwnershipTransferred_RecipientNotification)
-               ? true
-               : false;
+           isReceiving = typeId == _TYPEID_LSP9_OwnershipTransferred_RecipientNotification;
```

## [N-06] Code in `lsp20VerifyCall()` can be more succinct

The following code:

[LSP6KeyManagerCore.sol#L256-L262](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L256-L262)

```solidity
        bool isSetData = false;
        if (
            bytes4(data) == IERC725Y.setData.selector ||
            bytes4(data) == IERC725Y.setDataBatch.selector
        ) {
            isSetData = true;
        }
```

can be rewritten as a single expression:

```solidity
bool isSetData = bytes4(data) == IERC725Y.setData.selector || bytes4(data) == IERC725Y.setDataBatch.selector;
```

## [N-07] Use `type(uint128).max` instead of `~uint128(0)`

Consider changing the code below to use `type(uint128).max`, which is more readable and easier to understand:

[LSP6KeyManagerCore.sol#L445](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L445)

```solidity
        uint256 mask = ~uint128(0);
```

## [N-08] `_setupLSP6ReentrancyGuard()` is redundant

In [`LSP6KeyManagerCore.sol`](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol), `_setupLSP6ReentrancyGuard()` is used to initialize `_reentrancyStatus` to `false`:

[LSP6KeyManagerCore.sol#L515-L520](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L515-L520)

```solidity
    /**
     * @dev Initialise _reentrancyStatus to _NOT_ENTERED.
     */
    function _setupLSP6ReentrancyGuard() internal virtual {
        _reentrancyStatus = false;
    }
```

However, this is redundant as `_reentrancyStatus` is set to `false` by default.

## [N-09] Missing check in `_deployCreate2()`

In [`ERC725XCore.sol`](https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725XCore.sol), `_deployCreate()` ensures that `address(this).balance` is more than or equal to the `value` parameter:

[ERC725XCore.sol#L225-L227](https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725XCore.sol#L225-L227)

```solidity
        if (address(this).balance < value) {
            revert ERC725X_InsufficientBalance(address(this).balance, value);
        }
```

However, this check is missing in [`_deployCreate2()`](https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725XCore.sol#L247-L268). Currently, this isn't exploitable as [Openzeppelin's Create2 library](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/release-v4.9/contracts/utils/Create2.sol) has the following check:

[Create2.sol#L31](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/release-v4.9/contracts/utils/Create2.sol#L31)

```solidity
        require(address(this).balance >= amount, "Create2: insufficient balance");
```

Nevertheless, consider adding the `address(this) < value` check to `_deployCreate2()` to have consistent error handling.

## [N-10] Document `_INTERFACEID_LSP20_CALL_VERIFICATION` in the LSP-20 specification

In `LSP20Constants.sol`, there are two ERC165 interface IDs:

[LSP20Constants.sol#L4-L8](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP20CallVerification/LSP20Constants.sol#L4-L8)

```solidity
// bytes4(keccak256("LSP20CallVerification"))
bytes4 constant _INTERFACEID_LSP20_CALL_VERIFICATION = 0x1a0eb6a5;

// `lsp20VerifyCall(address,uint256,bytes)` selector XOR `lsp20VerifyCallResult(bytes32,bytes)` selector
bytes4 constant _INTERFACEID_LSP20_CALL_VERIFIER = 0x480c0ec2;
```

However, in the [LSP-20 specification](https://github.com/lukso-network/LIPs/blob/main/LSPs/LSP-20-CallVerification.md), only `_INTERFACEID_LSP20_CALL_VERIFICATION` is mentioned. This might be confusing for developers that wish to implement LSP-20, but are unable to figure out which interface ID to use. 

### Recommendation

Consider adding `_INTERFACEID_LSP20_CALL_VERIFICATION` to the specification and explaining when each interface ID should be used.

## [N-11] `LSP8CompatibleERC721.sol` doesn't have a `_safeMint()` function

`LSP8CompatibleERC721.sol` is meant to resemble the ERC721 standard. However, although it has a `safeTransferFrom()` function, its contract does not implement a `_safeMint()` function. 

This might confuse developers who are used to popular implementations of ERC721 (eg. [Openzepplein's ERC721](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC721/ERC721.sol#L245-L247)), which usually have an in-built `_safeMint()` function.