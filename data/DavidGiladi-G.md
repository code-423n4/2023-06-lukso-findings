

### Gas Optimization Issues
|Title|Issue|Instances|Total Gas Saved|
|-|:-|:-:|:-:|
| Multiplication and Division by 2 Should use in Bit Shifting | [Multiplication and Division by 2 Should use in Bit Shifting](#multiplication-and-division-by-2-should-use-in-bit-shifting) | 3 | 60 |
| Check Arguments Early | [Check Arguments Early](#check-arguments-early) | 2 | - |
| Potential Optimization by Combining Multiple Mappings into a Struct | [Potential Optimization by Combining Multiple Mappings into a Struct](#potential-optimization-by-combining-multiple-mappings-into-a-struct) | 1 | - |
| State variables that could be declared immutable | [State variables that could be declared immutable](#state-variables-that-could-be-declared-immutable) | 2 | 4194 |
| Inefficient Parameter Storage | [Inefficient Parameter Storage](#inefficient-parameter-storage) | 49 | - |
| Internal or Private Function that Called Once Should Be Inlined to Save Gas | [Internal or Private Function that Called Once Should Be Inlined to Save Gas](#internal-or-private-function-that-called-once-should-be-inlined-to-save-gas) | 10 | 200 |
| State variables should be cached in stack variables rather than re-reading them from storage | [State variables should be cached in stack variables rather than re-reading them from storage](#state-variables-should-be-cached-in-stack-variables-rather-than-re-reading-them-from-storage) | 8 | 776 |
| Unnecessary Casting of Variables | [Unnecessary Casting of Variables](#unnecessary-casting-of-variables) | 1 | - |
| Redundant Contract Existence Check in Consecutive External Calls | [Redundant Contract Existence Check in Consecutive External Calls](#redundant-contract-existence-check-in-consecutive-external-calls) | 2 | 200 |
| Unused Named Return Variables | [Unused Named Return Variables](#unused-named-return-variables) | 11 | - |
| Cache Array Length Outside of Loop | [Cache Array Length Outside of Loop](#cache-array-length-outside-of-loop) | 6 | 18 |

Total: 94 instances over 11 issues

## Multiplication and Division by 2 Should use in Bit Shifting
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 60

### Description
The expressions 'x * 2' and 'x / 2' can be optimized for gas efficiency by utilizing bitwise operations. In Solidity, you can achieve the same results by using bitwise left shift (x << 1) for multiplication and bitwise right shift (x >> 1) for division.

Using bitwise shift operations (SHL and SHR) instead of multiplication (MUL) and division (DIV) opcodes can lead to significant gas savings. The MUL and DIV opcodes cost 5 gas, while the SHL and SHR opcodes incur a lower cost of only 3 gas.

By leveraging these more efficient bitwise operations, you can reduce the gas consumption of your smart contracts and enhance their overall performance.

<details>

<summary>
There are 3 instances of this issue:

</summary>

###
- File: contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol
```
 
Line: 242          nbOfBytes < (offset + 32 + (arrayLength * 32))
```
 instead `32` use bit shifting `5` 

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol#L242](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol#L242)


- File: contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol
```
 
Line: 581                        0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff

                ) <<

                (8 * (32 - length));



            /*

             * tran
```
 instead `8` use bit shifting `3` 

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L581-L585](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L581-L585)


- File: contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol
```
 
Line: 701          ffffffffffffffffffffffffffffffffffffffff

                ) <<

                (8 * (32 - length));



            /*

             * transform the allowed data key situated from
```
 instead `8` use bit shifting `3` 

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L701-L705](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L701-L705)


</details>

# 

## Note:
I have reported only on issues that were overlooked by the winning bot.

## Potential Optimization by Combining Multiple Mappings into a Struct
- Severity: Gas Optimization
- Confidence: High

### Description
This detector identifies instances where multiple mappings of an address or ID could be combined into a single mapping to a struct. This optimisation not only saves a storage slot for each mapping that is combined, but also makes the contract more efficient in terms of gas usage. Depending on the circumstances and sizes of types, it can avoid a 20000 gas cost (Gsset) per combined mapping. Also, it can make reads and subsequent writes cheaper when a function requires both values and they both fit in the same storage slot. Lastly, if both fields are accessed in the same function, it can save approximately 42 gas per access due to not having to recalculate the key's keccak256 hash and its associated stack operations.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###

- File: contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol
```
 
Line: 46          mapping(bytes32 => address) private _tokenOwners
```
Consider to combine with <br>-    ```Line: 52 mapping(bytes32 => EnumerableSet.AddressSet) private _operators```<br>-    ```Line: 54 mapping(address => EnumerableSet.Bytes32Set) private _tokenIdsForOperator```<br><br><br>
[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L46](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L46)


</details>

# 

## State variables that could be declared immutable
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 4194

### Description
State variables that are not updated following deployment should be declared immutable to save gas.

<details>

<summary>
There are 2 instances of this issue:

</summary>

###
- File: contracts/LSP6KeyManager/LSP6KeyManagerCore.sol
```
 
Line: 79          address internal _target
```
 should be immutable 

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L79](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L79)


- File: contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol
```
 
Line: 47          bool internal _isNonDivisible
```
 should be immutable 

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L47](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L47)


</details>

# 

## Inefficient Parameter Storage
- Severity: Gas Optimization
- Confidence: Medium

### Description
When passing function parameters, using the `calldata` area instead of `memory` can improve gas efficiency. Calldata is a read-only area where function arguments and external function calls' parameters are stored.

By using `calldata` for function parameters, you avoid unnecessary gas costs associated with copying data from `calldata` to memory. This is particularly beneficial when the parameter is read-only and doesn't require modification within the contract.

Using `calldata` for function parameters can help optimize gas usage, especially when making external function calls or when the parameter values are provided externally and don't need to be stored persistently within the contract.

<details>

<summary>
There are 49 instances of this issue:

</summary>

###
- File: contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol
```
 
Line: 281          uint256[] memory operationsType
```
 should be declared as `calldata` instead 

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L281](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L281)


- File: contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol
```
 
Line: 282          address[] memory targets
```
 should be declared as `calldata` instead 

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L282](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L282)


- File: contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol
```
 
Line: 283          uint256[] memory values
```
 should be declared as `calldata` instead 

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L283](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L283)


- File: contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol
```
 
Line: 284          bytes[] memory datas
```
 should be declared as `calldata` instead 

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L284](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L284)


- File: contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol
```
 
Line: 381          bytes32[] memory dataKeys
```
 should be declared as `calldata` instead 

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L381](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L381)


- File: contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol
```
 
Line: 382          bytes[] memory dataValues
```
 should be declared as `calldata` instead 

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L382](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L382)


- File: contracts/LSP6KeyManager/LSP6KeyManagerCore.sol
```
 
Line: 205          bytes[] memory signatures
```
 should be declared as `calldata` instead 

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L205](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L205)


