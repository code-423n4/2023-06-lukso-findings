1. Redundant `msg.value` check

If `receive` is triggered then `msg.value` must be > 0 by default, so this check isn't neccessary.

[LSP9VaultCore.sol](https://github.com/code-423n4/2023-06-lukso/blob/bd49f57c32a522563fc42feeee23c83c8b373405/contracts/LSP9Vault/LSP9VaultCore.sol#L92-L94)
```
receive() external payable virtual {
        if (msg.value > 0) emit ValueReceived(msg.sender, msg.value); 
```

2. `_tokenIdsForOperator` is unused and can be removed

[LSP8IdentifiableDigitalAssetCore.sol#L54](https://github.com/code-423n4/2023-06-lukso/blob/bd49f57c32a522563fc42feeee23c83c8b373405/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L54)

`mapping(address => EnumerableSet.Bytes32Set) private _tokenIdsForOperator;`
