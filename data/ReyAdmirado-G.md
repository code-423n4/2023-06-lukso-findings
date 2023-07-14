| | issue |
| ----------- | ----------- |
| 1 | [Add `unchecked {}` for subtractions where the operands cannot underflow because of a previous `require()` or `if` statement](#1-add-unchecked--for-subtractions-where-the-operands-cannot-underflow-because-of-a-previous-require-or-if-statement) |
| 2 | [can make the variable outside the loop to save gas](#2-can-make-the-variable-outside-the-loop-to-save-gas) |
| 3 | [it costs more gas to initialize non-constant/non-immutable variables to zero than to let the default of zero be applied](#3-it-costs-more-gas-to-initialize-non-constantnon-immutable-variables-to-zero-than-to-let-the-default-of-zero-be-applied) |
| 4 | [using > 0 costs more gas than != 0 when used on a uint in a require() statement](#4-using--0-costs-more-gas-than--0-when-used-on-a-uint-in-a-require-statement) |
| 5 | [using `calldata` instead of `memory` for read-only arguments in external functions saves gas](#5-using-calldata-instead-of-memory-for-read-only-arguments-in-external-functions-saves-gas) |
| 6 | [internal functions only called once can be inlined to save gas](#6-internal-functions-only-called-once-can-be-inlined-to-save-gas) |
| 7 | [abi.encode() is less efficient than abi.encodepacked()](#7-abiencode-is-less-efficient-than-abiencodepacked) |
| 8 | [usage of uint/int smaller than 32 bytes (256 bits) incurs overhead](#8-usage-of-uintint-smaller-than-32-bytes-256-bits-incurs-overhead) |
| 9 | [avoid an unnecessary sstore by not writing a default value for bools](#9-avoid-an-unnecessary-sstore-by-not-writing-a-default-value-for-bools) |
| 10 | [Use assembly to check for address(0)](#10-use-assembly-to-check-for-address0) |
| 11 | [use existing cached version of state var](#11-use-existing-cached-version-of-state-var) |
| 12 | [>= costs less gas than >](#12--costs-less-gas-than) |
| 13 | [Use hardcoded address instead address(this)](#13-use-hardcoded-address-instead-addressthis) |
| 14 | [The use of a logical AND in place of double if is slightly less gas efficient in instances where there isn't a corresponding else statement for the given if statement](#14-the-use-of-a-logical-and-in-place-of-double-if-is-slightly-less-gas-efficient-in-instances-where-there-isnt-a-corresponding-else-statement-for-the-given-if-statement) |
| 15 | [Avoid updating storage when the value hasn't changed](#15-avoid-updating-storage-when-the-value-hasnt-changed) |


## 1. Add `unchecked {}` for subtractions where the operands cannot underflow because of a previous `require()` or `if` statement

require(a <= b); x = b - a => require(a <= b); unchecked { x = b - a }
if(a <= b); x = b - a => if(a <= b); unchecked { x = b - a }
this will stop the check for overflow and underflow so it will save gas

- [LSP8CompatibleERC721.sol#L138](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol#L138)

- [LSP6SetDataModule.sol#L585](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L585)
- [LSP6SetDataModule.sol#L705](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L705)

- [LSP7DigitalAssetCore.sol#L145](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L145)
- [LSP7DigitalAssetCore.sol#L245](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L245)


## 2. can make the variable outside the loop to save gas

make the variable outside the loop and only give the value to variable inside

- [LSP0ERC725AccountCore.sol#L176](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L176)

- [LSP8IdentifiableDigitalAssetCore.sol#L275](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L275)

- [LSP6ExecuteModule.sol#L289](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L289)


## 3. it costs more gas to initialize non-constant/non-immutable variables to zero than to let the default of zero be applied

- [LSP0ERC725AccountCore.sol#L396](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L396)
- [LSP0ERC725AccountCore.sol#L411](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L411)

- [LSP8IdentifiableDigitalAssetCore.sol#L227](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L227)
- [LSP8IdentifiableDigitalAssetCore.sol#L273](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L273)

- [LSP6KeyManagerCore.sol#L163](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L163)
- [LSP6KeyManagerCore.sol#L223](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L223)

- [LSP7DigitalAssetCore.sol#L171](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L171)


## 4. using > 0 costs more gas than != 0 when used on a uint in a require() statement

This change saves 6 gas per instance. The optimization works until solidity version 0.8.13 where there is a regression in gas costs.

- [LSP1UniversalReceiverDelegateUP.sol#L165](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol#L165)

- [LSP0ERC725AccountCore.sol#L182](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L182)
- [LSP0ERC725AccountCore.sol#L741](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L741)

- [LSP8CompatibleERC721.sol#L313](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol#L313)

- [LSP8IdentifiableDigitalAssetCore.sol#L493](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L493)

- [LSP7DigitalAssetCore.sol#L171](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L171)

- [LSP20CallVerification.sol#L86](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP20CallVerification/LSP20CallVerification.sol#L86)


## 5. using `calldata` instead of `memory` for read-only arguments in external functions saves gas

When a function with a memory array is called externally, the abi.decode() step has to use a for-loop to copy each index of the calldata to the memory index. Each iteration of this for-loop costs at least 60 gas (i.e. 60 * <mem_array>.length). Using calldata directly, obliviates the need for such a loop in the contract code and runtime execution. 

- [LSP0ERC725AccountCore.sol#L226](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L226)
- [LSP0ERC725AccountCore.sol#L280](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L280) 4 instances here  saves lots of gas
- [LSP0ERC725AccountCore.sol#L341](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L341)
- [LSP0ERC725AccountCore.sol#L381](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L381)
- [LSP0ERC725AccountCore.sol#L382](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L382)

- [LSP7DigitalAssetCore.sol#L160](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L160) 5 instances in this function. saves lots of gas

- [LSP7DigitalAssetCore.sol#L130](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L130)


## 6. internal functions only called once can be inlined to save gas

Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

- [LSP1UniversalReceiverDelegateUP.sol#L152](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol#L152)
- [LSP1UniversalReceiverDelegateUP.sol#L201](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol#L201)

- [LSP6KeyManagerCore.sol#L441](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L441)
- [LSP6KeyManagerCore.sol#L518](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L518)

- [LSP6SetDataModule.sol#L474](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L474)
- [LSP6SetDataModule.sol#L490](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L490)

- [LSP6ExecuteModule.sol#L153](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L153)
- [LSP6ExecuteModule.sol#L170](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L170)
- [LSP6ExecuteModule.sol#L188](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L188)
- [LSP6ExecuteModule.sol#L317](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L317)
- [LSP6ExecuteModule.sol#L339](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L339)
- [LSP6ExecuteModule.sol#L366](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L366)
- [LSP6ExecuteModule.sol#L382](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L382)
- [LSP6ExecuteModule.sol#L399](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L399)
- [LSP6ExecuteModule.sol#L418](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L418)


## 7. abi.encode() is less efficient than abi.encodepacked()

- [LSP0ERC725AccountCore.sol#L253](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L253)
- [LSP0ERC725AccountCore.sol#L319](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L319)
- [LSP0ERC725AccountCore.sol#L528](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L528)


## 8. usage of uint/int smaller than 32 bytes (256 bits) incurs overhead

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.
Each operation involving a uint8 costs an extra 22-28 gas (depending on whether the other operand is also a variable of type uint8) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so. Use a larger size then downcast where needed
https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html Use a larger size then downcast where needed

operations are being done on a smaller uint

- [LSP5Utils.sol#L85](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP5ReceivedAssets/LSP5Utils.sol#L85)
- [LSP5Utils.sol#L134](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP5ReceivedAssets/LSP5Utils.sol#L134)

- [LSP10Utils.sol#L83](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP10ReceivedVaults/LSP10Utils.sol#L83)
- [LSP10Utils.sol#L140](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP10ReceivedVaults/LSP10Utils.sol#L140)


## 9. avoid an unnecessary sstore by not writing a default value for bools

- [LSP6KeyManagerCore.sol#L256](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L256)
- [LSP6KeyManagerCore.sol#L323](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L323)
- [LSP6KeyManagerCore.sol#L369](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L369)

- [LSP6SetDataModule.sol#L127](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L127)


## 10. Use assembly to check for address(0)

saves 6 gas per instance

- [LSP0ERC725AccountCore.sol#L801](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L801)
- [LSP0ERC725AccountCore.sol#L804](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L804)

- [LSP8IdentifiableDigitalAssetCore.sol#L84](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L84)
- [LSP8IdentifiableDigitalAssetCore.sol#L115](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L115)
- [LSP8IdentifiableDigitalAssetCore.sol#L139](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L139)
- [LSP8IdentifiableDigitalAssetCore.sol#L291](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L291)
- [LSP8IdentifiableDigitalAssetCore.sol#L319](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L319)
- [LSP8IdentifiableDigitalAssetCore.sol#L410](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L410)

- [LSP7DigitalAssetCore.sol#L268](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L268)
- [LSP7DigitalAssetCore.sol#L300](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L300)
- [LSP7DigitalAssetCore.sol#L343](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L343)
- [LSP7DigitalAssetCore.sol#L406](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L406)

- [LSP17Extendable.sol#L50](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP17ContractExtension/LSP17Extendable.sol#L50)
- [LSP17Extendable.sol#L91](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP17ContractExtension/LSP17Extendable.sol#L91)


## 11. use existing cached version of state var

The instances below point to the second+ access of a state variable within a function. Caching of a state variable replace each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses. 


if instead of -= we use simple subtraction instead we will have a cheaper version if we use the cached version of `_tokenOwnerBalances[from]` before line 347
- [LSP7DigitalAssetCore.sol#L373](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L373)

same thing with `_operatorAuthorizedAmount[from][operator]` used in line 354 and 365
- [LSP7DigitalAssetCore.sol#L365](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L365)

same here `_tokenOwnerBalances[from]` in lines 410 419
- [LSP7DigitalAssetCore.sol#L419](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L419)


## 12. >= costs less gas than >

The compiler uses opcodes `GT` and `ISZERO` for solidity code that uses `>`, but only requires `LT` for `>=` saves 3 gas. - [Reference](https://gist.github.com/IllIllI000/3dc79d25acccfa16dee4e83ffdc6ffde)

- [LSP1UniversalReceiverDelegateUP.sol#L165](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol#L165)

- [LSP0ERC725AccountCore.sol#L182](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L182)
- [LSP0ERC725AccountCore.sol#L741](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L741)

- [LSP8CompatibleERC721.sol#L313](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol#L313)

- [LSP8IdentifiableDigitalAssetCore.sol#L493](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L493)

- [LSP7DigitalAssetCore.sol#L171](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L171)

- [LSP20CallVerification.sol#L86](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP20CallVerification/LSP20CallVerification.sol#L86)


## 13. Use hardcoded address instead address(this)

Instead of using `address(this)`, it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry's `script.sol` and solmate's `LibRlp.sol` contracts can help achieve this. - [Reference](https://twitter.com/transmissions11/status/1518507047943245824)

- [LSP0ERC725AccountCore.sol#L176](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L176)

- [LSP6KeyManagerCore.sol#L365](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L365)

- [LSP6ExecuteModule.sol#L86](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L86)


## 14. The use of a logical AND in place of double if is slightly less gas efficient in instances where there isn't a corresponding else statement for the given if statement

Using a double if statement instead of logical AND (&&) can provide similar short-circuiting behavior whereas double if is slightly more efficient.

- [LSP1UniversalReceiverDelegateUP.sol#L227](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol#L227)
- [LSP1UniversalReceiverDelegateUP.sol#L245](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol#L245)

- [LSP8CompatibleERC721.sol#L236](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol#L236)

- [LSP6KeyManagerCore.sol#L338](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L338)
- [LSP6KeyManagerCore.sol#L404](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L404)

- [LSP6SetDataModule.sol#L362](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L362)

12 instances here
- [LSP6ExecuteModule.sol](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L165)

- [LSP5Utils.sol#L78](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP5ReceivedAssets/LSP5Utils.sol#L78)

- [LSP10Utils.sol#L76](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP10ReceivedVaults/LSP10Utils.sol#L76)


## 15. Avoid updating storage when the value hasn't changed

In Solidity, manipulating contract storage comes with significant gas costs. One can optimize gas usage by preventing unnecessary storage updates when the new value is the same as the existing one. If an existing value is the same as the new one, not reassigning it to the storage could potentially save substantial amounts of gas, notably 2900 gas for a 'Gsreset'. This saving may come at the expense of a cold storage load operation ('Gcoldsload'), which costs 2100 gas, or a warm storage access operation ('Gwarmaccess'), which costs 100 gas. Therefore, the gas efficiency of your contract can be significantly improved by adding a check that compares the new value with the current one before any storage update operation. If the values are the same, you can bypass the storage operation, thereby saving gas.

- [LSP6KeyManagerInitAbstract.sol#L22](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6KeyManagerInitAbstract.sol#L22)

- [LSP6KeyManager.sol#L20](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6KeyManager.sol#L20)

- [LSP8CompatibleERC721.sol#L293](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol#L293)

- [LSP8CompatibleERC721InitAbstract.sol#L293](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol#L293)

- [LSP7CompatibleERC20InitAbstract.sol#L34](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/extensions/LSP7CompatibleERC20InitAbstract.sol#L34)

- [LSP7DigitalAssetInitAbstract.sol#L34](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetInitAbstract.sol#L34)

- [LSP7DigitalAssetCore.sol#L276](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L276)
