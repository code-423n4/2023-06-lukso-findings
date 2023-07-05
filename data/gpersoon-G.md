# 1 Use calldata instead of memory in arrays

## Impact
In several functions, `calldata` can be used instead of `memory` for arrays. Especially in low level functions, which are called frequently this will save some gas.

## Proof of Concept
Here is an example: [ERC725XCore.sol executeBatch](https://github.com/ERC725Alliance/ERC725/blob/develop/implementations/contracts/ERC725XCore.sol#L52-L59)
```solidity
    function executeBatch(
        uint256[] memory operationsType,
        address[] memory targets,
        uint256[] memory values,
        bytes[] memory datas
    ) public payable virtual override onlyOwner returns (bytes[] memory) {
        return _executeBatch(operationsType, targets, values, datas);
    }
```
Here are some more locations:
- [ERC725XCore.sol getDataBatch](https://github.com/ERC725Alliance/ERC725/blob/develop/implementations/contracts/ERC725YCore.sol#L42-L57)
- [ERC725XCore.sol setDataBatch](https://github.com/ERC725Alliance/ERC725/blob/develop/implementations/contracts/ERC725YCore.sol#L73-L96)
-  [ERC725XCore.sol executeBatch](https://github.com/lukso-network/lsp-smart-contracts/blob/v0.10.2/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L280-L324)
-  [LSP0ERC725AccountCore setDataBatch](https://github.com/lukso-network/lsp-smart-contracts/blob/v0.10.2/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L380-L424)
-  [LSP6KeyManagerCore executeRelayCallBatch](https://github.com/lukso-network/lsp-smart-contracts/blob/v0.10.2/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L202-L244)

## Tools Used

## Recommended Mitigation Steps
Replace `calldata` with `memory` in arrays the function definition. Also do this in the (internal) functions that are called from these functions.
Here is an example for `executeBatch`:
```diff
function executeBatch(
-   uint256[] memory operationsType,
+   uint256[] calldata operationsType,
-   address[] memory targets,
+   address[] calldata targets,
-   uint256[] memory values,
+   uint256[] calldata values,
-   bytes[] memory datas
+   bytes[] calldata datas
    ) public payable virtual override onlyOwner returns (bytes[] memory) {
    return _executeBatch(operationsType, targets, values, datas);
}
```
