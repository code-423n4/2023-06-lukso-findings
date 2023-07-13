# Functions Mutating Storage Should Emit Events

## Description
Functions mutating storage should emit an Event for easy off-chain monitoring.
When accepting transfer of ownership, an event is not issued.

Reference: https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol#L87-L101
```
    function acceptOwnership() public virtual {
        address previousOwner = owner();

        _acceptOwnership();

        previousOwner.tryNotifyUniversalReceiver(
            _TYPEID_LSP14_OwnershipTransferred_SenderNotification,
            ""
        );

        msg.sender.tryNotifyUniversalReceiver(
            _TYPEID_LSP14_OwnershipTransferred_RecipientNotification,
            ""
        );
    }
```
