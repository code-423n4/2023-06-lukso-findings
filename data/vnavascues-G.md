## Gas Optimizations

### [G-01] The increment expression of a loop should be wrapped in `unchecked{<increment_expression>}` when it is not possible to overflow

```solidity
File: contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol

// Description
                inputDataKeysAllowed++;

// Recommendation
                unchecked {
                    ++inputDataKeysAllowed;
                }
```

https://github.com/code-423n4/2023-06-lukso/blob/09bbdd68eeba6ed4dd624286c94a1947f79c1959/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L156

### [G-02] Check multiple arrays length equality with bitwise OR (`|`)

Not only it costs slightly less gas, but also it improves readability and makes the condition less prone to human error.

_There are 4 instances of this issue:_

<details>
<summary>see instances</summary>

```solidity
File: contracts/LSP6KeyManager/LSP6KeyManagerCore.sol

// Description
        if (
            signatures.length != nonces.length ||
            nonces.length != validityTimestamps.length ||
            validityTimestamps.length != values.length ||
            values.length != payloads.length
        ) {
            revert BatchExecuteRelayCallParamsLengthMismatch();
        }

// Recommendation
        uint256 arraysLength = payloads.length;
        if (
            arraysLength != signatures.length |
            nonces.length |
            validityTimestamps.length |
            values.length
        ) {
            revert BatchExecuteRelayCallParamsLengthMismatch();
        }
```

https://github.com/code-423n4/2023-06-lukso/blob/09bbdd68eeba6ed4dd624286c94a1947f79c1959/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L211-L218

```solidity
File: contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol

// Description
        uint256 fromLength = from.length;
        if (
            fromLength != to.length ||
            fromLength != amount.length ||
            fromLength != allowNonLSP1Recipient.length ||
            fromLength != data.length
        ) {
            revert LSP7InvalidTransferBatch();
        }

// Recommendation
        uint256 arraysLength = from.length;
        if (
            arraysLength != to.length |
            amount.length |
            allowNonLSP1Recipient.length |
            data.length
        ) {
            revert LSP7InvalidTransferBatch();
        }
```

https://github.com/code-423n4/2023-06-lukso/blob/09bbdd68eeba6ed4dd624286c94a1947f79c1959/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L161-L169

```solidity
File: contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol

// Description
        uint256 fromLength = from.length;
        if (
            fromLength != to.length ||
            fromLength != tokenId.length ||
            fromLength != allowNonLSP1Recipient.length ||
            fromLength != data.length
        ) {
            revert LSP8InvalidTransferBatch();
        }


// Recommendation
        uint256 arraysLength = from.length;
        if (
            arraysLength != to.length |
            tokenId.length |
            allowNonLSP1Recipient.length |
            data.length
        ) {
            revert LSP8InvalidTransferBatch();
        }
```

https://github.com/code-423n4/2023-06-lukso/blob/09bbdd68eeba6ed4dd624286c94a1947f79c1959/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L217-L225

```solidity
File: submodules/ERC725/implementations/contracts/ERC725XCore.sol

// Description
        if (
            operationsType.length != targets.length ||
            (targets.length != values.length || values.length != datas.length)
        ) {
            revert ERC725X_ExecuteParametersLengthMismatch();
        }


// Recommendation
        uint256 arraysLength = operationsType.length;
        if (
            arraysLength.length != targets.length |
            values.length |
            datas.length
        ) {
            revert ERC725X_ExecuteParametersLengthMismatch();
        }
```

https://github.com/ERC725Alliance/ERC725/blob/31574e1ac5259713e83c1b299800401a0737d483/implementations/contracts/ERC725XCore.sol#L137-L142

</details>

### [G-03] Improve low level bubbling up revert reasons

Save gas by checking the `result` length with `==` and removing the unnecessary `else` block.

_There are 2 instances of this issue:_

<details>
<summary>see instances</summary>

```solidity
File: contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol

// Description
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


// Recommendation
                // Look for revert reason and bubble it up if present
                if (result.length == 0) revert("LSP0: batchCalls reverted");
                // The easiest way to bubble the revert reason is using memory via assembly
                // solhint-disable no-inline-assembly
                /// @solidity memory-safe-assembly
                assembly {
                    let returndata_size := mload(result)
                    revert(add(32, result), returndata_size)
                }
```

https://github.com/code-423n4/2023-06-lukso/blob/09bbdd68eeba6ed4dd624286c94a1947f79c1959/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L181-L192

