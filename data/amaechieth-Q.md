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

3. `burn` & `mint` are missing validation post `_beforeTokenTransfer` hook

Both of these functions implement the `_beforeTokenTransfer` and don't validate the post conditions of this hook. In the case of minting, some implementations may mint the specified tokenId. In the case of burning, some implementations may transfer the token. In order to account for this in standard ERC721 implementations, post the `_beforeTokenTransfer`, the following checks are made.

[ERC721.sol#_mint](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/6e214227376acd688e60fcdca8cb7f0ae8cef98f/contracts/token/ERC721/ERC721.sol#L280-L285)

```
_beforeTokenTransfer(address(0), to, tokenId, 1);

        // Check that tokenId was not minted by `_beforeTokenTransfer` hook
        if (_exists(tokenId)) {
            revert ERC721InvalidSender(address(0));
        }
```

[ERC721.sol#_burn](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/6e214227376acd688e60fcdca8cb7f0ae8cef98f/contracts/token/ERC721/ERC721.sol#L316-L319)

```
beforeTokenTransfer(owner, address(0), tokenId, 1);

        // Update ownership in case tokenId was transferred by `_beforeTokenTransfer` hook
        owner = ownerOf(tokenId);
```

However, in `LSP8IdentifiableDigitalAssetCore` these checks aren't present which may break certain implementations of this standard.

[LSP8IdentifiableDigitalAssetCore](https://github.com/code-423n4/2023-06-lukso/blob/bd49f57c32a522563fc42feeee23c83c8b373405/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L313-L382)




