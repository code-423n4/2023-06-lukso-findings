### [L-01] LSP14Ownable2Step: warn about the side effects between these 2-step processes: transfer ownership & renounce ownership

#### Impact

The current best practices for transferring the ownership is a 2-step process (as an example, see [OpenZeppelin `Ownable2Step.transferOwnership`](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.9.2/contracts/access/Ownable2Step.sol#L35) and [Vectorized Solady `Ownable.transferOwnership`](https://github.com/Vectorized/solady/blob/main/src/auth/Ownable.sol#L124)). In the same way, best practices for renouncing the ownership is a 2-step process, although the most common access control libraries still implement it in a single-step fashion (as an example, see [OpenZeppelin `Ownable2Step.renounceOwnership` (from `Ownable.renounceOwnership`)](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.9.2/contracts/access/Ownable.sol#L61) and [Vectorized Solady `Ownable.renounceOwnership`](https://github.com/Vectorized/solady/blob/main/src/auth/Ownable.sol#L136)). And on this single-step implementation, renouncing the ownership does not interfere with transferring the ownership, which allows to:

```
1. Owner transfers the ownership to newOwner (as pending owner).
2. Owner renounces the ownership (Zero address is the owner, newOwner is the pending owner).
3. newOwner accepts the ownership (from pending owner to owner).
```

Lukso rightfully implements a 2-step renounce ownership process (one of the two options suggested by [Watchpug on LUKSO LSPs#2 Audit Report - [WP‐L6] LSP‐14: `_pendingOwner` should be deleted when `_renounceOwnership` is called for the first time to initialize it](https://github.com/code-423n4/2023-06-lukso/blob/09bbdd68eeba6ed4dd624286c94a1947f79c1959/audits/Watchpug_audit_2022_12_15.pdf)), however it interfers with a transferring ownership process and vice-versa:

- `transferOwnership()` resets a rounouncing ownership process by deleting `_renounceOwnershipStartedAt`.
- `renounceOwnership()` resets a transferring ownership process by deleting `_pendingOwner`.

This has a different outcome for the case described above:

```
1. Owner transfers the ownership to newOwner (as pending owner).
2. Owner renounces the ownership:
  2.1. Owner makes the 1st step for renouncing the ownership (Zero address is the pending owner).
  2.2. Owner makes the 2nd step for renouncing the ownership (Zero address is the owner and the pending owner).
3. newOwner can't accept the ownership (the contract is left without owner).
```

Lukso's [`ILSP14Ownable2Step` contract NatSpec](https://github.com/code-423n4/2023-06-lukso/blob/09bbdd68eeba6ed4dd624286c94a1947f79c1959/contracts/LSP14Ownable2Step/ILSP14Ownable2Step.sol#L74) and [LSP14Ownable2Step Official docs](https://docs.lukso.tech/contracts/contracts/LSP14Ownable2Step/#renounceownership) warn about the `renounceOwnership()` dangers and also inform about the `pendingOwner` value on different scenarios, but none of them mention the fact that `transferOwnership()` & `renounceOwnership()` cancel each other nor the differences with the single-step process common in the industry.

#### Recommended Mitigation Steps

Explain in greater detail (in the `LSP14Ownable2Step` module NatSpec and the official website docs) how `transferOwnership()` and `renounceOwnership()` interact with each other and warn about the differences between Lukso's 2-step process and the common single-step one.

Alternatively, for extra on-chain safety, consider introducing a lock to prevent to transfer the ownership during an ownership renouncement and vice-versa (already suggested by [Watchpug on LUKSO LSPs#2 Audit Report - [WP‐L6] LSP‐14: `_pendingOwner` should be deleted when `_renounceOwnership` is called for the first time to initialize it](https://github.com/code-423n4/2023-06-lukso/blob/09bbdd68eeba6ed4dd624286c94a1947f79c1959/audits/Watchpug_audit_2022_12_15.pdf) but discarded by Lukso).

https://github.com/code-423n4/2023-06-lukso/blob/09bbdd68eeba6ed4dd624286c94a1947f79c1959/contracts/LSP14Ownable2Step/ILSP14Ownable2Step.sol#L44-L54
https://github.com/code-423n4/2023-06-lukso/blob/09bbdd68eeba6ed4dd624286c94a1947f79c1959/contracts/LSP14Ownable2Step/ILSP14Ownable2Step.sol#L68-L76

### [L-02] LSP6SetDataModule: `_verifyCanSetData` reverts with panic if `inputDataKeys` or `inputDataValues` is an empty array

File: contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol

#### Impact

The function [`LSP6SetDataModule._verifyCanSetData`](https://github.com/code-423n4/2023-06-lukso/blob/09bbdd68eeba6ed4dd624286c94a1947f79c1959/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L116) reverts with `panic 0x32 (Array accessed at an out-of-bounds or negative index)` if `inputDataKeys` or `inputDataValues` is an empty array because of the `do..while` loop first iteration.

Currently, there is no risk for the `LSP6KeyManager` because it is exclusively called by [`LSP6KeyManagerCore._verifyPermissions`](https://github.com/code-423n4/2023-06-lukso/blob/09bbdd68eeba6ed4dd624286c94a1947f79c1959/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L489) when data has to be set in batch (and it does not make any sense to set no data). However, future extensions of `LSP6KeyManagerCore` should be aware of it.

#### Recommended Mitigation Steps

Revert if `inputDataKeys` or `inputDataValues` length is zero.

```solidity
File: contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol

// Description
        if (inputDataKeys.length != inputDataValues.length) {
            revert ERC725Y_DataKeysValuesLengthMismatch();
        }

// Recommendation
        uint256 inputDataKeysLength = inputDataKeys.length;
        if (inputDataKeys.length == 0) {
            revert ERC725Y_DataKeysValuesIsEmpty();
        }
        uint256 inputDataValuesLength = inputDataValues.length;
        if (inputDataKeysLength != inputDataValuesLength) {
            revert ERC725Y_DataKeysValuesLengthMismatch();
        }
```

https://github.com/code-423n4/2023-06-lukso/blob/09bbdd68eeba6ed4dd624286c94a1947f79c1959/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L123

## Non-critical Issues

### [N-01] Bad usage of `unchecked` block by wrapping the entire (`_updateOperator`) function

Future changes on `_updateOperator` could under- or overflow.

```solidity
File: contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol

// Description
        unchecked {
            _updateOperator(
                msg.sender,
                operator,
                currentAllowance - substractedAmount
            );
        }

// Recommendation
        uint256 updatedAllowance;
        unchecked {
            updatedAllowance = currentAllowance - substractedAmount;
        }
        _updateOperator(
            msg.sender,
            operator,
            updatedAllowance
        );
```

https://github.com/code-423n4/2023-06-lukso/blob/09bbdd68eeba6ed4dd624286c94a1947f79c1959/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L241-L247

### [N-02] Misleading and/or incorrect revert messages

_There is 1 instance of this issue:_

<details>
<summary>see instances</summary>

```solidity
File: contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol

// Description
        require(dataKey.length >= 2, "MUST be longer than 2 characters");

// Recommendation
        require(dataKey.length >= 2, "MUST be longer than 1 character"); // "LSP2: must be >= 2 characters"
```

https://github.com/code-423n4/2023-06-lukso/blob/09bbdd68eeba6ed4dd624286c94a1947f79c1959/contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol#L36

</details>

### [N-03] Misleading variable names

_There is 1 instance of this issue:_

<details>
<summary>see instances</summary>

```solidity
File: contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol

// Description
        bytes32 hashFunctionDigest = keccak256(bytes(hashFunction));
        bytes32 jsonDigest = keccak256(bytes(assetBytes));

        return abi.encodePacked(bytes4(hashFunctionDigest), jsonDigest, url);

// Recommendation
        bytes32 hashFunctionDigest = keccak256(bytes(hashFunction));
        bytes32 assetDigest = keccak256(bytes(assetBytes));

        return abi.encodePacked(bytes4(hashFunctionDigest), assetDigest, url);
```

https://github.com/code-423n4/2023-06-lukso/blob/09bbdd68eeba6ed4dd624286c94a1947f79c1959/contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol#L221-L224

</details>

### [N-04] NatSpec typos

_There are 6 instances of this issue:_

<details>
<summary>see instances</summary>

```solidity
File: contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol

// Description
     * 2. Query the ERC725Y storage with the data key `[_LSP1_UNIVERSAL_RECEIVER_DELEGATE_KEY] + <bytes32 typeId>`.

// Recommendation
     * 2. Query the ERC725Y storage with the data key `[_LSP1_UNIVERSAL_RECEIVER_DELEGATE_PREFIX] + <bytes32 typeId>`.
```

https://github.com/code-423n4/2023-06-lukso/blob/09bbdd68eeba6ed4dd624286c94a1947f79c1959/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L444

```solidity
File: contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol

// Description
        // Generate the data key {_LSP1_UNIVERSAL_RECEIVER_DELEGATE_KEY + <bytes32 typeId>}

// Recommendation
        // Generate the data key {_LSP1_UNIVERSAL_RECEIVER_DELEGATE_PREFIX + <bytes32 typeId>}
```

https://github.com/code-423n4/2023-06-lukso/blob/09bbdd68eeba6ed4dd624286c94a1947f79c1959/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L496

```solidity
File: contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol

// Description
        // Query the ERC725Y storage with the data key {_LSP1_UNIVERSAL_RECEIVER_DELEGATE_KEY + <bytes32 typeId>}

// Recommendation
        // Query the ERC725Y storage with the data key {_LSP1_UNIVERSAL_RECEIVER_DELEGATE_PREFIX + <bytes32 typeId>}
```

https://github.com/code-423n4/2023-06-lukso/blob/09bbdd68eeba6ed4dd624286c94a1947f79c1959/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L502

```solidity
File: contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol

// Description
    /**
     * @dev Generate a ASSETURL valueContent
     * @param hashFunction The function used to hash the JSON file
     * @param assetBytes Bytes value of the JSON file
     * @param url The URL where the JSON file is hosted
     */

// Recommendation
    /**
     * @dev Generate an ASSETURL valueContent
     * @param hashFunction The function used to hash the asset file
     * @param assetBytes Bytes value of the asset file
     * @param url The URL where the asset file is hosted
     */
```

https://github.com/code-423n4/2023-06-lukso/blob/09bbdd68eeba6ed4dd624286c94a1947f79c1959/contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol#L210-L215

```solidity
File: contracts/LSP5ReceivedAssets/LSP5Utils.sol

// Description
        // Updating the number of the received assets (decrementing by 1

// Recommendation
        // Updating the number of the received assets (decrementing by 1)
```

https://github.com/code-423n4/2023-06-lukso/blob/09bbdd68eeba6ed4dd624286c94a1947f79c1959/contracts/LSP5ReceivedAssets/LSP5Utils.sol#L136

```solidity
File: submodules/ERC725/implementations/contracts/ERC725XCore.sol

// Description
     * see `IERC725X,execute(uint256[],address[],uint256[],bytes[])`

// Recommendation
     * see `IERC725X.execute(uint256[],address[],uint256[],bytes[])`
```

https://github.com/ERC725Alliance/ERC725/blob/31574e1ac5259713e83c1b299800401a0737d483/implementations/contracts/ERC725XCore.sol#L129

</details>

### [N-05] NatSpec style

_There are 4 instances of this issue:_

<details>
<summary>see instances</summary>

```solidity
File: contracts/UniversalProfile.sol

// Description
 * @dev Implementation of the ERC725Account + LSP1 universalReceiver

// Recommendation
 * @dev Implementation of the ERC725Account + LSP1 UniversalReceiver
```

https://github.com/code-423n4/2023-06-lukso/blob/09bbdd68eeba6ed4dd624286c94a1947f79c1959/contracts/UniversalProfile.sol#L16

```solidity
File: contracts/UniversalProfileInit.sol

// Description
 * @dev Implementation of the ERC725Account + LSP1 universalReceiver

// Recommendation
 * @dev Implementation of the ERC725Account + LSP1 UniversalReceiver
```

https://github.com/code-423n4/2023-06-lukso/blob/09bbdd68eeba6ed4dd624286c94a1947f79c1959/contracts/UniversalProfileInit.sol#L10

```solidity
File: contracts/UniversalProfileInit.sol

// Description
     * @notice deploying a `UniversalProfileInit` base contract to be used behind proxy

// Recommendation
     * @notice Deploying a `UniversalProfileInit` base contract to be used behind proxy
```

https://github.com/code-423n4/2023-06-lukso/blob/09bbdd68eeba6ed4dd624286c94a1947f79c1959/contracts/UniversalProfileInit.sol#L14

```solidity
File: contracts/UniversalProfileInitAbstract.sol

// Description
 * @dev Implementation of the ERC725Account + LSP1 universalReceiver

// Recommendation
 * @dev Implementation of the ERC725Account + LSP1 UniversalReceiver
```

https://github.com/code-423n4/2023-06-lukso/blob/09bbdd68eeba6ed4dd624286c94a1947f79c1959/contracts/UniversalProfileInitAbstract.sol#L18

</details>

### [N-06] NatSpec missing `@inheritdoc`

_There are 4 instances of this issue:_

<details>
<summary>see instances</summary>

```solidity
File: contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol

// Description
    /**
     * @dev Allows a caller to batch different function calls in one call.
     * Perform a delegatecall on self, to call different functions with preserving the context
     * It is not possible to send value along the functions call due to the use of delegatecall.
     *
     * @param data An array of ABI encoded function calls to be called on the contract.
     * @return results An array of values returned by the executed functions.
     */

// Recommendation
    /**
     * @inheritdoc ILSP0ERC725Account
     */
```

https://github.com/code-423n4/2023-06-lukso/blob/09bbdd68eeba6ed4dd624286c94a1947f79c1959/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L163-L170

```solidity
File: LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol

// Description
    /**
     * @dev Handles two cases:
     * - Registers the address of received assets (exclusively LSP7 and LSP8) and vaults (exclusively LSP9) according
     *   to {LSP5-ReceivedAssets} and {LSP10-ReceivedVaults} respectively
     *
     * - Removes the address of registered assets and vaults when the full balance is sent from the LSP0ERC725Account contract
     *
     * Requirements:
     * - The contract should be able to setData the LSP5 and LSP10 data Keys according to the logic of the owner
     *    of the LSP0ERC725Account.
     *
     * - Cannot accept native tokens
     */

// Recommendation
    /**
     * @inheritdoc ILSP1UniversalReceiver
     * @dev Handles two cases:
     * - Registers the address of received assets (exclusively LSP7 and LSP8) and vaults (exclusively LSP9) according
     *   to {LSP5-ReceivedAssets} and {LSP10-ReceivedVaults} respectively
     *
     * - Removes the address of registered assets and vaults when the full balance is sent from the LSP0ERC725Account contract
     *
     * Requirements:
     * - The contract should be able to setData the LSP5 and LSP10 data Keys according to the logic of the owner
     *    of the LSP0ERC725Account.
     *
     * - Cannot accept native tokens
     */
```

https://github.com/code-423n4/2023-06-lukso/blob/09bbdd68eeba6ed4dd624286c94a1947f79c1959/contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol#L65-L77

```solidity
File: LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol

// Description
    /**
     * @dev Returns the name of the token.
     * @return The name of the token
     */

// Recommendation
    /**
     * @inheritdoc ILSP4Compatibility
     */
```

https://github.com/code-423n4/2023-06-lukso/blob/09bbdd68eeba6ed4dd624286c94a1947f79c1959/contracts/LSP4DigitalAssetMetadata/LSP4Compatibility.sol#L21-L24

```solidity
File: LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol

// Description
    /**
     * @dev Returns the symbol of the token, usually a shorter version of the name.
     * @return The symbol of the token
     */

// Recommendation
    /**
     * @inheritdoc ILSP4Compatibility
     */
```

https://github.com/code-423n4/2023-06-lukso/blob/09bbdd68eeba6ed4dd624286c94a1947f79c1959/contracts/LSP4DigitalAssetMetadata/LSP4Compatibility.sol#L30-L33

</details>

### [N-07] Standardise Custom Errors naming convention

Except for a few cases, it is useful to prepend to a Custom Error name its origin (e.g. module, contract). Almost all Lukso's Custom Errors are prepended with the standard name (e.g. `LSP1`, `LSP14`) and there is no reason why the following cases do not follow this pattern:

- All errors in [`LSP1Errors.sol`](https://github.com/code-423n4/2023-06-lukso/blob/09bbdd68eeba6ed4dd624286c94a1947f79c1959/contracts/LSP1UniversalReceiver/LSP1Errors.sol).
- Several errors in [`LSP6Errors.sol`](https://github.com/code-423n4/2023-06-lukso/blob/09bbdd68eeba6ed4dd624286c94a1947f79c1959/contracts/LSP6KeyManager/LSP6Errors.sol).
- All errors in [`LSP17Errors.sol`](https://github.com/code-423n4/2023-06-lukso/blob/09bbdd68eeba6ed4dd624286c94a1947f79c1959/contracts/LSP17ContractExtension/LSP17Errors.sol).

https://github.com/code-423n4/2023-06-lukso/blob/09bbdd68eeba6ed4dd624286c94a1947f79c1959/contracts/LSP1UniversalReceiver/LSP1Errors.sol
https://github.com/code-423n4/2023-06-lukso/blob/09bbdd68eeba6ed4dd624286c94a1947f79c1959/contracts/LSP17ContractExtension/LSP17Errors.sol
https://github.com/code-423n4/2023-06-lukso/blob/09bbdd68eeba6ed4dd624286c94a1947f79c1959/contracts/LSP6KeyManager/LSP6Errors.sol
