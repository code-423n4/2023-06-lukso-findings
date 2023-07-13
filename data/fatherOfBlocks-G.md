**ERC725YCore.sol**
- L15/66/78/81/85 - The entire error file is imported, but only 4 are used, this generates an extra gas expense, only the errors that are going to be used should be imported.


**OwnableUnset.sol**
- L22 - A getter function is created, since the variable in memory is private, but instead of making the external function, it is public. Therefore it should be set to external.

- L70/71/72 - Gas could be saved by implementing the setting in this way:

	emit OwnershipTransferred(_owner, newOwner);
	_owner = newOwner;


**LSP1Utils.sol**
- L17 - The LSP9Constants file is imported, but not all the constants are used, therefore gas could be saved by importing only the necessary constants.
 

**LSP1Errors.sol**
- L16 - An error CallerNotLSP6LinkedTarget is created, which is never used, therefore it should be eliminated, therefore gas would be saved by not creating it.


**LSP14Ownable2Step.sol**
- L59 - A getter function is created, since the variable in memory is private, but instead of making the external function, it is public. Therefore it should be set to external.


**LSP8Constants.sol**
- L10/13 - Constants are created that are never used: _LSP8_METADATA_ADDRESS_KEY_PREFIX, _LSP8_METADATA_JSON_KEY_PREFIX, therefore they should be eliminated, since unnecessary gas and storage costs are generated.


**LSP6SetDataModule.sol**
- L127 - Variables need not be initialized to false
The default value for variables is false, so initializing them to false is superfluous.


**LSP6KeyManagerCore.sol**
- L256/323 - Variables need not be initialized to false
The default value for variables is false, so initializing them to false is superfluous.