- File: contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol
```
 
Line: 120          bytes32[] memory inputDataKeys
```
 should be declared as `calldata` instead 

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L120](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L120)


- File: contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol
```
 
Line: 121          bytes[] memory inputDataValues
```
 should be declared as `calldata` instead 

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L121](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L121)


- File: contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol
```
 
Line: 624           bytes memory allowedERC725YDa
```
 should be declared as `calldata` instead 

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L624](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L624)


- File: contracts/LSP6KeyManager/LSP6Utils.sol
```
 
Line: 170          bytes32[] memory permissions
```
 should be declared as `calldata` instead 

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Utils.sol#L170](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Utils.sol#L170)


- File: contracts/LSP7DigitalAsset/ILSP7DigitalAsset.sol
```
 
Line: 190          address[] memory from
```
 should be declared as `calldata` instead 

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/ILSP7DigitalAsset.sol#L190](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/ILSP7DigitalAsset.sol#L190)


- File: contracts/LSP7DigitalAsset/ILSP7DigitalAsset.sol
```
 
Line: 191          address[] memory to
```
 should be declared as `calldata` instead 

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/ILSP7DigitalAsset.sol#L191](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/ILSP7DigitalAsset.sol#L191)


- File: contracts/LSP7DigitalAsset/ILSP7DigitalAsset.sol
```
 
Line: 192          uint256[] memory amount
```
 should be declared as `calldata` instead 

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/ILSP7DigitalAsset.sol#L192](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/ILSP7DigitalAsset.sol#L192)


