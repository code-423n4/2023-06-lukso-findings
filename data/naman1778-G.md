## [G-01] Using *calldata* instead of *memory* for read-only arguments in *external* functions saves gas

When a function with a *memory* array is called externally, the *abi.decode()* step has to use a for-loop to copy each index of the *calldata* to the *memory* index. Each iteration of this for-loop costs at least 60 gas (i.e. *60 * <mem_array>.length*). Using *calldata* directly, obliviates the need for such a loop in the contract code and runtime execution. Note that even if an interface defines a function as having *memory* arguments, it’s still valid for implementation contracs to use *calldata* arguments instead.

If the array is passed to an *internal* function which passes the array to another internal function where the array is modified and therefore *memory* is used in the *external* call, it’s still more gass-efficient to use *calldata* when the *external* function uses modifiers, since the modifiers may prevent the internal functions from being called. Structs have the same overhead as an array of length one

Note that I’ve also flagged instances where the function is *public* but can be marked as *external* since it’s not called by the contract, and cases where a constructor is involved

There are 17 instances of this issue in 12 files:

    File: IERC725X.sol	

    63: function execute(
    64:     uint256 operationType,
    65:     address target,
    66:     uint256 value,
    67:     bytes memory data
    68: ) external payable returns (bytes memory);

    93: function executeBatch(
    94:     uint256[] memory operationsType,
    95:     address[] memory targets,
    96:     uint256[] memory values,
    97:     bytes[] memory datas
    98: ) external payable returns (bytes[] memory);

https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/interfaces/IERC725X.sol

    File: IERC725Y.sol	

    33: function getDataBatch(
    34:     bytes32[] memory dataKeys
    35: ) external view returns (bytes[] memory dataValues);

    50: function setData(bytes32 dataKey, bytes memory dataValue) external payable;

    65: function setDataBatch(bytes32[] memory dataKeys, bytes[] memory dataValues) external payable;

