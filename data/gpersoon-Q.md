

# 1 Missing `emit` in `lsp20VerifyCall()`
## Impact
Function `lsp20VerifyCall()` has two calls to `_verifyPermissions()`, however only the first one is followed by an `emit`.
This might make debugging transaction more difficult. Also indexed data by protocols like TheGraph will be incomplete.

## Proof of Concept
[LSP6KeyManagerCore.sol#L249-L296](https://github.com/lukso-network/lsp-smart-contracts/blob/v0.10.2/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L249-L296)

```solidity
function lsp20VerifyCall(...) ... {
    ...
    if (msg.sender == _target) {
        ...
        _verifyPermissions(caller, msgValue, data);
        emit VerifiedCall(caller, msgValue, bytes4(data));
        return ...
    }
    else {
        ...
        _verifyPermissions(caller, msgValue, data);
        // no emit
        return ...
    }
}
```

## Tools Used

## Recommended Mitigation Steps
Add an `emit` after the second call to `_verifyPermissions()`:
```diff
function lsp20VerifyCall(...) ... {
    ...
    if (msg.sender == _target) {
        ...
        _verifyPermissions(caller, msgValue, data);
        emit VerifiedCall(caller, msgValue, bytes4(data));
        return ...
    }
    else {
        ...
        _verifyPermissions(caller, msgValue, data);
+       emit VerifiedCall(caller, msgValue, bytes4(data));
        return ...
    }
}
```


# 2 `renounceOwnership()` can't be done via controller

## Impact
The function `renounceOwnership()` or `LSP0ERC725AccountCore` can be called by a controller and the permissions of the controller are then checked via `_verifyCall`. This calls `_verifyPermissions()`, which allows for `transferOwnership()` and `acceptOwnership()`, but doesn't allow for `renounceOwnership()`.

This means  `renounceOwnership()` can only be done via the `owner` of the `KeyManager` and not via a `controller`.

## Proof of Concept
[LSP0ERC725AccountCore.sol#L655-L679](https://github.com/lukso-network/lsp-smart-contracts/blob/v0.10.2/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L655-L679)
```solidity
function renounceOwnership() ... { 
    ...
    bool verifyAfter = _verifyCall(_owner);
    LSP14Ownable2Step._renounceOwnership();
    ...
}
```
[LSP6KeyManagerCore.sol#L455-L511](https://github.com/lukso-network/lsp-smart-contracts/blob/v0.10.2/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L455-L511)
```solidity
function _verifyPermissions(...) ... {
    ...
    if (...) {
        ...
    } else if (
            erc725Function == ILSP14Ownable2Step.transferOwnership.selector ||
            erc725Function == ILSP14Ownable2Step.acceptOwnership.selector
        ) {
            LSP6OwnershipModule._verifyOwnershipPermissions(from, permissions);
        } else {
            revert InvalidERC725Function(erc725Function);
        }
    }
```

## Tools Used

## Recommended Mitigation Steps
Determine if a controller should be able to `renounceOwnership`.
If so, update `_verifyPermissions()` to allow this.
If not, remove the code from `renounceOwnership()` that prepares for this (e.g. the call to `_verifyCall()` )



# 3 `renounceOwnership()` doesn't notify UniversalReceiver
## Impact
Function `renounceOwnership()` doesn't notify UniversalReceiver, while `transferOwnership()`  and `acceptOwnership()`.
This is inconsistent and could mean the administration and verification of ownership isn't accurate, especially because the operation can be started via a `controller` that isn't the `owner` (see [LSP0ERC725AccountCore.sol renounceOwnership](https://github.com/lukso-network/lsp-smart-contracts/blob/v0.10.2/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L655-L679)).

Note: this situation exists in both `LSP0ERC725AccountCore` and `LSP14Ownable2Step`.

## Proof of Concept
[LSP0ERC725AccountCore.sol#L557-L697](https://github.com/lukso-network/lsp-smart-contracts/blob/v0.10.2/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L557-L697)
```solidity
abstract contract LSP0ERC725AccountCore is
    function transferOwnership(...) ... {
        ...
        pendingNewOwner.tryNotifyUniversalReceiver(_TYPEID_LSP0_OwnershipTransferStarted,"");
        ...
    }
    function acceptOwnership() ... {
        ...
        previousOwner.tryNotifyUniversalReceiver(_TYPEID_LSP0_OwnershipTransferred_SenderNotification,"");
        msg.sender.tryNotifyUniversalReceiver(_TYPEID_LSP0_OwnershipTransferred_RecipientNotification,"");
    }
    function renounceOwnership() .. {
       ... // no tryNotifyUniversalReceiver
    }
}
```
[LSP14Ownable2Step.sol#L66-L113](https://github.com/lukso-network/lsp-smart-contracts/blob/v0.10.2/contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol#L66-L113)
```solidity
abstract contract LSP14Ownable2Step is ... {
    function transferOwnership(...) ... { 
        ...
        newOwner.tryNotifyUniversalReceiver(_TYPEID_LSP14_OwnershipTransferStarted,"");
        ...
    }
    function acceptOwnership() public virtual {
        ...
        previousOwner.tryNotifyUniversalReceiver(_TYPEID_LSP14_OwnershipTransferred_SenderNotification,"");
        msg.sender.tryNotifyUniversalReceiver(_TYPEID_LSP14_OwnershipTransferred_RecipientNotification,"");
    }
   function renounceOwnership() .. {
       ... // no tryNotifyUniversalReceiver
    }
}
```

## Tools Used

## Recommended Mitigation Steps
Also send updates from `renounceOwnership()`.

# 4 Data corruption handled in different ways


## Impact
The function `_whenSending()` sometimes returns an error string when it encounters corrupt data. Sometimes it reverts, when the corruption is detected in `generateSentAssetKeys()`.

## Proof of Concept
[LSP1UniversalReceiverDelegateUP.sol#L201-L252](https://github.com/lukso-network/lsp-smart-contracts/blob/v0.10.2/contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol#L201-L252)
```solidity
function _whenSending(...) ... {
    if (typeId != _TYPEID_LSP9_OwnershipTransferred_SenderNotification) {
        ...
        (dataKeys, dataValues) = LSP5Utils.generateSentAssetKeys(...); // reverts on corrupt data
        if (dataKeys.length == 0 && dataValues.length == 0)
            return "LSP1: asset data corrupted"; // returns on corrupt data
        }
    }
}
```
[LSP5ReceivedAssets/LSP5Utils.sol#L117-L137](https://github.com/lukso-network/lsp-smart-contracts/blob/v0.10.2/contracts/LSP5ReceivedAssets/LSP5Utils.sol#L117-L137)
```solidity
function generateSentAssetKeys(...) ... {
    ...
    bytes memory lsp5ReceivedAssetsCountValue = getLSP5ReceivedAssetsCount(account);
    if (lsp5ReceivedAssetsCountValue.length != 16) {
        revert InvalidLSP5ReceivedAssetsArrayLength(...);
    }
}
```
## Tools Used

## Recommended Mitigation Steps
Consider handling all cases of data corruption in the same way.


# 5 Function `generateSentAssetKeys()` can revert

## Impact
Function `generateSentAssetKeys()` can revert on `uint128 newArrayLength = oldArrayLength - 1`.
This only happens when the data is already corrupt.
However in this case no useful error/revert message is given.

## Proof of Concept
[LSP5Utils.sol#L117-L137](https://github.com/lukso-network/lsp-smart-contracts/blob/32ad32f942888398bf99b039e98e238c3146c1b3/contracts/LSP5ReceivedAssets/LSP5Utils.sol#L117-L137)
```solidity
function generateSentAssetKeys(...) ... {
    ...
    uint128 oldArrayLength = uint128(bytes16(lsp5ReceivedAssetsCountValue));

    // Updating the number of the received assets (decrementing by 1
    uint128 newArrayLength = oldArrayLength - 1;  // could revert (only when data is already corrupt)
    ...
}
```

## Tools Used

## Recommended Mitigation Steps
Consider an extra check, for example in the following way:
```diff
function generateSentAssetKeys(...) ... {
    ...
    uint128 oldArrayLength = uint128(bytes16(lsp5ReceivedAssetsCountValue));
+   if (oldArrayLength == 0) revert Invalidlsp5ReceivedAssetsCountValue();
    // Updating the number of the received assets (decrementing by 1
    uint128 newArrayLength = oldArrayLength - 1;  // could revert (only when data is already corrupt)
    ...
}
```


# 6 Comments of `renounceOwnership()` on risks is incomplete

## Impact
After a complete `renounceOwnership()` the `owner` will be 0. Not only direct calls that check the owner will fail, but also the checks via `_verifyCall()` will fail.
This is because `_verifyCall()` interacts with the `owner`, which no longer exists.

This means that all (write/execute) actions on the Universal Profile will fail.
Read only actions and previousely set allowances will keep working.
So `renounceOwnership()` will make the Universal Profile largely unuseable.
The warnings in the source do not show the entire extend of the issue, see [LSP0ERC725AccountCore.sol#L642-L655](https://github.com/lukso-network/lsp-smart-contracts/blob/v0.10.2/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L642-L655).

## Proof of Concept
[LSP0ERC725AccountCore.sol#L222-L257](https://github.com/lukso-network/lsp-smart-contracts/blob/v0.10.2/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L222-L257)
```solidity
function execute(...) ... {
    ...
    address _owner = owner();
    ...
    bool verifyAfter = LSP20CallVerification._verifyCall(_owner);
    bytes memory result = ERC725XCore._execute(...);
    ...
}
```
[LSP20CallVerification.sol#L23-L43](https://github.com/lukso-network/lsp-smart-contracts/blob/v0.10.2/contracts/LSP20CallVerification/LSP20CallVerification.sol#L23-L43)
```solidity
function _verifyCall(address logicVerifier) ... {
    (bool success, bytes memory returnedData) = logicVerifier.call(...);  // call to address 0, will succeed, but no data returned
    _validateCall(false, success, returnedData);   // will revert due to lack of data
    bytes4 magicValue = abi.decode(returnedData, (bytes4));
    if (bytes3(magicValue) != bytes3(ILSP20.lsp20VerifyCall.selector)) // otherwise it will revert here
            revert LSP20InvalidMagicValue(false, returnedData);
    ...
}
```

## Tools Used

## Recommended Mitigation Steps
Consider enhancing the comments to include the risks:
```diff
@custom:danger Leaves the contract without an owner. 
Once ownership of the contract has been renounced, any functions that are restricted to be called 
by the owner 
+or a controller
will be permanently inaccessible, making these functions not callable anymore and unusable.
```


# 7 Function `_verifyCall()` doesn't check for EOAs

## Impact
The function `_verifyCall()` calls a `logicVerifier` without checking if this is a contract or an EOA.
Whereas function `isValidSignature()` first checks if the `owner` is an EOA.

The risk is limited because the checks in `_verifyCall()` will revert.

## Proof of Concept
[LSP0ERC725AccountCore.sol#L222-L257](https://github.com/lukso-network/lsp-smart-contracts/blob/v0.10.2/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L222-L257)
```solidity
function _verifyCall(address logicVerifier) ... {
    (bool success, bytes memory returnedData) = logicVerifier.call(...);  // call EOA will succeed, but no data returned
    _validateCall(false, success, returnedData);   // will revert due to lack of data
    bytes4 magicValue = abi.decode(returnedData, (bytes4));
    if (bytes3(magicValue) != bytes3(ILSP20.lsp20VerifyCall.selector)) // otherwise it will revert here
            revert LSP20InvalidMagicValue(false, returnedData);
    ...
}
```
[LSP0ERC725AccountCore.sol#L734-L774](https://github.com/lukso-network/lsp-smart-contracts/blob/v0.10.2/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L734-L774)

```solidity
function isValidSignature(...) ... {
    address _owner = owner();
    // If owner is a contract
    if (_owner.code.length > 0) {
        (bool success, bytes memory result) = _owner.staticcall( ... );

    }
    // If owner is an EOA
    else {
        ...
    }
}
```
## Tools Used

## Recommended Mitigation Steps
Consider checking it the `logicVerifier` is an EOA in function `_verifyCall()`.


# 8 Missing permissions in `getPermissionName()`

## Impact
The function `getPermissionName()` retrieves the string equivalent of permissions. This is used for error messages.
However some permissions are missing.

## Proof of Concept
[LSP6Utils.sol#L226-L247](https://github.com/lukso-network/lsp-smart-contracts/blob/v0.10.2/contracts/LSP6KeyManager/LSP6Utils.sol#L226-L247)
```solidity
    function getPermissionName(bytes32 permission) internal pure returns (string memory errorMessage) {
        if (permission == _PERMISSION_CHANGEOWNER) return "TRANSFEROWNERSHIP";
        if (permission == _PERMISSION_EDITPERMISSIONS) return "EDITPERMISSIONS";
        if (permission == _PERMISSION_ADDCONTROLLER) return "ADDCONTROLLER";
        if (permission == _PERMISSION_ADDEXTENSIONS) return "ADDEXTENSIONS";
        if (permission == _PERMISSION_CHANGEEXTENSIONS)
            return "CHANGEEXTENSIONS";
        if (permission == _PERMISSION_ADDUNIVERSALRECEIVERDELEGATE)
            return "ADDUNIVERSALRECEIVERDELEGATE";
        if (permission == _PERMISSION_CHANGEUNIVERSALRECEIVERDELEGATE)
            return "CHANGEUNIVERSALRECEIVERDELEGATE";
        if (permission == _PERMISSION_REENTRANCY) return "REENTRANCY";
        if (permission == _PERMISSION_SETDATA) return "SETDATA";
        if (permission == _PERMISSION_CALL) return "CALL";
        if (permission == _PERMISSION_STATICCALL) return "STATICCALL";
        if (permission == _PERMISSION_DELEGATECALL) return "DELEGATECALL";
        if (permission == _PERMISSION_DEPLOY) return "DEPLOY";
        if (permission == _PERMISSION_TRANSFERVALUE) return "TRANSFERVALUE";
        if (permission == _PERMISSION_SIGN) return "SIGN";
    }
```

## Tools Used

## Recommended Mitigation Steps
Add the missing permissions:
```diff
function getPermissionNameComplete(bytes32 permission) internal pure returns (string memory errorMessage) {
    ...
+   if (permission == _PERMISSION_SUPER_TRANSFERVALUE) return "SUPER_TRANSFERVALUE";
+   if (permission == _PERMISSION_SUPER_TRANSFERVALUE) return "SUPER_TRANSFERVALUE";
+   if (permission == _PERMISSION_SUPER_CALL) return "SUPER_CALL";
+   if (permission == _PERMISSION_SUPER_STATICCALL) return "SUPER_STATICCALL";
+   if (permission == _PERMISSION_SUPER_DELEGATECALL) return "SUPER_DELEGATECALL";
+   if (permission == _PERMISSION_SUPER_SETDATA) return "SUPER_SETDATA";
+   if (permission == _PERMISSION_ENCRYPT) return "ENCRYPT";
+   if (permission == _PERMISSION_DECRYPT) return "DECRYPT";
}
```

# 9 Missing check in `setDataBatch()` of `LSP0ERC725AccountCore`

## Impact
Function  `setDataBatch()` of ERC725YCore has an extra check compared to the version in `LSP0ERC725AccountCore`.
The extra check verifies if `dataKeys.length == 0` and then reverts.
The risk of this is very low though, however it is not consistent.

Note: several other Batch functions don't check for array lenghts of 0 either.

## Proof of Concept
[ERC725YCore.sol#L73-L96](https://github.com/ERC725Alliance/ERC725/blob/develop/implementations/contracts/ERC725YCore.sol#L73-L96)
```solidity
    function setDataBatch(bytes32[] memory dataKeys, ...) ... {
        ...
        if (dataKeys.length == 0) {
            revert ERC725Y_DataKeysValuesEmptyArray();
        }
    }
```
[LSP0ERC725AccountCore.sol#L380-L424](https://github.com/lukso-network/lsp-smart-contracts/blob/develop/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L380-L424)
```solidity
    function setDataBatch(bytes32[] memory dataKeys, ...) ... {
        // no check on dataKeys.length == 0
    }
```

## Tools Used

## Recommended Mitigation Steps
Consider making the code consistent.

# 10 The `_pendingOwner`  can be reset via `_renounceOwnership()`

## Impact
Assume `transferOwnership()` has been called and the `_pendingOwner` has been set.
The function `_renounceOwnership()` can now be used to reset the `_pendingOwner`, which means that the next `acceptOwnership()` will fail. 
Note: After the timeout periond of  `_renounceOwnership()`, the original situation is restored, so `_renounceOwnership()` has to be done twice to renounce ownership.
Note: Calling `transferOwnership()` again will also update the `_pendingOwner`.

This might an unexpected situation.

## Proof of Concept
https://github.com/lukso-network/lsp-smart-contracts/blob/v0.10.2/contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol#L106C2-L179
```solidity
abstract contract LSP14Ownable2Step is ... {

    function _renounceOwnership() ...  {
        ...
        if (currentBlock > confirmationPeriodEnd || _renounceOwnershipStartedAt == 0 ) {
            _renounceOwnershipStartedAt = currentBlock;
            delete _pendingOwner;
            ...
            return;
        }
        ...
    }
}
```

## Tools Used

## Recommended Mitigation Steps
Doublecheck if this is the intended behaviour.
Consider to notify the `UniversalReceiver` of the fact that `OwnershipTransferStarted` is no longer relevant.

