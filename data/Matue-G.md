# Don't Initialize Variables with Default Value
## Description
Uninitialized variables are assigned with the types default value.

Explicitly initializing a variable with it's default value costs unnecessary gas.

The case described was seen in the isSetData variable, more details in the code snippet below.

## Code Snippet
```
    function lsp20VerifyCall(
        address caller,
        uint256 msgValue,
        bytes calldata data
    ) external returns (bytes4) {
        bool isSetData = false;
        if (
            bytes4(data) == IERC725Y.setData.selector ||
            bytes4(data) == IERC725Y.setDataBatch.selector
        ) {
            isSetData = true;
        }

        // If target is invoking the verification, emit the event and change the reentrancy guard
        if (msg.sender == _target) {
            bool isReentrantCall = _nonReentrantBefore(isSetData, caller);

            _verifyPermissions(caller, msgValue, data);
            emit VerifiedCall(caller, msgValue, bytes4(data));

            // if it's a setData call, do not invoke the `lsp20VerifyCallResult(..)` function
            return
                isSetData || isReentrantCall
                    ? _LSP20_VERIFY_CALL_MAGIC_VALUE_WITHOUT_POST_VERIFICATION
                    : _LSP20_VERIFY_CALL_MAGIC_VALUE_WITH_POST_VERIFICATION;
        }
        // If a different address is invoking the verification, do not change the state
        // and do read-only verification
        else {
            bool isReentrantCall = _reentrancyStatus;

            if (isReentrantCall) {
                _requirePermissions(
                    caller,
                    ERC725Y(_target).getPermissionsFor(caller),
                    _PERMISSION_REENTRANCY
                );
            }

            _verifyPermissions(caller, msgValue, data);

            // if it's a setData call, do not invoke the `lsp20VerifyCallResult(..)` function
            return
                isSetData || isReentrantCall
                    ? _LSP20_VERIFY_CALL_MAGIC_VALUE_WITHOUT_POST_VERIFICATION
                    : _LSP20_VERIFY_CALL_MAGIC_VALUE_WITH_POST_VERIFICATION;
        }
    }
```
## Reference
https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L256

# Condition that will never be met
## Description
The condition validated in the aforementioned require (currentOwner == owner()) will never be met because the function does not manipulate the currentOwner address, therefore, a cost of gas is not necessary.

## Code Snippet
```
    function transferOwnership(
        address newOwner
    ) public virtual override(OwnableUnset, ILSP14Ownable2Step) onlyOwner {
        _transferOwnership(newOwner);

        address currentOwner = owner();
        emit OwnershipTransferStarted(currentOwner, newOwner);

        newOwner.tryNotifyUniversalReceiver(
            _TYPEID_LSP14_OwnershipTransferStarted,
            ""
        );
        require(
            currentOwner == owner(),
            "LSP14: newOwner MUST accept ownership in a separate transaction"
        );
    }
```

## Reference
https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol#L79

# Use != 0 instead of > 0 for Unsigned Integer Comparison
## Description
When dealing with unsigned integer types, comparisons with != 0 are cheaper than with > 0.

*There are some instances of this problem:*

```
    function batchCalls(
        bytes[] calldata data
    ) public returns (bytes[] memory results) {
        results = new bytes[](data.length);
        for (uint256 i; i < data.length; ) {
            (bool success, bytes memory result) = address(this).delegatecall(
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
    }
```
Reference: https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L182

```
    function isValidSignature(
        bytes32 dataHash,
        bytes memory signature
    ) public view virtual returns (bytes4 magicValue) {
        address _owner = owner();

        // If owner is a contract
        if (_owner.code.length > 0) {
            (bool success, bytes memory result) = _owner.staticcall(
                abi.encodeWithSelector(
                    IERC1271.isValidSignature.selector,
                    dataHash,
                    signature
                )
            );

            bool isValid = (success &&
                result.length == 32 &&
                abi.decode(result, (bytes32)) == bytes32(_ERC1271_MAGICVALUE));

            return isValid ? _ERC1271_MAGICVALUE : _ERC1271_FAILVALUE;
        }
        // If owner is an EOA
        else {
            // if isValidSignature fail, the error is catched in returnedError
            (address recoveredAddress, ECDSA.RecoverError returnedError) = ECDSA
                .tryRecover(dataHash, signature);

            // if recovering throws an error, return the fail value
            if (returnedError != ECDSA.RecoverError.NoError)
                return _ERC1271_FAILVALUE;

            // if recovering is successful and the recovered address matches the owner's address,
            // return the ERC1271 magic value. Otherwise, return the ERC1271 fail value
            // matches the address of the owner, otherwise return fail value
            return
                recoveredAddress == _owner
                    ? _ERC1271_MAGICVALUE
                    : _ERC1271_FAILVALUE;
        }
    }
```
Reference: https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L741

