
# VULN 1 

## [LOW] Empty receive()/payable fallback() function does not authenticate requests
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 117 at 2023-06-lukso/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol:

    receive() external payable virtual {

------------------------------------------------------------------------ 

### Mitigation 

If the intention is for the Ether to be used, the function should call another function, otherwise it should revert (e.g. require(msg.sender == address(weth))).Empty receive()/fallback() payable functions that are not used, can be removed to save deployment gas. Having no access control on the function means that someone may send Ether to the contract, and have no way to get anything back out, which is a loss of funds. If the concern is having to spend a small amount of gas to check the sender against an immutable address, the code should at least have a function to rescue unused Ether.










# VULN 2 

## [LOW] Use Ownable2Step rather than Ownable
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 7 at 2023-06-lukso/contracts/LSP0ERC725Account/LSP0ERC725Account.sol:

    OwnableUnset



Found in line 55 at 2023-06-lukso/contracts/LSP0ERC725Account/LSP0ERC725Account.sol:

        OwnableUnset._setOwner(initialOwner);


Found in line 25 at 2023-06-lukso/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol:

    OwnableUnset



Found in line 559 at 2023-06-lukso/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol:

    ) public virtual override(LSP14Ownable2Step, OwnableUnset) {


Found in line 658 at 2023-06-lukso/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol:

        override(LSP14Ownable2Step, OwnableUnset)


Found in line 10 at 2023-06-lukso/contracts/LSP0ERC725Account/LSP0ERC725AccountInitAbstract.sol:

    OwnableUnset


Found in line 65 at 2023-06-lukso/contracts/LSP0ERC725Account/LSP0ERC725AccountInitAbstract.sol:

        OwnableUnset._setOwner(initialOwner);


Found in line 9 at 2023-06-lukso/contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol:

    OwnableUnset
    

Found in line 32 at 2023-06-lukso/contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol:

abstract contract LSP14Ownable2Step is ILSP14Ownable2Step, OwnableUnset {


Found in line 68 at 2023-06-lukso/contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol:

    ) public virtual override(OwnableUnset, ILSP14Ownable2Step) onlyOwner {


Found in line 109 at 2023-06-lukso/contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol:

        override(OwnableUnset, ILSP14Ownable2Step)

------------------------------------------------------------------------ 

### Mitigation 

Ownable2Step and Ownable2StepUpgradeable prevent the contract ownership from mistakenly being transferred to an address that cannot handle it (e.g. due to a typo in the address), by requiring that the recipient of the owner permissions actively accept via a contract call of its own.










# VULN 3 

## [LOW] Avoid using tx.origin
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 103 at 2023-06-lukso/contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol:

        if (notifier == tx.origin) revert CannotRegisterEOAsAsAssets(notifier);

------------------------------------------------------------------------ 

### Mitigation 

tx.origin is a global variable in Solidity that returns the address of the account that sent the transaction. Using the variable could make a contract vulnerable if an authorized account calls a malicious contract. You can impersonate a user using a third party contract.










# VULN 4 

## [LOW] Use the safe variant and ERC721.mint
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 313 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol:

    function _mint(


Found in line 40 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/presets/LSP8MintableInitAbstract.sol:

        _mint(to, tokenId, allowNonLSP1Recipient, data);


Found in line 38 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/presets/LSP8Mintable.sol:

        _mint(to, tokenId, allowNonLSP1Recipient, data);


Found in line 19 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/presets/LSP8CompatibleERC721Mintable.sol:

        _mint(to, tokenId, allowNonLSP1Recipient, data);


Found in line 29 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/presets/LSP8CompatibleERC721MintableInitAbstract.sol:

        _mint(to, tokenId, allowNonLSP1Recipient, data);


Found in line 56 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CappedSupplyInitAbstract.sol:

    function _mint(


Found in line 66 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CappedSupplyInitAbstract.sol:

        super._mint(to, tokenId, allowNonLSP1Recipient, data);


Found in line 56 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CappedSupply.sol:

    function _mint(


Found in line 66 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CappedSupply.sol:

        super._mint(to, tokenId, allowNonLSP1Recipient, data);


Found in line 259 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol:

    function _mint(


Found in line 266 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol:

        super._mint(to, tokenId, allowNonLSP1Recipient, data);


Found in line 259 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol:

    function _mint(


Found in line 266 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol:

        super._mint(to, tokenId, allowNonLSP1Recipient, data);


Found in line 294 at 2023-06-lukso/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol:

    function _mint(


Found in line 29 at 2023-06-lukso/contracts/LSP7DigitalAsset/presets/LSP7CompatibleERC20MintableInitAbstract.sol:

        _mint(to, amount, allowNonLSP1Recipient, data);


Found in line 43 at 2023-06-lukso/contracts/LSP7DigitalAsset/presets/LSP7MintableInitAbstract.sol:

        _mint(to, amount, allowNonLSP1Recipient, data);


Found in line 40 at 2023-06-lukso/contracts/LSP7DigitalAsset/presets/LSP7Mintable.sol:

        _mint(to, amount, allowNonLSP1Recipient, data);


Found in line 19 at 2023-06-lukso/contracts/LSP7DigitalAsset/presets/LSP7CompatibleERC20Mintable.sol:

        _mint(to, amount, allowNonLSP1Recipient, data);


Found in line 135 at 2023-06-lukso/contracts/LSP7DigitalAsset/extensions/LSP7CompatibleERC20InitAbstract.sol:

    function _mint(


Found in line 142 at 2023-06-lukso/contracts/LSP7DigitalAsset/extensions/LSP7CompatibleERC20InitAbstract.sol:

        super._mint(to, amount, allowNonLSP1Recipient, data);


Found in line 53 at 2023-06-lukso/contracts/LSP7DigitalAsset/extensions/LSP7CappedSupplyInitAbstract.sol:

    function _mint(


Found in line 63 at 2023-06-lukso/contracts/LSP7DigitalAsset/extensions/LSP7CappedSupplyInitAbstract.sol:

        super._mint(to, amount, allowNonLSP1Recipient, data);


Found in line 53 at 2023-06-lukso/contracts/LSP7DigitalAsset/extensions/LSP7CappedSupply.sol:

    function _mint(


Found in line 63 at 2023-06-lukso/contracts/LSP7DigitalAsset/extensions/LSP7CappedSupply.sol:

        super._mint(to, amount, allowNonLSP1Recipient, data);


Found in line 127 at 2023-06-lukso/contracts/LSP7DigitalAsset/extensions/LSP7CompatibleERC20.sol:

    function _mint(


Found in line 134 at 2023-06-lukso/contracts/LSP7DigitalAsset/extensions/LSP7CompatibleERC20.sol:

        super._mint(to, amount, allowNonLSP1Recipient, data);

------------------------------------------------------------------------ 

### Mitigation 

.mint won’t check if the recipient is able to receive the NFT. If an incorrect address is passed, it will result in a silent failure and loss of asset. OpenZeppelin recommendation is to use the safe variant of _mint. Replace _mint() with _safeMint().










# VULN 5 

## [LOW] Use safeTransferOwnership instead of transferOwnership function
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 557 at 2023-06-lukso/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol:

    function transferOwnership(


Found in line 66 at 2023-06-lukso/contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol:

    function transferOwnership(


Found in line 54 at 2023-06-lukso/contracts/LSP14Ownable2Step/ILSP14Ownable2Step.sol:

    function transferOwnership(address newOwner) external;


Found in line 506 at 2023-06-lukso/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol:

            erc725Function == ILSP14Ownable2Step.transferOwnership.selector ||

------------------------------------------------------------------------ 

### Mitigation 

Use a 2 structure transferOwnership which is safer. safeTransferOwnership, use it is more secure due to 2-stage ownership transfer. https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable2Step.sol










# VULN 6 

## [LOW] Add parameter to event-emit
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 24 at 2023-06-lukso/contracts/LSP14Ownable2Step/ILSP14Ownable2Step.sol:

    event RenounceOwnershipStarted();


Found in line 29 at 2023-06-lukso/contracts/LSP14Ownable2Step/ILSP14Ownable2Step.sol:

    event OwnershipRenounced();

------------------------------------------------------------------------ 

### Mitigation 

Some event-emit description doesn’t have parameter. Add parameter for front-end website or client app, they can has that something has happened on the blockchain.










# VULN 7 

## [LOW] Immutables should be in uppercase
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 19 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CappedSupply.sol:

    uint256 private immutable _tokenSupplyCap;


Found in line 17 at 2023-06-lukso/contracts/LSP7DigitalAsset/extensions/LSP7CappedSupply.sol:

    uint256 private immutable _tokenSupplyCap;

------------------------------------------------------------------------ 

### Mitigation 

Immutables should be in uppercase, it helps to distinguish immutables from other types of variables and provides better code readability.
