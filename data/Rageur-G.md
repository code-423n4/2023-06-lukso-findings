## GAS-1: <X> * 2 costs more gas then <X> << 1

### Description

Using bitwize operator instead of '*' or '/' saves gas.

### Affected file

* LSP2Utils.sol (Line: 242)

## GAS-2: <X> <= <Y> costs more gas than <X> < <Y> + 1

### Description

In Solididy, the opcode 'less or equal' doesn't exist. So the EVM will translate this by two distinct operation, first the inferior, and then the equal which cost more gas then a strict less.

### Affected file

* LSP0ERC725AccountCore.sol (Line: 474, 506)
* LSP2Utils.sol (Line: 36, 335)
* LSP5Utils.sol (Line: 190)
* LSP6ExecuteModule.sol (Line: 353)
* LSP6Utils.sol (Line: 97, 126)

## GAS-3: Caching global variables is more expensive than using the actual variable

### Description

 It’s cheaper to use global variable as compared to caching.

### Affected file

* LSP14Ownable2Step.sol (Line: 151)
* LSP7DigitalAssetCore.sol (Line: 133, 304, 352, 415)
* LSP8CompatibleERC721.sol (Line: 233)
* LSP8CompatibleERC721InitAbstract.sol (Line: 233)
* LSP8IdentifiableDigitalAssetCore.sol (Line: 198, 327, 361, 414)

## GAS-4: Empty blocks should be removed or emit something

### Description

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting.

### Affected file

* LSP7CompatibleERC20.sol (Line: 32)
* LSP7CompatibleERC20Mintable.sol (Line: 7)
* LSP7DigitalAssetCore.sol (Line: 442)
* LSP7Mintable.sol (Line: 24)
* LSP8CompatibleERC721.sol (Line: 67)
* LSP8CompatibleERC721Mintable.sol (Line: 7)
* LSP8IdentifiableDigitalAsset.sol (Line: 36)
* LSP8IdentifiableDigitalAssetCore.sol (Line: 444)
* LSP8Mintable.sol (Line: 23)

## GAS-5: Internal functions not called by the contract should be removed to save deployment gas

### Description

If the functions are required by an interface, the contract should inherit from that interface and use the override keyword.

### Affected file

* LSP0ERC725AccountInitAbstract.sol (Line: 59)
* LSP17Extendable.sol (Line: 44, 86)
* LSP17Extension.sol (Line: 32, 44, 54)
* LSP20CallVerification.sol (Line: 23, 49)
* LSP6ExecuteModule.sol (Line: 61)
* LSP6KeyManagerCore.sol (Line: 518)
* LSP6KeyManagerInitAbstract.sol (Line: 20)
* LSP6OwnershipModule.sol (Line: 14)
* LSP6SetDataModule.sol (Line: 62, 116)
* LSP7CappedSupplyInitAbstract.sol (Line: 21)
* LSP7DigitalAssetCore.sol (Line: 294, 338)
* LSP8CappedSupplyInitAbstract.sol (Line: 23)
* LSP8IdentifiableDigitalAssetCore.sol (Line: 313, 359)

## GAS-6: Internal functions only called once can be inlined to save gas

### Description

Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

### Affected file

* LSP0ERC725AccountCore.sol (Line: 796, 851)
* LSP14Ownable2Step.sol (Line: 126, 136, 150)
* LSP1UniversalReceiverDelegateUP.sol (Line: 152, 201)
* LSP20CallVerification.sol (Line: 84)
* LSP4DigitalAssetMetadataInitAbstract.sol (Line: 27)
* LSP6ExecuteModule.sol (Line: 153, 170, 188, 317, 339, 366, 382, 399, 418)
* LSP6KeyManagerCore.sol (Line: 441)
* LSP6SetDataModule.sol (Line: 334, 385, 406, 438, 474, 490, 507, 622)
* LSP7CappedSupply.sol (Line: 53)
* LSP7CappedSupplyInitAbstract.sol (Line: 53)
* LSP7CompatibleERC20.sol (Line: 107, 116, 127, 137, 146)
* LSP7CompatibleERC20InitAbstract.sol (Line: 33, 115, 124, 135, 145, 154)
* LSP7CompatibleERC20MintableInitAbstract.sol (Line: 15)
* LSP7DigitalAssetCore.sol (Line: 399)
* LSP7DigitalAssetInitAbstract.sol (Line: 28)
* LSP7MintableInitAbstract.sol (Line: 20)
* LSP8CappedSupply.sol (Line: 56)
* LSP8CappedSupplyInitAbstract.sol (Line: 56)
* LSP8CompatibleERC721.sol (Line: 259, 269, 284, 341)
* LSP8CompatibleERC721InitAbstract.sol (Line: 61, 259, 269, 284, 341)
* LSP8CompatibleERC721MintableInitAbstract.sol (Line: 15)
* LSP8Enumerable.sol (Line: 31)
* LSP8EnumerableInitAbstract.sol (Line: 33)
* LSP8IdentifiableDigitalAssetCore.sol (Line: 394)
* LSP8IdentifiableDigitalAssetInitAbstract.sol (Line: 30)
* LSP8MintableInitAbstract.sol (Line: 19)
* UniversalProfileInitAbstract.sol (Line: 27)

