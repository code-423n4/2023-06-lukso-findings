# QA Report

## Summary

### Low

| id | title |
| --- | --- |
| [L-01](#l-01-bytes-memory-data-is-used-but-msgdata-is-validated) | `bytes memory data` is used but `msg.data` is validated     |

### Suggestions

| id | title |
| --- | --- |
| [S-01](#s-01-consider-adding-the-possiblity-to-configure-how-much-value-you-can-transfer) | consider adding the possiblity to configure how much value you can transfer |
| [S-02](#s-02-create2-lets-you-create-any-contract) | `create`/`2` lets you create any contract |

### Non critical/Informational

| id | title |
| --- | --- |
| [NC-01](#nc-01-incorrect-comment) | incorrect comment |
| [NC-02](#nc-02-bit-mask-description-is-off) | bit mask description is off |
| [NC-03](#nc-03-wrong-pointers-in-allowederc725Ysinglekey-comment) | wrong pointers in `AllowedERC725YSingleKey` comment |
| [NC-04](#nc-04-typo) | typo |

## Low

### L-01 `bytes memory data` is used but `msg.data` is validated

In `LSP0ERC725AccountCore` when calling any of the `setData/execute` (and batch alternatives) the call can be sent for validation using `LSP20CallVerification`:

Lets use `setData` as an example (the same goes for `execute`):

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L339-L356
```solidity
File: LSP0ERC725Account/LSP0ERC725AccountCore.sol

339:    function setData(
340:        bytes32 dataKey,
341:        bytes memory dataValue // <--- data passed here
342:    ) public payable virtual override {
...
354:        // If the caller is not the owner, call {lsp20VerifyCall} on the owner
355:        // Depending on the magicValue returned, a second call is done after setting data
356:        bool verifyAfter = _verifyCall(_owner);
```

`_verifyCall` goes to `LSP20CallVerification`:

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP20CallVerification/LSP20CallVerification.sol#L23-L33
```solidity
File: LSP20CallVerification/LSP20CallVerification.sol

23:    function _verifyCall(
24:        address logicVerifier
25:    ) internal virtual returns (bool verifyAfter) {
26:        (bool success, bytes memory returnedData) = logicVerifier.call(
27:            abi.encodeWithSelector(
28:                ILSP20.lsp20VerifyCall.selector,
29:                msg.sender,
30:                msg.value,
31:                msg.data // <--- msg.data used here
32:            )
33:        );
```

The issue is that `setData` takes `bytes memory dataValue`, while, what is being passed to be validated is `msg.data`. `msg.data` could potentially have more bytes at the end. However, using `LSP6KeyManager` I could find no way to misuse this other than some minor data corruption (in `AllowedERC725YDataKeys`).

#### Recommendation
Send what is being passed as parameters to be validated, not msg.data.


## Suggestions

### S-01 consider adding the possibility to configure how much value you can transfer

Right now you can only allow to send either noting or everything. There could be use cases for only allowing a user to send a certain amount on behalf of the account.

### S-02 `create`/`2` lets you create any contract

Same here, right now, if you allow a user to create contracts, they can create any contract. There could be use cases where a user should only be permitted to create a certain specific contract, say a proxy.

## Non critical/Informational

### NC-01 incorrect comment

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Utils.sol#L136-L137
```solidity
File: LSP6KeyManager/LSP6Utils.sol

136:            // the length of the allowed data key must be not under 33 bytes and not 0
137:            if (elementLength == 0 || elementLength > 32) return false;
```
The comment should be `must be under 33 bytes and not 0` or `must be not above 33 bytes and not 0`

### NC-02 bit mask description is off

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L577-L579
```solidity
File: LSP6Modules/LSP6SetDataModule.sol
577:             *             &                              discard this part
578:             *                       vvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvv
579:             *           mask = 0xffffff0000000000000000000000000000000000000000000000000000000000
```

the description should move by three columns/bytes:
```diff
             *             &                              discard this part
-            *                       vvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvv
+            *                          vvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvv
             *           mask = 0xffffff0000000000000000000000000000000000000000000000000000000000
```

Same at:

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L697-L699

### NC-03 wrong pointers in `AllowedERC725YSingleKey` comment

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L519-L523
```solidity
File: LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol

519:         * 0003 a00000
520:         * 0005 fff83a0011
521:         * 0020 aa0000000000000000000000000000000000000000000000000000000000cafe
522:         * 0012 bb000000000000000000000000000000beef
523:         * 0019 cc00000000000000000000000000000000000000000000deed
```

The first two, `0003 a00000` and `0005 fff83a0011` are correct but the next three, the data fields are too long. Should be:

```solidity
521:         * 0020 aa0000000000000000000000000000000000cafe
522:         * 0012 bb000000000000000000beef
523:         * 0019 cc00000000000000000000000000000000deed
```

Same applies to second comment as well:
https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L641-L643

### NC-04 typo

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP5ReceivedAssets/LSP5Utils.sol#L136
```solidity
File: LSP5ReceivedAssets/LSP5Utils.sol

134:        // Updating the number of the received assets (decrementing by 1
```