- File: contracts/LSP7DigitalAsset/ILSP7DigitalAsset.sol
```
 
Line: 193          bool[] memory allowNonLSP1Recipient
```
 should be declared as `calldata` instead 

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/ILSP7DigitalAsset.sol#L193](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/ILSP7DigitalAsset.sol#L193)


- File: contracts/LSP7DigitalAsset/ILSP7DigitalAsset.sol
```
 
Line: 194          bytes[] memory data
```
 should be declared as `calldata` instead 

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/ILSP7DigitalAsset.sol#L194](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/ILSP7DigitalAsset.sol#L194)


- File: contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol
```
 
Line: 155          address[] memory from
```
 should be declared as `calldata` instead 

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L155](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L155)


- File: contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol
```
 
Line: 156          address[] memory to
```
 should be declared as `calldata` instead 

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L156](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L156)


- File: contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol
```
 
Line: 157          uint256[] memory amount
```
 should be declared as `calldata` instead 

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L157](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L157)


- File: contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol
```
 
Line: 158          bool[] memory allowNonLSP1Recipient
```
 should be declared as `calldata` instead 

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L158](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L158)


- File: contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol
```
 
Line: 159          bytes[] memory data
```
 should be declared as `calldata` instead 

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L159](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L159)


- File: contracts/LSP8IdentifiableDigitalAsset/ILSP8IdentifiableDigitalAsset.sol
```
 
Line: 217          address[] memory from
```
 should be declared as `calldata` instead 

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/ILSP8IdentifiableDigitalAsset.sol#L217](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/ILSP8IdentifiableDigitalAsset.sol#L217)


- File: contracts/LSP8IdentifiableDigitalAsset/ILSP8IdentifiableDigitalAsset.sol
```
 
Line: 218          address[] memory to
```
 should be declared as `calldata` instead 

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/ILSP8IdentifiableDigitalAsset.sol#L218](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/ILSP8IdentifiableDigitalAsset.sol#L218)


- File: contracts/LSP8IdentifiableDigitalAsset/ILSP8IdentifiableDigitalAsset.sol
```
 
Line: 219          bytes32[] memory tokenId
```
 should be declared as `calldata` instead 

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/ILSP8IdentifiableDigitalAsset.sol#L219](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/ILSP8IdentifiableDigitalAsset.sol#L219)


- File: contracts/LSP8IdentifiableDigitalAsset/ILSP8IdentifiableDigitalAsset.sol
```
 
Line: 220          bool[] memory allowNonLSP1Recipient
```
 should be declared as `calldata` instead 

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/ILSP8IdentifiableDigitalAsset.sol#L220](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/ILSP8IdentifiableDigitalAsset.sol#L220)


- File: contracts/LSP8IdentifiableDigitalAsset/ILSP8IdentifiableDigitalAsset.sol
```
 
Line: 221          bytes[] memory data
```
 should be declared as `calldata` instead 

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/ILSP8IdentifiableDigitalAsset.sol#L221](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/ILSP8IdentifiableDigitalAsset.sol#L221)


- File: contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol
```
 
Line: 211          address[] memory from
```
 should be declared as `calldata` instead 

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L211](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L211)


