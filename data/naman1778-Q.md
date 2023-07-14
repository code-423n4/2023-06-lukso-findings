## [N-01] Use a modifier for access control

Consider using a modifier to implement access control instead of inlining the condition/requirement in the function’s body.

There are 7 instances of this issue in 5 files

    File: LSP8IdentifiableDigitalAssetCore.sol	

    111: if (tokenOwner != msg.sender) {
    112:     revert LSP8NotTokenOwner(tokenOwner, tokenId, msg.sender);
    113: }

    135: if (tokenOwner != msg.sender) {
    136      revert LSP8NotTokenOwner(tokenOwner, tokenId, msg.sender);
    137: }

    198: address operator = msg.sender;
    200: if (!_isOperatorOrOwner(operator, tokenId)) {
    201:     revert LSP8NotTokenOperator(tokenId, operator);
    202: }

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol
 
    File: LSP8CompatibleERC721InitAbstract.sol	

    233: address operator = msg.sender;
    235: if (
    236:     !isApprovedForAll(from, operator) &&
    237:     !_isOperatorOrOwner(operator, tokenId)
    238: ) {
    239:     revert LSP8NotTokenOperator(tokenId, operator);
    240: }

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol

    File: LSP8CompatibleERC721.sol	

    233: address operator = msg.sender;
    235: if (
    236:     !isApprovedForAll(from, operator) &&
    237:     !_isOperatorOrOwner(operator, tokenId)
    238: ) {
    239:     revert LSP8NotTokenOperator(tokenId, operator);
    240: }

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol

    File: LSP8Burnable.sol	

    17: if (!_isOperatorOrOwner(msg.sender, tokenId)) {
    18:     revert LSP8NotTokenOperator(tokenId, msg.sender);
    19: }

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8Burnable.sol

    File: LSP14Ownable2Step.sol	

    137: require(
    138:     msg.sender == pendingOwner(),
    139:     "LSP14: caller is not the pendingOwner"
    140: );

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol


## [N-02] Contract files should define a locked compiler version

Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.

All the contracts mentioned in scope are using floating compiler version


## [N-03] According to the syntax rules, use *=> mapping (* instead of *=> mapping(* using spaces as keyword

There are 4 instances of this issue in 4 files:

    File: LSP6KeyManagerCore.sol	

    85: mapping(address => mapping(uint256 => uint256)) internal _nonceStore;

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol

    File: LSP7DigitalAssetCore.sol	

    42: mapping(address => mapping(address => uint256))

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol

    File: LSP8CompatibleERC721InitAbstract.sol	

    59: mapping(address => mapping(address => bool)) private _operatorApprovals;

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol

    File: LSP8CompatibleERC721.sol	

    59: mapping(address => mapping(address => bool)) private _operatorApprovals;

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol

## [N-04] Inconsistent Solidity Versions

The contracts listed in scope are using multiple compiler verion. Instead of making use of multiple compiler versions make use of same compiler version across all the contracts

## [N-05] Contract Owner Has Too Many Privileges

The owner of the contracts has too many privileges relative to standard users. The consequence is disastrous if the contract owner’s private key has been compromised. And, in the event the key was lost or unrecoverable, no implementation upgrades and system parameter updates will ever be possible.

For a project this grand, it increases the likelihood that the owner will be targeted by an attacker, especially given the insufficient protection on sensitive owner private keys. The concentration of privileges creates a single point of failure; and, here are some of the incidents that could possibly transpire:

Transfer ownership and mess up with all the setter functions, hijacking the entire protocol.

Consider:

-- splitting privileges (e.g. via the multisig option) to ensure that no one address has excessive ownership of the system,
-- clearly documenting the functions and implementations the owner can change,
-- documenting the risks associated with privileged users and single points of failure, and
-- ensuring that users are aware of all the risks associated with the system.


## [N-06] Empty/Unused Function Parameters

Empty or unused function parameters should be commented out as a better and declarative way to silence runtime warning messages. As an example, the following function may have these parameters refactored to:

There are 4 instances of this issue in 4 files:

    File: LSP1UniversalReceiverDelegateUP.sol	

    78: function universalReceiver(
    79:     bytes32 typeId,
    80:     bytes memory /* data */
    81: ) public payable virtual returns (bytes memory result) {

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol

    File: LSP6KeyManagerCore.sol	

    303: function lsp20VerifyCallResult(
    304:     bytes32 /*callHash*/,
    305:     bytes memory /*result*/
    306: ) external returns (bytes4) {

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol

    File: LSP8CompatibleERC721InitAbstract.sol	

    94: function tokenURI(
    95:     uint256 /* tokenId */
    96: ) public view virtual returns (string memory) {

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol

    File: LSP8CompatibleERC721.sol	

    94: function tokenURI(
    95:     uint256 /* tokenId */
    96: ) public view virtual returns (string memory) {

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol
