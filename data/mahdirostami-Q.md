# 1. Consider all possible values in LSP6ExecuteModule - _verifyCanExecute
_verifyCanExecute must revert if operationType isn't operation0,1,2,3,4.
MITIGATION
add else: revert
[https://github.com/code-423n4/2023-06-lukso/blob/9dbc96410b3052fc0fd9d423249d1fa42958cae8/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L61-L151](https://github.com/code-423n4/2023-06-lukso/blob/9dbc96410b3052fc0fd9d423249d1fa42958cae8/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L61-L151)

# 2. In LSP2Utils there is no function for generateMappingWithGroupingKey for three words
in documentation, it's mentioned but it isn't implemented
solidity example in documentation
```solidity
bytes32 dataKey = bytes32(
    bytes.concat(
        bytes6(keccak256(bytes(firstWord))),
        bytes4(keccak256(bytes(secondWord))),
        bytes2(0),
        bytes20(keccak256(bytes(thirdWord)))
    )
);
```
end of the page in [https://docs.lukso.tech/standards/generic-standards/lsp2-json-schema/#mappingwithgrouping](https://docs.lukso.tech/standards/generic-standards/lsp2-json-schema/#mappingwithgrouping)

# 3. Check that new owner has targeted this account
Whenever we want to transferring ownership to LSP6 it's better to first check that LSP6 target is our contract.

# 4. Use important data in the event
in RenounceOwnershipStarted(), it's better to use currentBlock.
[https://github.com/code-423n4/2023-06-lukso/blob/9dbc96410b3052fc0fd9d423249d1fa42958cae8/contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol#L165](https://github.com/code-423n4/2023-06-lukso/blob/9dbc96410b3052fc0fd9d423249d1fa42958cae8/contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol#L165)

# 5. Notify the current owner like the new owner with "OwnershipTransferStarted"
in transferOwnership in LSP14Ownable2Step.sol it just notifies newOwner, it's better to notify current owner as well.


# 6. Like other LSPs use different file for errors In LSP10.
In each LSP if it has error functions we have a different file for errors instead of LSP10.

# 7. Use del instead of delete
You can use the delete keyword instead of setting the variable as zero.
[https://github.com/code-423n4/2023-06-lukso/blob/9dbc96410b3052fc0fd9d423249d1fa42958cae8/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L518-L520](https://github.com/code-423n4/2023-06-lukso/blob/9dbc96410b3052fc0fd9d423249d1fa42958cae8/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L518-L520)
```diff
+        delete _reentrancyStatus; 
-        _reentrancyStatus = false;
     }
```
