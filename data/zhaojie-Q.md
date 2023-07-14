
`returns (bool verifyAfter)` Variable `verifyAfter` not used.

https://github.com/code-423n4/2023-06-lukso/blob/9dbc96410b3052fc0fd9d423249d1fa42958cae8/contracts/LSP20CallVerification/LSP20CallVerification.sol#L23
 
```
   function _verifyCall(
        address logicVerifier
    ) internal virtual returns (bool verifyAfter) {
        (bool success, bytes memory returnedData) = logicVerifier.call(
            abi.encodeWithSelector(
                ILSP20.lsp20VerifyCall.selector,
                msg.sender,
                msg.value,
                msg.data
            )
        );

        _validateCall(false, success, returnedData);

        bytes4 magicValue = abi.decode(returnedData, (bytes4));

        if (bytes3(magicValue) != bytes3(ILSP20.lsp20VerifyCall.selector))
            revert LSP20InvalidMagicValue(false, returnedData);

        return bytes1(magicValue[3]) == 0x01 ? true : false;
    }
```