https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/interfaces/IERC725Y.sol

    File: LSP6KeyManagerCore.sol	

    303: function lsp20VerifyCallResult(
    304:     bytes32 /*callHash*/,
    305:     bytes memory /*result*/
    306: ) external returns (bytes4) {

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol

    File: ILSP7DigitalAsset.sol	

    159: function transfer(
    160:     address from,
    161:     address to,
    162:     uint256 amount,
    163:     bool allowNonLSP1Recipient,
    164:     bytes memory data
    165: ) external;

    189: function transferBatch(
    190:     address[] memory from,
    191:     address[] memory to,
    192:     uint256[] memory amount,
    193:     bool[] memory allowNonLSP1Recipient,
    194:     bytes[] memory data
    195: ) external;

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/ILSP7DigitalAsset.sol

    File: LSP7CompatibleERC20MintableInit.sol	

    25: function initialize(
    26:     string memory name_,
    27:     string memory symbol_,
    28:     address newOwner_
    29: ) external virtual initializer {

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/presets/LSP7CompatibleERC20MintableInit.sol

    File: LSP7MintableInit.sol	

    26: function initialize(
    27:     string memory name_,
    28:     string memory symbol_,
    29:     address newOwner_,
    30:     bool isNonDivisible_
    31: ) external virtual initializer {

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/presets/LSP7MintableInit.sol

    File: ILSP7Mintable.sol	

    26: function mint(
    27:     address to,
    28:     uint256 amount,
    29:     bool allowNonLSP1Recipient,
    30:     bytes memory data
    31: ) external;

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/presets/ILSP7Mintable.sol

    File: ILSP8IdentifiableDigitalAsset.sol	

    186: function transfer(
    187:     address from,
    188:     address to,
    189:     bytes32 tokenId,
    190:     bool allowNonLSP1Recipient,
    191:     bytes memory data
    192: ) external;

    216: function transferBatch(
    217:     address[] memory from,
    218:     address[] memory to,
    219:     bytes32[] memory tokenId,
    220:     bool[] memory allowNonLSP1Recipient,
    221:     bytes[] memory data
    222: ) external;

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/ILSP8IdentifiableDigitalAsset.sol

    File: ILSP8CompatibleERC721.sol	

    80: function safeTransferFrom(
    81:     address from,
    82:     address to,
    83:     uint256 tokenId,
    84:     bytes memory data
    85: ) external;

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/ILSP8CompatibleERC721.sol

    File: LSP8CompatibleERC721MintableInit.sol	

    25: function initialize(
    26:     string memory name_,
    27:     string memory symbol_,
    28:     address newOwner_
    29: ) external virtual initializer {

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/presets/LSP8CompatibleERC721MintableInit.sol

    File: LSP8MintableInit.sol	

    25: function initialize(
    26:     string memory name_,
    27:     string memory symbol_,
    28:     address newOwner_
    29: ) external virtual initializer {

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/presets/LSP8MintableInit.sol

    File: ILSP8Mintable.sol	

    28: function mint(
    29:     address to,
    30:     bytes32 tokenId,
    31:     bool allowNonLSP1Recipient,
    32:     bytes memory data
    33: ) external;

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/presets/ILSP8Mintable.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized("Naman");
            c1.optimized("Naman");
        }
    }
    
    contract Contract0 {
    
         function not_optimized(string memory a) public returns(string memory){
             return a;
         }
    }
    
    contract Contract1 {
    
         function optimized(string calldata a) public returns(string calldata){
             return a;
         }
    }

#### Gas Report

| Contract0 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 100747                                    | 535             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| not_optimized                             | 790             | 790 | 790    | 790 | 1       |


| Contract1 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 66917                                     | 366             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| optimized                                 | 556             | 556 | 556    | 556 | 1       |

## [G-02] abi.encode() is less efficient than abi.encodePacked()

Changing abi.encode function to abi.encodePacked can save gas since the abi.encode function pads extra null bytes at the end of the call data, which is unnecessary. Also, in general, abi.encodePacked is more gas-efficient (see [Solidity-Encode-Gas-Comparison](https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison)).

Use abi.encodePacked() where possible to save gas

There are 3 isntances of this issue in 1 file:

    File: LSP0ERC725AccountCore.sol	

    253: LSP20CallVerification._verifyCallResult(_owner, abi.encode(result));

    319: abi.encode(results)

    528: returnedValues = abi.encode(

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized();
            c1.optimized();
        }
    }
    contract Contract0 {
        string a = "Code4rena";
        function not_optimized() public returns(bytes32){
            return keccak256(abi.encode(a));
        }
    }
    contract Contract1 {
        string a = "Code4rena";
        function optimized() public returns(bytes32){
            return keccak256(abi.encodePacked(a));
        }
    }

#### Gas Report

| Contract0 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 101871                                    | 683             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| not_optimized                             | 2661            | 2661 | 2661   | 2661 | 1       |


| Contract1 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 99465                                     | 671             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| optimized                                 | 2608            | 2608 | 2608   | 2608 | 1       |

## [G-03] Use *assembly* to write *address* storage values

When writing value for variables whose type is address, make use of assembly code instead of solidity code.

There are 3 isntances of this issue in 3 files:

    File: OwnableUnset.sol	

    65: _owner = newOwner;

https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/custom/OwnableUnset.sol

    File: LSP6KeyManager.sol	

    47: _target = target_;

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManager.sol

    File: LSP14Ownable2Step.sol	

    129: _pendingOwner = newOwner;

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;

        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }

        function testGas() public {
            c0.setOwnerAssembly(0xFD2dabe9DFcc4d88a12A9D0D40D834E81217Cccf);
            c1.setOwner(0xFD2dabe9DFcc4d88a12A9D0D40D834E81217Cccf);
        }
    }

    contract Contract0 {

        address owner;
        function setOwnerAssembly(address _owner) public {
            assembly{
                sstore(owner.slot,_owner)
            }
        }

    }

    contract Contract1 {
        address owner;
        function setOwner(address _owner) public {
            owner = _owner;
        }

    }

#### Gas Report

|  Contract0 contract                       |                 |       |        |       |         |
|-------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                           | Deployment Size |       |        |       |         |
| 35287                                     | 207             |       |        |       |         |
| Function Name                             | min             | avg   | median | max   | # calls |
| setOwnerAssembly                          | 22324           | 22324 | 22324  | 22324 | 1       |