```solidity
File: contracts/LSP20CallVerification/LSP20CallVerification.sol

// Description
        // Look for revert reason and bubble it up if present
        if (returnedData.length > 0) {
            // The easiest way to bubble the revert reason is using memory via assembly
            // solhint-disable no-inline-assembly
            /// @solidity memory-safe-assembly
            assembly {
                let returndata_size := mload(returnedData)
                revert(add(32, returnedData), returndata_size)
            }
        } else {
            revert LSP20CallingVerifierFailed(postCall);
        }


// Recommendation
        // Look for revert reason and bubble it up if present
        if (returnedData.length == 0) revert LSP20CallingVerifierFailed(postCall);
        // The easiest way to bubble the revert reason is using memory via assembly
        // solhint-disable no-inline-assembly
        /// @solidity memory-safe-assembly
        assembly {
            let returndata_size := mload(returnedData)
            revert(add(32, returnedData), returndata_size)
        }
```

https://github.com/code-423n4/2023-06-lukso/blob/09bbdd68eeba6ed4dd624286c94a1947f79c1959/contracts/LSP20CallVerification/LSP20CallVerification.sol#L85-L96

</details>

### [G-04] `else` block not required

_There are 3 instances of this issue:_

<details>
<summary>see instances</summary>

```solidity
File: contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol

// Description
        bytes32 requiredPermission = _getPermissionRequiredToSetDataKey(
            controlledContract,
            controllerPermissions,
            inputDataKey,
            inputDataValue
        );

        // CHECK if allowed to set an ERC725Y Data Key
        if (requiredPermission == _PERMISSION_SETDATA) {
            // Skip if caller has SUPER permissions
            if (controllerPermissions.hasPermission(_PERMISSION_SUPER_SETDATA))
                return;

            _requirePermissions(
                controllerAddress,
                controllerPermissions,
                _PERMISSION_SETDATA
            );

            _verifyAllowedERC725YSingleKey(
                controllerAddress,
                inputDataKey,
                ERC725Y(controlledContract).getAllowedERC725YDataKeysFor(
                    controllerAddress
                )
            );
        } else {
            // Do not check again if we already checked that the controller had the permissions inside `_getPermissionRequiredToSetDataKey(...)`
            if (requiredPermission == bytes32(0)) return;

            // Otherwise CHECK the required permission if setting LSP6 permissions, LSP1 Delegate or LSP17 Extensions.
            _requirePermissions(
                controllerAddress,
                controllerPermissions,
                requiredPermission
            );
        }


// Recommendation
        bytes32 requiredPermission = _getPermissionRequiredToSetDataKey(
            controlledContract,
            controllerPermissions,
            inputDataKey,
            inputDataValue
        );

        if (requiredPermission == bytes32(0)) return;

        // CHECK if allowed to set an ERC725Y Data Key
        if (requiredPermission == _PERMISSION_SETDATA) {
            // Skip if caller has SUPER permissions
            if (controllerPermissions.hasPermission(_PERMISSION_SUPER_SETDATA))
                return;

            _requirePermissions(
                controllerAddress,
                controllerPermissions,
                _PERMISSION_SETDATA
            );

            _verifyAllowedERC725YSingleKey(
                controllerAddress,
                inputDataKey,
                ERC725Y(controlledContract).getAllowedERC725YDataKeysFor(
                    controllerAddress
                )
            );
            return;
        }
        // Check the required permission if setting LSP6 permissions, LSP1 Delegate or LSP17 Extensions.
        _requirePermissions(
            controllerAddress,
            controllerPermissions,
            requiredPermission
        );
```

https://github.com/code-423n4/2023-06-lukso/blob/09bbdd68eeba6ed4dd624286c94a1947f79c1959/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L69-L105

```solidity
File: contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol

// Description
                if (reason.length == 0) {
                    revert(
                        "LSP8CompatibleERC721: transfer to non ERC721Receiver implementer"
                    );
                } else {
                    // solhint-disable no-inline-assembly
                    /// @solidity memory-safe-assembly
                    assembly {
                        revert(add(32, reason), mload(reason))
                    }
                }


// Recommendation
                if (reason.length == 0) revert("LSP8CompatibleERC721: transfer to non ERC721Receiver implementer");
                // solhint-disable no-inline-assembly
                /// @solidity memory-safe-assembly
                assembly {
                    revert(add(32, reason), mload(reason))
                }
```

https://github.com/code-423n4/2023-06-lukso/blob/09bbdd68eeba6ed4dd624286c94a1947f79c1959/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol#L324-L334

```solidity
File: contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol

// Description
                if (reason.length == 0) {
                    revert(
                        "LSP8CompatibleERC721: transfer to non ERC721Receiver implementer"
                    );
                } else {
                    // solhint-disable no-inline-assembly
                    /// @solidity memory-safe-assembly
                    assembly {
                        revert(add(32, reason), mload(reason))
                    }
                }


// Recommendation
                if (reason.length == 0) revert("LSP8CompatibleERC721: transfer to non ERC721Receiver implementer");
                // solhint-disable no-inline-assembly
                /// @solidity memory-safe-assembly
                assembly {
                    revert(add(32, reason), mload(reason))
                }

```

https://github.com/code-423n4/2023-06-lukso/blob/09bbdd68eeba6ed4dd624286c94a1947f79c1959/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol#L324-L333

</details>