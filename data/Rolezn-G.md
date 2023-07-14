## GAS Summary<a name="GAS Summary">

### Gas Optimizations
| |Issue|Contexts|Estimated Gas Saved|
|-|:-|:-|:-:|
| [GAS&#x2011;1](#gas1-abiencode-is-less-efficient-than-abiencodepacked) | `abi.encode()` is less efficient than `abi.encodepacked()` | 3 | 159 |
| [GAS&#x2011;2](#gas2-consider-activating-viair-for-deploying) | Consider activating `via-ir` for deploying | 1 | 250 |
| [GAS&#x2011;3](#gas3-use-assembly-to-emit-events) | Use assembly to emit events | 48 | 1824 |
| [GAS&#x2011;4](#gas4-counting-down-in-for-statements-is-more-gas-efficient) | Counting down in `for` statements is more gas efficient | 14 | 3598 |
| [GAS&#x2011;5](#gas5-using-delete-statement-can-save-gas) | Using `delete` statement can save gas | 6 | 48 |
| [GAS&#x2011;6](#gas6-use-assembly-to-write-address-storage-values) | Use `assembly` to write address storage values | 14 | 1036 |
| [GAS&#x2011;7](#gas7-the-result-of-a-function-call-should-be-cached-rather-than-recalling-the-function) | The result of a function call should be cached rather than re-calling the function | 25 | 1250 |
| [GAS&#x2011;8](#gas8-help-the-optimizer-by-saving-a-storage-variable’s-reference-instead-of-repeatedly-fetching-it) | Help The Optimizer By Saving A Storage Variable's Reference Instead Of Repeatedly Fetching It | 13 | 702 |
| [GAS&#x2011;9](#gas9-usage-of-uintsints-smaller-than-32-bytes-256-bits-incurs-overhead) | Usage of `uints`/`ints` smaller than 32 bytes (256 bits) incurs overhead | 15 | 90 |
| [GAS&#x2011;10](#gas10-use-nested-if-and-avoid-multiple-check-combinations) | Use nested `if` and avoid multiple check combinations | 14 | 84 |
| [GAS&#x2011;11](#gas11-using-openzeppelin-ownable2stepsol-is-more-gas-efficient) | Using Openzeppelin `Ownable2Step.sol` is more gas efficient | 4 | 32 |
| [GAS&#x2011;12](#gas12-using-xor-^-and-and-&-bitwise-equivalents) | Using XOR (^) and AND (&) bitwise equivalents | 161 | 2093 |

Total: 326 contexts over 12 issues

## Gas Optimizations

### <a href="#gas-summary">[GAS&#x2011;1]</a><a name="GAS&#x2011;1"> `abi.encode()` is less efficient than `abi.encodepacked()`

See for more information: https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison 

#### <ins>Proof Of Concept</ins>


```solidity
File: LSP0ERC725AccountCore.sol

253: LSP20CallVerification._verifyCallResult(_owner, abi.encode(result));
```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L253

```solidity
File: LSP0ERC725AccountCore.sol

319: abi.encode(results)
```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L319

```solidity
File: LSP0ERC725AccountCore.sol

528: returnedValues = abi.encode(
```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L528



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

#### Gas Test Report

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



### <a href="#gas-summary">[GAS&#x2011;2]</a><a name="GAS&#x2011;2"> Consider activating `via-ir` for deploying

The IR-based code generator was introduced with an aim to not only allow code generation to be more transparent and auditable but also to enable more powerful optimization passes that span across functions.

You can enable it on the command line using `--via-ir` or with the option `{"viaIR": true}`.

This will take longer to compile, but you can just simple test it before deploying and if you got a better benchmark then you can add --via-ir to your deploy command

More on: https://docs.soliditylang.org/en/v0.8.17/ir-breaking-changes.html






### <a href="#gas-summary">[GAS&#x2011;3]</a><a name="GAS&#x2011;3"> Use assembly to emit events

We can use assembly to emit events efficiently by utilizing `scratch space` and the `free memory pointer`. This will allow us to potentially avoid memory expansion costs.
Note: In order to do this optimization safely, we will need to cache and restore the free memory pointer.

For example, for a generic `emit` event for `eventSentAmountExample`:
```solidity
// uint256 id, uint256 value, uint256 amount
emit eventSentAmountExample(id, value, amount);
```

We can use the following assembly emit events:

```solidity
        assembly {
            let memptr := mload(0x40)
            mstore(0x00, calldataload(0x44))
            mstore(0x20, calldataload(0xa4))
            mstore(0x40, amount)
            log1(
                0x00,
                0x60,
                // keccak256("eventSentAmountExample(uint256,uint256,uint256)")
                0xa622cf392588fbf2cd020ff96b2f4ebd9c76d7a4bc7f3e6b2f18012312e76bc3
            )
            mstore(0x40, memptr)
        }
```

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: LSP0ERC725AccountCore.sol

229: emit ValueReceived(msg.sender, msg.value);

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L229

```solidity
File: LSP0ERC725AccountCore.sol

287: emit ValueReceived(msg.sender, msg.value);

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L287

```solidity
File: LSP0ERC725AccountCore.sol

344: emit ValueReceived(msg.sender, msg.value);

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L344

```solidity
File: LSP0ERC725AccountCore.sol

385: emit ValueReceived(msg.sender, msg.value);

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L385

```solidity
File: LSP0ERC725AccountCore.sol

465: emit ValueReceived(msg.sender, msg.value);

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L465

```solidity
File: LSP0ERC725AccountCore.sol

566: emit OwnershipTransferStarted(currentOwner, pendingNewOwner);
566: emit OwnershipTransferStarted(currentOwner, pendingNewOwner);

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L566

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L566



```solidity
File: LSP0ERC725AccountInitAbstract.sol

63: emit ValueReceived(msg.sender, msg.value);

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountInitAbstract.sol#L63

```solidity
File: LSP14Ownable2Step.sol

72: emit OwnershipTransferStarted(currentOwner, newOwner);

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol#L72

```solidity
File: LSP14Ownable2Step.sol

165: emit RenounceOwnershipStarted();
178: emit OwnershipRenounced();

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol#L165

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol#L178



```solidity
File: LSP6KeyManagerCore.sol

269: emit VerifiedCall(caller, msgValue, bytes4(data));

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L269

```solidity
File: LSP6KeyManagerCore.sol

334: emit VerifiedCall(msg.sender, msgValue, bytes4(payload));

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L334

```solidity
File: LSP6KeyManagerCore.sol

400: emit VerifiedCall(signer, msgValue, bytes4(payload));

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L400

```solidity
File: LSP7DigitalAssetCore.sol

279: emit AuthorizedOperator(operator, tokenOwner, amount);
281: emit RevokedOperator(operator, tokenOwner);

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L279

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L281



```solidity
File: LSP7DigitalAssetCore.sol

375: emit Transfer(operator, from, address(0), amount, false, data);

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L375

```solidity
File: LSP7DigitalAssetCore.sol

422: emit Transfer(operator, from, to, amount, allowNonLSP1Recipient, data);

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L422

```solidity
File: LSP7CompatibleERC20.sol

113: emit Approval(tokenOwner, operator, amount);

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/extensions/LSP7CompatibleERC20.sol#L113

```solidity
File: LSP7CompatibleERC20.sol

123: emit Transfer(from, to, amount);

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/extensions/LSP7CompatibleERC20.sol#L123

```solidity
File: LSP7CompatibleERC20.sol

133: emit Transfer(address(0), to, amount);

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/extensions/LSP7CompatibleERC20.sol#L133

```solidity
File: LSP7CompatibleERC20.sol

142: emit Transfer(from, address(0), amount);

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/extensions/LSP7CompatibleERC20.sol#L142

```solidity
File: LSP7CompatibleERC20InitAbstract.sol

121: emit Approval(tokenOwner, operator, amount);

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/extensions/LSP7CompatibleERC20InitAbstract.sol#L121

```solidity
File: LSP7CompatibleERC20InitAbstract.sol

131: emit Transfer(from, to, amount);

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/extensions/LSP7CompatibleERC20InitAbstract.sol#L131

```solidity
File: LSP7CompatibleERC20InitAbstract.sol

141: emit Transfer(address(0), to, amount);

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/extensions/LSP7CompatibleERC20InitAbstract.sol#L141

```solidity
File: LSP7CompatibleERC20InitAbstract.sol

150: emit Transfer(from, address(0), amount);

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/extensions/LSP7CompatibleERC20InitAbstract.sol#L150

```solidity
File: LSP8IdentifiableDigitalAssetCore.sol

126: emit AuthorizedOperator(operator, tokenOwner, tokenId);

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L126

```solidity
File: LSP8IdentifiableDigitalAssetCore.sol

252: emit RevokedOperator(operator, tokenOwner, tokenId);

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L252

```solidity
File: LSP8IdentifiableDigitalAssetCore.sol

373: emit Transfer(operator, tokenOwner, address(0), tokenId, false, data);

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L373

```solidity
File: LSP8IdentifiableDigitalAssetCore.sol

424: emit Transfer(operator, from, to, tokenId, allowNonLSP1Recipient, data);

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L424

```solidity
File: LSP8CompatibleERC721.sol

157: emit Approval(tokenOwnerOf(bytes32(tokenId)), operator, tokenId);

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol#L157

```solidity
File: LSP8CompatibleERC721.sol

223: emit Approval(tokenOwnerOf(tokenId), operator, uint256(tokenId));

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol#L223

```solidity
File: LSP8CompatibleERC721.sol

242: emit Transfer(from, to, uint256(tokenId));

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol#L242

```solidity
File: LSP8CompatibleERC721.sol

265: emit Transfer(address(0), to, uint256(tokenId));

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol#L265

```solidity
File: LSP8CompatibleERC721.sol

275: emit Transfer(tokenOwner, address(0), uint256(tokenId));

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol#L275

```solidity
File: LSP8CompatibleERC721.sol

294: emit ApprovalForAll(tokensOwner, operator, approved);

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol#L294

```solidity
File: LSP8CompatibleERC721InitAbstract.sol

157: emit Approval(tokenOwnerOf(bytes32(tokenId)), operator, tokenId);

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol#L157

```solidity
File: LSP8CompatibleERC721InitAbstract.sol

223: emit Approval(tokenOwnerOf(tokenId), operator, uint256(tokenId));

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol#L223

```solidity
File: LSP8CompatibleERC721InitAbstract.sol

242: emit Transfer(from, to, uint256(tokenId));

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol#L242

```solidity
File: LSP8CompatibleERC721InitAbstract.sol

265: emit Transfer(address(0), to, uint256(tokenId));

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol#L265

```solidity
File: LSP8CompatibleERC721InitAbstract.sol

275: emit Transfer(tokenOwner, address(0), uint256(tokenId));

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol#L275

```solidity
File: LSP8CompatibleERC721InitAbstract.sol

294: emit ApprovalForAll(tokensOwner, operator, approved);

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol#L294

```solidity
File: ERC725XCore.sol

183: emit Executed(OPERATION_0_CALL, target, value, bytes4(data));

```

https://github.com/code-423n4/2023-06-lukso/tree/main/submodules/ERC725/implementations/contracts/ERC725XCore.sol#L183

```solidity
File: ERC725XCore.sol

207: emit Executed(OPERATION_3_STATICCALL, target, 0, bytes4(data));

```

https://github.com/code-423n4/2023-06-lukso/tree/main/submodules/ERC725/implementations/contracts/ERC725XCore.sol#L207

```solidity
File: ERC725XCore.sol

229: emit Executed(OPERATION_4_DELEGATECALL, target, 0, bytes4(data));

```

https://github.com/code-423n4/2023-06-lukso/tree/main/submodules/ERC725/implementations/contracts/ERC725XCore.sol#L229

```solidity
File: ERC725XCore.sol

307: emit ContractCreated(OPERATION_2_CREATE2, contractAddress, value, salt);

```

https://github.com/code-423n4/2023-06-lukso/tree/main/submodules/ERC725/implementations/contracts/ERC725XCore.sol#L307

```solidity
File: ERC725YCore.sol

109: emit DataChanged(dataKey, dataValue);

```

https://github.com/code-423n4/2023-06-lukso/tree/main/submodules/ERC725/implementations/contracts/ERC725YCore.sol#L109

```solidity
File: OwnableUnset.sol

72: emit OwnershipTransferred(oldOwner, newOwner);

```

https://github.com/code-423n4/2023-06-lukso/tree/main/submodules/ERC725/implementations/contracts/custom/OwnableUnset.sol#L72



</details>





### <a href="#gas-summary">[GAS&#x2011;4]</a><a name="GAS&#x2011;4"> Counting down in `for` statements is more gas efficient

Counting down is more gas efficient than counting up because neither we are making zero variable to non-zero variable and also we will get gas refund in the last transaction when making non-zero to zero variable.

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: LSP0ERC725AccountCore.sol

175: for (uint256 i; i < data.length; ) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L175

```solidity
File: LSP0ERC725AccountCore.sol

396: for (uint256 i = 0; i < dataKeys.length; ) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L396



```solidity
File: LSP2Utils.sol

261: for (uint256 ii = 0; ii < arrayLength; ) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol#L261

```solidity
File: LSP2Utils.sol

292: for (uint256 ii = 0; ii < arrayLength; ) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol#L292

```solidity
File: LSP6KeyManagerCore.sol

163: for (uint256 ii = 0; ii < payloads.length; ) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L163

```solidity
File: LSP6KeyManagerCore.sol

223: for (uint256 ii = 0; ii < payloads.length; ) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L223

```solidity
File: LSP6Utils.sol

173: for (uint256 i = 0; i < permissions.length; i++) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Utils.sol#L173

```solidity
File: LSP6ExecuteModule.sol

272: for (uint256 ii = 0; ii < allowedCalls.length; ii += 34) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L272

```solidity
File: LSP7DigitalAssetCore.sol

171: for (uint256 i = 0; i < fromLength; ) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L171

```solidity
File: LSP8IdentifiableDigitalAssetCore.sol

227: for (uint256 i = 0; i < fromLength; ) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L227

```solidity
File: LSP8IdentifiableDigitalAssetCore.sol

273: for (uint256 i = 0; i < operatorListLength; ) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L273

```solidity
File: ERC725XCore.sol

150: for (uint256 i = 0; i < operationsType.length; ) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/submodules/ERC725/implementations/contracts/ERC725XCore.sol#L150

```solidity
File: ERC725YCore.sol

47: for (uint256 i = 0; i < dataKeys.length; ) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/submodules/ERC725/implementations/contracts/ERC725YCore.sol#L47

```solidity
File: ERC725YCore.sol

88: for (uint256 i = 0; i < dataKeys.length; ) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/submodules/ERC725/implementations/contracts/ERC725YCore.sol#L88



</details>


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

#### Gas Test Report

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



### <a href="#gas-summary">[GAS&#x2011;5]</a><a name="GAS&#x2011;5"> Using `delete` statement can save gas

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: LSP2Utils.sol

328: uint256 pointer = 0;

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol#L328

```solidity
File: LSP6Utils.sol

94: uint256 pointer = 0;

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Utils.sol#L94

```solidity
File: LSP6Utils.sol

123: uint256 pointer = 0;

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Utils.sol#L123

```solidity
File: LSP6Utils.sol

172: uint256 result = 0;

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Utils.sol#L172

```solidity
File: LSP6SetDataModule.sol

129: uint256 inputDataKeysAllowed = 0;
133: uint256 ii = 0;

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L129

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L133





</details>





### <a href="#gas-summary">[GAS&#x2011;6]</a><a name="GAS&#x2011;6"> Use `assembly` to write address storage values

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: LSP14Ownable2Step.sol

129: _pendingOwner = newOwner;
```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol#L129

```solidity
File: LSP14Ownable2Step.sol

163: _renounceOwnershipStartedAt = currentBlock;
```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol#L163

```solidity
File: LSP6KeyManager.sol

20: _target = target_;
```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManager.sol#L20

```solidity
File: LSP6KeyManagerCore.sol

519: _reentrancyStatus = false;
```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L519

```solidity
File: LSP6KeyManagerCore.sol

553: _reentrancyStatus = false;
```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L553

```solidity
File: LSP6KeyManagerCore.sol

541: _reentrancyStatus = true;
```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L541

```solidity
File: LSP6KeyManagerInitAbstract.sol

22: _target = target_;
```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerInitAbstract.sol#L22

```solidity
File: LSP7DigitalAsset.sol

45: _isNonDivisible = isNonDivisible_;
```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/LSP7DigitalAsset.sol#L45

```solidity
File: LSP7DigitalAssetInitAbstract.sol

34: _isNonDivisible = isNonDivisible_;
```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetInitAbstract.sol#L34

```solidity
File: LSP7CappedSupply.sol

28: _tokenSupplyCap = tokenSupplyCap_;
```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/extensions/LSP7CappedSupply.sol#L28

```solidity
File: LSP7CappedSupplyInitAbstract.sol

28: _tokenSupplyCap = tokenSupplyCap_;
```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/extensions/LSP7CappedSupplyInitAbstract.sol#L28

```solidity
File: LSP8CappedSupply.sol

30: _tokenSupplyCap = tokenSupplyCap_;
```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CappedSupply.sol#L30

```solidity
File: LSP8CappedSupplyInitAbstract.sol

30: _tokenSupplyCap = tokenSupplyCap_;
```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CappedSupplyInitAbstract.sol#L30

```solidity
File: OwnableUnset.sol

71: _owner = newOwner;
```

https://github.com/code-423n4/2023-06-lukso/tree/main/submodules/ERC725/implementations/contracts/custom/OwnableUnset.sol#L71



</details>




### <a href="#gas-summary">[GAS&#x2011;7]</a><a name="GAS&#x2011;7"> The result of a function call should be cached rather than re-calling the function

External calls are expensive. Results of external function calls should be cached rather than call them multiple times. Consider caching the following:

#### <ins>Proof Of Concept</ins>


<details>

```solidity
File: LSP0ERC725AccountCore.sol

560: address currentOwner = owner();
577: currentOwner == owner(),
598: currentOwner == owner(),

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L560

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L577

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L598



```solidity
File: LSP0ERC725AccountCore.sol

664: return LSP14Ownable2Step._renounceOwnership();
672: LSP14Ownable2Step._renounceOwnership();

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L664


https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L672



```solidity
File: LSP0ERC725AccountCore.sol

811: calldatacopy(0, 0, calldatasize())
815: mstore(calldatasize(), shl(96, caller()))
818: mstore(add(calldatasize(), 20), callvalue())
826: add(calldatasize(), 52),
832: returndatacopy(0, 0, returndatasize())
837: revert(0, returndatasize())
840: return(0, returndatasize())

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L811

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L815

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L818

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L826

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L832

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L837

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L840



```solidity
File: LSP14Ownable2Step.sol

71: address currentOwner = owner();
79: currentOwner == owner(),

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol#L71

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol#L79



```solidity
File: LSP17Extendable.sol

98: calldatacopy(0, 0, calldatasize())
102: mstore(calldatasize(), shl(96, caller()))
105: mstore(add(calldatasize(), 20), callvalue())
113: add(calldatasize(), 52),
119: returndatacopy(0, 0, returndatasize())
124: revert(0, returndatasize())
127: return(0, returndatasize())

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP17ContractExtension/LSP17Extendable.sol#L98

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP17ContractExtension/LSP17Extendable.sol#L102

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP17ContractExtension/LSP17Extendable.sol#L105

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP17ContractExtension/LSP17Extendable.sol#L113

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP17ContractExtension/LSP17Extendable.sol#L119

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP17ContractExtension/LSP17Extendable.sol#L124

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP17ContractExtension/LSP17Extendable.sol#L127



```solidity
File: LSP8Enumerable.sol

37: uint256 index = totalSupply();
41: uint256 lastIndex = totalSupply() - 1;

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8Enumerable.sol#L37

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8Enumerable.sol#L41



```solidity
File: LSP8EnumerableInitAbstract.sol

39: uint256 index = totalSupply();
43: uint256 lastIndex = totalSupply() - 1;

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8EnumerableInitAbstract.sol#L39

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8EnumerableInitAbstract.sol#L43





</details>




### <a href="#gas-summary">[GAS&#x2011;8]</a><a name="GAS&#x2011;8"> Help The Optimizer By Saving A Storage Variable's Reference Instead Of Repeatedly Fetching It

To help the optimizer, declare a storage type variable and use it instead of repeatedly fetching the reference in a map or an array.
The effect can be quite significant.
As an example, instead of repeatedly calling someMap[someIndex], save its reference like this: SomeStruct storage someStruct = someMap[someIndex] and use it.

#### <ins>Proof Of Concept</ins>


```solidity
File: LSP6KeyManagerCore.sol

164: if ((totalValues += values[ii]) > msg.value) {
```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L164

```solidity
File: LSP6KeyManagerCore.sol

168: results[ii] = _execute(values[ii], payloads[ii]);
```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L168

```solidity
File: LSP6KeyManagerCore.sol

224: if ((totalValues += values[ii]) > msg.value) {
```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L224

```solidity
File: LSP6KeyManagerCore.sol

229: signatures[ii],
```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L229

```solidity
File: LSP6KeyManagerCore.sol

230: nonces[ii],
```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L230

```solidity
File: LSP6KeyManagerCore.sol

231: validityTimestamps[ii],
```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L231

```solidity
File: LSP6KeyManagerCore.sol

232: values[ii],
```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L232

```solidity
File: LSP6KeyManagerCore.sol

233: payloads[ii]
```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L233

```solidity
File: LSP6SetDataModule.sol

667: allowedERC725YDataKeysCompacted[pointer],
```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L667

```solidity
File: LSP6SetDataModule.sol

729: if (validatedInputKeysList[ii]) {
```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L729

```solidity
File: LSP6SetDataModule.sol

737: if ((inputDataKeys[ii] & mask) == allowedKey) {
```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L737

```solidity
File: LSP6SetDataModule.sol

763: if (!validatedInputKeysList[jj]) {
```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L763

```solidity
File: LSP6SetDataModule.sol

766: inputDataKeys[jj]
```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L766








### <a href="#gas-summary">[GAS&#x2011;9]</a><a name="GAS&#x2011;9"> Usage of `uints`/`ints` smaller than 32 bytes (256 bits) incurs overhead

When using elements that are smaller than 32 bytes, your contract's gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html
Each operation involving a `uint8` costs an extra 22-28 gas (depending on whether the other operand is also a variable of type `uint8`) as compared to ones involving `uint256`, due to the compiler having to clear the higher bits of the memory word before operating on the `uint8`, as well as the associated stack operations of doing so. Use a larger size then downcast where needed

#### <ins>Proof Of Concept</ins>


<details>

```solidity
File: LSP10Utils.sol

83: uint128 oldArrayLength = uint128(bytes16(encodedArrayLength));
```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP10ReceivedVaults/LSP10Utils.sol#L83

```solidity
File: LSP10Utils.sol

89: uint128 newArrayLength = oldArrayLength + 1;
```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP10ReceivedVaults/LSP10Utils.sol#L89

```solidity
File: LSP10Utils.sol

133: uint128 oldArrayLength = uint128(bytes16(lsp10VaultsCountValue));
```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP10ReceivedVaults/LSP10Utils.sol#L133

```solidity
File: LSP10Utils.sol

140: uint128 newArrayLength = oldArrayLength - 1;
```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP10ReceivedVaults/LSP10Utils.sol#L140

```solidity
File: LSP1Utils.sol

89: interfaceId = _INTERFACEID_LSP7;
```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP1UniversalReceiver/LSP1Utils.sol#L89

```solidity
File: LSP1Utils.sol

96: interfaceId = _INTERFACEID_LSP8;
```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP1UniversalReceiver/LSP1Utils.sol#L96

```solidity
File: LSP1Utils.sol

103: interfaceId = _INTERFACEID_LSP9;
```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP1UniversalReceiver/LSP1Utils.sol#L103

```solidity
File: LSP1UniversalReceiverDelegateUP.sol

139: uint128 arrayIndex = uint128(uint160(bytes20(notifierMapValue)));
```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol#L139

```solidity
File: LSP5Utils.sol

85: uint128 oldArrayLength = uint128(bytes16(encodedArrayLength));
```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP5ReceivedAssets/LSP5Utils.sol#L85

```solidity
File: LSP5Utils.sol

134: uint128 oldArrayLength = uint128(bytes16(lsp5ReceivedAssetsCountValue));
```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP5ReceivedAssets/LSP5Utils.sol#L134

```solidity
File: LSP5Utils.sol

137: uint128 newArrayLength = oldArrayLength - 1;
```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP5ReceivedAssets/LSP5Utils.sol#L137

```solidity
File: LSP6KeyManagerCore.sol

387: uint128 startingTimestamp = uint128(validityTimestamps >> 128);
```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L387

```solidity
File: LSP6KeyManagerCore.sol

388: uint128 endingTimestamp = uint128(validityTimestamps);
```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L388

```solidity
File: LSP6Utils.sol

205: uint128 newArrayLength = arrayLength + 1;
```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Utils.sol#L205

```solidity
File: LSP6SetDataModule.sol

346: uint128 newLength = uint128(bytes16(inputDataValue));
```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L346



</details>





### <a href="#gas-summary">[GAS&#x2011;10]</a><a name="GAS&#x2011;10"> Use nested `if` and avoid multiple check combinations

Using nested `if`, is cheaper than using `&&` multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: LSP0ERC725AccountCore.sol

801: if (msg.sig == bytes4(0) && extension == address(0)) return;

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L801

```solidity
File: LSP10Utils.sol

76: if (encodedArrayLength.length != 0 && encodedArrayLength.length != 16) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP10ReceivedVaults/LSP10Utils.sol#L76

```solidity
File: LSP1UniversalReceiverDelegateUP.sol

227: if (dataKeys.length == 0 && dataValues.length == 0)

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol#L227



```solidity
File: LSP5Utils.sol

78: if (encodedArrayLength.length != 0 && encodedArrayLength.length != 16) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP5ReceivedAssets/LSP5Utils.sol#L78

```solidity
File: LSP6KeyManagerCore.sol

338: if (!isReentrantCall && !isSetData) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L338

```solidity
File: LSP6KeyManagerCore.sol

404: if (!isReentrantCall && !isSetData) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L404

```solidity
File: LSP6ExecuteModule.sol

165: if (isFundingContract && !hasSuperTransferValue) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L165

```solidity
File: LSP6ExecuteModule.sol

214: if (isTransferringValue && !hasSuperTransferValue) {
224: if (!hasSuperCall && !isCallDataPresent && !isTransferringValue) {
228: if (isCallDataPresent && !hasSuperCall) {
233: if (hasSuperCall && !isTransferringValue) return;
236: if (hasSuperTransferValue && !isCallDataPresent && isTransferringValue)
240: if (hasSuperCall && hasSuperTransferValue) return;

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L214

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L224

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L228

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L233

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L236

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L240



```solidity
File: LSP6SetDataModule.sol

362: if (inputDataValue.length != 0 && inputDataValue.length != 20) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L362



</details>


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

#### Gas Test Report

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




### <a href="#gas-summary">[GAS&#x2011;11]</a><a name="GAS&#x2011;11"> Using Openzeppelin `Ownable2Step.sol` is more gas efficient

The project makes secure Owner changes using the following function:

#### <ins>Proof Of Concept</ins>

```solidity
File: LSP0ERC725AccountCore.sol

624: function acceptOwnership() public virtual override {
        address previousOwner = owner();

        _acceptOwnership();

        
        previousOwner.tryNotifyUniversalReceiver(
            _TYPEID_LSP0_OwnershipTransferred_SenderNotification,
            ""
        );

        
        msg.sender.tryNotifyUniversalReceiver(
            _TYPEID_LSP0_OwnershipTransferred_RecipientNotification,
            ""
        );
    }

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L624



```solidity
File: LSP14Ownable2Step.sol

87: function acceptOwnership() public virtual {
        address previousOwner = owner();

        _acceptOwnership();

        previousOwner.tryNotifyUniversalReceiver(
            _TYPEID_LSP14_OwnershipTransferred_SenderNotification,
            ""
        );

        msg.sender.tryNotifyUniversalReceiver(
            _TYPEID_LSP14_OwnershipTransferred_RecipientNotification,
            ""
        );
    }

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol#L87


```solidity
File: LSP14Ownable2Step.sol

136: function _acceptOwnership() internal virtual {
        require(
            msg.sender == pendingOwner(),
            "LSP14: caller is not the pendingOwner"
        );

        _setOwner(msg.sender);
        delete _pendingOwner;
    }

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol#L136


However, it is recommended to use the more gas-optimized Openzeppelin project of `Ownable2Step.sol` as it is much more gas-efficient.

```solidity
function acceptOwnership() public virtual {
        address sender = _msgSender();
        require(pendingOwner() == sender, "Ownable2Step: caller is not the new owner");
        _transferOwnership(sender);
    }
```
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable2Step.sol




### <a href="#gas-summary">[GAS&#x2011;12]</a><a name="GAS&#x2011;12"> Using XOR (^) and AND (&) bitwise equivalents

Given 4 variables a, b, c and d represented as such:

    0 0 0 0 0 1 1 0 <- a
    0 1 1 0 0 1 1 0 <- b
    0 0 0 0 0 0 0 0 <- c
    1 1 1 1 1 1 1 1 <- d

To have a == b means that every 0 and 1 match on both variables. Meaning that a XOR (operator ^) would evaluate to 0 ((a ^ b) == 0), as it excludes by definition any equalities.Now, if a != b, this means that there’s at least somewhere a 1 and a 0 not matching between a and b, making (a ^ b) != 0.Both formulas are logically equivalent and using the XOR bitwise operator costs actually the same amount of gas.However, it is much cheaper to use the bitwise OR operator (|) than comparing the truthy or falsy values.These are logically equivalent too, as the OR bitwise operator (|) would result in a 1 somewhere if any value is not 0 between the XOR (^) statements, meaning if any XOR (^) statement verifies that its arguments are different.

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: LSP0ERC725AccountCore.sol

235: if (msg.sender == _owner) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L235

```solidity
File: LSP0ERC725AccountCore.sol

293: if (msg.sender == _owner) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L293

```solidity
File: LSP0ERC725AccountCore.sol

350: if (msg.sender == _owner) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L350

```solidity
File: LSP0ERC725AccountCore.sol

395: if (msg.sender == _owner) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L395

```solidity
File: LSP0ERC725AccountCore.sol

563: if (msg.sender == currentOwner) {
577: currentOwner == owner(),

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L563

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L577



```solidity
File: LSP0ERC725AccountCore.sol

663: if (msg.sender == _owner) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L663

```solidity
File: LSP0ERC725AccountCore.sol

702: interfaceId == _INTERFACEID_ERC1271 ||
703: interfaceId == _INTERFACEID_LSP0 ||
704: interfaceId == _INTERFACEID_LSP1 ||
705: interfaceId == _INTERFACEID_LSP14 ||
706: interfaceId == _INTERFACEID_LSP20_CALL_VERIFICATION ||

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L702-L706

```solidity
File: LSP0ERC725AccountCore.sol

751: result.length == 32 &&
770: recoveredAddress == _owner

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L751

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L770



```solidity
File: LSP0ERC725AccountCore.sol

801: if (msg.sig == bytes4(0) && extension == address(0)) return;
804: if (extension == address(0))

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L801

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L804



```solidity
File: LSP10Utils.sol

85: if (oldArrayLength == type(uint128).max) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP10ReceivedVaults/LSP10Utils.sol#L85

```solidity
File: LSP10Utils.sol

149: if (vaultIndex == newArrayLength) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP10ReceivedVaults/LSP10Utils.sol#L149

```solidity
File: LSP14Ownable2Step.sol

79: currentOwner == owner(),

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol#L79

```solidity
File: LSP14Ownable2Step.sol

127: if (newOwner == address(this)) revert CannotTransferOwnershipToSelf();

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol#L127

```solidity
File: LSP14Ownable2Step.sol

138: msg.sender == pendingOwner(),

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol#L138

```solidity
File: LSP14Ownable2Step.sol

158: _renounceOwnershipStartedAt == 0

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol#L158

```solidity
File: LSP17Extendable.sol

32: interfaceId == _INTERFACEID_LSP17_EXTENDABLE ||

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP17ContractExtension/LSP17Extendable.sol#L32

```solidity
File: LSP17Extendable.sol

50: if (erc165Extension == address(0)) return false;

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP17ContractExtension/LSP17Extendable.sol#L50

```solidity
File: LSP17Extendable.sol

91: if (extension == address(0))

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP17ContractExtension/LSP17Extendable.sol#L91

```solidity
File: LSP17Extension.sol

24: interfaceId == _INTERFACEID_LSP17_EXTENSION ||

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP17ContractExtension/LSP17Extension.sol#L24

```solidity
File: LSP1Utils.sol

85: typeId == _TYPEID_LSP7_TOKENSSENDER ||
86: typeId == _TYPEID_LSP7_TOKENSRECIPIENT
90: isReceiving = typeId == _TYPEID_LSP7_TOKENSRECIPIENT ? true : false;
92: typeId == _TYPEID_LSP8_TOKENSSENDER ||
93: typeId == _TYPEID_LSP8_TOKENSRECIPIENT
97: isReceiving = typeId == _TYPEID_LSP8_TOKENSRECIPIENT ? true : false;
99: typeId == _TYPEID_LSP9_OwnershipTransferred_SenderNotification ||
100: typeId == _TYPEID_LSP9_OwnershipTransferred_RecipientNotification

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP1UniversalReceiver/LSP1Utils.sol#L85

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP1UniversalReceiver/LSP1Utils.sol#L86

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP1UniversalReceiver/LSP1Utils.sol#L90

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP1UniversalReceiver/LSP1Utils.sol#L92

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP1UniversalReceiver/LSP1Utils.sol#L93

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP1UniversalReceiver/LSP1Utils.sol#L97

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP1UniversalReceiver/LSP1Utils.sol#L99

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP1UniversalReceiver/LSP1Utils.sol#L100



```solidity
File: LSP1UniversalReceiverDelegateUP.sol

103: if (notifier == tx.origin) revert CannotRegisterEOAsAsAssets(notifier);

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol#L103

```solidity
File: LSP1UniversalReceiverDelegateUP.sol

170: if (balance == 0) return "LSP1: balance not updated";

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol#L170

```solidity
File: LSP1UniversalReceiverDelegateUP.sol

227: if (dataKeys.length == 0 && dataValues.length == 0)

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol#L227




```solidity
File: LSP1UniversalReceiverDelegateUP.sol

263: interfaceId == _INTERFACEID_LSP1 ||

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol#L263

```solidity
File: LSP2Utils.sol

346: if (pointer == compactBytesArray.length) return true;

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol#L346

```solidity
File: LSP4DigitalAssetMetadata.sol

56: if (dataKey == _LSP4_TOKEN_NAME_KEY) {
58: } else if (dataKey == _LSP4_TOKEN_SYMBOL_KEY) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP4DigitalAssetMetadata/LSP4DigitalAssetMetadata.sol#L56

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP4DigitalAssetMetadata/LSP4DigitalAssetMetadata.sol#L58



```solidity
File: LSP4DigitalAssetMetadataInitAbstract.sol

54: if (dataKey == _LSP4_TOKEN_NAME_KEY) {
56: } else if (dataKey == _LSP4_TOKEN_SYMBOL_KEY) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP4DigitalAssetMetadata/LSP4DigitalAssetMetadataInitAbstract.sol#L54

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP4DigitalAssetMetadata/LSP4DigitalAssetMetadataInitAbstract.sol#L56



```solidity
File: LSP5Utils.sol

87: if (oldArrayLength == type(uint128).max) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP5ReceivedAssets/LSP5Utils.sol#L87

```solidity
File: LSP5Utils.sol

146: if (assetIndex == newArrayLength) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP5ReceivedAssets/LSP5Utils.sol#L146

```solidity
File: LSP6KeyManagerCore.sol

98: interfaceId == _INTERFACEID_LSP6 ||
99: interfaceId == _INTERFACEID_ERC1271 ||
100: interfaceId == _INTERFACEID_LSP20_CALL_VERIFIER ||

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L98-L100

```solidity
File: LSP6KeyManagerCore.sol

265: if (msg.sender == _target) {
273: isSetData || isReentrantCall

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L265

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L273



```solidity
File: LSP6KeyManagerCore.sol

309: if (msg.sender == _target) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L309

```solidity
File: LSP6KeyManagerCore.sol

463: if (permissions == bytes32(0)) revert NoPermissionsSet(from);
468: if (erc725Function == IERC725Y.setData.selector) {
484: } else if (erc725Function == IERC725Y.setDataBatch.selector) {
498: } else if (erc725Function == IERC725X.execute.selector) {
506: erc725Function == ILSP14Ownable2Step.transferOwnership.selector ||
507: erc725Function == ILSP14Ownable2Step.acceptOwnership.selector

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L463

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L468

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L484

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L498

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L506

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L507



```solidity
File: LSP6KeyManagerInitAbstract.sol

21: if (target_ == address(0)) revert InvalidLSP6Target();

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerInitAbstract.sol#L21

```solidity
File: LSP6Utils.sol

110: if (pointer == allowedCallsCompacted.length) return true;

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Utils.sol#L110

```solidity
File: LSP6Utils.sol

137: if (elementLength == 0 || elementLength > 32) return false;
140: if (pointer == allowedERC725YDataKeysCompacted.length) return true;

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Utils.sol#L137

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Utils.sol#L140



```solidity
File: LSP6Utils.sol

229: if (permission == _PERMISSION_CHANGEOWNER) return "TRANSFEROWNERSHIP";
230: if (permission == _PERMISSION_EDITPERMISSIONS) return "EDITPERMISSIONS";
231: if (permission == _PERMISSION_ADDCONTROLLER) return "ADDCONTROLLER";
232: if (permission == _PERMISSION_ADDEXTENSIONS) return "ADDEXTENSIONS";
233: if (permission == _PERMISSION_CHANGEEXTENSIONS)
235: if (permission == _PERMISSION_ADDUNIVERSALRECEIVERDELEGATE)
237: if (permission == _PERMISSION_CHANGEUNIVERSALRECEIVERDELEGATE)
239: if (permission == _PERMISSION_REENTRANCY) return "REENTRANCY";
240: if (permission == _PERMISSION_SETDATA) return "SETDATA";
241: if (permission == _PERMISSION_CALL) return "CALL";
242: if (permission == _PERMISSION_STATICCALL) return "STATICCALL";
243: if (permission == _PERMISSION_DELEGATECALL) return "DELEGATECALL";
244: if (permission == _PERMISSION_DEPLOY) return "DEPLOY";
245: if (permission == _PERMISSION_TRANSFERVALUE) return "TRANSFERVALUE";
246: if (permission == _PERMISSION_SIGN) return "SIGN";

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Utils.sol#L229

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Utils.sol#L230

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Utils.sol#L231

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Utils.sol#L232

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Utils.sol#L233

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Utils.sol#L235

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Utils.sol#L237

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Utils.sol#L239

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Utils.sol#L240

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Utils.sol#L241

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Utils.sol#L242

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Utils.sol#L243

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Utils.sol#L244

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Utils.sol#L245

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Utils.sol#L246



```solidity
File: LSP6ExecuteModule.sol

86: if (to == address(this)) {
107: if (operationType == OPERATION_0_CALL) {
119: operationType == OPERATION_1_CREATE ||
120: operationType == OPERATION_2_CREATE2
137: if (operationType == OPERATION_3_STATICCALL) {
148: if (operationType == OPERATION_4_DELEGATECALL) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L86

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L107

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L119

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L120

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L137

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L148



```solidity
File: LSP6ExecuteModule.sol

262: if (allowedCalls.length == 0) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L262

```solidity
File: LSP6ExecuteModule.sol

329: if (operationType == OPERATION_0_CALL) {
331: } else if (operationType == OPERATION_3_STATICCALL) {
333: } else if (operationType == OPERATION_4_DELEGATECALL) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L329

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L331

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L333



```solidity
File: LSP6ExecuteModule.sol

362: executeCalldata.length == 164

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L362

```solidity
File: LSP6ExecuteModule.sol

378: allowedAddress == address(bytes20(type(uint160).max)) ||
379: to == allowedAddress;

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L378-L379

```solidity
File: LSP6ExecuteModule.sol

395: allowedStandard == bytes4(type(uint32).max) ||

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L395

```solidity
File: LSP6ExecuteModule.sol

414: allowedFunction == bytes4(type(uint32).max) ||
415: (isFunctionCall && (requiredFunction == allowedFunction));

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L414-L415

```solidity
File: LSP6ExecuteModule.sol

430: return (allowedCallType & requiredCallTypes == requiredCallTypes);

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L430

```solidity
File: LSP6SetDataModule.sol

142: if (requiredPermission == _PERMISSION_SETDATA) {
97: if (requiredPermission == bytes32(0)) return;

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L142

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L97



```solidity
File: LSP6SetDataModule.sol

142: if (requiredPermission == _PERMISSION_SETDATA) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L142

```solidity
File: LSP6SetDataModule.sol

282: inputDataKey == _LSP1_UNIVERSAL_RECEIVER_DELEGATE_KEY ||

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L282

```solidity
File: LSP6SetDataModule.sol

341: if (inputDataKey == _LSP6KEY_ADDRESSPERMISSIONS_ARRAY) {
374: ERC725Y(controlledContract).getData(inputDataKey).length == 0

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L341

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L374



```solidity
File: LSP6SetDataModule.sol

426: ERC725Y(controlledContract).getData(dataKey).length == 0

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L426

```solidity
File: LSP6SetDataModule.sol

461: ERC725Y(controlledContract).getData(dataKey).length == 0

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L461

```solidity
File: LSP6SetDataModule.sol

479: ERC725Y(controlledContract).getData(lsp1DelegateDataKey).length == 0

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L479

```solidity
File: LSP6SetDataModule.sol

512: if (allowedERC725YDataKeysCompacted.length == 0)

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L512

```solidity
File: LSP6SetDataModule.sol

629: if (allowedERC725YDataKeysCompacted.length == 0)
747: if (allowedDataKeysFound == inputKeysLength) return;

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L629

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6SetDataModule.sol#L747



```solidity
File: LSP7DigitalAsset.sol

55: interfaceId == _INTERFACEID_LSP7 ||

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/LSP7DigitalAsset.sol#L55

```solidity
File: LSP7DigitalAssetCore.sol

112: if (tokenOwner == operator) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L112

```solidity
File: LSP7DigitalAssetCore.sol

131: if (from == to) revert LSP7CannotSendToSelf();

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L131

```solidity
File: LSP7DigitalAssetCore.sol

268: if (operator == address(0)) {
272: if (operator == tokenOwner) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L268

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L272



```solidity
File: LSP7DigitalAssetCore.sol

300: if (to == address(0)) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L300

```solidity
File: LSP7DigitalAssetCore.sol

343: if (from == address(0)) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L343

```solidity
File: LSP7DigitalAssetCore.sol

406: if (from == address(0) || to == address(0)) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetCore.sol#L406

```solidity
File: LSP7DigitalAssetInitAbstract.sol

49: interfaceId == _INTERFACEID_LSP7 ||

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetInitAbstract.sol#L49

```solidity
File: LSP7CappedSupplyInitAbstract.sol

24: if (tokenSupplyCap_ == 0) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/extensions/LSP7CappedSupplyInitAbstract.sol#L24

```solidity
File: LSP8IdentifiableDigitalAsset.sol

49: interfaceId == _INTERFACEID_LSP8 ||

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAsset.sol#L49

```solidity
File: LSP8IdentifiableDigitalAssetCore.sol

84: if (tokenOwner == address(0)) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L84

```solidity
File: LSP8IdentifiableDigitalAssetCore.sol

115: if (operator == address(0)) {
119: if (tokenOwner == operator) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L115

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L119



```solidity
File: LSP8IdentifiableDigitalAssetCore.sol

139: if (operator == address(0)) {
143: if (tokenOwner == operator) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L139

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L143



```solidity
File: LSP8IdentifiableDigitalAssetCore.sol

183: return (caller == tokenOwner || _operators[tokenId].contains(caller));

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L183

```solidity
File: LSP8IdentifiableDigitalAssetCore.sol

319: if (to == address(0)) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L319

```solidity
File: LSP8IdentifiableDigitalAssetCore.sol

401: if (from == to) {
410: if (to == address(0)) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L401

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L410



```solidity
File: LSP8IdentifiableDigitalAssetInitAbstract.sol

49: interfaceId == _INTERFACEID_LSP8 ||

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetInitAbstract.sol#L49

```solidity
File: LSP8CappedSupplyInitAbstract.sol

26: if (tokenSupplyCap_ == 0) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CappedSupplyInitAbstract.sol#L26

```solidity
File: LSP8CompatibleERC721.sol

86: interfaceId == _INTERFACEID_ERC721 ||
87: interfaceId == _INTERFACEID_ERC721METADATA ||

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol#L86-L87

```solidity
File: LSP8CompatibleERC721.sol

129: if (operatorListLength == 0) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol#L129

```solidity
File: LSP8CompatibleERC721.sol

322: return retval == IERC721Receiver.onERC721Received.selector;
324: if (reason.length == 0) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol#L322

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol#L324



```solidity
File: LSP8CompatibleERC721InitAbstract.sol

86: interfaceId == _INTERFACEID_ERC721 ||
87: interfaceId == _INTERFACEID_ERC721METADATA ||

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol#L86-L87

```solidity
File: LSP8CompatibleERC721InitAbstract.sol

129: if (operatorListLength == 0) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol#L129

```solidity
File: LSP8CompatibleERC721InitAbstract.sol

322: return retval == IERC721Receiver.onERC721Received.selector;
324: if (reason.length == 0) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol#L322

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol#L324



```solidity
File: LSP8Enumerable.sol

36: if (from == address(0)) {
40: } else if (to == address(0)) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8Enumerable.sol#L36

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8Enumerable.sol#L40



```solidity
File: LSP8EnumerableInitAbstract.sol

38: if (from == address(0)) {
42: } else if (to == address(0)) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8EnumerableInitAbstract.sol#L38

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8EnumerableInitAbstract.sol#L42



```solidity
File: ERC725.sol

41: interfaceId == _INTERFACEID_ERC725X ||
42: interfaceId == _INTERFACEID_ERC725Y ||

```

https://github.com/code-423n4/2023-06-lukso/tree/main/submodules/ERC725/implementations/contracts/ERC725.sol#L41-L42

```solidity
File: ERC725InitAbstract.sol

43: interfaceId == _INTERFACEID_ERC725X ||
44: interfaceId == _INTERFACEID_ERC725Y ||

```

https://github.com/code-423n4/2023-06-lukso/tree/main/submodules/ERC725/implementations/contracts/ERC725InitAbstract.sol#L43-L44

```solidity
File: ERC725XCore.sol

68: interfaceId == _INTERFACEID_ERC725X ||

```

https://github.com/code-423n4/2023-06-lukso/tree/main/submodules/ERC725/implementations/contracts/ERC725XCore.sol#L68

```solidity
File: ERC725XCore.sol

83: if (operationType == OPERATION_0_CALL) {
88: if (operationType == OPERATION_1_CREATE) {
95: if (operationType == OPERATION_2_CREATE2) {
102: if (operationType == OPERATION_3_STATICCALL) {
119: if (operationType == OPERATION_4_DELEGATECALL) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/submodules/ERC725/implementations/contracts/ERC725XCore.sol#L83

https://github.com/code-423n4/2023-06-lukso/tree/main/submodules/ERC725/implementations/contracts/ERC725XCore.sol#L88

https://github.com/code-423n4/2023-06-lukso/tree/main/submodules/ERC725/implementations/contracts/ERC725XCore.sol#L95

https://github.com/code-423n4/2023-06-lukso/tree/main/submodules/ERC725/implementations/contracts/ERC725XCore.sol#L102

https://github.com/code-423n4/2023-06-lukso/tree/main/submodules/ERC725/implementations/contracts/ERC725XCore.sol#L119



```solidity
File: ERC725XCore.sol

139: (targets.length != values.length || values.length != datas.length)
144: if (operationsType.length == 0) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/submodules/ERC725/implementations/contracts/ERC725XCore.sol#L139

https://github.com/code-423n4/2023-06-lukso/tree/main/submodules/ERC725/implementations/contracts/ERC725XCore.sol#L144



```solidity
File: ERC725XCore.sol

255: if (creationCode.length == 0) {
269: if (contractAddress == address(0)) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/submodules/ERC725/implementations/contracts/ERC725XCore.sol#L255

https://github.com/code-423n4/2023-06-lukso/tree/main/submodules/ERC725/implementations/contracts/ERC725XCore.sol#L269



```solidity
File: ERC725XCore.sol

292: if (creationCode.length == 0) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/submodules/ERC725/implementations/contracts/ERC725XCore.sol#L292

```solidity
File: ERC725YCore.sol

84: if (dataKeys.length == 0) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/submodules/ERC725/implementations/contracts/ERC725YCore.sol#L84

```solidity
File: ERC725YCore.sol

119: interfaceId == _INTERFACEID_ERC725Y ||

```

https://github.com/code-423n4/2023-06-lukso/tree/main/submodules/ERC725/implementations/contracts/ERC725YCore.sol#L119



</details>


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

#### Gas Test Report

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