## GAS-7: Public functions not called by the contract should be declared external instead

### Description

Contracts are allowed to override their parents’ functions and change the visibility from external to public and can save gas by doing so. 

### Affected file

* LSP0ERC725AccountCore.sol (Line: 171, 380, 460, 734)
* LSP1UniversalReceiverDelegateUP.sol (Line: 78)
* LSP4Compatibility.sol (Line: 25, 34)
* LSP6KeyManagerCore.sol (Line: 87, 107, 118, 152, 204)
* LSP7CompatibleERC20.sol (Line: 56, 66, 79)
* LSP7CompatibleERC20InitAbstract.sol (Line: 64, 74, 87)
* LSP7DigitalAssetCore.sol (Line: 54, 61, 70, 91, 101, 154, 203, 232)
* LSP8CompatibleERC721.sol (Line: 94, 113, 120, 155, 172, 185, 197)
* LSP8CompatibleERC721InitAbstract.sol (Line: 94, 113, 120, 155, 172, 185, 197)
* LSP8Enumerable.sol (Line: 27)
* LSP8EnumerableInitAbstract.sol (Line: 29)
* LSP8IdentifiableDigitalAssetCore.sol (Line: 61, 70, 94, 105, 153, 165, 210)

## GAS-8: Storage pointer to a structure is cheaper than copying each value of the structure into memory, same for array and mapping

### Description

It may not be obvious, but every time you copy a storage struct/array/mapping to a memory variable, you are literally copying each member by reading it from storage, which is expensive. And when you use the storage keyword, you are just storing a pointer to the storage, which is much cheaper.

### Affected file

* LSP0ERC725AccountCore.sol (Line: 176, 244, 308, 469, 472, 503, 504, 742)
* LSP10Utils.sol (Line: 72, 121)
* LSP1UniversalReceiverDelegateUP.sol (Line: 112, 158, 159, 207, 208)
* LSP1Utils.sol (Line: 47, 57)
* LSP20CallVerification.sol (Line: 26, 53)
* LSP2Utils.sol (Line: 35, 56, 78, 100, 119, 144, 166, 185)
* LSP4Compatibility.sol (Line: 26, 35)
* LSP5Utils.sol (Line: 74, 123, 216, 219)
* LSP6ExecuteModule.sol (Line: 259, 289, 445)
* LSP6KeyManagerCore.sol (Line: 160, 220, 336, 356, 402, 420, 424, 470, 486)
* LSP6OwnershipModule.sol (Line: 24)
* LSP6SetDataModule.sol (Line: 128, 788)
* LSP6Utils.sol (Line: 29, 155)
* LSP7DigitalAssetCore.sol (Line: 322, 377, 424)
* LSP8CompatibleERC721.sol (Line: 97, 102, 126, 323)
* LSP8CompatibleERC721InitAbstract.sol (Line: 97, 102, 126, 323)
* LSP8IdentifiableDigitalAssetCore.sol (Line: 346, 375, 426)

## GAS-9: Usage of uint/int smaller than 32 bytes (256 bits) incurs overhead

### Description

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.
Each operation involving a uint8 costs an extra 22-28 gas (depending on whether the other operand is also a variable of type uint8) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so. Use a larger size then downcast where needed.

### Affected file

* LSP10Utils.sol (Line: 83, 89, 115, 133, 136, 140)
* LSP1UniversalReceiverDelegateUP.sol (Line: 139, 139, 139, 201)
* LSP2Utils.sol (Line: 52, 296, 336)
* LSP5Utils.sol (Line: 85, 117, 134, 137, 191)
* LSP6ExecuteModule.sol (Line: 378)
* LSP6KeyManagerCore.sol (Line: 107, 387, 388, 445)
* LSP6SetDataModule.sol (Line: 346, 350, 544, 664)
* LSP6Utils.sol (Line: 98, 128, 202, 205)
* LSP7DigitalAssetCore.sol (Line: 54)

