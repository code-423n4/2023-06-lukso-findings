Gas Optimization by removing the "Payable" modifier and removing the "msg.value" check. 

In Line 78, the function is mark as payable. But the first operation is to revert if the "msg.value" is 0. 
It's more gas efficient to remove the payable modifier and remove as well this "if" statement.
The function will revert is the user try to send some ETH as is not a "payable" function anymore.

 ```solidity
    function setDataBatch(
        bytes32[] memory dataKeys,
        bytes[] memory dataValues
    ) public payable virtual override onlyOwner {
        /// @dev do not allow to send value by default when setting data in ERC725Y
        if (msg.value != 0) revert ERC725Y_MsgValueDisallowed();
        ...
    }
```
[ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725YCore.sol](https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725YCore.sol)
