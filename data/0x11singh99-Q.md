# Low Level Issues :

## [L-01] Check function address type params is not address(0)

### Instance#1 : Check that `newOwner` is not address(0) before transferring ownership

```solidity
File: contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol

66:  function transferOwnership(
        address newOwner
    ) public virtual override(OwnableUnset, ILSP14Ownable2Step) onlyOwner {
69:        _transferOwnership(newOwner);
```

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol#L66C5-L69C38

## [L-02] Missing zero address check in constructor or initialize() function,specially for owner

### Instance#1 : check `newOwner_` is not address(0) before calling `_initialize`

```solidity
File:  /contracts/LSP8IdentifiableDigitalAsset/presets/LSP8CompatibleERC721MintableInit.sol
25:  function initialize(
        string memory name_,
        string memory symbol_,
        address newOwner_
    ) external virtual initializer {
        LSP8CompatibleERC721MintableInitAbstract._initialize(
            name_,
            symbol_,
            newOwner_
        );
```

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/presets/LSP8CompatibleERC721MintableInit.sol#L25C4-L34C11

# Non Critcal Issues

## [NC-01] : Named return is confusing, use explicit returns

Return from function explicitly make code readable

_1 Instances_

### Instance# 1 :

```solidity
File: contracts/LSP20CallVerification/LSP20CallVerification.sol

23:   function _verifyCall(
24:        address logicVerifier
25:    ) internal virtual returns (bool verifyAfter) {
```

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP20CallVerification/LSP20CallVerification.sol#L23C3-L25C52

## [NC-02] : No caching needed if state var used only once.

### Instance#1: `tokenOwnerOf(tokenId)` need not be cached used only once.

```solidity
File: /contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol

177: function _isOperatorOrOwner(
        address caller,
        bytes32 tokenId
    ) internal view virtual returns (bool) {
        address tokenOwner = tokenOwnerOf(tokenId);

        return (caller == tokenOwner || _operators[tokenId].contains(caller));
184:   }
```

https://github.com/code-423n4/2023-06-lukso/blob/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L177C5-L184C6
