# Incorrect Comments and NatSpec

### `LSP2Utils.generateMappingKey()`

The current description describes the key for a mapping to be the following:

```solidity=65
     // <bytes10(keccak256(firstWord))>:<bytes2(0)>:<bytes20(keccak256(firstWord))>
```

However, according to [LIP-2](https://github.com/lukso-network/LIPs/blob/main/LSPs/LSP-2-ERC725YJSONSchema.md#mapping) and the current code implementation, the `bytes20` should use a potentially different word, such as `lastWord`.

```solidity=78
        bytes memory temporaryBytes = bytes.concat(
            bytes10(firstWordHash),
            bytes2(0),
            bytes20(lastWordHash)
        );
```

-----

### `LSP2Utils.generateASSETURLValue()`

The current NatSpec of the function `generateASSETURLValue()` is as follows:

```solidity=210
    /**
     * @dev Generate a ASSETURL valueContent
     * @param hashFunction The function used to hash the JSON file
     * @param assetBytes Bytes value of the JSON file
     * @param url The URL where the JSON file is hosted
     */
```

The above comments are meant for a JSON file when `generateASSETURLValue()` can be used for an arbitrary asset.

-----

### `LSP6Utils.isCompactBytesArrayOfAllowedCalls()`

The NatSpec of `isCompactBytesArrayOfAllowedCalls()` states that the function checks if the input `allowedCallsCompacted` is a compact bytes array where each element is 28 bytes.

```solidity=85
    /**
     * @dev same as LSP2Utils.isCompactBytesArray with the additional requirement that each element must be 28 bytes long.
     *
     * @param allowedCallsCompacted a compact bytes array of tuples (bytes4,address,bytes4) to check. //@audit incorrect comment. should be 32 (bytes4,address,bytes4,bytes4)
     * @return true if the value passed is a valid compact bytes array of bytes28 elements according to LSP2, false otherwise.
     */
```

However, this is inconsistent with [LIP-6](https://github.com/lukso-network/LIPs/blob/main/LSPs/LSP-6-KeyManager.md#addresspermissionsallowedcallsaddress), where each element of the compact bytes array is 32 bytes, of type `(bytes4,address,bytes4,bytes4)`.

This also differs from the current implementation, where the function actually checks that each element has length 32.

```solidity=107
            if (elementLength != 32) return false;
```

# Possible issue on Validating Execution Calls For A Fallback Function

1. In `_extractExecuteParameters()`, if there are not enough bytes for a selector, the selector is chosen to be `bytes4(0)`, suggesting both `receive()` and `fallback()` should have `bytes4(0)` as the selector, but [LIP-6](https://github.com/lukso-network/LIPs/blob/main/LSPs/LSP-6-KeyManager.md) does not mention an agreed upon selector.

2. In the case where call is not sending any value and is an empty call, `_extractCallType()` will return `bytes4(0)`, which is not a supported call type. Is this intentional or should `_ALLOWEDCALLS_CALL` be the required call type in this situation?