|  Contract1 contract                       |                 |       |        |       |         |
|-------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                           | Deployment Size |       |        |       |         |
| 48499                                     | 273             |       |        |       |         |
| Function Name                             | min             | avg   | median | max   | # calls |
| setOwner                                  | 22363           | 22363 | 22363  | 22363 | 1       |

## [G-04] Use nested *if* and, avoid multiple check combinations

Using nesting is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

There are 17 instances of this issue in 8 files:

    File: LSP0ERC725AccountCore.sol	

    801: if (msg.sig == bytes4(0) && extension == address(0)) return;

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol

    File: LSP1UniversalReceiverDelegateUP.sol	

    227: if (dataKeys.length == 0 && dataValues.length == 0)

    245: if (dataKeys.length == 0 && dataValues.length == 0)

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol

    File: LSP6SetDataModule.sol	

    362: if (inputDataValue.length != 0 && inputDataValue.length != 20) {

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol

    File: LSP6KeyManagerCore.sol	

    338: if (!isReentrantCall && !isSetData) {

    404: if (!isReentrantCall && !isSetData) {

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol

    File: LSP6ExecuteModule.sol	

    165: if (isFundingContract && !hasSuperTransferValue) {

    214: if (isTransferringValue && !hasSuperTransferValue) {

    224: if (!hasSuperCall && !isCallDataPresent && !isTransferringValue) {

    228: if (isCallDataPresent && !hasSuperCall) {

    233: if (hasSuperCall && !isTransferringValue) return;

    236: if (hasSuperTransferValue && !isCallDataPresent && isTransferringValue)

    240: if (hasSuperCall && hasSuperTransferValue) return;

    301: if (
    302:     _isAllowedCallType(allowedCall, requiredCallTypes) &&
    303:     _isAllowedAddress(allowedCall, to) &&
    304:     _isAllowedStandard(allowedCall, to) &&
    305:     _isAllowedFunction(allowedCall, selector)
    306: ) return;

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol

    File: LSP8CompatibleERC721InitAbstract.sol	

    235: if (
    236:     !isApprovedForAll(from, operator) &&
    237:     !_isOperatorOrOwner(operator, tokenId)
    238: ) {

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol

    File: LSP8CompatibleERC721.sol	

    235: if (
    236:     !isApprovedForAll(from, operator) &&
    237:     !_isOperatorOrOwner(operator, tokenId)
    238: ) {

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol

    File: LSP10Utils.sol	

    76: if (encodedArrayLength.length != 0 && encodedArrayLength.length != 16) {

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP10ReceivedVaults/LSP10Utils.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;

        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }

        function testGas() public {
            c0.checkAge(19);
            c1.checkAgeOptimized(19);
        }
    }

    contract Contract0 {

        function checkAge(uint8 _age) public returns(string memory){
            if(_age>18 && _age<22){
                return "Eligible";
            }
        }

    }

    contract Contract1 {

        function checkAgeOptimized(uint8 _age) public returns(string memory){
            if(_age>18){
                if(_age<22){
                    return "Eligible";
                }
            }
        }
    }

#### Gas Report

| Contract0 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 76923                                     | 416             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| checkAge                                  | 651             | 651 | 651    | 651 | 1       |


| Contract1 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 76323                                     | 413             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| checkAgeOptimized                         | 645             | 645 | 645    | 645 | 1       |

## [G-05] Using XOR (^) and AND (&) bitwise equivalents

On Remix, given only uint256 types, the following are logical equivalents, but don’t cost the same amount of gas:

(a != b || c != d || e != f) costs 571
((a ^ b) | (c ^ d) | (e ^ f)) != 0 costs 498 (saving 73 gas)
Consider rewriting as following to save gas:

To have a == b means that every 0 and 1 match on both variables. Meaning that a XOR (operator ^) would evaluate to 0 ((a ^ b) == 0), as it excludes by definition any equalities.
Now, if a != b, this means that there’s at least somewhere a 1 and a 0 not matching between a and b, making (a ^ b) != 0.

Both formulas are logically equivalent and using the XOR bitwise operator costs actually the same amount of gas:

However, it is much cheaper to use the bitwise OR operator (|) than comparing the truthy or falsy values

There are 14 instances of this issue in 7 files:

    File: ERC725XCore.sol	
 
    133: if (
    134:     operationsType.length != targets.length ||
    135:     (targets.length != values.length || values.length != datas.length)
    136: ) {

https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725XCore.sol

    File: LSP1Utils.sol	

    84: if (
    85:     typeId == _TYPEID_LSP7_TOKENSSENDER ||
    86:     typeId == _TYPEID_LSP7_TOKENSRECIPIENT
    87: ) {

    91: } else if (
    92:     typeId == _TYPEID_LSP8_TOKENSSENDER ||
    93:     typeId == _TYPEID_LSP8_TOKENSRECIPIENT
    94: ) {

    98: } else if (
    99:     typeId == _TYPEID_LSP9_OwnershipTransferred_SenderNotification ||
    100:     typeId == _TYPEID_LSP9_OwnershipTransferred_RecipientNotification
    101: ) {

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP1UniversalReceiver/LSP1Utils.sol

    File: LSP6SetDataModule.sol	

    281: } else if (
    282:     inputDataKey == _LSP1_UNIVERSAL_RECEIVER_DELEGATE_KEY ||
    283:     bytes12(inputDataKey) == _LSP1_UNIVERSAL_RECEIVER_DELEGATE_PREFIX
    284: ) {

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol

    File: LSP6KeyManagerCore.sol	

    211: if (
    212:     signatures.length != nonces.length ||
    213:     nonces.length != validityTimestamps.length ||
    214:     validityTimestamps.length != values.length ||
    215:     values.length != payloads.length
    216: ) {

    257: if (
    258:     bytes4(data) == IERC725Y.setData.selector ||
    259:     bytes4(data) == IERC725Y.setDataBatch.selector
    260: ) {

    324: if (
    325:     bytes4(payload) == IERC725Y.setData.selector ||
    326:     bytes4(payload) == IERC725Y.setDataBatch.selector
    327: ) {

    370: if (
    371:     bytes4(payload) == IERC725Y.setData.selector ||
    372:     bytes4(payload) == IERC725Y.setDataBatch.selector
    373: ) {

    505: } else if (
    506:     erc725Function == ILSP14Ownable2Step.transferOwnership.selector ||
    507:     erc725Function == ILSP14Ownable2Step.acceptOwnership.selector
    508: ) {

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol
 
    File: LSP6ExecuteModule.sol	

    118: if (
    119:     operationType == OPERATION_1_CREATE ||
    120:     operationType == OPERATION_2_CREATE2
    121: ) {

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol

    File: LSP7DigitalAssetCore.sol	

    162: if (
    164:     fromLength != to.length ||
    165:     fromLength != amount.length ||
    166:     fromLength != allowNonLSP1Recipient.length ||
    167:     fromLength != data.length
    168: ) {

    406: if (from == address(0) || to == address(0)) {

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol

    File: LSP8IdentifiableDigitalAssetCore.sol	

    218: if (
    219:     fromLength != to.length ||
    220:     fromLength != tokenId.length ||
    221:     fromLength != allowNonLSP1Recipient.length ||
    222:     fromLength != data.length
    223: ) {

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized(1,2);
            c1.optimized(1,2);
        }
    }
    
    contract Contract0 {
        function not_optimized(uint8 a,uint8 b) public returns(bool){
            return ((a==1) || (b==1));
        }
    }
    
    contract Contract1 {
        function optimized(uint8 a,uint8 b) public returns(bool){
            return ((a ^ 1) & (b ^ 1)) == 0;
        }
    }

#### Gas Report

| Contract0 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 46099                                     | 261             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| not_optimized                             | 456             | 456 | 456    | 456 | 1       |

| Contract1 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 42493                                     | 243             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| optimized                                 | 430             | 430 | 430    | 430 | 1       |

## [G-06] Instead of counting down in *for* statements, count up

Counting down is more gas efficient than counting up.

There are 17 instances of this issue in 10 files:

    File: ERC725XCore.sol	
 
    146: for (uint256 i = 0; i < operationsType.length; ) {

https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725XCore.sol

    File: ERC725YCore.sol	

    47: for (uint256 i = 0; i < dataKeys.length; ) {

    88: for (uint256 i = 0; i < dataKeys.length; ) {

https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725YCore.sol

    File: LSP0ERC725AccountCore.sol	

    175: for (uint256 i; i < data.length; ) {

    396: for (uint256 i = 0; i < dataKeys.length; ) {

    411: for (uint256 i = 0; i < dataKeys.length; ) {

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol

    File: LSP6SetDataModule.sol	

    726:  for (uint256 ii; ii < inputKeysLength; ) {

    762: for (uint256 jj; jj < inputKeysLength; ) {

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol

    File: LSP6KeyManagerCore.sol	

    163: for (uint256 ii = 0; ii < payloads.length; ) {

    223: for (uint256 ii = 0; ii < payloads.length; ) {

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol

    File: LSP6ExecuteModule.sol	

    272: for (uint256 ii = 0; ii < allowedCalls.length; ii += 34) {

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol

    File: LSP6Utils.sol	

    173: for (uint256 i = 0; i < permissions.length; i++) {

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Utils.sol

    File: LSP7DigitalAssetCore.sol	

    171: for (uint256 i = 0; i < fromLength; ) {

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol

    File: LSP8IdentifiableDigitalAssetCore.sol	

    227: for (uint256 i = 0; i < fromLength; ) {

    273: for (uint256 i = 0; i < operatorListLength; ) {

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol

    File: LSP2Utils.sol

    261: for (uint256 ii = 0; ii < arrayLength; ) {

    292: for (uint256 ii = 0; ii < arrayLength; ) {

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.AddNum();
            c1.AddNum();
        }
    }


    contract Contract0 {
        uint256 num = 3;
        function AddNum() public {
            uint256 _num = num;
            for(uint i=0;i<=9;i++){
                _num = _num +1;
            }
            num = _num;
        }
    }

    contract Contract1 {
        uint256 num = 3;
        function AddNum() public {
            uint256 _num = num;
            for(uint i=9;i>=0;i--){
                _num = _num +1;
            }
            num = _num;
        }
    }

#### Gas Report

| Contract0 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 77011                                     | 311             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| AddNum                                    | 7040            | 7040 | 7040   | 7040 | 1       |

| Contract1 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 73811                                     | 295             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| AddNum                                    | 3819            | 3819 | 3819   | 3819 | 1       |

## [G-07] Use assembly to check for *address(0)*

Checking zero address can be improved by replacing the require statement with Assembly.Solidity has a lot of guardrails that can be removed (with care) for optimization purposes, especially for simple functionality like checking if an address is zero.

There are 25 instances of this issue in 15 files:

    File: ERC725XCore.sol	
 
    87: if (target != address(0)) revert ERC725X_CreateOperationsRequireEmptyRecipientAddress();

    93: if (target != address(0)) revert ERC725X_CreateOperationsRequireEmptyRecipientAddress();

    239: if (contractAddress == address(0)) {

https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725XCore.sol

    File: OwnableUnset.sol	

    47: require(newOwner != address(0), "Ownable: new owner is the zero address");

https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/custom/OwnableUnset.sol

    File: ERC725.sol	

    24: require(newOwner != address(0), "Ownable: new owner is the zero address");

https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725.sol

    File: ERC725XInitAbstract.sol	

    22: require(newOwner != address(0), "Ownable: new owner is the zero address");

https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725XInitAbstract.sol

    File: ERC725YInitAbstract.sol	

    22: require(newOwner != address(0), "Ownable: new owner is the zero address");

https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725YInitAbstract.sol

    File: ERC725Y.sol	
 
    22: require(newOwner != address(0), "Ownable: new owner is the zero address");

https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725Y.sol

    File: ERC725X.sol	

    21: require(newOwner != address(0), "Ownable: new owner is the zero address");

https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725X.sol

    File: LSP0ERC725AccountCore.sol	   

    801: if (msg.sig == bytes4(0) && extension == address(0)) return;

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol

    File: LSP6KeyManagerInitAbstract.sol	

    21: if (target_ == address(0)) revert InvalidLSP6Target();

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerInitAbstract.sol

    File: LSP6KeyManager.sol	

    19: if (target_ == address(0)) revert InvalidLSP6Target();

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManager.sol

    File: LSP7DigitalAssetCore.sol	

    268: if (operator == address(0)) {

    300: if (to == address(0)) {

    343: if (from == address(0)) {

    406: if (from == address(0) || to == address(0)) {

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol

    File: LSP8IdentifiableDigitalAssetCore.sol	

    84: if (tokenOwner == address(0)) {

    115: if (operator == address(0)) {

    139: if (operator == address(0)) {

    319: if (to == address(0)) {

    410: if (to == address(0)) {

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol

    File: LSP8EnumerableInitAbstract.sol	

    38:if (from == address(0)) {

    42: } else if (to == address(0)) {

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8EnumerableInitAbstract.sol

    File: LSP8Enumerable.sol	

    36: if (from == address(0)) {

    40: } else if (to == address(0)) {

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8Enumerable.sol

    File: LSP17Extendable.sol	

    50: if (erc165Extension == address(0)) return false;

    91: if (extension == address(0))

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP17ContractExtension/LSP17Extendable.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.solidity_notZero(0xFD2dabe9DFcc4d88a12A9D0D40D834E81217Cccf);
            c1.assembly_notZero(0xFD2dabe9DFcc4d88a12A9D0D40D834E81217Cccf);
        }
    }


    contract Contract0 {

        error ZeroAddress();

        function solidity_notZero(address toCheck) public pure returns(bool success) {
            if(toCheck == address(0)) revert ZeroAddress();

            return true;
        }
    }

    contract Contract1{

    error ZeroAddress();
    function assembly_notZero(address toCheck) public pure returns(bool success) {
        assembly {
            if iszero(toCheck) {
                let ptr := mload(0x40)
                mstore(ptr, 0xd92e233d00000000000000000000000000000000000000000000000000000000)
                revert(ptr, 0x4)
            }
        }
        return true;
    }
    }

#### Gas Report

| Contract0 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 55905                                     | 311             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| solidity_notZero                          | 323             | 323 | 323    | 323 | 1       |

| Contract1 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 50099                                     | 281             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| assembly_notZero                          | 317             | 317 | 317    | 317 | 1       |

## [G-08] *internal* functions only called once can be inlined to save gas

Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

There are 29 instances of this issue in 8 files:

    File: ERC725XCore.sol	

    127: function _executeBatch(
    128:     uint256[] memory operationsType,
    129:     address[] memory targets,
    130:     uint256[] memory values,
    131:     bytes[] memory datas
    132: ) internal virtual returns (bytes[] memory) {

    165: function _executeCall(
    166:     address target,
    167:     uint256 value,
    168:     bytes memory data
    169: ) internal virtual returns (bytes memory result) {

    204: function _executeDelegateCall(
    205:     address target,
    206:     bytes memory data
    207: ) internal virtual returns (bytes memory result) {

    221: function _deployCreate(
    222:     uint256 value,
    223:     bytes memory creationCode
    224: ) internal virtual returns (bytes memory newContract) {

    253: function _deployCreate2(
    254:     uint256 value,
    255:     bytes memory creationCode
    256: ) internal virtual returns (bytes memory newContract) {

https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/ERC725XCore.sol

    File: OwnableUnset.sol	

    54: function _checkOwner() internal view virtual {

https://github.com/ERC725Alliance/ERC725/blob/v5.1.0/implementations/contracts/custom/OwnableUnset.sol

    File: LSP0ERC725AccountCore.sol	

    796: function _fallbackLSP17Extendable() internal virtual override {

    851: function _getExtension(
    852:     bytes4 functionSelector
    853: ) internal view virtual override returns (address) {

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol

    File: LSP1UniversalReceiverDelegateUP.sol	
 
    152: function _whenReceiving(
    153:     bytes32 typeId,
    154:     address notifier,
    155:     bytes32 notifierMapKey,
    156:     bytes4 interfaceID
    157: ) internal virtual returns (bytes memory) {

    201: function _whenSending(
    202:     bytes32 typeId,
    203:     address notifier,
    204:     bytes32 notifierMapKey,
    205:     uint128 arrayIndex
    206: ) internal virtual returns (bytes memory) {

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol

    File: LSP6SetDataModule.sol	

    334: function _getPermissionToSetPermissionsArray(
    335:     address controlledContract,
    336:     bytes32 inputDataKey,
    337:     bytes memory inputDataValue,
    338:     bool hasBothAddControllerAndEditPermissions
    339: ) internal view virtual returns (bytes32) {

    385: function _getPermissionToSetControllerPermissions(
    386:     address controlledContract,
    387:     bytes32 inputPermissionDataKey
    388: ) internal view virtual returns (bytes32) {

    406: function _getPermissionToSetAllowedCalls(
    407:     address controlledContract,
    408:     bytes32 dataKey,
    409:     bytes memory dataValue,
    410:     bool hasBothAddControllerAndEditPermissions
    411: ) internal view virtual returns (bytes32) {

    438: function _getPermissionToSetAllowedERC725YDataKeys(
    439:     address controlledContract,
    440:     bytes32 dataKey,
    441:     bytes memory dataValue,
    442:     bool hasBothAddControllerAndEditPermissions
    443: ) internal view returns (bytes32) {

    474: function _getPermissionToSetLSP1Delegate(
    475:     address controlledContract,
    476:     bytes32 lsp1DelegateDataKey
    477: ) internal view virtual returns (bytes32) {

    490: function _getPermissionToSetLSP17Extension(
    491:     address controlledContract,
    492:     bytes32 lsp17ExtensionDataKey
    493: ) internal view virtual returns (bytes32) {

    507: function _verifyAllowedERC725YSingleKey(
    508:     address controllerAddress,
    509:     bytes32 inputDataKey,
    510:     bytes memory allowedERC725YDataKeysCompacted
    511: ) internal pure virtual {

    622: function _verifyAllowedERC725YDataKeys(
    623:     address controllerAddress,
    624:     bytes32[] memory inputDataKeys,
    625:     bytes memory allowedERC725YDataKeysCompacted,
    626:     bool[] memory validatedInputKeysList,
    627:     uint256 allowedDataKeysFound
    628: ) internal pure virtual {

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol

    File: LSP6KeyManagerCore.sol	

    441: function _isValidNonce(
    442:     address from,
    443:     uint256 idx
    444: ) internal view virtual returns (bool) {

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol

    File: LSP6ExecuteModule.sol	

    153: function _verifyCanDeployContract(
    154:     address controller,
    155:     bytes32 permissions,
    156:     bool isFundingContract
    157: ) internal view virtual {

    170: function _verifyCanStaticCall(
    171:     address controlledContract,
    172:     address controller,
    173:     bytes32 permissions,
    174:     bytes calldata payload
    175: ) internal view virtual {

    188: function _verifyCanCall(
    189:     address controlledContract,
    190:     address controller,
    191:     bytes32 permissions,
    192:     bytes calldata payload
    193: ) internal view virtual {

    317: function _extractCallType(
    318:     uint256 operationType,
    319:     uint256 value,
    320:     bool isEmptyCall
    321: ) internal pure returns (bytes4 requiredCallTypes) {

    339: function _extractExecuteParameters(
    340:     bytes calldata executeCalldata
    341: ) internal pure returns (uint256, address, uint256, bytes4, bool) {

    366: function _isAllowedAddress(
    367:     bytes memory allowedCall,
    368:     address to
    369: ) internal pure returns (bool) {

    382: function _isAllowedStandard(
    383:     bytes memory allowedCall,
    384:     address to
    385: ) internal view returns (bool) {

    399: function _isAllowedFunction(
    400:     bytes memory allowedCall,
    401:     bytes4 requiredFunction
    402: ) internal pure returns (bool) {

    418: function _isAllowedCallType(
    419:     bytes memory allowedCall,
    420:     bytes4 requiredCallTypes
    421: ) internal pure returns (bool) {

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol

    File: LSP20CallVerification.sol	

    84: function _revert(bool postCall, bytes memory returnedData) internal pure {

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP20CallVerification/LSP20CallVerification.sol
