## _checkOnERC721Received Method Duplication Definition

https://github.com/code-423n4/2023-06-lukso/blob/9dbc96410b3052fc0fd9d423249d1fa42958cae8/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol#L297-L339

https://github.com/code-423n4/2023-06-lukso/blob/9dbc96410b3052fc0fd9d423249d1fa42958cae8/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol#L297-L339

## supportsInterface Method Duplication Definition

https://github.com/code-423n4/2023-06-lukso/blob/9dbc96410b3052fc0fd9d423249d1fa42958cae8/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol#L73-L89

https://github.com/code-423n4/2023-06-lukso/blob/9dbc96410b3052fc0fd9d423249d1fa42958cae8/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol#L73-L89

## Recommended

contracts/LSP8IdentifiableDigitalAsset/extensions 

There are many duplicate codes in this directory, it is recommended to define a parent class
