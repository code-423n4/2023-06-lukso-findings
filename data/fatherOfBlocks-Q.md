**ERC725InitAbstract.sol**
- L5 - imports of contract that is not used are made, apart from generating an extra amount of gas in the deploy, it also generates little readability in the code when viewing the code in the blockchain explorer.


**LSP0Constants.sol**
- L15/18/21 - 3 constants are created, which include words in lower case within their name, this does not follow the standard of a constant, for example _TYPEID_LSP0_OwnershipTransferStarted, it should be written like this: "_TYPEID_LSP0_OWNERSHIP_TRANSFER_STARTED".


**LSP14Constants.sol**
- L9/12/15 - 3 constants are created, which include words in lower case within their name, this does not follow the standard of a constant, for example _TYPEID_LSP14_OwnershipTransferStarted, it should be written like this: "_TYPEID_LSP14_OWNERSHIP_TRANSFER_STARTED".


**LSP20CallVerification.sol**
- L56 - abi.encodePacked() should not be used with dynamic types when passing the result to a hash function such as keccak256()
Use abi.encode() instead which will pad items to 32 bytes, which will prevent hash collisions (e.g. abi.encodePacked(0x123,0x456) => 0x123456 => abi.encodePacked(0x1,0x23456), but abi.encode(0x123,0x456) => 0x0...1230...456). “Unless there is a compelling reason, abi.encode should be preferred”. If there is only one argument to abi.encodePacked() it can often be cast to bytes() or bytes32() instead.
If all arguments are strings and or bytes, bytes.concat() should be used instead.


**LSP10Utils.sol**
- L76/125 - The number 16 is used, without giving any meaning to that number. There is an explanation in the comments, but it should be in the code defined with a name.


**LSP6SetDataModule.sol**
- L679/705/715 - The number 32 is used twice, without any explanation of what the meaning of this number is, for a better understanding of the code it would be better to name it with a local variable to this number.


**LSP6ExecuteModule.sol**
- L272/284/289/295/374 - The number 34 is used twice, without any explanation of what the meaning of this number is, for a better understanding of the code it would be better to name it with a local variable to this number.
The same thing happens with number 32, which although it has more explanation in the documentation, should have a variable with a defined name.