- File: contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol
```
 
Line: 212          address[] memory to
```
 should be declared as `calldata` instead 

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L212](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L212)


- File: contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol
```
 
Line: 213          bytes32[] memory tokenId
```
 should be declared as `calldata` instead 

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L213](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L213)


- File: contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol
```
 
Line: 214          bool[] memory allowNonLSP1Recipient
```
 should be declared as `calldata` instead 

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L214](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L214)


- File: contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol
```
 
Line: 215          bytes[] memory data
```
 should be declared as `calldata` instead 

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L215](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L215)


- File: submodules/ERC725/implementations/contracts/ERC725XCore.sol
```
 
Line: 53          uint256[] memory operationsType
```
 should be declared as `calldata` instead 

[https://github.com/code-423n4/2023-06-lukso/blob/main/submodules/ERC725/implementations/contracts/ERC725XCore.sol#L53](https://github.com/code-423n4/2023-06-lukso/blob/main/submodules/ERC725/implementations/contracts/ERC725XCore.sol#L53)


- File: submodules/ERC725/implementations/contracts/ERC725XCore.sol
```
 
Line: 54          address[] memory targets
```
 should be declared as `calldata` instead 

[https://github.com/code-423n4/2023-06-lukso/blob/main/submodules/ERC725/implementations/contracts/ERC725XCore.sol#L54](https://github.com/code-423n4/2023-06-lukso/blob/main/submodules/ERC725/implementations/contracts/ERC725XCore.sol#L54)


- File: submodules/ERC725/implementations/contracts/ERC725XCore.sol
```
 
Line: 55          uint256[] memory values
```
 should be declared as `calldata` instead 

[https://github.com/code-423n4/2023-06-lukso/blob/main/submodules/ERC725/implementations/contracts/ERC725XCore.sol#L55](https://github.com/code-423n4/2023-06-lukso/blob/main/submodules/ERC725/implementations/contracts/ERC725XCore.sol#L55)


- File: submodules/ERC725/implementations/contracts/ERC725XCore.sol
```
 
Line: 56          bytes[] memory datas
```
 should be declared as `calldata` instead 

[https://github.com/code-423n4/2023-06-lukso/blob/main/submodules/ERC725/implementations/contracts/ERC725XCore.sol#L56](https://github.com/code-423n4/2023-06-lukso/blob/main/submodules/ERC725/implementations/contracts/ERC725XCore.sol#L56)


- File: submodules/ERC725/implementations/contracts/ERC725XCore.sol
```
 
Line: 132          uint256[] memory operationsType
```
 should be declared as `calldata` instead 

[https://github.com/code-423n4/2023-06-lukso/blob/main/submodules/ERC725/implementations/contracts/ERC725XCore.sol#L132](https://github.com/code-423n4/2023-06-lukso/blob/main/submodules/ERC725/implementations/contracts/ERC725XCore.sol#L132)


- File: submodules/ERC725/implementations/contracts/ERC725XCore.sol
```
 
Line: 133          address[] memory targets
```
 should be declared as `calldata` instead 

[https://github.com/code-423n4/2023-06-lukso/blob/main/submodules/ERC725/implementations/contracts/ERC725XCore.sol#L133](https://github.com/code-423n4/2023-06-lukso/blob/main/submodules/ERC725/implementations/contracts/ERC725XCore.sol#L133)


- File: submodules/ERC725/implementations/contracts/ERC725XCore.sol
```
 
Line: 134          uint256[] memory values
```
 should be declared as `calldata` instead 

[https://github.com/code-423n4/2023-06-lukso/blob/main/submodules/ERC725/implementations/contracts/ERC725XCore.sol#L134](https://github.com/code-423n4/2023-06-lukso/blob/main/submodules/ERC725/implementations/contracts/ERC725XCore.sol#L134)


- File: submodules/ERC725/implementations/contracts/ERC725XCore.sol
```
 
Line: 135          bytes[] memory datas
```
 should be declared as `calldata` instead 

[https://github.com/code-423n4/2023-06-lukso/blob/main/submodules/ERC725/implementations/contracts/ERC725XCore.sol#L135](https://github.com/code-423n4/2023-06-lukso/blob/main/submodules/ERC725/implementations/contracts/ERC725XCore.sol#L135)


- File: submodules/ERC725/implementations/contracts/ERC725YCore.sol
```
 
Line: 43          bytes32[] memory dataKeys
```
 should be declared as `calldata` instead 

[https://github.com/code-423n4/2023-06-lukso/blob/main/submodules/ERC725/implementations/contracts/ERC725YCore.sol#L43](https://github.com/code-423n4/2023-06-lukso/blob/main/submodules/ERC725/implementations/contracts/ERC725YCore.sol#L43)


- File: submodules/ERC725/implementations/contracts/ERC725YCore.sol
```
 
Line: 74          bytes32[] memory dataKeys
```
 should be declared as `calldata` instead 

[https://github.com/code-423n4/2023-06-lukso/blob/main/submodules/ERC725/implementations/contracts/ERC725YCore.sol#L74](https://github.com/code-423n4/2023-06-lukso/blob/main/submodules/ERC725/implementations/contracts/ERC725YCore.sol#L74)


- File: submodules/ERC725/implementations/contracts/ERC725YCore.sol
```
 
Line: 75          bytes[] memory dataValues
```
 should be declared as `calldata` instead 

[https://github.com/code-423n4/2023-06-lukso/blob/main/submodules/ERC725/implementations/contracts/ERC725YCore.sol#L75](https://github.com/code-423n4/2023-06-lukso/blob/main/submodules/ERC725/implementations/contracts/ERC725YCore.sol#L75)


- File: submodules/ERC725/implementations/contracts/interfaces/IERC725X.sol
```
 
Line: 94          uint256[] memory operationsType
```
 should be declared as `calldata` instead 

[https://github.com/code-423n4/2023-06-lukso/blob/main/submodules/ERC725/implementations/contracts/interfaces/IERC725X.sol#L94](https://github.com/code-423n4/2023-06-lukso/blob/main/submodules/ERC725/implementations/contracts/interfaces/IERC725X.sol#L94)


- File: submodules/ERC725/implementations/contracts/interfaces/IERC725X.sol
```
 
Line: 95          address[] memory targets
```
 should be declared as `calldata` instead 

[https://github.com/code-423n4/2023-06-lukso/blob/main/submodules/ERC725/implementations/contracts/interfaces/IERC725X.sol#L95](https://github.com/code-423n4/2023-06-lukso/blob/main/submodules/ERC725/implementations/contracts/interfaces/IERC725X.sol#L95)


- File: submodules/ERC725/implementations/contracts/interfaces/IERC725X.sol
```
 
Line: 96          uint256[] memory values
```
 should be declared as `calldata` instead 

[https://github.com/code-423n4/2023-06-lukso/blob/main/submodules/ERC725/implementations/contracts/interfaces/IERC725X.sol#L96](https://github.com/code-423n4/2023-06-lukso/blob/main/submodules/ERC725/implementations/contracts/interfaces/IERC725X.sol#L96)


- File: submodules/ERC725/implementations/contracts/interfaces/IERC725X.sol
```
 
Line: 97          bytes[] memory datas
```
 should be declared as `calldata` instead 

[https://github.com/code-423n4/2023-06-lukso/blob/main/submodules/ERC725/implementations/contracts/interfaces/IERC725X.sol#L97](https://github.com/code-423n4/2023-06-lukso/blob/main/submodules/ERC725/implementations/contracts/interfaces/IERC725X.sol#L97)


- File: submodules/ERC725/implementations/contracts/interfaces/IERC725Y.sol
```
 
Line: 36          bytes32[] memory dataKeys
```
 should be declared as `calldata` instead 

[https://github.com/code-423n4/2023-06-lukso/blob/main/submodules/ERC725/implementations/contracts/interfaces/IERC725Y.sol#L36](https://github.com/code-423n4/2023-06-lukso/blob/main/submodules/ERC725/implementations/contracts/interfaces/IERC725Y.sol#L36)


- File: submodules/ERC725/implementations/contracts/interfaces/IERC725Y.sol
```
 
Line: 68          bytes32[] memory dataKeys
```
 should be declared as `calldata` instead 

[https://github.com/code-423n4/2023-06-lukso/blob/main/submodules/ERC725/implementations/contracts/interfaces/IERC725Y.sol#L68](https://github.com/code-423n4/2023-06-lukso/blob/main/submodules/ERC725/implementations/contracts/interfaces/IERC725Y.sol#L68)


- File: submodules/ERC725/implementations/contracts/interfaces/IERC725Y.sol
```
 
Line: 69          bytes[] memory dataValues
```
 should be declared as `calldata` instead 

[https://github.com/code-423n4/2023-06-lukso/blob/main/submodules/ERC725/implementations/contracts/interfaces/IERC725Y.sol#L69](https://github.com/code-423n4/2023-06-lukso/blob/main/submodules/ERC725/implementations/contracts/interfaces/IERC725Y.sol#L69)


</details>

# 


## Internal or Private Function that Called Once Should Be Inlined to Save Gas
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 200

### Description
Setting the internal/private function that called once to inline would save gas

<details>

<summary>
There are 10 instances of this issue:

</summary>

###
- File: contracts/LSP17ContractExtension/LSP17Extension.sol
```
 
Line: 32          function _extendableMsgData()

        internal

        view

        virtual

        returns (bytes calldata)

    
```
 this function can be delete, the function is never called:

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP17ContractExtension/LSP17Extension.sol#L32-L39](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP17ContractExtension/LSP17Extension.sol#L32-L39)


- File: contracts/LSP17ContractExtension/LSP17Extension.sol
```
 
Line: 44          function _extendableMsgSender() internal view virtual returns (address) 
```
 this function can be delete, the function is never called:

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP17ContractExtension/LSP17Extension.sol#L44-L49](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP17ContractExtension/LSP17Extension.sol#L44-L49)


- File: contracts/LSP17ContractExtension/LSP17Extension.sol
```
 
Line: 54          function _extendableMsgValue() internal view virtual returns (uint256) 
```
 this function can be delete, the function is never called:

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP17ContractExtension/LSP17Extension.sol#L54-L56](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP17ContractExtension/LSP17Extension.sol#L54-L56)


- File: contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol
```
 
Line: 152          function _whenReceiving(

        bytes32 typeId,

        address notifier,

        bytes32 notifierMapKey,

        bytes4 interfaceID

    ) internal virtual returns (bytes memory) 
```
 should be inline to save gas:

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol#L152-L194](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol#L152-L194)


- File: contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol
```
 
Line: 201          function _whenSending(

        bytes32 typeId,

        address notifier,

        bytes32 notifierMapKey,

        uint128 arrayIndex

    ) internal virtual returns (bytes memory) 
```
 should be inline to save gas:

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol#L201-L252](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol#L201-L252)


- File: contracts/LSP6KeyManager/LSP6KeyManagerInitAbstract.sol
```
 
Line: 20          function _initialize(address target_) internal virtual onlyInitializing 
```
 should be inline to save gas:

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6KeyManagerInitAbstract.sol#L20-L24](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6KeyManagerInitAbstract.sol#L20-L24)


- File: contracts/LSP7DigitalAsset/extensions/LSP7CappedSupplyInitAbstract.sol
```
 
Line: 21          function _initialize(

        uint256 tokenSupplyCap_

    ) internal virtual onlyInitializing 
```
 should be inline to save gas:

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/extensions/LSP7CappedSupplyInitAbstract.sol#L21-L29](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/extensions/LSP7CappedSupplyInitAbstract.sol#L21-L29)


- File: contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CappedSupplyInitAbstract.sol
```
 
Line: 23          function _initialize(

        uint256 tokenSupplyCap_

    ) internal virtual onlyInitializing 
```
 should be inline to save gas:

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CappedSupplyInitAbstract.sol#L23-L31](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CappedSupplyInitAbstract.sol#L23-L31)


- File: contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol
```
 
Line: 307          function _checkOnERC721Received(

        address from,

        address to,

        uint256 tokenId,

        bytes memory data

    ) private returns (bool) 
```
 should be inline to save gas:

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol#L307-L339](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol#L307-L339)


- File: contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol
```
 
Line: 307          function _checkOnERC721Received(

        address from,

        address to,

        uint256 tokenId,

        bytes memory data

    ) private returns (bool) 
```
 should be inline to save gas:

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol#L307-L339](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol#L307-L339)


</details>

# 



## Unnecessary Casting of Variables
- Severity: Gas Optimization
- Confidence: High

### Description
This detector scans for instances where a variable is casted to its own type. This is unnecessary and can be safely removed to improve code readability.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
- File: contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol
```
 
Line: 76          bytes32(payload[100:132]) != bytes32(uint256(128))
```
Unnecessary cast: `uint256(128)` it cast to the same type.<br>
[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L76](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L76)


</details>

# 

## Redundant Contract Existence Check in Consecutive External Calls
- Severity: Gas Optimization
- Confidence: Medium
- Total Gas Saved: 200

### Description
This detector identifies instances where there are consecutive external calls to the same contract, where the subsequent calls could use a low-level call to save gas.

Note: This detector only triggers if the function call does not return any value. 

Prior to 0.8.10, the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent Solidity versions the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low-level calls never check for contract existence. 

<details>

<summary>
There are 2 instances of this issue:

</summary>

###
- File: contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol
```
 
Line: 636          msg.sender.tryNotifyUniversalReceiver(

            _TYPEID_LSP0_OwnershipTransferred_RecipientNotification,

            ""

        )
```
This call could be replaced with a low-level call because the contract `LSP1Utils` has already been checked in <br>`Line: 630    previousOwner.tryNotifyUniversalReceiver(

            _TYPEID_LSP0_OwnershipTransferred_SenderNotification,

            ""

        )`<br>
[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L636-L639](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L636-L639)


- File: contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol
```
 
Line: 97          msg.sender.tryNotifyUniversalReceiver(

            _TYPEID_LSP14_OwnershipTransferred_RecipientNotification,

            ""

        )
```
This call could be replaced with a low-level call because the contract `LSP1Utils` has already been checked in <br>`Line: 92    previousOwner.tryNotifyUniversalReceiver(

            _TYPEID_LSP14_OwnershipTransferred_SenderNotification,

            ""

        )`<br>
[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol#L97-L100](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol#L97-L100)


</details>

# 


## Check Arguments Early
- Severity: Gas Optimization
- Confidence: High

### Description
Checks that require() or revert() statements that check input arguments are at the top of the function. Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a gas load in a function that may ultimately revert in the unhappy case.

<details>

<summary>
There are 2 instances of this issue:

</summary>

###
- File: contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol
```
 
Line: 253          require(

            _checkOnERC721Received(from, to, tokenId, data),

            "LSP8CompatibleERC721: transfer to non ERC721Receiver implementer"

        )
```

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol#L253-L256](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol#L253-L256)


- File: contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol
```
 
Line: 253          require(

            _checkOnERC721Received(from, to, tokenId, data),

            "LSP8CompatibleERC721: transfer to non ERC721Receiver implementer"

        )
```

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol#L253-L256](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol#L253-L256)


</details>

# 

## Note:
I have reported only on issues that were overlooked by the winning bot, specifically mappings and arrays that weren't checked but could potentially be cached.

## State variables should be cached in stack variables rather than re-reading them from storage
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 776

### Description
State variables should be cached instead of re-reading them.Subsequent reads incur a 100 gas penalty.Caching such variables replaces the Gwarmaccess with much cheaper stack reads.

<details>

<summary>
There are 5 instances of this issue:

</summary>

###


- File: contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol
```
 
Line: 338          function _burn(

        address from,

        uint256 amount,

        bytes memory data

    ) internal virtual 
```
 should cache the following state variables: `_tokenOwnerBalances`,  `_operatorAuthorizedAmount`
 

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L338-L384](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L338-L384)


- File: contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol
```
 
Line: 399          function _transfer(

        address from,

        address to,

        uint256 amount,

        bool allowNonLSP1Recipient,

        bytes memory data

    ) internal virtual 
```
 should cache the following state variables:  
 `_tokenOwnerBalances`


[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L399-L428](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L399-L428)


- File: contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol
```
 
Line: 394          function _transfer(

        address from,

        address to,

        bytes32 tokenId,

        bool allowNonLSP1Recipient,

        bytes memory data

    ) internal virtual 
```
 should cache the following state variables:  
 `_ownedTokens`


[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L394-L430](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L394-L430)


- File: contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8Enumerable.sol
```
 
Line: 31          function _beforeTokenTransfer(

        address from,

        address to,

        bytes32 tokenId

    ) internal virtual override(LSP8IdentifiableDigitalAssetCore) 
```
 should cache the following state variables:  
 `_tokenIndex`,  
 `_indexToken`



[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8Enumerable.sol#L31-L52](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8Enumerable.sol#L31-L52)


- File: contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8EnumerableInitAbstract.sol
```
 
Line: 33          function _beforeTokenTransfer(

        address from,

        address to,

        bytes32 tokenId

    ) internal virtual override(LSP8IdentifiableDigitalAssetCore) 
```
 should cache the following state variables:   
 `_tokenIndex`,  
`_indexToken`
	

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8EnumerableInitAbstract.sol#L33-L54](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8EnumerableInitAbstract.sol#L33-L54)


</details>

# 

## Note:
I've only reported issues that were overlooked by the winning bot, specifically on 'while' loops that could potentially be optimized with caching.

## Cache Array Length Outside of Loop
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 18

### Description
When using arrays in a for loop, calling .length on every iteration will result in unnecessary gas expenses. Instead, it is recommended to cache the array length beforehand, which saves 3 gas per iteration. For storage arrays, an additional 100 gas per iteration (except the first) can be saved by pre-caching the length

<details>

<summary>
There are 6 instances of this issue:

</summary>

###



- File: contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol
```
 
Line: 334          while (pointer < compactBytesArray.length) {

           
```
 caching the length of array
 
[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol#L334](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol#L334)





- File: contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol
```
 
Line: 162         while (ii < inputDataKeys.length)
```
 caching the length of array
 
[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L162](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L162)


- File: contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol
```
 
Line: 542          while (pointer < allowedERC725YDataKeysCompacted.length) {

```
 caching the length of array
 
[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L542](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L542)


- File: contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol
```
 
Line: 662         while (pointer < allowedERC725YDataKeysCompacted.length) {
```
 caching the length of array
 
[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L662](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L662)


- File: contracts/LSP6KeyManager/LSP6Utils.sol
```
 
Line: 96          while (pointer < allowedCallsCompacted.length) {
```
 caching the length of array
 
[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Utils.sol#L96](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Utils.sol#L96)


- File: contracts/LSP6KeyManager/LSP6Utils.sol
```
 
Line: 125          while (pointer < allowedERC725YDataKeysCompacted.length) {
```
 caching the length of array
 
[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Utils.sol#L125](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Utils.sol#L125)



</details>

# 
