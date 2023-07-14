# Low Risk Findings


## LSP6KeyManager#constructor: `target_` can be EOA

**Description:**
`target_` should be checked more strictly. In the current contract, it only checks if `target_` is zero address.

## Impact

The target of LSP6KeyManager(ERC725 Account) can be set wrong, thus making the key manager contract useless.

## Proof of Concept
```solidity
// LSP6KeyManager.sol
// #L18

contract LSP6KeyManager is LSP6KeyManagerCore {
    constructor(address target_) {
        if (target_ == address(0)) revert InvalidLSP6Target();
        _target = target_;
        _setupLSP6ReentrancyGuard();
    }
```

 

## Tools Used
Hand

## Recommended Mitigation Steps

Checking the codelength of `target_` will be sufficient.

```solidity

contract LSP6KeyManager is LSP6KeyManagerCore {
    constructor(address target_) {
        if (target_ == address(0) || target_.code.length == 0) revert InvalidLSP6Target();
        _target = target_;
        _setupLSP6ReentrancyGuard();
    }
```