## GAS-10: Use ```assembly``` to write address storage values

### Affected file

* LSP14Ownable2Step.sol (Line: 129, 163)
* LSP6KeyManagerCore.sol (Line: 519, 541, 553)
* LSP7CappedSupply.sol (Line: 28)
* LSP7CappedSupplyInitAbstract.sol (Line: 28)
* LSP8CappedSupply.sol (Line: 30)
* LSP8CappedSupplyInitAbstract.sol (Line: 30)
* LSP8IdentifiableDigitalAssetCore.sol (Line: 268)

## GAS-11: Use constants instead of type(uintx).max

### Description

It uses more gas in the distribution process and also for each transaction than constant usage.

### Affected file

* LSP10Utils.sol (Line: 85, 135)
* LSP5Utils.sol (Line: 87, 190)
* LSP6ExecuteModule.sol (Line: 296, 378, 395, 414)

## GAS-12: Use hardcoded address instead address(this)

### Description

Instead of using ```address(this)```, it is more gas-efficient to pre-calculate and use the hardcoded ```address```

### Affected file

* LSP0ERC725AccountCore.sol (Line: 176)
* LSP14Ownable2Step.sol (Line: 127)
* LSP6ExecuteModule.sol (Line: 86)
* LSP6KeyManagerCore.sol (Line: 365)

## GAS-13: Use nested if and avoid multiple check combinations

### Description

Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

### Affected file

* LSP0ERC725AccountCore.sol (Line: 801)
* LSP10Utils.sol (Line: 76)
* LSP1UniversalReceiverDelegateUP.sol (Line: 227, 245)
* LSP5Utils.sol (Line: 78)
* LSP6ExecuteModule.sol (Line: 165, 214, 224, 228, 233, 236, 240, 301)
* LSP6KeyManagerCore.sol (Line: 338, 404)
* LSP6SetDataModule.sol (Line: 362)
* LSP8CompatibleERC721.sol (Line: 235)
* LSP8CompatibleERC721InitAbstract.sol (Line: 235)

## GAS-14: Using > 0 costs more gas than != 0 when used on a uint in a require() statement

### Description

When dealing with unsigned integer types, comparisons with != 0 are cheaper then with > 0. This change saves 6 gas per instance.

### Affected file

* LSP0ERC725AccountCore.sol (Line: 182, 741)
* LSP1UniversalReceiverDelegateUP.sol (Line: 165)
* LSP20CallVerification.sol (Line: 86)
* LSP7DigitalAssetCore.sol (Line: 491)
* LSP8CompatibleERC721.sol (Line: 313)
* LSP8CompatibleERC721InitAbstract.sol (Line: 313)
* LSP8IdentifiableDigitalAssetCore.sol (Line: 493)

## GAS-15: Variable initialized with their default value

### Description

Uninitialized variables are assigned with the types default value. Explicitly initializing a variable with it’s default value costs unnecesary gas.

### Affected file

* LSP0ERC725AccountCore.sol (Line: 396, 411)
* LSP2Utils.sol (Line: 261, 292, 328)
* LSP6ExecuteModule.sol (Line: 124, 194, 272)
* LSP6KeyManagerCore.sol (Line: 163, 223, 256, 323, 369)
* LSP6SetDataModule.sol (Line: 127, 129, 133)
* LSP6Utils.sol (Line: 94, 123, 172, 173)
* LSP7DigitalAssetCore.sol (Line: 171)
* LSP8IdentifiableDigitalAssetCore.sol (Line: 227, 273)

## GAS-16: With assembly, ```.call (bool success)``` transfer can be done gas-optimized

### Description

```return``` data ```(bool success,)``` has to be stored due to EVM architecture, but in a usage in assembly, 'out' and 'outsize' values are given (0,0), this storage disappears and gas optimization is provided.

### Affected file

* LSP0ERC725AccountCore.sol (Line: 176, 742)
* LSP1Utils.sol (Line: 57)
* LSP20CallVerification.sol (Line: 26, 53)
* LSP6KeyManagerCore.sol (Line: 420)

## GAS-17: abi.encode() is less efficient than abi.encodePacked()

### Description

Use abi.encodePacked() where possible to save gas.

### Affected file

* LSP0ERC725AccountCore.sol (Line: 253, 319, 528)