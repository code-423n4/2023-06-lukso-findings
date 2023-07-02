

# VULN 1 

## [GAS] ++i/i++ should be unchecked{++i}/unchecked{i++} and ++i costs less gas than i++
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 173 at 2023-06-lukso/contracts/LSP6KeyManager/LSP6Utils.sol:

        for (uint256 i = 0; i < permissions.length; i++) {


Found in line 384 at 2023-06-lukso/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol:

        _nonceStore[signer][nonce >> 128]++;


Found in line 156 at 2023-06-lukso/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol:

                inputDataKeysAllowed++;


Found in line 743 at 2023-06-lukso/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol:

                        allowedDataKeysFound++;


Found in line 771 at 2023-06-lukso/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol:

                jj++;

------------------------------------------------------------------------ 

### Mitigation 

++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, as is the case when used in for and while-loops. Moreover ++i costs less gas than i++, especially when its used in for-loops (--i/i-- too).










# VULN 2 

## [GAS] Don’t initialize variables with default value
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 227 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol:

        for (uint256 i = 0; i < fromLength; ) {


Found in line 273 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol:

        for (uint256 i = 0; i < operatorListLength; ) {


Found in line 396 at 2023-06-lukso/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol:

            for (uint256 i = 0; i < dataKeys.length; ) {


Found in line 411 at 2023-06-lukso/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol:

        for (uint256 i = 0; i < dataKeys.length; ) {


Found in line 261 at 2023-06-lukso/contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol:

        for (uint256 ii = 0; ii < arrayLength; ) {


Found in line 292 at 2023-06-lukso/contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol:

        for (uint256 ii = 0; ii < arrayLength; ) {


Found in line 328 at 2023-06-lukso/contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol:

        uint256 pointer = 0;


Found in line 171 at 2023-06-lukso/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol:

        for (uint256 i = 0; i < fromLength; ) {


Found in line 94 at 2023-06-lukso/contracts/LSP6KeyManager/LSP6Utils.sol:

        uint256 pointer = 0;


Found in line 123 at 2023-06-lukso/contracts/LSP6KeyManager/LSP6Utils.sol:

        uint256 pointer = 0;


Found in line 172 at 2023-06-lukso/contracts/LSP6KeyManager/LSP6Utils.sol:

        uint256 result = 0;


Found in line 173 at 2023-06-lukso/contracts/LSP6KeyManager/LSP6Utils.sol:

        for (uint256 i = 0; i < permissions.length; i++) {


Found in line 163 at 2023-06-lukso/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol:

        for (uint256 ii = 0; ii < payloads.length; ) {


Found in line 223 at 2023-06-lukso/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol:

        for (uint256 ii = 0; ii < payloads.length; ) {


Found in line 272 at 2023-06-lukso/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol:

        for (uint256 ii = 0; ii < allowedCalls.length; ii += 34) {


Found in line 129 at 2023-06-lukso/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol:

        uint256 inputDataKeysAllowed = 0;


Found in line 133 at 2023-06-lukso/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol:

        uint256 ii = 0;

------------------------------------------------------------------------ 

### Mitigation 

In such cases, initializing the variables with default values would be unnecessary and can be considered a waste of gas. Additionally, initializing variables with default values can sometimes lead to unnecessary storage operations, which can increase gas costs. For example, if you have a large array of variables, initializing them all with default values can result in a lot of unnecessary storage writes, which can increase the gas costs of your contract.










# VULN 3 

## [GAS] Use Custom Errors
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 191 at 2023-06-lukso/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol:

                    revert("LSP0: batchCalls reverted");


Found in line 36 at 2023-06-lukso/contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol:

        require(dataKey.length >= 2, "MUST be longer than 2 characters");


Found in line 166 at 2023-06-lukso/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol:

            revert NotAuthorised(controller, "SUPER_TRANSFERVALUE");

------------------------------------------------------------------------ 

### Mitigation 

Instead of using error strings, to reduce deployment and runtime cost, you should use Custom Errors. This would save both deployment and runtime cost.










# VULN 4 

## [GAS] Use calldata instead of memory for function arguments that do not get mutated
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 359 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol:

    function _burn(bytes32 tokenId, bytes memory data) internal virtual {


Found in line 16 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8Burnable.sol:

    function burn(bytes32 tokenId, bytes memory data) public {


Found in line 231 at 2023-06-lukso/contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol:

    function isEncodedArray(bytes memory data) internal pure returns (bool) {


Found in line 17 at 2023-06-lukso/contracts/LSP7DigitalAsset/extensions/LSP7Burnable.sol:

    function burn(address from, uint256 amount, bytes memory data) public {


Found in line 19 at 2023-06-lukso/contracts/LSP7DigitalAsset/extensions/LSP7BurnableInitAbstract.sol:

    function burn(address from, uint256 amount, bytes memory data) public {


Found in line 84 at 2023-06-lukso/contracts/LSP20CallVerification/LSP20CallVerification.sol:

    function _revert(bool postCall, bytes memory returnedData) internal pure {

------------------------------------------------------------------------ 

### Mitigation 

Mark data types as calldata instead of memory where possible. This makes it so that the data is not automatically loaded into memory. If the data passed into the function does not need to be changed (like updating values in an array), it can be passed in as calldata. The one exception to this is if the argument must later be passed into another function that takes an argument that specifies memory storage.










# VULN 5 

## [GAS] Cache array length outside of loop
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 175 at 2023-06-lukso/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol:

        for (uint256 i; i < data.length; ) {


Found in line 396 at 2023-06-lukso/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol:

            for (uint256 i = 0; i < dataKeys.length; ) {


Found in line 411 at 2023-06-lukso/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol:

        for (uint256 i = 0; i < dataKeys.length; ) {


Found in line 173 at 2023-06-lukso/contracts/LSP6KeyManager/LSP6Utils.sol:

        for (uint256 i = 0; i < permissions.length; i++) {


Found in line 163 at 2023-06-lukso/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol:

        for (uint256 ii = 0; ii < payloads.length; ) {


Found in line 223 at 2023-06-lukso/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol:

        for (uint256 ii = 0; ii < payloads.length; ) {


Found in line 272 at 2023-06-lukso/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol:

        for (uint256 ii = 0; ii < allowedCalls.length; ii += 34) {

------------------------------------------------------------------------ 

### Mitigation 

If not cached, the solidity compiler will always read the length of the array during each iteration. That is, if it is a storage array, this is an extra sload operation (100 additional extra gas for each iteration except for the first) and if it is a memory array, this is an extra mload operation (3 additional gas for each iteration except for the first).










# VULN 6 

## [GAS] Use assembly to check for address(0)
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 50 at 2023-06-lukso/contracts/LSP17ContractExtension/LSP17Extendable.sol:

        if (erc165Extension == address(0)) return false;


Found in line 91 at 2023-06-lukso/contracts/LSP17ContractExtension/LSP17Extendable.sol:

        if (extension == address(0))


Found in line 84 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol:

        if (tokenOwner == address(0)) {


Found in line 115 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol:

        if (operator == address(0)) {


Found in line 139 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol:

        if (operator == address(0)) {


Found in line 319 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol:

        if (to == address(0)) {


Found in line 410 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol:

        if (to == address(0)) {


Found in line 36 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8Enumerable.sol:

        if (from == address(0)) {


Found in line 40 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8Enumerable.sol:

        } else if (to == address(0)) {


Found in line 38 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8EnumerableInitAbstract.sol:

        if (from == address(0)) {


Found in line 42 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8EnumerableInitAbstract.sol:

        } else if (to == address(0)) {


Found in line 801 at 2023-06-lukso/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol:

        if (msg.sig == bytes4(0) && extension == address(0)) return;


Found in line 804 at 2023-06-lukso/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol:

        if (extension == address(0))


Found in line 268 at 2023-06-lukso/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol:

        if (operator == address(0)) {


Found in line 300 at 2023-06-lukso/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol:

        if (to == address(0)) {


Found in line 343 at 2023-06-lukso/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol:

        if (from == address(0)) {


Found in line 406 at 2023-06-lukso/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol:

        if (from == address(0) || to == address(0)) {


Found in line 19 at 2023-06-lukso/contracts/LSP6KeyManager/LSP6KeyManager.sol:

        if (target_ == address(0)) revert InvalidLSP6Target();


Found in line 21 at 2023-06-lukso/contracts/LSP6KeyManager/LSP6KeyManagerInitAbstract.sol:

        if (target_ == address(0)) revert InvalidLSP6Target();

------------------------------------------------------------------------ 

### Mitigation 

Using assembly to check for the zero address can result in significant gas savings compared to using a Solidity expression, especially if the check is performed frequently or in a loop. However, it's important to note that using assembly can make the code less readable and harder to maintain, so it should be used judiciously and with caution.










# VULN 7 

## [GAS] Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 336 at 2023-06-lukso/contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol:

            uint256 elementLength = uint16(


Found in line 65 at 2023-06-lukso/contracts/LSP7DigitalAsset/ILSP7DigitalAsset.sol:

    function decimals() external view returns (uint8);


Found in line 54 at 2023-06-lukso/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol:

    function decimals() public view virtual returns (uint8) {


Found in line 98 at 2023-06-lukso/contracts/LSP6KeyManager/LSP6Utils.sol:

            uint256 elementLength = uint16(


Found in line 128 at 2023-06-lukso/contracts/LSP6KeyManager/LSP6Utils.sol:

            uint256 elementLength = uint16(


Found in line 395 at 2023-06-lukso/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol:

            allowedStandard == bytes4(type(uint32).max) ||


Found in line 414 at 2023-06-lukso/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol:

            allowedFunction == bytes4(type(uint32).max) ||


Found in line 544 at 2023-06-lukso/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol:

            length = uint16(


Found in line 664 at 2023-06-lukso/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol:

            length = uint16(

------------------------------------------------------------------------ 

### Mitigation 

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size. https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html. Each operation involving a uint8 costs an extra 22-28 gas (depending on whether the other operand is also a variable of type uint8) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so. Use a larger size then downcast where needed.










# VULN 8 

## [GAS] Public Functions to external
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 624 at 2023-06-lukso/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol:

    function acceptOwnership() public virtual override {


Found in line 25 at 2023-06-lukso/contracts/LSP4DigitalAssetMetadata/LSP4Compatibility.sol:

    function name() public view virtual override returns (string memory) {


Found in line 34 at 2023-06-lukso/contracts/LSP4DigitalAssetMetadata/LSP4Compatibility.sol:

    function symbol() public view virtual override returns (string memory) {

------------------------------------------------------------------------ 

### Mitigation 

The following functions could be set external to save gas and improve code quality. External call cost is less expensive than of public functions.










# VULN 9 

## [GAS] <x> += <y> Costs More Gas Than <x> = <x> + <y> For State Variables
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 332 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol:

        _existingTokens += 1;


Found in line 269 at 2023-06-lukso/contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol:

            pointer += 32;


Found in line 299 at 2023-06-lukso/contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol:

            pointer += 32;


Found in line 344 at 2023-06-lukso/contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol:

            pointer += elementLength + 2;


Found in line 309 at 2023-06-lukso/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol:

        _existingTokens += amount;


Found in line 311 at 2023-06-lukso/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol:

        _tokenOwnerBalances[to] += amount;


Found in line 420 at 2023-06-lukso/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol:

        _tokenOwnerBalances[to] += amount;


Found in line 108 at 2023-06-lukso/contracts/LSP6KeyManager/LSP6Utils.sol:

            pointer += elementLength + 2;


Found in line 138 at 2023-06-lukso/contracts/LSP6KeyManager/LSP6Utils.sol:

            pointer += elementLength + 2;


Found in line 174 at 2023-06-lukso/contracts/LSP6KeyManager/LSP6Utils.sol:

            result += uint256(permissions[i]);


Found in line 164 at 2023-06-lukso/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol:

            if ((totalValues += values[ii]) > msg.value) {


Found in line 224 at 2023-06-lukso/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol:

            if ((totalValues += values[ii]) > msg.value) {


Found in line 272 at 2023-06-lukso/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol:

        for (uint256 ii = 0; ii < allowedCalls.length; ii += 34) {

------------------------------------------------------------------------ 

### Mitigation 

When you use the += operator on a state variable, the EVM has to perform three operations: load the current value of the state variable, add the new value to it, and then store the result back in the state variable. On the other hand, when you use the = operator and then add the values separately, the EVM only needs to perform two operations: load the current value of the state variable and add the new value to it. Better use <x> = <x> + <y> For State Variables.










# VULN 10 

## [GAS] Use hardcode address instead address(this)
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 176 at 2023-06-lukso/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol:

            (bool success, bytes memory result) = address(this).delegatecall(


Found in line 127 at 2023-06-lukso/contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol:

        if (newOwner == address(this)) revert CannotTransferOwnershipToSelf();


Found in line 365 at 2023-06-lukso/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol:

        address signer = address(this)


Found in line 86 at 2023-06-lukso/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol:

        if (to == address(this)) {

------------------------------------------------------------------------ 

### Mitigation 

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this.










# VULN 11 

## [GAS] Do not calculate constants
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 23 at 2023-06-lukso/contracts/LSP6KeyManager/LSP6Constants.sol:

bytes10 constant _LSP6KEY_ADDRESSPERMISSIONS_PERMISSIONS_PREFIX = 0x4b80742de2bf82acb363; // AddressPermissions:Permissions:<address> --> bytes32


Found in line 26 at 2023-06-lukso/contracts/LSP6KeyManager/LSP6Constants.sol:

bytes10 constant _LSP6KEY_ADDRESSPERMISSIONS_AllowedERC725YDataKeys_PREFIX = 0x4b80742de2bf866c2911; // AddressPermissions:AllowedERC725YDataKeys:<address> --> bytes[CompactBytesArray]


Found in line 29 at 2023-06-lukso/contracts/LSP6KeyManager/LSP6Constants.sol:

bytes10 constant _LSP6KEY_ADDRESSPERMISSIONS_ALLOWEDCALLS_PREFIX = 0x4b80742de2bf393a64c7; // AddressPermissions:AllowedCalls:<address>


Found in line 62 at 2023-06-lukso/contracts/LSP6KeyManager/LSP6Constants.sol:

bytes4 constant _ALLOWEDCALLS_TRANSFERVALUE   = 0x00000001; // 0000 0001


Found in line 63 at 2023-06-lukso/contracts/LSP6KeyManager/LSP6Constants.sol:

bytes4 constant _ALLOWEDCALLS_CALL            = 0x00000002; // 0000 0010


Found in line 64 at 2023-06-lukso/contracts/LSP6KeyManager/LSP6Constants.sol:

bytes4 constant _ALLOWEDCALLS_STATICCALL      = 0x00000004; // 0000 0100


Found in line 65 at 2023-06-lukso/contracts/LSP6KeyManager/LSP6Constants.sol:

bytes4 constant _ALLOWEDCALLS_DELEGATECALL    = 0x00000008; // 0000 1000

------------------------------------------------------------------------ 

### Mitigation 

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.










# VULN 12 

## [GAS] >= costs less gas than >
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 35 at 2023-06-lukso/contracts/LSP0ERC725Account/LSP0Utils.sol:

     * @return the bytes value stored under the `LSP1UniversalReceiverDelegate:<bytes32>` data key.


Found in line 236 at 2023-06-lukso/contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol:

        if (nbOfBytes < offset + 32) return false;


Found in line 242 at 2023-06-lukso/contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol:

        if (nbOfBytes < (offset + 32 + (arrayLength * 32))) return false;


Found in line 79 at 2023-06-lukso/contracts/LSP20CallVerification/LSP20CallVerification.sol:

            returnedData.length < 32 ||


Found in line 86 at 2023-06-lukso/contracts/LSP20CallVerification/LSP20CallVerification.sol:

        if (returnedData.length > 0) {


Found in line 137 at 2023-06-lukso/contracts/LSP6KeyManager/LSP6Utils.sol:

            if (elementLength == 0 || elementLength > 32) return false;

------------------------------------------------------------------------ 

### Mitigation 

The compiler uses opcodes GT and ISZERO for solidity code that uses >, but only requires LT for >=, which saves 3 gas.










# VULN 13 

## [GAS] With assembly, .call (bool success) transfer can be done gas-optimized
------------------------------------------------------------------------ 

### Proof of concept 

Found in 2023-06-lukso/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol (function batchCalls():

    function batchCalls(

        bytes[] calldata data

    ) public returns (bytes[] memory results) {

        results = new bytes[](data.length);

        for (uint256 i; i < data.length; ) {

            (bool success, bytes memory result) = address(this).delegatecall(

                data[i]

            );



            if (!success) {

                // Look for revert reason and bubble it up if present



Found in 2023-06-lukso/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol (function isValidSignature():

    ) public view virtual returns (bytes4 magicValue) {

        address _owner = owner();



        // If owner is a contract

        if (_owner.code.length > 0) {

            (bool success, bytes memory result) = _owner.staticcall(

                abi.encodeWithSelector(

                    IERC1271.isValidSignature.selector,

                    dataHash,

                    signature

                )



Found in 2023-06-lukso/contracts/LSP1UniversalReceiver/LSP1Utils.sol (function callUniversalReceiverWithCallerInfos():

            ),

            msgSender,

            msgValue

        );



        (bool success, bytes memory result) = universalReceiverDelegate.call(

            callData

        );

        Address.verifyCallResult(

            success,

            result,



Found in 2023-06-lukso/contracts/LSP20CallVerification/LSP20CallVerification.sol (function _verifyCall():

     * Returns whether a verification after the execution should happen based on the last byte of the magicValue

     */

    function _verifyCall(

        address logicVerifier

    ) internal virtual returns (bool verifyAfter) {

        (bool success, bytes memory returnedData) = logicVerifier.call(

            abi.encodeWithSelector(

                ILSP20.lsp20VerifyCall.selector,

                msg.sender,

                msg.value,

                msg.data



Found in 2023-06-lukso/contracts/LSP20CallVerification/LSP20CallVerification.sol (function _verifyCallResult():

     */

    function _verifyCallResult(

        address logicVerifier,

        bytes memory callResult

    ) internal virtual {

        (bool success, bytes memory returnedData) = logicVerifier.call(

            abi.encodeWithSelector(

                ILSP20.lsp20VerifyCallResult.selector,

                keccak256(abi.encodePacked(msg.sender, msg.value, msg.data)),

                callResult

            )



Found in 2023-06-lukso/contracts/LSP20CallVerification/LSP20CallVerification.sol (function _validateCall():

        ) revert LSP20InvalidMagicValue(true, returnedData);

    }



    function _validateCall(

        bool postCall,

        bool success,

        bytes memory returnedData

    ) internal pure {

        if (!success) _revert(postCall, returnedData);



        // check if the returned data contains at least 32 bytes, potentially an abi encoded bytes4 value



Found in 2023-06-lukso/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol (function _executePayload():

     */

    function _executePayload(

        uint256 msgValue,

        bytes calldata payload

    ) internal virtual returns (bytes memory) {

        (bool success, bytes memory returnData) = _target.call{

            value: msgValue,

            gas: gasleft()

        }(payload);

        bytes memory result = Address.verifyCallResult(

            success,


------------------------------------------------------------------------ 

### Mitigation 

return data (bool success,) has to be stored due to EVM architecture, but in a usage like below, ‘out’ and ‘outsize’ values are given (0,0), this storage disappears and gas optimization is provided.










# VULN 14 

## [GAS] Empty blocks should be removed or emit something
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 40 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAsset.sol:

    ) LSP4DigitalAssetMetadata(name_, symbol_, newOwner_) {}


Found in line 448 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol:

    ) internal virtual {}


Found in line 27 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/presets/LSP8Mintable.sol:

    ) LSP8IdentifiableDigitalAsset(name_, symbol_, newOwner_) {}


Found in line 11 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/presets/LSP8CompatibleERC721Mintable.sol:

    ) LSP8CompatibleERC721(name_, symbol_, newOwner_) {}


Found in line 71 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol:

    ) LSP8IdentifiableDigitalAsset(name_, symbol_, newOwner_) {}


Found in line 446 at 2023-06-lukso/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol:

    ) internal virtual {}


Found in line 29 at 2023-06-lukso/contracts/LSP7DigitalAsset/presets/LSP7Mintable.sol:

    ) LSP7DigitalAsset(name_, symbol_, newOwner_, isNonDivisible_) {}


Found in line 11 at 2023-06-lukso/contracts/LSP7DigitalAsset/presets/LSP7CompatibleERC20Mintable.sol:

    ) LSP7CompatibleERC20(name_, symbol_, newOwner_) {}


Found in line 36 at 2023-06-lukso/contracts/LSP7DigitalAsset/extensions/LSP7CompatibleERC20.sol:

    ) LSP7DigitalAsset(name_, symbol_, newOwner_, false) {}

------------------------------------------------------------------------ 

### Mitigation 

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting. If the contract is meant to be extended, the contract should be abstract and the function signatures be added without any default implementation. If the block is an empty if-statement block to avoid doing subsequent checks in the else-if/else conditions, the else-if/else conditions should be nested under the negation of the if-statement, because they involve different classes of checks, which may lead to the introduction of errors when the code is later modified (if(x){}else if(y){...}else{...} => if(!x){if(y){...}else{...}}).










# VULN 15 

## [GAS] Use double require instead of using &&
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 302 at 2023-06-lukso/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol:

                _isAllowedCallType(allowedCall, requiredCallTypes) &&



Found in line 415 at 2023-06-lukso/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol:

            (isFunctionCall && (requiredFunction == allowedFunction));


------------------------------------------------------------------------ 

### Mitigation 

Using double require instead of operator && can save more gas. When having a require statement with 2 or more expressions needed, place the expression that cost less gas first.










# VULN 16 

## [GAS] Use assembly to write address storage values
------------------------------------------------------------------------ 

### Proof of concept 

Found in lines 36 to 40 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAsset.sol:

    constructor(
        string memory name_,
        string memory symbol_,
        address newOwner_
    ) LSP4DigitalAssetMetadata(name_, symbol_, newOwner_) {}


Found in lines 23 to 27 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/presets/LSP8Mintable.sol:

    constructor(
        string memory name_,
        string memory symbol_,
        address newOwner_
    ) LSP8IdentifiableDigitalAsset(name_, symbol_, newOwner_) {}


Found in lines 7 to 11 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/presets/LSP8CompatibleERC721Mintable.sol:

    constructor(
        string memory name_,
        string memory symbol_,
        address newOwner_
    ) LSP8CompatibleERC721(name_, symbol_, newOwner_) {}


Found in lines 67 to 71 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol:

    constructor(
        string memory name_,
        string memory symbol_,
        address newOwner_
    ) LSP8IdentifiableDigitalAsset(name_, symbol_, newOwner_) {}


Found in lines 31 to 35 at 2023-06-lukso/contracts/LSP4DigitalAssetMetadata/LSP4DigitalAssetMetadata.sol:

    constructor(
        string memory name_,
        string memory symbol_,
        address newOwner_
    ) ERC725Y(newOwner_) {


Found in lines 39 to 44 at 2023-06-lukso/contracts/LSP7DigitalAsset/LSP7DigitalAsset.sol:

    constructor(
        string memory name_,
        string memory symbol_,
        address newOwner_,
        bool isNonDivisible_
    ) LSP4DigitalAssetMetadata(name_, symbol_, newOwner_) {


Found in lines 24 to 29 at 2023-06-lukso/contracts/LSP7DigitalAsset/presets/LSP7Mintable.sol:

    constructor(
        string memory name_,
        string memory symbol_,
        address newOwner_,
        bool isNonDivisible_
    ) LSP7DigitalAsset(name_, symbol_, newOwner_, isNonDivisible_) {}


Found in lines 7 to 11 at 2023-06-lukso/contracts/LSP7DigitalAsset/presets/LSP7CompatibleERC20Mintable.sol:

    constructor(
        string memory name_,
        string memory symbol_,
        address newOwner_
    ) LSP7CompatibleERC20(name_, symbol_, newOwner_) {}


Found in lines 32 to 36 at 2023-06-lukso/contracts/LSP7DigitalAsset/extensions/LSP7CompatibleERC20.sol:

    constructor(
        string memory name_,
        string memory symbol_,
        address newOwner_
    ) LSP7DigitalAsset(name_, symbol_, newOwner_, false) {}

------------------------------------------------------------------------ 

### Mitigation 

Use assembly to write address storage values. Here are a few reasons: * Reduced opcode usage: When using assembly, you can directly manipulate storage values using lower-level instructions like sstore (storage store) instead of relying on higher-level Solidity storage assignments. These direct operations typically result in fewer opcode executions, reducing gas costs. * Avoiding unnecessary checks: Solidity storage assignments often involve additional checks and operations, such as enforcing security modifiers or triggering events. By using assembly, you can bypass these additional checks and perform the necessary storage operations directly, resulting in gas savings. * Optimized packing: Assembly provides greater flexibility in packing and unpacking data structures. By carefully arranging and manipulating the storage layout in assembly, you can achieve more efficient storage utilization and minimize wasted storage space. * Fine-grained control: Assembly allows for precise control over gas-consuming operations. You can optimize gas usage by selecting specific instructions and minimizing unnecessary operations or data copying.










# VULN 17 

## [GAS] Use ERC721A instead ERC721
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 6 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/presets/LSP8CompatibleERC721MintableInit.sol:

    LSP8CompatibleERC721MintableInitAbstract


Found in line 7 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/presets/LSP8CompatibleERC721MintableInit.sol:

} from "./LSP8CompatibleERC721MintableInitAbstract.sol";


Found in line 9 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/presets/LSP8CompatibleERC721MintableInit.sol:

contract LSP8CompatibleERC721MintableInit is


Found in line 10 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/presets/LSP8CompatibleERC721MintableInit.sol:

    LSP8CompatibleERC721MintableInitAbstract


Found in line 30 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/presets/LSP8CompatibleERC721MintableInit.sol:

        LSP8CompatibleERC721MintableInitAbstract._initialize(


Found in line 6 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/presets/LSP8CompatibleERC721Mintable.sol:

contract LSP8CompatibleERC721Mintable is LSP8CompatibleERC721 {


Found in line 11 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/presets/LSP8CompatibleERC721Mintable.sol:

    ) LSP8CompatibleERC721(name_, symbol_, newOwner_) {}


Found in line 6 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/presets/LSP8CompatibleERC721MintableInitAbstract.sol:

    LSP8CompatibleERC721InitAbstract


Found in line 7 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/presets/LSP8CompatibleERC721MintableInitAbstract.sol:

} from "../extensions/LSP8CompatibleERC721InitAbstract.sol";


Found in line 9 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/presets/LSP8CompatibleERC721MintableInitAbstract.sol:

contract LSP8CompatibleERC721MintableInitAbstract is


Found in line 10 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/presets/LSP8CompatibleERC721MintableInitAbstract.sol:

    LSP8CompatibleERC721InitAbstract


Found in line 20 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/presets/LSP8CompatibleERC721MintableInitAbstract.sol:

        LSP8CompatibleERC721InitAbstract._initialize(name_, symbol_, newOwner_);


Found in line 7 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol:

    IERC721Receiver


Found in line 8 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol:

} from "@openzeppelin/contracts/token/ERC721/IERC721Receiver.sol";


Found in line 41 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol:

    _INTERFACEID_ERC721,


Found in line 42 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol:

    _INTERFACEID_ERC721METADATA


Found in line 43 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol:

} from "././ILSP8CompatibleERC721.sol";


Found in line 48 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol:

abstract contract LSP8CompatibleERC721 is


Found in line 49 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol:

    ILSP8CompatibleERC721,


Found in line 86 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol:

            interfaceId == _INTERFACEID_ERC721 ||


Found in line 87 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol:

            interfaceId == _INTERFACEID_ERC721METADATA ||


Found in line 254 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol:

            _checkOnERC721Received(from, to, tokenId, data),


Found in line 255 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol:

            "LSP8CompatibleERC721: transfer to non ERC721Receiver implementer"


Found in line 291 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol:

            "LSP8CompatibleERC721: approve to caller"


Found in line 307 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol:

    function _checkOnERC721Received(


Found in line 315 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol:

                IERC721Receiver(to).onERC721Received(


Found in line 322 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol:

                return retval == IERC721Receiver.onERC721Received.selector;


Found in line 326 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol:

                        "LSP8CompatibleERC721: transfer to non ERC721Receiver implementer"


Found in line 7 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol:

    IERC721Receiver


Found in line 8 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol:

} from "@openzeppelin/contracts/token/ERC721/IERC721Receiver.sol";


Found in line 41 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol:

    _INTERFACEID_ERC721,


Found in line 42 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol:

    _INTERFACEID_ERC721METADATA


Found in line 43 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol:

} from "./ILSP8CompatibleERC721.sol";


Found in line 48 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol:

abstract contract LSP8CompatibleERC721InitAbstract is


Found in line 49 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol:

    ILSP8CompatibleERC721,


Found in line 86 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol:

            interfaceId == _INTERFACEID_ERC721 ||


Found in line 87 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol:

            interfaceId == _INTERFACEID_ERC721METADATA ||


Found in line 254 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol:

            _checkOnERC721Received(from, to, tokenId, data),


Found in line 255 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol:

            "LSP8CompatibleERC721: transfer to non ERC721Receiver implementer"


Found in line 291 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol:

            "LSP8CompatibleERC721: approve to caller"


Found in line 307 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol:

    function _checkOnERC721Received(


Found in line 315 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol:

                IERC721Receiver(to).onERC721Received(


Found in line 322 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol:

                return retval == IERC721Receiver.onERC721Received.selector;


Found in line 326 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol:

                        "LSP8CompatibleERC721: transfer to non ERC721Receiver implementer"


Found in line 10 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions/ILSP8CompatibleERC721.sol:

bytes4 constant _INTERFACEID_ERC721 = 0x80ac58cd;


Found in line 11 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions/ILSP8CompatibleERC721.sol:

bytes4 constant _INTERFACEID_ERC721METADATA = 0x5b5e139f;


Found in line 16 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions/ILSP8CompatibleERC721.sol:

interface ILSP8CompatibleERC721 is ILSP8IdentifiableDigitalAsset {

------------------------------------------------------------------------ 

### Mitigation 

ERC721A standard, ERC721A is an improvement standard for ERC721 tokens. It was proposed by the Azuki team and used for developing their NFT collection. Compared with ERC721, ERC721A is a more gas-efficient standard to mint a lot of of NFTs simultaneously. It allows developers to mint multiple NFTs at the same gas price. This has been a great improvement due to Ethereum’s sky-rocketing gas fee. Reference: https://nextrope.com/erc721-vs-erc721a-2/










# VULN 18 

## [GAS] require() or revert() statements that check input arguments should be at the top of the function (Also restructured some if’s)
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 124 at 2023-06-lukso/contracts/LSP17ContractExtension/LSP17Extendable.sol:

                revert(0, returndatasize())


Found in line 253 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol:

        require(


Found in line 325 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol:

                    revert(


Found in line 332 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol:

                        revert(add(32, reason), mload(reason))


Found in line 253 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol:

        require(


Found in line 325 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol:

                    revert(


Found in line 332 at 2023-06-lukso/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol:

                        revert(add(32, reason), mload(reason))


Found in line 188 at 2023-06-lukso/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol:

                        revert(add(32, result), returndata_size)


Found in line 191 at 2023-06-lukso/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol:

                    revert("LSP0: batchCalls reverted");


Found in line 576 at 2023-06-lukso/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol:

            require(


Found in line 597 at 2023-06-lukso/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol:

            require(


Found in line 837 at 2023-06-lukso/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol:

                revert(0, returndatasize())


Found in line 78 at 2023-06-lukso/contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol:

        require(


Found in line 92 at 2023-06-lukso/contracts/LSP20CallVerification/LSP20CallVerification.sol:

                revert(add(32, returnedData), returndata_size)

------------------------------------------------------------------------ 

### Mitigation 

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting alot of gas in a function that may ultimately revert in the unhappy case.
