# Note: 
I reported issues that were overlooked by the winning bot. I reported only on instances that were missed

### Low Issues
|Title|Issue|Instances|
|-|:-|:-:|
|[L-1] Array out of bounds accesses | [Array out of bounds accesses](#array-out-of-bounds-accesses) | 1 |
|[L-2] Burn functions must be protected with a modifier | [Burn functions must be protected with a modifier](#burn-functions-must-be-protected-with-a-modifier) | 9 |
|[L-3] Controlled Delegatecall | [Controlled Delegatecall](#controlled-delegatecall) | 1 |
|[L-4] Payable functions using `delegatecall` inside a loop | [Payable functions using `delegatecall` inside a loop](#payable-functions-using-delegatecall-inside-a-loop) | 1 |
|[L-5] NFT contract redefines mint()/safeMint(), but not both | [NFT contract redefines mint()/safeMint(), but not both](#nft-contract-redefines-mintsafemint-but-not-both) | 1 |
|[L-6] Local variable shadowing | [Local variable shadowing](#local-variable-shadowing) | 6 |
|[L-7] Contracts that lock Ether | [Contracts that lock Ether](#contracts-that-lock-ether) | 9 |
|[L-8] Missing gap Storage Variable in Upgradeable Contract | [Missing gap Storage Variable in Upgradeable Contract](#missing-gap-storage-variable-in-upgradeable-contract) | 24 |
|[L-9] Missing zero address validation | [Missing zero address validation](#missing-zero-address-validation) | 4 |
|[L-10] `msg.value` inside a loop | [`msg.value` inside a loop](#msgvalue-inside-a-loop) | 4 |
|[L-11] Calls inside a loop | [Calls inside a loop](#calls-inside-a-loop) | 16 |
|[L-12] Reentrancy vulnerabilities | [Reentrancy vulnerabilities](#reentrancy-vulnerabilities-2) | 4 |
|[L-13] Unbounded gas for external call | [Unbounded gas for external call](#unbounded-gas-for-external-call) | 4 |
|[L-14] Unused return | [Unused return](#unused-return) | 12 |
|[L-15] Use 'abi.encodeCall()' Instead of 'abi.encodeSignature()'/abi.encodeSelector() | [Use 'abi.encodeCall()' Instead of 'abi.encodeSignature()'/abi.encodeSelector()](#use-abiencodecall-instead-of-abiencodesignatureabiencodeselector) | 5 |
|[L-16] Use of transferFrom() rather than safeTransferFrom() for NFTs | [Use of transferFrom() rather than safeTransferFrom() for NFTs](#use-of-transferfrom-rather-than-safetransferfrom-for-nfts) | 2 |

Total: 16 issues

## Array out of bounds accesses
- Severity: Low
- Confidence: Medium

### Description
If the lengths of arrays are not checked before they're accessed, user operations may not be executed properly due to the potential issue of accessing an array out of its bounds. In Solidity, accessing an array beyond its boundaries can cause a runtime error known as an out-of-bounds exception. This error arises when attempting to read or write to an element of an array that does not exist. Therefore, it is critical to avoid out-of-bounds exceptions while accessing arrays in Solidity code to prevent unexpected halts and possible loss of funds.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
- File: contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol
```
 
Line: 626                 bool[] memory validatedInputKeysList

```
 is accessed on File: contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol
```
 
Line: 729                  validatedInputKeysList[ii];

           
```
 index that might be out-of-bounds

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L626](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L626)


</details>

# 


## Burn functions must be protected with a modifier
- Severity: Low
- Confidence: High

### Description
If burn functions are not protected by a modifier, any address may be able to burn tokens, potentially leading to financial loss. A common modifier to use is `onlyOwner`.

<details>

<summary>
There are 9 instances of this issue:

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
Burn function without a protective modifier.
[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L338-L384](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L338-L384)


- File: contracts/LSP7DigitalAsset/extensions/LSP7Burnable.sol
```
 
Line: 17          function burn(address from, uint256 amount, bytes memory data) public 
```
Burn function without a protective modifier.
[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/extensions/LSP7Burnable.sol#L17-L19](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/extensions/LSP7Burnable.sol#L17-L19)


- File: contracts/LSP7DigitalAsset/extensions/LSP7BurnableInitAbstract.sol
```
 
Line: 19          function burn(address from, uint256 amount, bytes memory data) public 
```
Burn function without a protective modifier.
[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/extensions/LSP7BurnableInitAbstract.sol#L19-L21](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/extensions/LSP7BurnableInitAbstract.sol#L19-L21)


- File: contracts/LSP7DigitalAsset/extensions/LSP7CompatibleERC20.sol
```
 
Line: 137          function _burn(

        address from,

        uint256 amount,

        bytes memory data

    ) internal virtual override 
```
Burn function without a protective modifier.
[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/extensions/LSP7CompatibleERC20.sol#L137-L144](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/extensions/LSP7CompatibleERC20.sol#L137-L144)


- File: contracts/LSP7DigitalAsset/extensions/LSP7CompatibleERC20InitAbstract.sol
```
 
Line: 145          function _burn(

        address from,

        uint256 amount,

        bytes memory data

    ) internal virtual override 
```
Burn function without a protective modifier.
[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/extensions/LSP7CompatibleERC20InitAbstract.sol#L145-L152](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/extensions/LSP7CompatibleERC20InitAbstract.sol#L145-L152)


- File: contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol
```
 
Line: 359          function _burn(bytes32 tokenId, bytes memory data) internal virtual 
```
Burn function without a protective modifier.
[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L359-L382](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L359-L382)


- File: contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8Burnable.sol
```
 
Line: 16          function burn(bytes32 tokenId, bytes memory data) public 
```
Burn function without a protective modifier.
[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8Burnable.sol#L16-L21](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8Burnable.sol#L16-L21)


- File: contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol
```
 
Line: 269          function _burn(

        bytes32 tokenId,

        bytes memory data

    ) internal virtual override 
```
Burn function without a protective modifier.
[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol#L269-L277](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol#L269-L277)


- File: contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol
```
 
Line: 269          function _burn(

        bytes32 tokenId,

        bytes memory data

    ) internal virtual override 
```
Burn function without a protective modifier.
[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol#L269-L277](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol#L269-L277)


</details>

# 



## Controlled Delegatecall
- Severity: Low
- Confidence: Medium

### Description
This detector identifies instances where a contract's code allows a user to control the target of a delegatecall or callcode operation. 
Delegatecalls and callcodes are powerful features in Solidity that allow one contract to execute the code of another contract within its own context.

However, they also pose significant security risks if misused or not properly secured.
When the target of a delegatecall or callcode is controlled by a user, that user may be able to manipulate the contract's state or behavior in unexpected and potentially harmful ways.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
- File: submodules/ERC725/implementations/contracts/ERC725XCore.sol
```
 
Line: 225          function _executeDelegateCall(

        address target,

        bytes memory data

    ) internal virtual returns (bytes memory result) 
```
 uses delegatecall to a input-controlled function id
	- File: submodules/ERC725/implementations/contracts/ERC725XCore.sol
```
 
Line: 232          (bool success, bytes memory returnData) = target.delegatecall(data)
```


[https://github.com/code-423n4/2023-06-lukso/blob/main/submodules/ERC725/implementations/contracts/ERC725XCore.sol#L225-L239](https://github.com/code-423n4/2023-06-lukso/blob/main/submodules/ERC725/implementations/contracts/ERC725XCore.sol#L225-L239)


</details>

# 


## Payable functions using `delegatecall` inside a loop
- Severity: Low
- Confidence: Medium

### Description
Detect the use of `delegatecall` inside a loop in a payable function.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
- File: submodules/ERC725/implementations/contracts/ERC725XCore.sol
```
 
Line: 225          function _executeDelegateCall(

        address target,

        bytes memory data

    ) internal virtual returns (bytes memory result) 
```
 has delegatecall inside a loop in a payable function: File: submodules/ERC725/implementations/contracts/ERC725XCore.sol
```
 
Line: 232          (bool success, bytes memory returnData) = target.delegatecall(data)
```


[https://github.com/code-423n4/2023-06-lukso/blob/main/submodules/ERC725/implementations/contracts/ERC725XCore.sol#L225-L239](https://github.com/code-423n4/2023-06-lukso/blob/main/submodules/ERC725/implementations/contracts/ERC725XCore.sol#L225-L239)


</details>

# 




## NFT contract redefines mint()/safeMint(), but not both
- Severity: Low
- Confidence: High

### Description
ERC721 contracts should have both _mint() and _safeMint() consistently redefined for spec compatibility. The _mint() variant is supposed to skip onERC721Received() checks, whereas _safeMint() does not. If one of these functions is re-implemented or has new arguments, the other should as well. 

<details>

<summary>
There are 1 instances of this issue:

</summary>

###


- File: contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol
```
 
Line: 48          abstract contract LSP8CompatibleERC721InitAbstract is

    ILSP8CompatibleERC721,

    LSP8IdentifiableDigitalAssetInitAbstract,

    LSP4Compatibility


```
Redefine _mint() without redefine _safeMint().
[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol#L48-L351](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol#L48-L351)


</details>

# 



## Local variable shadowing
- Severity: Low
- Confidence: High

### Description
Detection of shadowing using local variables.

<details>

<summary>
There are 6 instances of this issue:

</summary>

###
- File: contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol
```
 
Line: 232          address _owner = owner()
```
 shadows:
	- File: submodules/ERC725/implementations/contracts/custom/OwnableUnset.sol
```
 
Line: 12          address private _owner
```
 (state variable)

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L232](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L232)


- File: contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol
```
 
Line: 290          address _owner = owner()
```
 shadows:
	- File: submodules/ERC725/implementations/contracts/custom/OwnableUnset.sol
```
 
Line: 12          address private _owner
```
 (state variable)

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L290](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L290)


- File: contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol
```
 
Line: 347          address _owner = owner()
```
 shadows:
	- File: submodules/ERC725/implementations/contracts/custom/OwnableUnset.sol
```
 
Line: 12          address private _owner
```
 (state variable)

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L347](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L347)


- File: contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol
```
 
Line: 392          address _owner = owner()
```
 shadows:
	- File: submodules/ERC725/implementations/contracts/custom/OwnableUnset.sol
```
 
Line: 12          address private _owner
```
 (state variable)

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L392](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L392)


- File: contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol
```
 
Line: 660          address _owner = owner()
```
 shadows:
	- File: submodules/ERC725/implementations/contracts/custom/OwnableUnset.sol
```
 
Line: 12          address private _owner
```
 (state variable)

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L660](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L660)


- File: contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol
```
 
Line: 738          address _owner = owner()
```
 shadows:
	- File: submodules/ERC725/implementations/contracts/custom/OwnableUnset.sol
```
 
Line: 12          address private _owner
```
 (state variable)

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L738](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L738)


</details>

# 


## Contracts that lock Ether
- Severity: Low
- Confidence: High

### Description
Contract with a `payable` function, but without a withdrawal capacity.

<details>

<summary>
There are 9 instances of this issue:

</summary>

###
- Contract locking ether found:
	Contract File: contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol
```
 
Line: 62          contract LSP1UniversalReceiverDelegateUP is ERC165, ILSP1UniversalReceiver 
```
 has payable functions:
	 - File: contracts/LSP1UniversalReceiver/ILSP1UniversalReceiver.sol
```
 
Line: 32          function universalReceiver(

        bytes32 typeId,

        bytes calldata data

    ) external payable returns (bytes memory);
```

	 - File: contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol
```
 
Line: 78          function universalReceiver(

        bytes32 typeId,

        bytes memory /* data */

    ) public payable virtual returns (bytes memory result) 
```

	But does not have a function to withdraw the ether

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol#L62-L266](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol#L62-L266)


- Contract locking ether found:
	Contract File: contracts/LSP7DigitalAsset/presets/LSP7CompatibleERC20Mintable.sol
```
 
Line: 6          contract LSP7CompatibleERC20Mintable is LSP7CompatibleERC20 
```
 has payable functions:
	 - File: submodules/ERC725/implementations/contracts/interfaces/IERC725Y.sol
```
 
Line: 52          function setData(bytes32 dataKey, bytes memory dataValue) external payable;
```

	 - File: submodules/ERC725/implementations/contracts/interfaces/IERC725Y.sol
```
 
Line: 67          function setDataBatch(

        bytes32[] memory dataKeys,

        bytes[] memory dataValues

    ) external payable;
```

	 - File: submodules/ERC725/implementations/contracts/ERC725Y.sol
```
 
Line: 21          constructor(address newOwner) payable 
```

	 - File: submodules/ERC725/implementations/contracts/ERC725YCore.sol
```
 
Line: 62          function setData(

        bytes32 dataKey,

        bytes memory dataValue

    ) public payable virtual override onlyOwner 
```

	 - File: submodules/ERC725/implementations/contracts/ERC725YCore.sol
```
 
Line: 73          function setDataBatch(

        bytes32[] memory dataKeys,

        bytes[] memory dataValues

    ) public payable virtual override onlyOwner 
```

	But does not have a function to withdraw the ether

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/presets/LSP7CompatibleERC20Mintable.sol#L6-L21](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/presets/LSP7CompatibleERC20Mintable.sol#L6-L21)


- Contract locking ether found:
	Contract File: contracts/LSP7DigitalAsset/presets/LSP7CompatibleERC20MintableInit.sol
```
 
Line: 9          contract LSP7CompatibleERC20MintableInit is

    LSP7CompatibleERC20MintableInitAbstract


```
 has payable functions:
	 - File: submodules/ERC725/implementations/contracts/interfaces/IERC725Y.sol
```
 
Line: 52          function setData(bytes32 dataKey, bytes memory dataValue) external payable;
```

	 - File: submodules/ERC725/implementations/contracts/interfaces/IERC725Y.sol
```
 
Line: 67          function setDataBatch(

        bytes32[] memory dataKeys,

        bytes[] memory dataValues

    ) external payable;
```

	 - File: submodules/ERC725/implementations/contracts/ERC725YCore.sol
```
 
Line: 62          function setData(

        bytes32 dataKey,

        bytes memory dataValue

    ) public payable virtual override onlyOwner 
```

	 - File: submodules/ERC725/implementations/contracts/ERC725YCore.sol
```
 
Line: 73          function setDataBatch(

        bytes32[] memory dataKeys,

        bytes[] memory dataValues

    ) public payable virtual override onlyOwner 
```

	But does not have a function to withdraw the ether

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/presets/LSP7CompatibleERC20MintableInit.sol#L9-L36](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/presets/LSP7CompatibleERC20MintableInit.sol#L9-L36)


- Contract locking ether found:
	Contract File: contracts/LSP7DigitalAsset/presets/LSP7Mintable.sol
```
 
Line: 16          contract LSP7Mintable is LSP7DigitalAsset, ILSP7Mintable 
```
 has payable functions:
	 - File: submodules/ERC725/implementations/contracts/interfaces/IERC725Y.sol
```
 
Line: 52          function setData(bytes32 dataKey, bytes memory dataValue) external payable;
```

	 - File: submodules/ERC725/implementations/contracts/interfaces/IERC725Y.sol
```
 
Line: 67          function setDataBatch(

        bytes32[] memory dataKeys,

        bytes[] memory dataValues

    ) external payable;
```

	 - File: submodules/ERC725/implementations/contracts/ERC725Y.sol
```
 
Line: 21          constructor(address newOwner) payable 
```

	 - File: submodules/ERC725/implementations/contracts/ERC725YCore.sol
```
 
Line: 62          function setData(

        bytes32 dataKey,

        bytes memory dataValue

    ) public payable virtual override onlyOwner 
```

	 - File: submodules/ERC725/implementations/contracts/ERC725YCore.sol
```
 
Line: 73          function setDataBatch(

        bytes32[] memory dataKeys,

        bytes[] memory dataValues

    ) public payable virtual override onlyOwner 
```

	But does not have a function to withdraw the ether

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/presets/LSP7Mintable.sol#L16-L42](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/presets/LSP7Mintable.sol#L16-L42)


- Contract locking ether found:
	Contract File: contracts/LSP7DigitalAsset/presets/LSP7MintableInit.sol
```
 
Line: 11          contract LSP7MintableInit is LSP7MintableInitAbstract 
```
 has payable functions:
	 - File: submodules/ERC725/implementations/contracts/interfaces/IERC725Y.sol
```
 
Line: 52          function setData(bytes32 dataKey, bytes memory dataValue) external payable;
```

	 - File: submodules/ERC725/implementations/contracts/interfaces/IERC725Y.sol
```
 
Line: 67          function setDataBatch(

        bytes32[] memory dataKeys,

        bytes[] memory dataValues

    ) external payable;
```

	 - File: submodules/ERC725/implementations/contracts/ERC725YCore.sol
```
 
Line: 62          function setData(

        bytes32 dataKey,

        bytes memory dataValue

    ) public payable virtual override onlyOwner 
```

	 - File: submodules/ERC725/implementations/contracts/ERC725YCore.sol
```
 
Line: 73          function setDataBatch(

        bytes32[] memory dataKeys,

        bytes[] memory dataValues

    ) public payable virtual override onlyOwner 
```

	But does not have a function to withdraw the ether

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/presets/LSP7MintableInit.sol#L11-L39](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/presets/LSP7MintableInit.sol#L11-L39)


- Contract locking ether found:
	Contract File: contracts/LSP8IdentifiableDigitalAsset/presets/LSP8CompatibleERC721Mintable.sol
```
 
Line: 6          contract LSP8CompatibleERC721Mintable is LSP8CompatibleERC721 
```
 has payable functions:
	 - File: submodules/ERC725/implementations/contracts/interfaces/IERC725Y.sol
```
 
Line: 52          function setData(bytes32 dataKey, bytes memory dataValue) external payable;
```

	 - File: submodules/ERC725/implementations/contracts/interfaces/IERC725Y.sol
```
 
Line: 67          function setDataBatch(

        bytes32[] memory dataKeys,

        bytes[] memory dataValues

    ) external payable;
```

	 - File: submodules/ERC725/implementations/contracts/ERC725Y.sol
```
 
Line: 21          constructor(address newOwner) payable 
```

	 - File: submodules/ERC725/implementations/contracts/ERC725YCore.sol
```
 
Line: 62          function setData(

        bytes32 dataKey,

        bytes memory dataValue

    ) public payable virtual override onlyOwner 
```

	 - File: submodules/ERC725/implementations/contracts/ERC725YCore.sol
```
 
Line: 73          function setDataBatch(

        bytes32[] memory dataKeys,

        bytes[] memory dataValues

    ) public payable virtual override onlyOwner 
```

	But does not have a function to withdraw the ether

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/presets/LSP8CompatibleERC721Mintable.sol#L6-L21](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/presets/LSP8CompatibleERC721Mintable.sol#L6-L21)


- Contract locking ether found:
	Contract File: contracts/LSP8IdentifiableDigitalAsset/presets/LSP8CompatibleERC721MintableInit.sol
```
 
Line: 9          contract LSP8CompatibleERC721MintableInit is

    LSP8CompatibleERC721MintableInitAbstract


```
 has payable functions:
	 - File: submodules/ERC725/implementations/contracts/ERC725YCore.sol
```
 
Line: 62          function setData(

        bytes32 dataKey,

        bytes memory dataValue

    ) public payable virtual override onlyOwner 
```

	 - File: submodules/ERC725/implementations/contracts/ERC725YCore.sol
```
 
Line: 73          function setDataBatch(

        bytes32[] memory dataKeys,

        bytes[] memory dataValues

    ) public payable virtual override onlyOwner 
```

	 - File: submodules/ERC725/implementations/contracts/interfaces/IERC725Y.sol
```
 
Line: 52          function setData(bytes32 dataKey, bytes memory dataValue) external payable;
```

	 - File: submodules/ERC725/implementations/contracts/interfaces/IERC725Y.sol
```
 
Line: 67          function setDataBatch(

        bytes32[] memory dataKeys,

        bytes[] memory dataValues

    ) external payable;
```

	But does not have a function to withdraw the ether

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/presets/LSP8CompatibleERC721MintableInit.sol#L9-L36](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/presets/LSP8CompatibleERC721MintableInit.sol#L9-L36)


- Contract locking ether found:
	Contract File: contracts/LSP8IdentifiableDigitalAsset/presets/LSP8Mintable.sol
```
 
Line: 16          contract LSP8Mintable is LSP8IdentifiableDigitalAsset, ILSP8Mintable 
```
 has payable functions:
	 - File: submodules/ERC725/implementations/contracts/interfaces/IERC725Y.sol
```
 
Line: 52          function setData(bytes32 dataKey, bytes memory dataValue) external payable;
```

	 - File: submodules/ERC725/implementations/contracts/interfaces/IERC725Y.sol
```
 
Line: 67          function setDataBatch(

        bytes32[] memory dataKeys,

        bytes[] memory dataValues

    ) external payable;
```

	 - File: submodules/ERC725/implementations/contracts/ERC725Y.sol
```
 
Line: 21          constructor(address newOwner) payable 
```

	 - File: submodules/ERC725/implementations/contracts/ERC725YCore.sol
```
 
Line: 62          function setData(

        bytes32 dataKey,

        bytes memory dataValue

    ) public payable virtual override onlyOwner 
```

	 - File: submodules/ERC725/implementations/contracts/ERC725YCore.sol
```
 
Line: 73          function setDataBatch(

        bytes32[] memory dataKeys,

        bytes[] memory dataValues

    ) public payable virtual override onlyOwner 
```

	But does not have a function to withdraw the ether

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/presets/LSP8Mintable.sol#L16-L40](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/presets/LSP8Mintable.sol#L16-L40)


- Contract locking ether found:
	Contract File: contracts/LSP8IdentifiableDigitalAsset/presets/LSP8MintableInit.sol
```
 
Line: 11          contract LSP8MintableInit is LSP8MintableInitAbstract 
```
 has payable functions:
	 - File: submodules/ERC725/implementations/contracts/interfaces/IERC725Y.sol
```
 
Line: 52          function setData(bytes32 dataKey, bytes memory dataValue) external payable;
```

	 - File: submodules/ERC725/implementations/contracts/interfaces/IERC725Y.sol
```
 
Line: 67          function setDataBatch(

        bytes32[] memory dataKeys,

        bytes[] memory dataValues

    ) external payable;
```

	 - File: submodules/ERC725/implementations/contracts/ERC725YCore.sol
```
 
Line: 62          function setData(

        bytes32 dataKey,

        bytes memory dataValue

    ) public payable virtual override onlyOwner 
```

	 - File: submodules/ERC725/implementations/contracts/ERC725YCore.sol
```
 
Line: 73          function setDataBatch(

        bytes32[] memory dataKeys,

        bytes[] memory dataValues

    ) public payable virtual override onlyOwner 
```

	But does not have a function to withdraw the ether

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/presets/LSP8MintableInit.sol#L11-L32](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/presets/LSP8MintableInit.sol#L11-L32)


</details>

# 


## Missing gap Storage Variable in Upgradeable Contract
- Severity: Low
- Confidence: High

### Description
upgradeable contracts that are missing a '__gap' storage variable. In upgradeable contracts, it is important to reserve storage slots for future versions to introduce new storage variables. The '__gap' storage variable acts as a placeholder, allowing for seamless upgrades without affecting existing storage layout.When a contract is not designed with a '__gap' storage variable, adding new storage variables in subsequent versions becomes problematic. It can lead to storage collisions or layout incompatibilities, making it difficult to upgrade the contract without requiring costly data migrations or redeployments.

<details>

<summary>
There are 24 instances of this issue:

</summary>

###
- File: contracts/LSP0ERC725Account/LSP0ERC725AccountInit.sol
```
 
Line: 40          contract LSP0ERC725AccountInit is LSP0ERC725AccountInitAbstract 
```
 missing __gap storage variable

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountInit.sol#L40-L67](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountInit.sol#L40-L67)


- File: contracts/LSP0ERC725Account/LSP0ERC725AccountInitAbstract.sol
```
 
Line: 44          abstract contract LSP0ERC725AccountInitAbstract is

    Initializable,

    LSP0ERC725AccountCore


```
 missing __gap storage variable

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountInitAbstract.sol#L44-L67](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountInitAbstract.sol#L44-L67)


- File: contracts/LSP4DigitalAssetMetadata/LSP4DigitalAssetMetadataInitAbstract.sol
```
 
Line: 26          abstract contract LSP4DigitalAssetMetadataInitAbstract is ERC725YInitAbstract 
```
 missing __gap storage variable

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP4DigitalAssetMetadata/LSP4DigitalAssetMetadataInitAbstract.sol#L26-L68](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP4DigitalAssetMetadata/LSP4DigitalAssetMetadataInitAbstract.sol#L26-L68)


- File: contracts/LSP6KeyManager/LSP6KeyManagerInit.sol
```
 
Line: 12          contract LSP6KeyManagerInit is LSP6KeyManagerInitAbstract 
```
 missing __gap storage variable

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6KeyManagerInit.sol#L12-L27](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6KeyManagerInit.sol#L12-L27)


- File: contracts/LSP6KeyManager/LSP6KeyManagerInitAbstract.sol
```
 
Line: 16          abstract contract LSP6KeyManagerInitAbstract is

    Initializable,

    LSP6KeyManagerCore


```
 missing __gap storage variable

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6KeyManagerInitAbstract.sol#L16-L25](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6KeyManagerInitAbstract.sol#L16-L25)


- File: contracts/LSP7DigitalAsset/LSP7DigitalAssetInitAbstract.sol
```
 
Line: 24          abstract contract LSP7DigitalAssetInitAbstract is

    LSP4DigitalAssetMetadataInitAbstract,

    LSP7DigitalAssetCore


```
 missing __gap storage variable

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetInitAbstract.sol#L24-L52](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetInitAbstract.sol#L24-L52)


- File: contracts/LSP7DigitalAsset/extensions/LSP7BurnableInitAbstract.sol
```
 
Line: 13          abstract contract LSP7BurnableInitAbstract is LSP7DigitalAssetInitAbstract 
```
 missing __gap storage variable

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/extensions/LSP7BurnableInitAbstract.sol#L13-L22](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/extensions/LSP7BurnableInitAbstract.sol#L13-L22)


- File: contracts/LSP7DigitalAsset/extensions/LSP7CappedSupplyInitAbstract.sol
```
 
Line: 13          abstract contract LSP7CappedSupplyInitAbstract is LSP7DigitalAssetInitAbstract 
```
 missing __gap storage variable

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/extensions/LSP7CappedSupplyInitAbstract.sol#L13-L65](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/extensions/LSP7CappedSupplyInitAbstract.sol#L13-L65)


- File: contracts/LSP7DigitalAsset/extensions/LSP7CompatibleERC20InitAbstract.sol
```
 
Line: 21          abstract contract LSP7CompatibleERC20InitAbstract is

    ILSP7CompatibleERC20,

    LSP4Compatibility,

    LSP7DigitalAssetInitAbstract


```
 missing __gap storage variable

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/extensions/LSP7CompatibleERC20InitAbstract.sol#L21-L164](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/extensions/LSP7CompatibleERC20InitAbstract.sol#L21-L164)


- File: contracts/LSP7DigitalAsset/presets/LSP7CompatibleERC20MintableInit.sol
```
 
Line: 9          contract LSP7CompatibleERC20MintableInit is

    LSP7CompatibleERC20MintableInitAbstract


```
 missing __gap storage variable

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/presets/LSP7CompatibleERC20MintableInit.sol#L9-L36](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/presets/LSP7CompatibleERC20MintableInit.sol#L9-L36)


- File: contracts/LSP7DigitalAsset/presets/LSP7CompatibleERC20MintableInitAbstract.sol
```
 
Line: 9          contract LSP7CompatibleERC20MintableInitAbstract is

    LSP7CompatibleERC20InitAbstract


```
 missing __gap storage variable

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/presets/LSP7CompatibleERC20MintableInitAbstract.sol#L9-L31](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/presets/LSP7CompatibleERC20MintableInitAbstract.sol#L9-L31)


- File: contracts/LSP7DigitalAsset/presets/LSP7MintableInit.sol
```
 
Line: 11          contract LSP7MintableInit is LSP7MintableInitAbstract 
```
 missing __gap storage variable

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/presets/LSP7MintableInit.sol#L11-L39](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/presets/LSP7MintableInit.sol#L11-L39)


- File: contracts/LSP7DigitalAsset/presets/LSP7MintableInitAbstract.sol
```
 
Line: 16          abstract contract LSP7MintableInitAbstract is

    LSP7DigitalAssetInitAbstract,

    ILSP7Mintable


```
 missing __gap storage variable

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/presets/LSP7MintableInitAbstract.sol#L16-L45](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/presets/LSP7MintableInitAbstract.sol#L16-L45)


- File: contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetInitAbstract.sol
```
 
Line: 26          abstract contract LSP8IdentifiableDigitalAssetInitAbstract is

    LSP4DigitalAssetMetadataInitAbstract,

    LSP8IdentifiableDigitalAssetCore


```
 missing __gap storage variable

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetInitAbstract.sol#L26-L52](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetInitAbstract.sol#L26-L52)


- File: contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CappedSupplyInitAbstract.sol
```
 
Line: 13          abstract contract LSP8CappedSupplyInitAbstract is

    LSP8IdentifiableDigitalAssetInitAbstract


```
 missing __gap storage variable

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CappedSupplyInitAbstract.sol#L13-L68](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CappedSupplyInitAbstract.sol#L13-L68)


- File: contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol
```
 
Line: 48          abstract contract LSP8CompatibleERC721InitAbstract is

    ILSP8CompatibleERC721,

    LSP8IdentifiableDigitalAssetInitAbstract,

    LSP4Compatibility


```
 missing __gap storage variable

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol#L48-L351](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol#L48-L351)


- File: contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8EnumerableInitAbstract.sol
```
 
Line: 16          abstract contract LSP8EnumerableInitAbstract is

    LSP8IdentifiableDigitalAssetInitAbstract


```
 missing __gap storage variable

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8EnumerableInitAbstract.sol#L16-L55](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8EnumerableInitAbstract.sol#L16-L55)


- File: contracts/LSP8IdentifiableDigitalAsset/presets/LSP8CompatibleERC721MintableInit.sol
```
 
Line: 9          contract LSP8CompatibleERC721MintableInit is

    LSP8CompatibleERC721MintableInitAbstract


```
 missing __gap storage variable

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/presets/LSP8CompatibleERC721MintableInit.sol#L9-L36](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/presets/LSP8CompatibleERC721MintableInit.sol#L9-L36)


- File: contracts/LSP8IdentifiableDigitalAsset/presets/LSP8CompatibleERC721MintableInitAbstract.sol
```
 
Line: 9          contract LSP8CompatibleERC721MintableInitAbstract is

    LSP8CompatibleERC721InitAbstract


```
 missing __gap storage variable

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/presets/LSP8CompatibleERC721MintableInitAbstract.sol#L9-L31](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/presets/LSP8CompatibleERC721MintableInitAbstract.sol#L9-L31)


- File: contracts/LSP8IdentifiableDigitalAsset/presets/LSP8MintableInit.sol
```
 
Line: 11          contract LSP8MintableInit is LSP8MintableInitAbstract 
```
 missing __gap storage variable

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/presets/LSP8MintableInit.sol#L11-L32](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/presets/LSP8MintableInit.sol#L11-L32)


- File: contracts/LSP8IdentifiableDigitalAsset/presets/LSP8MintableInitAbstract.sol
```
 
Line: 15          abstract contract LSP8MintableInitAbstract is

    LSP8IdentifiableDigitalAssetInitAbstract,

    ILSP8Mintable


```
 missing __gap storage variable

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/presets/LSP8MintableInitAbstract.sol#L15-L42](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/presets/LSP8MintableInitAbstract.sol#L15-L42)


- File: contracts/UniversalProfileInit.sol
```
 
Line: 12          contract UniversalProfileInit is UniversalProfileInitAbstract 
```
 missing __gap storage variable

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/UniversalProfileInit.sol#L12-L39](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/UniversalProfileInit.sol#L12-L39)


- File: contracts/UniversalProfileInitAbstract.sol
```
 
Line: 20          abstract contract UniversalProfileInitAbstract is

    LSP0ERC725AccountInitAbstract


```
 missing __gap storage variable

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/UniversalProfileInitAbstract.sol#L20-L38](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/UniversalProfileInitAbstract.sol#L20-L38)


- File: submodules/ERC725/implementations/contracts/ERC725YInitAbstract.sol
```
 
Line: 18          abstract contract ERC725YInitAbstract is Initializable, ERC725YCore 
```
 missing __gap storage variable

[https://github.com/code-423n4/2023-06-lukso/blob/main/submodules/ERC725/implementations/contracts/ERC725YInitAbstract.sol#L18-L26](https://github.com/code-423n4/2023-06-lukso/blob/main/submodules/ERC725/implementations/contracts/ERC725YInitAbstract.sol#L18-L26)


</details>

# 


## Missing zero address validation
- Severity: Low
- Confidence: Medium

### Description
Detect missing zero address validation.

<details>

<summary>
There are 4 instances of this issue:

</summary>

###
- File: contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol
```
 
Line: 129          _pendingOwner = newOwner
```
 state variable `_pendingOwner` assign with `newOwner` without checking

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol#L129](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol#L129)


- File: contracts/LSP6KeyManager/LSP6KeyManager.sol
```
 
Line: 20          _target = target_
```
 state variable `_target` assign with `target_` without checking

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6KeyManager.sol#L20](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6KeyManager.sol#L20)


- File: contracts/LSP6KeyManager/LSP6KeyManagerInitAbstract.sol
```
 
Line: 22          _target = target_
```
 state variable `_target` assign with `target_` without checking

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6KeyManagerInitAbstract.sol#L22](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6KeyManagerInitAbstract.sol#L22)


- File: submodules/ERC725/implementations/contracts/custom/OwnableUnset.sol
```
 
Line: 71          _owner = newOwner
```
 state variable `_owner` assign with `newOwner` without checking

[https://github.com/code-423n4/2023-06-lukso/blob/main/submodules/ERC725/implementations/contracts/custom/OwnableUnset.sol#L71](https://github.com/code-423n4/2023-06-lukso/blob/main/submodules/ERC725/implementations/contracts/custom/OwnableUnset.sol#L71)


</details>

# 


## `msg.value` inside a loop
- Severity: Low
- Confidence: Medium

### Description
Detect the use of `msg.value` inside a loop.

<details>

<summary>
There are 4 instances of this issue:

</summary>

###
- File: contracts/LSP6KeyManager/LSP6KeyManagerCore.sol
```
 
Line: 152          function executeBatch(

        uint256[] calldata values,

        bytes[] calldata payloads

    ) public payable virtual returns (bytes[] memory) 
```
 use msg.value in a loop: File: contracts/LSP6KeyManager/LSP6KeyManagerCore.sol
```
 
Line: 164          (totalValues += values[ii]) > msg.value
```


[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L152-L180](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L152-L180)


- File: contracts/LSP6KeyManager/LSP6KeyManagerCore.sol
```
 
Line: 152          function executeBatch(

        uint256[] calldata values,

        bytes[] calldata payloads

    ) public payable virtual returns (bytes[] memory) 
```
 use msg.value in a loop: File: contracts/LSP6KeyManager/LSP6KeyManagerCore.sol
```
 
Line: 165          revert LSP6BatchInsufficientValueSent(totalValues, msg.value)
```


[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L152-L180](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L152-L180)


- File: contracts/LSP6KeyManager/LSP6KeyManagerCore.sol
```
 
Line: 204          function executeRelayCallBatch(

        bytes[] memory signatures,

        uint256[] calldata nonces,

        uint256[] calldata validityTimestamps,

        uint256[] calldata values,

        bytes[] calldata payloads

    ) public payable virtual returns (bytes[] memory) 
```
 use msg.value in a loop: File: contracts/LSP6KeyManager/LSP6KeyManagerCore.sol
```
 
Line: 224          (totalValues += values[ii]) > msg.value
```


[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L204-L246](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L204-L246)


- File: contracts/LSP6KeyManager/LSP6KeyManagerCore.sol
```
 
Line: 204          function executeRelayCallBatch(

        bytes[] memory signatures,

        uint256[] calldata nonces,

        uint256[] calldata validityTimestamps,

        uint256[] calldata values,

        bytes[] calldata payloads

    ) public payable virtual returns (bytes[] memory) 
```
 use msg.value in a loop: File: contracts/LSP6KeyManager/LSP6KeyManagerCore.sol
```
 
Line: 225          revert LSP6BatchInsufficientValueSent(totalValues, msg.value)
```


[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L204-L246](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L204-L246)


</details>

# 


## Calls inside a loop
- Severity: Low
- Confidence: Medium

### Description
Calls inside a loop might lead to a denial-of-service attack.

<details>

<summary>
There are 16 instances of this issue:

</summary>

###
- File: contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol
```
 
Line: 171          function batchCalls(

        bytes[] calldata data

    ) public returns (bytes[] memory results) 
```
 has external calls inside a loop: File: contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol
```
 
Line: 176          (bool success, bytes memory result) = address(this).delegatecall(

                data[i]

            )
```


[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L171-L201](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L171-L201)


- File: contracts/LSP6KeyManager/LSP6KeyManagerCore.sol
```
 
Line: 416          function _executePayload(

        uint256 msgValue,

        bytes calldata payload

    ) internal virtual returns (bytes memory) 
```
 has external calls inside a loop: File: contracts/LSP6KeyManager/LSP6KeyManagerCore.sol
```
 
Line: 420          (bool success, bytes memory returnData) = _target.call{

            value: msgValue,

            gas: gasleft()

        }(payload)
```


[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L416-L431](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L416-L431)


- File: contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol
```
 
Line: 334          function _getPermissionToSetPermissionsArray(

        address controlledContract,

        bytes32 inputDataKey,

        bytes memory inputDataValue,

        bool hasBothAddControllerAndEditPermissions

    ) internal view virtual returns (bytes32) 
```
 has external calls inside a loop: File: contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol
```
 
Line: 349          newLength >

                    uint128(

                        bytes16(

                            ERC725Y(controlledContract).getData(inputDataKey)

                        )

                    )
```


[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L334-L377](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L334-L377)


- File: contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol
```
 
Line: 334          function _getPermissionToSetPermissionsArray(

        address controlledContract,

        bytes32 inputDataKey,

        bytes memory inputDataValue,

        bool hasBothAddControllerAndEditPermissions

    ) internal view virtual returns (bytes32) 
```
 has external calls inside a loop: File: contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol
```
 
Line: 374          ERC725Y(controlledContract).getData(inputDataKey).length == 0
```


[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L334-L377](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L334-L377)


- File: contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol
```
 
Line: 385          function _getPermissionToSetControllerPermissions(

        address controlledContract,

        bytes32 inputPermissionDataKey

    ) internal view virtual returns (bytes32) 
```
 has external calls inside a loop: File: contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol
```
 
Line: 392          bytes32(

                ERC725Y(controlledContract).getData(inputPermissionDataKey)

            ) == bytes32(0)
```


[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L385-L397](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L385-L397)


- File: contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol
```
 
Line: 406          function _getPermissionToSetAllowedCalls(

        address controlledContract,

        bytes32 dataKey,

        bytes memory dataValue,

        bool hasBothAddControllerAndEditPermissions

    ) internal view virtual returns (bytes32) 
```
 has external calls inside a loop: File: contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol
```
 
Line: 426          ERC725Y(controlledContract).getData(dataKey).length == 0
```


[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L406-L429](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L406-L429)


- File: contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol
```
 
Line: 438          function _getPermissionToSetAllowedERC725YDataKeys(

        address controlledContract,

        bytes32 dataKey,

        bytes memory dataValue,

        bool hasBothAddControllerAndEditPermissions

    ) internal view returns (bytes32) 
```
 has external calls inside a loop: File: contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol
```
 
Line: 461          ERC725Y(controlledContract).getData(dataKey).length == 0
```


[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L438-L464](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L438-L464)


- File: contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol
```
 
Line: 474          function _getPermissionToSetLSP1Delegate(

        address controlledContract,

        bytes32 lsp1DelegateDataKey

    ) internal view virtual returns (bytes32) 
```
 has external calls inside a loop: File: contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol
```
 
Line: 479          ERC725Y(controlledContract).getData(lsp1DelegateDataKey).length == 0
```


[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L474-L482](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L474-L482)


- File: contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol
```
 
Line: 490          function _getPermissionToSetLSP17Extension(

        address controlledContract,

        bytes32 lsp17ExtensionDataKey

    ) internal view virtual returns (bytes32) 
```
 has external calls inside a loop: File: contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol
```
 
Line: 495          ERC725Y(controlledContract).getData(lsp17ExtensionDataKey).length ==

                0
```


[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L490-L499](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L490-L499)


- File: contracts/LSP6KeyManager/LSP6Utils.sol
```
 
Line: 25          function getPermissionsFor(

        IERC725Y target,

        address caller

    ) internal view returns (bytes32) 
```
 has external calls inside a loop: File: contracts/LSP6KeyManager/LSP6Utils.sol
```
 
Line: 29          target.getData(

            LSP2Utils.generateMappingWithGroupingKey(

                _LSP6KEY_ADDRESSPERMISSIONS_PERMISSIONS_PREFIX,

                bytes20(caller)

            )

        )
```


[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Utils.sol#L25-L37](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Utils.sol#L25-L37)


- File: contracts/LSP6KeyManager/LSP6Utils.sol
```
 
Line: 58          function getAllowedERC725YDataKeysFor(

        IERC725Y target,

        address caller

    ) internal view returns (bytes memory) 
```
 has external calls inside a loop: File: contracts/LSP6KeyManager/LSP6Utils.sol
```
 
Line: 63          target.getData(

                LSP2Utils.generateMappingWithGroupingKey(

                    _LSP6KEY_ADDRESSPERMISSIONS_AllowedERC725YDataKeys_PREFIX,

                    bytes20(caller)

                )

            )
```


[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Utils.sol#L58-L69](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Utils.sol#L58-L69)


- File: contracts/LSP6KeyManager/LSP6Utils.sol
```
 
Line: 39          function getAllowedCallsFor(

        IERC725Y target,

        address from

    ) internal view returns (bytes memory) 
```
 has external calls inside a loop: File: contracts/LSP6KeyManager/LSP6Utils.sol
```
 
Line: 44          target.getData(

                LSP2Utils.generateMappingWithGroupingKey(

                    _LSP6KEY_ADDRESSPERMISSIONS_ALLOWEDCALLS_PREFIX,

                    bytes20(from)

                )

            )
```


[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Utils.sol#L39-L50](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Utils.sol#L39-L50)


- File: contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol
```
 
Line: 452          function _notifyTokenSender(

        address from,

        bytes memory lsp1Data

    ) internal virtual 
```
 has external calls inside a loop: File: contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol
```
 
Line: 462          ILSP1UniversalReceiver(from).universalReceiver(

                _TYPEID_LSP7_TOKENSSENDER,

                lsp1Data

            )
```


[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L452-L467](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L452-L467)


- File: contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol
```
 
Line: 475          function _notifyTokenReceiver(

        address to,

        bool allowNonLSP1Recipient,

        bytes memory lsp1Data

    ) internal virtual 
```
 has external calls inside a loop: File: contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol
```
 
Line: 486          ILSP1UniversalReceiver(to).universalReceiver(

                _TYPEID_LSP7_TOKENSRECIPIENT,

                lsp1Data

            )
```


[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L475-L497](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L475-L497)


- File: contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol
```
 
Line: 454          function _notifyTokenSender(

        address from,

        bytes memory lsp1Data

    ) internal virtual 
```
 has external calls inside a loop: File: contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol
```
 
Line: 464          ILSP1UniversalReceiver(from).universalReceiver(

                _TYPEID_LSP8_TOKENSSENDER,

                lsp1Data

            )
```


[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L454-L469](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L454-L469)


- File: contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol
```
 
Line: 477          function _notifyTokenReceiver(

        address to,

        bool allowNonLSP1Recipient,

        bytes memory lsp1Data

    ) internal virtual 
```
 has external calls inside a loop: File: contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol
```
 
Line: 488          ILSP1UniversalReceiver(to).universalReceiver(

                _TYPEID_LSP8_TOKENSRECIPIENT,

                lsp1Data

            )
```


[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L477-L499](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L477-L499)


</details>

# 


## Reentrancy vulnerabilities
- Severity: Low
- Confidence: Medium

### Description

Detection of the [reentrancy bug](https://github.com/trailofbits/not-so-smart-contracts/tree/master/reentrancy).
Only report reentrancy that acts as a double call (see `reentrancy-eth`, `reentrancy-no-eth`).

<details>

<summary>
There are 2 instances of this issue:

</summary>

###
- Reentrancy in File: contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol
```
 
Line: 655          function renounceOwnership()

        public

        virtual

        override(LSP14Ownable2Step, OwnableUnset)

    
```
:
	External calls:
	- File: contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol
```
 
Line: 669          bool verifyAfter = _verifyCall(_owner)
```

		- File: contracts/LSP20CallVerification/LSP20CallVerification.sol
```
 
Line: 26          (bool success, bytes memory returnedData) = logicVerifier.call(

            abi.encodeWithSelector(

                ILSP20.lsp20VerifyCall.selector,

                msg.sender,

                msg.value,

                msg.data

            )

        )
```

	State variables written after the call(s):
	- File: contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol
```
 
Line: 672          LSP14Ownable2Step._renounceOwnership()
```

		- File: contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol
```
 
Line: 164          delete _pendingOwner
```

	- File: contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol
```
 
Line: 672          LSP14Ownable2Step._renounceOwnership()
```

		- File: contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol
```
 
Line: 163          _renounceOwnershipStartedAt = currentBlock
```

		- File: contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol
```
 
Line: 177          delete _renounceOwnershipStartedAt
```


[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L655-L679](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L655-L679)


- Reentrancy in File: contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol
```
 
Line: 557          function transferOwnership(

        address pendingNewOwner

    ) public virtual override(LSP14Ownable2Step, OwnableUnset) 
```
:
	External calls:
	- File: contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol
```
 
Line: 583          bool verifyAfter = _verifyCall(currentOwner)
```

		- File: contracts/LSP20CallVerification/LSP20CallVerification.sol
```
 
Line: 26          (bool success, bytes memory returnedData) = logicVerifier.call(

            abi.encodeWithSelector(

                ILSP20.lsp20VerifyCall.selector,

                msg.sender,

                msg.value,

                msg.data

            )

        )
```

	State variables written after the call(s):
	- File: contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol
```
 
Line: 586          LSP14Ownable2Step._transferOwnership(pendingNewOwner)
```

		- File: contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol
```
 
Line: 129          _pendingOwner = newOwner
```

	- File: contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol
```
 
Line: 586          LSP14Ownable2Step._transferOwnership(pendingNewOwner)
```

		- File: contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol
```
 
Line: 130          delete _renounceOwnershipStartedAt
```


[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L557-L608](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L557-L608)


</details>

# 


## Reentrancy vulnerabilities
- Severity: Low
- Confidence: Medium

### Description

Detection of the [reentrancy bug](https://github.com/trailofbits/not-so-smart-contracts/tree/master/reentrancy).
Do not report reentrancies that don't involve Ether (see `reentrancy-no-eth`)

<details>

<summary>
There are 2 instances of this issue:

</summary>

###
- Reentrancy in File: contracts/LSP6KeyManager/LSP6KeyManagerCore.sol
```
 
Line: 315          function _execute(

        uint256 msgValue,

        bytes calldata payload

    ) internal virtual returns (bytes memory) 
```
:
	External calls:
	- File: contracts/LSP6KeyManager/LSP6KeyManagerCore.sol
```
 
Line: 336          bytes memory result = _executePayload(msgValue, payload)
```

		- File: contracts/LSP6KeyManager/LSP6KeyManagerCore.sol
```
 
Line: 420          (bool success, bytes memory returnData) = _target.call{

            value: msgValue,

            gas: gasleft()

        }(payload)
```

	State variables written after the call(s):
	- File: contracts/LSP6KeyManager/LSP6KeyManagerCore.sol
```
 
Line: 339          _nonReentrantAfter()
```

		- File: contracts/LSP6KeyManager/LSP6KeyManagerCore.sol
```
 
Line: 553          _reentrancyStatus = false
```

	File: contracts/LSP6KeyManager/LSP6KeyManagerCore.sol
```
 
Line: 83          bool private _reentrancyStatus
```
 can be used in cross function reentrancies:
	- File: contracts/LSP6KeyManager/LSP6KeyManagerCore.sol
```
 
Line: 550          function _nonReentrantAfter() internal virtual 
```

	- File: contracts/LSP6KeyManager/LSP6KeyManagerCore.sol
```
 
Line: 527          function _nonReentrantBefore(

        bool isSetData,

        address from

    ) internal virtual returns (bool isReentrantCall) 
```

	- File: contracts/LSP6KeyManager/LSP6KeyManagerCore.sol
```
 
Line: 251          function lsp20VerifyCall(

        address caller,

        uint256 msgValue,

        bytes calldata data

    ) external returns (bytes4) 
```


[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L315-L343](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L315-L343)


- Reentrancy in File: contracts/LSP6KeyManager/LSP6KeyManagerCore.sol
```
 
Line: 345          function _executeRelayCall(

        bytes memory signature,

        uint256 nonce,

        uint256 validityTimestamps,

        uint256 msgValue,

        bytes calldata payload

    ) internal virtual returns (bytes memory) 
```
:
	External calls:
	- File: contracts/LSP6KeyManager/LSP6KeyManagerCore.sol
```
 
Line: 402          bytes memory result = _executePayload(msgValue, payload)
```

		- File: contracts/LSP6KeyManager/LSP6KeyManagerCore.sol
```
 
Line: 420          (bool success, bytes memory returnData) = _target.call{

            value: msgValue,

            gas: gasleft()

        }(payload)
```

	State variables written after the call(s):
	- File: contracts/LSP6KeyManager/LSP6KeyManagerCore.sol
```
 
Line: 405          _nonReentrantAfter()
```

		- File: contracts/LSP6KeyManager/LSP6KeyManagerCore.sol
```
 
Line: 553          _reentrancyStatus = false
```

	File: contracts/LSP6KeyManager/LSP6KeyManagerCore.sol
```
 
Line: 83          bool private _reentrancyStatus
```
 can be used in cross function reentrancies:
	- File: contracts/LSP6KeyManager/LSP6KeyManagerCore.sol
```
 
Line: 550          function _nonReentrantAfter() internal virtual 
```

	- File: contracts/LSP6KeyManager/LSP6KeyManagerCore.sol
```
 
Line: 527          function _nonReentrantBefore(

        bool isSetData,

        address from

    ) internal virtual returns (bool isReentrantCall) 
```

	- File: contracts/LSP6KeyManager/LSP6KeyManagerCore.sol
```
 
Line: 251          function lsp20VerifyCall(

        address caller,

        uint256 msgValue,

        bytes calldata data

    ) external returns (bytes4) 
```


[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L345-L409](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L345-L409)


</details>

# 



## Unbounded gas for external call
- Severity: Low
- Confidence: High

### Description
There is no limit specified on the amount of gas used, so the recipient can use up all of the transaction's gas, causing it to revert. Use `addr.call{gas: <amount>}()` or [this](https://github.com/nomad-xyz/ExcessivelySafeCall) library instead.

<details>

<summary>
There are 4 instances of this issue:

</summary>

###
- File: contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol
```
 
Line: 176          (bool success, bytes memory result) = address(this).delegatecall(

                data[i]

            )
```

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L176-L178](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L176-L178)


- File: contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol
```
 
Line: 742          (bool success, bytes memory result) = _owner.staticcall(

                abi.encodeWithSelector(

                    IERC1271.isValidSignature.selector,

                    dataHash,

                    signature

                )

            )
```

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L742-L748](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L742-L748)






- File: submodules/ERC725/implementations/contracts/ERC725XCore.sol
```
 
Line: 232          (bool success, bytes memory returnData) = target.delegatecall(data)
```

[https://github.com/code-423n4/2023-06-lukso/blob/main/submodules/ERC725/implementations/contracts/ERC725XCore.sol#L232](https://github.com/code-423n4/2023-06-lukso/blob/main/submodules/ERC725/implementations/contracts/ERC725XCore.sol#L232)



- File: submodules/ERC725/implementations/contracts/ERC725XCore.sol
```
 
Line: 210          (bool success, bytes memory returnData) = target.staticcall(data)
```

[https://github.com/code-423n4/2023-06-lukso/blob/main/submodules/ERC725/implementations/contracts/ERC725XCore.sol#L210](https://github.com/code-423n4/2023-06-lukso/blob/main/submodules/ERC725/implementations/contracts/ERC725XCore.sol#L210)


</details>

# 



## Unused return
- Severity: Low
- Confidence: Medium

### Description
The return value of an external call is not stored in a local or state variable.

<details>

<summary>
There are 12 instances of this issue:

</summary>

###
- File: contracts/LSP1UniversalReceiver/LSP1Utils.sol
```
 
Line: 33          ILSP1UniversalReceiver(lsp1Implementation).universalReceiver(

                typeId,

                data

            )
```
 ignores return value 

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP1UniversalReceiver/LSP1Utils.sol#L33-L36](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP1UniversalReceiver/LSP1Utils.sol#L33-L36)


- File: contracts/LSP1UniversalReceiver/LSP1Utils.sol
```
 
Line: 60          Address.verifyCallResult(

            success,

            result,

            "Call to universalReceiver failed"

        )
```
 ignores return value 

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP1UniversalReceiver/LSP1Utils.sol#L60-L64](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP1UniversalReceiver/LSP1Utils.sol#L60-L64)


- File: contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol
```
 
Line: 462          ILSP1UniversalReceiver(from).universalReceiver(

                _TYPEID_LSP7_TOKENSSENDER,

                lsp1Data

            )
```
 ignores return value 

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L462-L465](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L462-L465)


- File: contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol
```
 
Line: 486          ILSP1UniversalReceiver(to).universalReceiver(

                _TYPEID_LSP7_TOKENSRECIPIENT,

                lsp1Data

            )
```
 ignores return value 

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L486-L489](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L486-L489)


- File: contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol
```
 
Line: 334          _ownedTokens[to].add(tokenId)
```
 ignores return value 

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L334](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L334)


- File: contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol
```
 
Line: 370          _ownedTokens[tokenOwner].remove(tokenId)
```
 ignores return value 

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L370](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L370)


- File: contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol
```
 
Line: 420          _ownedTokens[from].remove(tokenId)
```
 ignores return value 

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L420](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L420)


- File: contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol
```
 
Line: 421          _ownedTokens[to].add(tokenId)
```
 ignores return value 

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L421](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L421)


- File: contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol
```
 
Line: 464          ILSP1UniversalReceiver(from).universalReceiver(

                _TYPEID_LSP8_TOKENSSENDER,

                lsp1Data

            )
```
 ignores return value 

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L464-L467](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L464-L467)


- File: contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol
```
 
Line: 488          ILSP1UniversalReceiver(to).universalReceiver(

                _TYPEID_LSP8_TOKENSRECIPIENT,

                lsp1Data

            )
```
 ignores return value 

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L488-L491](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L488-L491)


- File: contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol
```
 
Line: 314          try

                IERC721Receiver(to).onERC721Received(

                    msg.sender,

                    from,

                    tokenId,

                    data

                )

            returns (bytes4 retval) {

                return retval == IERC721Receiver.onERC721Received.selector;

            } catch (bytes memory reason) {

                if (reason.length == 0) {

                    revert(

                        "LSP8CompatibleERC721: transfer to non ERC721Receiver implementer"

                    );

                } else {

                    // solhint-disable no-inline-assembly

                    /// @solidity memory-safe-assembly

                    assembly {

                        revert(add(32, reason), mload(reason))

                    }

                }

            }
```
 ignores return value 

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol#L314-L335](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol#L314-L335)


- File: contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol
```
 
Line: 314          try

                IERC721Receiver(to).onERC721Received(

                    msg.sender,

                    from,

                    tokenId,

                    data

                )

            returns (bytes4 retval) {

                return retval == IERC721Receiver.onERC721Received.selector;

            } catch (bytes memory reason) {

                if (reason.length == 0) {

                    revert(

                        "LSP8CompatibleERC721: transfer to non ERC721Receiver implementer"

                    );

                } else {

                    // solhint-disable no-inline-assembly

                    /// @solidity memory-safe-assembly

                    assembly {

                        revert(add(32, reason), mload(reason))

                    }

                }

            }
```
 ignores return value 

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol#L314-L335](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol#L314-L335)


</details>

# 


## Use 'abi.encodeCall()' Instead of 'abi.encodeSignature()'/abi.encodeSelector()
- Severity: Low
- Confidence: High

### Description
abi.encodeCall() has compiler type safety, whereas the other two functions do not.

<details>

<summary>
There are 5 instances of this issue:

</summary>

###
- File: contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol
```
 
Line: 743          abi.encodeWithSelector(

                    IERC1271.isValidSignature.selector,

                    dataHash,

                    signature

                )
```
 use abi.encodeCall()

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L743-L747](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L743-L747)


- File: contracts/LSP1UniversalReceiver/LSP1Utils.sol
```
 
Line: 48          abi.encodeWithSelector(

                ILSP1UniversalReceiver.universalReceiver.selector,

                typeId,

                receivedData

            )
```
 use abi.encodeCall()

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP1UniversalReceiver/LSP1Utils.sol#L48-L52](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP1UniversalReceiver/LSP1Utils.sol#L48-L52)


- File: contracts/LSP20CallVerification/LSP20CallVerification.sol
```
 
Line: 27          abi.encodeWithSelector(

                ILSP20.lsp20VerifyCall.selector,

                msg.sender,

                msg.value,

                msg.data

            )
```
 use abi.encodeCall()

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP20CallVerification/LSP20CallVerification.sol#L27-L32](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP20CallVerification/LSP20CallVerification.sol#L27-L32)


- File: contracts/LSP20CallVerification/LSP20CallVerification.sol
```
 
Line: 54          abi.encodeWithSelector(

                ILSP20.lsp20VerifyCallResult.selector,

                keccak256(abi.encodePacked(msg.sender, msg.value, msg.data)),

                callResult

            )
```
 use abi.encodeCall()

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP20CallVerification/LSP20CallVerification.sol#L54-L58](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP20CallVerification/LSP20CallVerification.sol#L54-L58)


- File: contracts/LSP6KeyManager/LSP6Utils.sol
```
 
Line: 155          abi.encodeWithSelector(

            IERC725Y.setDataBatch.selector,

            keys,

            values

        )
```
 use abi.encodeCall()

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Utils.sol#L155-L159](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP6KeyManager/LSP6Utils.sol#L155-L159)


</details>

# 


## Use of transferFrom() rather than safeTransferFrom() for NFTs
- Severity: Low
- Confidence: High

### Description

Use of `transferFrom()` rather than `safeTransferFrom()` for NFTs in will lead to the loss of NFTs
The EIP-721 standard says the following about `transferFrom()`:
```solidity
    /// @notice Transfer ownership of an NFT -- THE CALLER IS RESPONSIBLE
    ///  TO CONFIRM THAT `_to` IS CAPABLE OF RECEIVING NFTS OR ELSE
    ///  THEY MAY BE PERMANENTLY LOST
    /// @dev Throws unless `msg.sender` is the current owner, an authorized
    ///  operator, or the approved address for this NFT. Throws if `_from` is
    ///  not the current owner. Throws if `_to` is the zero address. Throws if
    ///  `_tokenId` is not a valid NFT.
    /// @param _from The current owner of the NFT
    /// @param _to The new owner
    /// @param _tokenId The NFT to transfer
    function transferFrom(address _from, address _to, uint256 _tokenId) external payable;
```
https://github.com/ethereum/EIPs/blob/78e2c297611f5e92b6a5112819ab71f74041ff25/EIPS/eip-721.md?plain=1#L103-L113
Code must use the `safeTransferFrom()` flavor if it hasn't otherwise verified that the receiving address can handle it

    

<details>

<summary>
There are 2 instances of this issue:

</summary>

###
- File: contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol
```
 
Line: 173          transfer(

                from[i],

                to[i],

                amount[i],

                allowNonLSP1Recipient[i],

                data[i]

            )
```

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L173-L179](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L173-L179)


- File: contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol
```
 
Line: 228          transfer(

                from[i],

                to[i],

                tokenId[i],

                allowNonLSP1Recipient[i],

                data[i]

            )
```

[https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L228-L234](https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L228-L234)


</details>

# 