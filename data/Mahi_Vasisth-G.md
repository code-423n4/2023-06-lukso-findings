[G-01] Use nested-if instead of '&&':
Proof:
    227. if (dataKeys.length == 0 && dataValues.length == 0)
    245. if (dataKeys.length == 0 && dataValues.length == 0)
Link: https://github.com/code-423n4/2023-06-lukso/blob/bd49f57c32a522563fc42feeee23c83c8b373405/contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol
[G-02] Use calldata as function argument instead of memory when argument is not mutated:
Proof:   
    33. string memory keyName
Link: https://github.com/code-423n4/2023-06-lukso/blob/bd49f57c32a522563fc42feeee23c83c8b373405/contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol
[G-03] Try to use do-while loop instead of for loop for gas optimization.
proof:
   256.for (uint256 ii = 0; ii < arrayLength; ) 
   292. for (uint256 ii = 0; ii < arrayLength; ) {
Link: https://github.com/code-423n4/2023-06-lukso/blob/bd49f57c32a522563fc42feeee23c83c8b373405/contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol
Proof:
    227. for (uint256 i = 0; i < fromLength; )
Link:https://github.com/code-423n4/2023-06-lukso/blob/bd49f57c32a522563fc42feeee23c83c8b373405/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol
[G-04] Don't initialize variables with default value.
Proof:
    256.for (uint256 ii = 0; ii < arrayLength; ) 
    292.for (uint256 ii = 0; ii < arrayLength; )
    328.uint256 pointer = 0;
Link: https://github.com/code-423n4/2023-06-lukso/blob/bd49f57c32a522563fc42feeee23c83c8b373405/contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol
Proof:
    227.for (uint256 i = 0; i < fromLength; )
Link: https://github.com/code-423n4/2023-06-lukso/blob/bd49f57c32a522563fc42feeee23c83c8b373405/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol
[G-05] Use <x>=<x>+1 format for incrementing or decrementing the variable value.
Proof:    
    328.pointer += 32;
    344.pointer += elementLength + 2;
Link: https://github.com/code-423n4/2023-06-lukso/blob/bd49f57c32a522563fc42feeee23c83c8b373405/contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol
[G-06] Use calldata instead of memory for the arguments which not get mutated.
Proof:
    37.string memory name_,
    38.string memory symbol_,
Link: https://github.com/code-423n4/2023-06-lukso/blob/bd49f57c32a522563fc42feeee23c83c8b373405/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAsset.sol
Proof:
    25.function setName(string memory name) public {
    29.function setName(string memory name) public {
Link: https://github.com/code-423n4/2023-06-lukso/blob/bd49f57c32a522563fc42feeee23c83c8b373405/contracts/Mocks/TargetContract.sol
        