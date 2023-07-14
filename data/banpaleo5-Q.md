# Table of Contents
| Number | Issues Details                                                                                         | Count |
| :----- | :----------------------------------------------------------------------------------------------------- | :---- |
| [L-1] | Incorporating Value Comparison in `Set` Functions | 1 |
| [L-2] | Utilize `_safeMint` for Safer Minting | 15 |
| [L-3] | Add Contract-Existence Check to `safeTransfer` Function | 3 |
| [L-4] | Validate Address Parameters to Avoid Issues | 1 |
| [N-1] | Circumvent Usage of Overlapping Variable Names | 1 |
| [N-2] | Update OpenZeppelin Contract Dependency to Latest Version | 1 |
| [N-3] | Implement Two-Step TransferOwnership | 1 |
| [N-4] | Omitting Initialization of `uint` Variables to Zero | 3 |
| [N-5] | Changing Visibility of Uninvoked Functions to External | 1 |




## [L-1]</a><a name="L-1"> Incorporating Value Comparison in `Set` Functions

To enhance the functionality of set functions, it is recommended to incorporate a comparison mechanism to verify that the current value and the new value are distinct.

<details>
<summary><i>There are 1 instances of this issue:</i></summary>

```solidity
File: LSP6Utils.sol
function setDataViaKeyManager(
        address keyManagerAddress,
        bytes32[] memory keys,
        bytes[] memory values
    ) internal returns (bytes memory result) {
        bytes memory payload = abi.encodeWithSelector(
            IERC725Y.setDataBatch.selector,
            keys,
            values
        );
        result = ILSP6KeyManager(keyManagerAddress).execute(payload);
    }
```
</details>





## [L-2]</a><a name="L-2"> Utilize `_safeMint` for Safer Minting

To enhance the safety and security of the minting process in your contract, it is recommended to use the `_safeMint` function instead of `_mint`. The `_safeMint` function incorporates additional checks and safeguards to mitigate potential risks.

<details>
<summary><i>There are 15 instances of this issue:</i></summary>

```solidity
File: LSP7CompatibleERC20Mintable.sol
_mint(to, amount, allowNonLSP1Recipient, data);
```

```solidity
File: LSP8CappedSupplyInitAbstract.sol
super._mint(to, tokenId, allowNonLSP1Recipient, data);
```

```solidity
File: LSP7CappedSupplyInitAbstract.sol
super._mint(to, amount, allowNonLSP1Recipient, data);
```

```solidity
File: LSP7MintableInitAbstract.sol
_mint(to, amount, allowNonLSP1Recipient, data);
```

```solidity
File: LSP8CompatibleERC721MintableInitAbstract.sol
_mint(to, tokenId, allowNonLSP1Recipient, data);
```

```solidity
File: LSP7CompatibleERC20InitAbstract.sol
super._mint(to, amount, allowNonLSP1Recipient, data);
```

```solidity
File: LSP8CappedSupply.sol
super._mint(to, tokenId, allowNonLSP1Recipient, data);
```

```solidity
File: LSP7CompatibleERC20.sol
super._mint(to, amount, allowNonLSP1Recipient, data);
```

```solidity
File: LSP8Mintable.sol
_mint(to, tokenId, allowNonLSP1Recipient, data);
```

```solidity
File: LSP8CompatibleERC721InitAbstract.sol
super._mint(to, tokenId, allowNonLSP1Recipient, data);
```

```solidity
File: LSP8CompatibleERC721Mintable.sol
_mint(to, tokenId, allowNonLSP1Recipient, data);
```

```solidity
File: LSP7CappedSupply.sol
super._mint(to, amount, allowNonLSP1Recipient, data);
```

```solidity
File: LSP7CompatibleERC20MintableInitAbstract.sol
_mint(to, amount, allowNonLSP1Recipient, data);
```

```solidity
File: LSP8MintableInitAbstract.sol
_mint(to, tokenId, allowNonLSP1Recipient, data);
```

```solidity
File: LSP7Mintable.sol
_mint(to, amount, allowNonLSP1Recipient, data);
```
</details>



## [L-3]</a><a name="L-3"> Add Contract-Existence Check to `safeTransfer` Function

<details>
<summary><i>There are 3 instances of this issue:</i></summary>

```solidity
File: LSP8CompatibleERC721.sol
function safeTransferFrom(
```

```solidity
File: LSP8CompatibleERC721.sol
function _safeTransfer(
```

```solidity
File: LSP8CompatibleERC721InitAbstract.sol
function safeTransferFrom(
```
</details>



## [L-4]</a><a name="L-4"> Validate Address Parameters to Avoid Issues

To prevent transaction reverts and gas wastage, validate address parameters to ensure they are not set to the zero address (0x0).

<details>
<summary><i>There are 1 instances of this issue:</i></summary>

```solidity
File: LSP7CompatibleERC20.sol
from
```

```solidity
File: LSP7CompatibleERC20.sol
to
```
</details>






## [N-1]</a><a name="N-1"> Circumvent Usage of Overlapping Variable Names

Employing global variable designations such as `call{value: value }` can lead to overshadowing caused by the resemblance in argument names. This can potentially reduce the comprehensibility of the code and degrade its overall readability.

<details>
<summary><i>There are 1 instances of this issue:</i></summary>

```solidity
File: ERC725XCore.sol
(bool success, bytes memory returnData) = target.call{value: value}(
```
</details>


## [N-2]</a><a name="N-2"> Update OpenZeppelin Contract Dependency to Latest Version

<details>
<summary><i>There are 1 instances of this issue:</i></summary>

```solidity
File: package.json
    "@openzeppelin/contracts": "^4.9.2"
```

```solidity
File: package.json
    "@openzeppelin/contracts-upgradeable": "^4.9.2"
```
</details>




## [N-3]</a><a name="N-3"> Implement Two-Step TransferOwnership

<details>
<summary><i>There are 1 instances of this issue:</i></summary>

```solidity
File: OwnableUnset.sol
function transferOwnership(address newOwner) public virtual onlyOwner {
```
</details>







## [N-4]</a><a name="N-4"> Omitting Initialization of `uint` Variables to Zero

Initializing `uint` variables to zero is unnecessary since their default value is already `0`.

<details>
<summary><i>There are 3 instances of this issue:</i></summary>

```solidity
File: LSP6SetDataModule.sol
uint256 inputDataKeysAllowed = 0;
uint256 ii = 0;
```

```solidity
File: LSP6Utils.sol
uint256 pointer = 0;
```

```solidity
File: LSP6Utils.sol
uint256 pointer = 0;
```
</details>




## [N-5]</a><a name="N-5"> Changing Visibility of Uninvoked Functions to External

In cases where public functions in a contract are not being called by the contract itself, it is advisable to change their visibility from public to external. Contracts have the flexibility to override parent functions and modify their visibility from external to public as needed. By making this adjustment, code clarity and adherence to best practices can be maintained, improving the overall readability and intent of the contract.

<details>
<summary><i>There are 1 instances of this issue:</i></summary>

```solidity
File: ERC725.sol
function supportsInterface(
        bytes4 interfaceId
    ) public view virtual override(ERC725XCore, ERC725YCore) returns (bool) {
```
</details>



