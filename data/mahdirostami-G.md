# check if first
by checking conditions first, avoid extra computation
[https://github.com/code-423n4/2023-06-lukso/blob/9dbc96410b3052fc0fd9d423249d1fa42958cae8/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L369-L397](https://github.com/code-423n4/2023-06-lukso/blob/9dbc96410b3052fc0fd9d423249d1fa42958cae8/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L369-L397)
```diff
-        bool isSetData = false;
-        if (
-            bytes4(payload) == IERC725Y.setData.selector ||
-            bytes4(payload) == IERC725Y.setDataBatch.selector
-        ) {
-            isSetData = true;
-        }
-
-        bool isReentrantCall = _nonReentrantBefore(isSetData, signer);
-
        if (!_isValidNonce(signer, nonce)) { 
             revert InvalidRelayNonce(signer, nonce, signature);
         }
 
-        // increase nonce after successful verification
-        _nonceStore[signer][nonce >> 128]++;
-
+        if (validityTimestamps != 0) { 
            uint128 startingTimestamp = uint128(validityTimestamps >> 128);
            uint128 endingTimestamp = uint128(validityTimestamps);

            // solhint-disable not-rely-on-time
            if (block.timestamp < startingTimestamp) {
                revert RelayCallBeforeStartTime();
            }
            if (block.timestamp > endingTimestamp) {
                revert RelayCallExpired();
            }
        }

+        // increase nonce after successful verification
+        _nonceStore[signer][nonce >> 128]++;
+
+        bool isSetData = false;
+        if (
+            bytes4(payload) == IERC725Y.setData.selector ||
+            bytes4(payload) == IERC725Y.setDataBatch.selector
+        ) {
+            isSetData = true;
+        }
+
+        bool isReentrantCall = _nonReentrantBefore(isSetData, signer); //if not setdata true if is setdata false
```
2. check ```if (notifier == tx.origin)``` at top of the function
[https://github.com/code-423n4/2023-06-lukso/blob/9dbc96410b3052fc0fd9d423249d1fa42958cae8/contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol#L103](https://github.com/code-423n4/2023-06-lukso/blob/9dbc96410b3052fc0fd9d423249d1fa42958cae8/contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol#L103)

# avoid extra computation 
here this if check is unnecessary because newArrayLength is ```uint128 newArrayLength = oldArrayLength - 1;```
[https://github.com/code-423n4/2023-06-lukso/blob/9dbc96410b3052fc0fd9d423249d1fa42958cae8/contracts/LSP5ReceivedAssets/LSP5Utils.sol#L190-L191](https://github.com/code-423n4/2023-06-lukso/blob/9dbc96410b3052fc0fd9d423249d1fa42958cae8/contracts/LSP5ReceivedAssets/LSP5Utils.sol#L190-L191)
```solidity
        uint128 newArrayLength = oldArrayLength - 1;
		
		
            if (newArrayLength >= type(uint128).max) {
                revert ReceivedAssetsIndexSuperiorToUint128(newArrayLength);
```
# avoid extra code with better implementation
functions call during for loop, it's better to check if first and then call.
2 instances:
1.[https://github.com/code-423n4/2023-06-lukso/blob/9dbc96410b3052fc0fd9d423249d1fa42958cae8/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L152-L179](https://github.com/code-423n4/2023-06-lukso/blob/9dbc96410b3052fc0fd9d423249d1fa42958cae8/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L152-L179)

2. [https://github.com/code-423n4/2023-06-lukso/blob/9dbc96410b3052fc0fd9d423249d1fa42958cae8/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L204-L245](https://github.com/code-423n4/2023-06-lukso/blob/9dbc96410b3052fc0fd9d423249d1fa42958cae8/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L204-L245)
```
-        bytes[] memory results = new bytes[](payloads.length);
         uint256 totalValues;

         for (uint256 ii = 0; ii < payloads.length; ) { //@audit gas
             if ((totalValues += values[ii]) > msg.value) {
                 revert LSP6BatchInsufficientValueSent(totalValues, msg.value);
            unchecked {
            ++ii;
             }

            }
        }
+        if (totalValues < msg.value) {
+            revert LSP6BatchExcessiveValueSent(totalValues, msg.value);
        }
+        bytes[] memory results = new bytes[](payloads.length);
+        for (uint256 ii = 0; ii < payloads.length; ) {
             results[ii] = _executeRelayCall(
                 signatures[ii],
                 nonces[ii],
				 
                 ++ii;
             }
         }

-        if (totalValues < msg.value) {
-            revert LSP6BatchExcessiveValueSent(totalValues, msg.value);
-        }
         return results;
     }
```

# cast array length if it used more times
here payloads.length used 3 times 
```
        if (values.length != payloads.length) {
```
1.[https://github.com/code-423n4/2023-06-lukso/blob/9dbc96410b3052fc0fd9d423249d1fa42958cae8/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L156](https://github.com/code-423n4/2023-06-lukso/blob/9dbc96410b3052fc0fd9d423249d1fa42958cae8/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L156)
2.[https://github.com/code-423n4/2023-06-lukso/blob/9dbc96410b3052fc0fd9d423249d1fa42958cae8/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L215](https://github.com/code-423n4/2023-06-lukso/blob/9dbc96410b3052fc0fd9d423249d1fa42958cae8/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L215)

# Use unit16 instead of uint256
use unit16 for RENOUNCE_OWNERSHIP_CONFIRMATION_DELAY & RENOUNCE_OWNERSHIP_CONFIRMATION_PERIOD
and add _pendingOwner address above them, which use 1 slot storage totally
```diff
+    /**
+     * @dev see `pendingOwner()`
+     */
+    address private _pendingOwner;
+
     /**
      * @dev The number of block that MUST pass before one is able to
      *  confirm renouncing ownership
      */
-    uint256 public constant RENOUNCE_OWNERSHIP_CONFIRMATION_DELAY = 200;
+    uint16 public constant RENOUNCE_OWNERSHIP_CONFIRMATION_DELAY = 200; //@audit gas use uint8
 
     /**
      * @dev The number of blocks during which one can renounce ownership
      */
-    uint256 public constant RENOUNCE_OWNERSHIP_CONFIRMATION_PERIOD = 200;
+    uint16 public constant RENOUNCE_OWNERSHIP_CONFIRMATION_PERIOD = 200; //@audit gas use uint8

```