```
    function _whenReceiving(
        bytes32 typeId,
        address notifier,
        bytes32 notifierMapKey,
        bytes4 interfaceID
    ) internal virtual returns (bytes memory) {
        bytes32[] memory dataKeys;
        bytes[] memory dataValues;

        // if it's a token transfer (LSP7/LSP8)
        if (typeId != _TYPEID_LSP9_OwnershipTransferred_RecipientNotification) {
            // CHECK balance only when the Token contract is already deployed,
            // not when tokens are being transferred on deployment through the `constructor`
            if (notifier.code.length > 0) {
                // if the amount sent is 0, then do not update the keys
                uint256 balance = ILSP7DigitalAsset(notifier).balanceOf(
                    msg.sender
                );
                if (balance == 0) return "LSP1: balance not updated";
            }

            (dataKeys, dataValues) = LSP5Utils.generateReceivedAssetKeys(
                msg.sender,
                notifier,
                notifierMapKey,
                interfaceID
            );

            // Set the LSP5 generated data keys on the account
            IERC725Y(msg.sender).setDataBatch(dataKeys, dataValues);
            return "";
        } else {
            (dataKeys, dataValues) = LSP10Utils.generateReceivedVaultKeys(
                msg.sender,
                notifier,
                notifierMapKey
            );

            // Set the LSP10 generated data keys on the account
            IERC725Y(msg.sender).setDataBatch(dataKeys, dataValues);
            return "";
        }
    }
```
Reference: https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol#L165

```
    function _revert(bool postCall, bytes memory returnedData) internal pure {
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
    }
}
```
Reference: https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP20CallVerification/LSP20CallVerification.sol#L86

```
    function _notifyTokenReceiver(
        address to,
        bool allowNonLSP1Recipient,
        bytes memory lsp1Data
    ) internal virtual {
        if (
            ERC165Checker.supportsERC165InterfaceUnchecked(
                to,
                _INTERFACEID_LSP1
            )
        ) {
            ILSP1UniversalReceiver(to).universalReceiver(
                _TYPEID_LSP7_TOKENSRECIPIENT,
                lsp1Data
            );
        } else if (!allowNonLSP1Recipient) {
            if (to.code.length > 0) {
                revert LSP7NotifyTokenReceiverContractMissingLSP1Interface(to);
            } else {
                revert LSP7NotifyTokenReceiverIsEOA(to);
            }
        }
    }
}
```
Reference: https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L491

```
    function _notifyTokenReceiver(
        address to,
        bool allowNonLSP1Recipient,
        bytes memory lsp1Data
    ) internal virtual {
        if (
            ERC165Checker.supportsERC165InterfaceUnchecked(
                to,
                _INTERFACEID_LSP1
            )
        ) {
            ILSP1UniversalReceiver(to).universalReceiver(
                _TYPEID_LSP8_TOKENSRECIPIENT,
                lsp1Data
            );
        } else if (!allowNonLSP1Recipient) {
            if (to.code.length > 0) {
                revert LSP8NotifyTokenReceiverContractMissingLSP1Interface(to);
            } else {
                revert LSP8NotifyTokenReceiverIsEOA(to);
            }
        }
    }
}
```
Reference: https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L493

```
    function _checkOnERC721Received(
        address from,
        address to,
        uint256 tokenId,
        bytes memory data
    ) private returns (bool) {
        if (to.code.length > 0) {
            try
                IERC721Receiver(to).onERC721Received(
                    msg.sender,
                    from,
                    tokenId,
                    data
                )
            returns (bytes4 retval) {
                return retval == IERC721Receiver.onERC721Received.selector;
            } catch (bytes memory reason) {
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
            }
        } else {
            return true;
        }
    }
```
Reference: https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol#L313

```
    function _checkOnERC721Received(
        address from,
        address to,
        uint256 tokenId,
        bytes memory data
    ) private returns (bool) {
        if (to.code.length > 0) {
            try
                IERC721Receiver(to).onERC721Received(
                    msg.sender,
                    from,
                    tokenId,
                    data
                )
            returns (bytes4 retval) {
                return retval == IERC721Receiver.onERC721Received.selector;
            } catch (bytes memory reason) {
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
            }
        } else {
            return true;
        }
    }
```
Reference: https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol#L313

