## QA Summary<a name="QA Summary">

### Low Risk Issues
| |Issue|Contexts|
|-|:-|:-:|
| [LOW&#x2011;1](#low1-double-type-casts-create-complexity-within-the-code) | Double type casts create complexity within the code | 2 |
| [LOW&#x2011;2](#low2-constructor-is-missing-zero-address-check) | Constructor is missing zero address check | 1 |
| [LOW&#x2011;3](#low3-assembly-calldata-forwarded-to-external-contract-is-missing-contract-existence-checks) | Assembly calldata forwarded to external contract is missing contract existence checks | 2 |
| [LOW&#x2011;4](#low4-missing-checks-for-address0x0-) | Missing Checks for Address(0x0)  | 1 |
| [LOW&#x2011;5](#low5-contracts-are-designed-to-receive-eth-but-do-not-implement-function-for-withdrawal) | Contracts are designed to receive ETH but do not implement function for withdrawal | 1 |
| [LOW&#x2011;6](#low6-unnecessary-low-level-calls-should-be-avoided) | Unnecessary Low level calls should be avoided | 1 |
| [LOW&#x2011;7](#low7-empty-receivepayable-fallback-function-does-not-authorize-requests) | Empty `receive()`/`payable fallback()` function does not authorize requests | 3 |

Total: 11 contexts over 7 issues

### Non-critical Issues
| |Issue|Contexts|
|-|:-|:-:|
| [NC&#x2011;1](#nc1-event-emit-should-emit-a-parameter) | Event emit should emit a parameter | 2 |
| [NC&#x2011;2](#nc2-function-creates-dirty-bits) | Function creates dirty bits | 2 |
| [NC&#x2011;3](#nc3-generate-perfect-code-headers-for-better-solidity-code-layout-and-readability) | Generate perfect code headers for better solidity code layout and readability | 7 |
| [NC&#x2011;4](#nc4-missing-event-for-critical-parameter-change) | Missing event for critical parameter change | 2 |
| [NC&#x2011;5](#nc5-consider-moving-msgsender-checks-to-a-common-authorization-modifier) | Consider moving `msg.sender` checks to a common authorization `modifier` | 9 |
| [NC&#x2011;6](#nc6-nonexternalpublic-function-names-should-begin-with-an-underscore) | Non-`external`/`public` function names should begin with an underscore | 37 |
| [NC&#x2011;7](#nc7-override-function-arguments-that-are-unused-should-have-the-variable-name-removed-or-commented-out-to-avoid-compiler-warnings) | `override` function arguments that are unused should have the variable name removed or commented out to avoid compiler warnings | 1 |
| [NC&#x2011;8](#nc8-using-underscore-at-the-end-of-variable-name) | Using underscore at the end of variable name | 8 |
| [NC&#x2011;9](#nc9-unusual-loop-variable) | Unusual loop variable | 4 |
| [NC&#x2011;10](#nc10-use-named-function-calls) | Use named function calls | 1 |
| [NC&#x2011;11](#nc11-use-of-_renounceownership-can-lead-to-unexpected-behaviors) | Use of `_renounceOwnership` can lead to unexpected behaviors | 10 |
| [NC&#x2011;12](#nc12-use-smtchecker) | Use SMTChecker | 1 |

Total: 84 contexts over 12 issues

## Low Risk Issues




### <a href="#qa-summary">[LOW&#x2011;1]</a><a name="LOW&#x2011;1"> Double type casts create complexity within the code

Double type casting should be avoided in Solidity contracts to prevent unintended consequences and ensure accurate data representation. Performing multiple type casts in succession can lead to unexpected truncation, rounding errors, or loss of precision, potentially compromising the contract's functionality and reliability. Furthermore, double type casting can make the code less readable and harder to maintain, increasing the likelihood of errors and misunderstandings during development and debugging. To ensure precise and consistent data handling, developers should use appropriate data types and avoid unnecessary or excessive type casting, promoting a more robust and dependable contract execution.

#### <ins>Proof Of Concept</ins>

```solidity
File: LSP1UniversalReceiverDelegateUP.sol

139: uint128 arrayIndex = uint128(uint160(bytes20(notifierMapValue)));

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP1UniversalReceiver/LSP1UniversalReceiverDelegateUP/LSP1UniversalReceiverDelegateUP.sol#L139

```solidity
File: LSP2Utils.sol

296: if (uint224(uint256(key)) != 0) return false;

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol#L296







### <a href="#qa-summary">[LOW&#x2011;2]</a><a name="LOW&#x2011;2"> Constructor is missing zero address check

In Solidity, constructors often take address parameters to initialize important components of a contract, such as owner or linked contracts. However, without a check, there's a risk that an address parameter could be mistakenly set to the zero address (0x0). This could occur due to a mistake or oversight during contract deployment. A zero address in a crucial role can cause serious issues, as it cannot perform actions like a normal address, and any funds sent to it are irretrievable. Therefore, it's crucial to include a zero address check in constructors to prevent such potential problems. If a zero address is detected, the constructor should revert the transaction.

#### <ins>Proof Of Concept</ins>

```solidity
File: LSP7DigitalAsset.sol

39: constructor: bool isNonDivisible_
```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/LSP7DigitalAsset.sol#L39







### <a href="#qa-summary">[LOW&#x2011;3]</a><a name="LOW&#x2011;3"> Assembly calldata forwarded to external contract is missing contract existence checks

Low-level calls return success if there is no code present at the specified address. In addition to the zero-address checks, add a check to verify that `extcodesize()` is non-zero.

#### <ins>Proof Of Concept</ins>

```solidity
File: LSP0ERC725AccountCore.sol

821: let success := call(

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L821

```solidity
File: LSP17Extendable.sol

108: let success := call(

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP17ContractExtension/LSP17Extendable.sol#L108






### <a href="#qa-summary">[LOW&#x2011;4]</a><a name="LOW&#x2011;4"> Missing Checks for Address(0x0) 

Lack of zero-address validation on address parameters may lead to transaction reverts, waste gas, require resubmission of transactions and may even force contract redeployments in certain cases within the protocol.

#### <ins>Proof Of Concept</ins>


```solidity
File: LSP1Utils.sol

41: function callUniversalReceiverWithCallerInfos: address universalReceiverDelegate
```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP1UniversalReceiver/LSP1Utils.sol#L41



#### <ins>Recommended Mitigation Steps</ins>

Consider adding explicit zero-address validation on input parameters of address type.




### <a href="#qa-summary">[LOW&#x2011;5]</a><a name="LOW&#x2011;5"> Contracts are designed to receive ETH but do not implement function for withdrawal

The following contracts can receive ETH but can not withdraw, ETH is occasionally sent by users will be stuck in those contracts.
This functionality also applies to baseTokens resulting in locked tokens and loss of funds.


#### <ins>Proof Of Concept</ins>

```solidity
File: LSP0ERC725AccountCore.sol

117: receive() external payable virtual {
```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L117



#### <ins>Recommended Mitigation Steps</ins>
Provide a rescue ETH and rescueTokens function




### <a href="#qa-summary">[LOW&#x2011;6]</a><a name="LOW&#x2011;6"> Unnecessary Low level calls should be avoided

Avoid making unnecessary low-level calls to the system whenever possible, as they function differently from contract-type calls. For instance:
Using `address.call(abi.encodeWithSelector("fancy(bytes32)", mybytes)` does not confirm whether the target is genuinely a contract, whereas `ContractInterface(address).fancy(mybytes)` does.

Moreover, when invoking functions declared as `view`/`pure`, the Solidity compiler executes a `staticcall`, offering extra security guarantees that are absent in low-level calls. Additionally, return values must be manually decoded when making low-level calls.

Note: If a low-level call is required, consider using `Contract.function.selector` instead of employing a hardcoded ABI string for encoding.

#### <ins>Proof Of Concept</ins>


```solidity
File: LSP20CallVerification.sol


function _verifyCall(
        address logicVerifier
    ) internal virtual returns (bool verifyAfter) {
        (bool success, bytes memory returnedData) = logicVerifier.call(
            abi.encodeWithSelector(
                ILSP20.lsp20VerifyCall.selector,
                msg.sender,
                msg.value,
                msg.data
            )
        );

        _validateCall(false, success, returnedData);

        bytes4 magicValue = abi.decode(returnedData, (bytes4));

        if (bytes3(magicValue) != bytes3(ILSP20.lsp20VerifyCall.selector))
            revert LSP20InvalidMagicValue(false, returnedData);

        return bytes1(magicValue[3]) == 0x01 ? true : false;
    }

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP20CallVerification/LSP20CallVerification.sol#L26





#### <ins>Recommended Mitigation Steps</ins>

When reaching out to a known contract within the system, always opt for typed contract calls (interfaces/contracts) rather than low-level calls.
This approach helps prevent mistakes, potentially unverified return values, and ensures security guarantees.



### <a href="#qa-summary">[LOW&#x2011;7]</a><a name="LOW&#x2011;7"> Empty `receive()`/`payable fallback()` function does not authorize requests

If the intention is for the Ether to be used, the function should call another function, otherwise it should revert (e.g. `require(msg.sender == address(weth))`). Having no access control on the function means that someone may send Ether to the contract, and have no way to get anything back out, which is a loss of funds. If the concern is having to spend a small amount of gas to check the sender against an immutable address, the code should at least have a function to rescue unused Ether.

#### <ins>Proof Of Concept</ins>


```solidity
File: ERC725.sol

18: receive() external payable {}
```

https://github.com/code-423n4/2023-06-lukso/tree/main/submodules/ERC725/implementations/contracts/ERC725.sol#L18

```solidity
File: ERC725Init.sol

12: receive() external payable {}
```

https://github.com/code-423n4/2023-06-lukso/tree/main/submodules/ERC725/implementations/contracts/ERC725Init.sol#L12

```solidity
File: ERC725InitAbstract.sol

34: receive() external payable {}
```

https://github.com/code-423n4/2023-06-lukso/tree/main/submodules/ERC725/implementations/contracts/ERC725InitAbstract.sol#L34






## Non Critical Issues


### <a href="#qa-summary">[NC&#x2011;1]</a><a name="NC&#x2011;1"> Event emit should emit a parameter

Some emitted events do not have any emitted parameters. It is recommended to add some parameter such as state changes or value changes when events are emitted

#### <ins>Proof Of Concept</ins>


```solidity
File: LSP14Ownable2Step.sol

165: emit RenounceOwnershipStarted()
178: emit OwnershipRenounced()

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol#L165

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol#L178






### <a href="#qa-summary">[NC&#x2011;2]</a><a name="NC&#x2011;2"> Function creates dirty bits

This explanation should be added in the NatSpec comments of this function that sends ether with call;

Note that this code probably isn’t secure or a good use case for assembly because a lot of memory management and security checks are bypassed.
Use with caution! Some functions in this contract knowingly create dirty bits at the destination of the free memory pointer.

#### <ins>Proof Of Concept</ins>


```solidity
File: LSP0ERC725AccountCore.sol

821: function _fallbackLSP17Extendable() internal virtual override {
        
        address extension = _getExtension(msg.sig);

        
        if (msg.sig == bytes4(0) && extension == address(0)) return;

        
        if (extension == address(0))
            revert NoExtensionFoundForFunctionSelector(msg.sig);

        
        
        
        assembly {
            calldatacopy(0, 0, calldatasize())

            
            
            mstore(calldatasize(), shl(96, caller()))

            
            mstore(add(calldatasize(), 20), callvalue())

            
            let success := call(
                gas(),
                extension,
                0,
                0,
                add(calldatasize(), 52),
                0,
                0
            )

            
            returndatacopy(0, 0, returndatasize())

            switch success
            
            case 0 {
                revert(0, returndatasize())
            }
            default {
                return(0, returndatasize())
            }
        }
    }

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L821

```solidity
File: LSP17Extendable.sol

108: function _getExtension(
        bytes4 functionSelector
    ) internal view virtual returns (address);

    
    function _fallbackLSP17Extendable() internal virtual {
        
        address extension = _getExtension(msg.sig);

        
        if (extension == address(0))
            revert NoExtensionFoundForFunctionSelector(msg.sig);

        
        
        
        assembly {
            calldatacopy(0, 0, calldatasize())

            
            
            mstore(calldatasize(), shl(96, caller()))

            
            mstore(add(calldatasize(), 20), callvalue())

            
            let success := call(
                gas(),
                extension,
                0,
                0,
                add(calldatasize(), 52),
                0,
                0
            )

            
            returndatacopy(0, 0, returndatasize())

            switch success
            
            case 0 {
                revert(0, returndatasize())
            }
            default {
                return(0, returndatasize())
            }
        }
    }
}

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP17ContractExtension/LSP17Extendable.sol#L108




#### <ins>Recommended Mitigation Steps</ins>
Add this comment to the function:
```
/// @dev Use with caution! Some functions in this contract knowingly create dirty bits at the destination of the free memory pointer. Note that this code probably isn’t secure or a good use case for assembly because a lot of memory management and security checks are bypassed.
```






### <a href="#qa-summary">[NC&#x2011;3]</a><a name="NC&#x2011;3"> Generate perfect code headers for better solidity code layout and readability

It is recommended to use pre-made headers for Solidity code layout and readability:
https://github.com/transmissions11/headers

```solidity
/*//////////////////////////////////////////////////////////////
                           TESTING 123
//////////////////////////////////////////////////////////////*/
```


#### <ins>Proof Of Concept</ins>


<details>

```solidity
File: LSP14Ownable2Step.sol

5: LSP14Ownable2Step.sol

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol#L5

```solidity
File: LSP4Compatibility.sol

5: LSP4Compatibility.sol

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP4DigitalAssetMetadata/LSP4Compatibility.sol#L5

```solidity
File: LSP7CompatibleERC20.sol

6: LSP7CompatibleERC20.sol

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/extensions/LSP7CompatibleERC20.sol#L6

```solidity
File: LSP7Mintable.sol

6: LSP7Mintable.sol

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/presets/LSP7Mintable.sol#L6

```solidity
File: LSP8CompatibleERC721.sol

9: LSP8CompatibleERC721.sol
43: LSP8CompatibleERC721.sol

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol#L9

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721.sol#L43



```solidity
File: LSP8Mintable.sol

6: LSP8Mintable.sol

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/presets/LSP8Mintable.sol#L6



</details>



### <a href="#qa-summary">[NC&#x2011;4]</a><a name="NC&#x2011;4"> Missing event for critical parameter change

When changing state variables events are not emitted. Emitting events allows monitoring activities with off-chain monitoring tools.

#### <ins>Proof Of Concept</ins>


```solidity
File: LSP6Utils.sol

150: function setDataViaKeyManager(
```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Utils.sol#L150

```solidity
File: ERC725YCore.sol

73: function setDataBatch(
```

https://github.com/code-423n4/2023-06-lukso/tree/main/submodules/ERC725/implementations/contracts/ERC725YCore.sol#L73





### <a href="#qa-summary">[NC&#x2011;5]</a><a name="NC&#x2011;5"> Consider moving `msg.sender` checks to a common authorization `modifier`

#### <ins>Proof Of Concept</ins>


<details>

```solidity
File: LSP0ERC725AccountCore.sol

235: if (msg.sender == _owner) {
293: if (msg.sender == _owner) {
350: if (msg.sender == _owner) {
395: if (msg.sender == _owner) {
663: if (msg.sender == _owner) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L235

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L293

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L350

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L395

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L663



```solidity
File: LSP0ERC725AccountCore.sol

563: if (msg.sender == currentOwner) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L563


```solidity
File: LSP6KeyManagerCore.sol

265: if (msg.sender == _target) {
309: if (msg.sender == _target) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L265

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerCore.sol#L309



```solidity
File: OwnableUnset.sol

61: require(owner() == msg.sender, "Ownable: caller is not the owner");

```

https://github.com/code-423n4/2023-06-lukso/tree/main/submodules/ERC725/implementations/contracts/custom/OwnableUnset.sol#L61



</details>





### <a href="#qa-summary">[NC&#x2011;6]</a><a name="NC&#x2011;6"> Non-`external`/`public` function names should begin with an underscore

According to the Solidity Style Guide, Non-`external`/`public` function names should begin with an <a href="https://docs.soliditylang.org/en/latest/style-guide.html#underscore-prefix-for-non-external-functions-and-variables">underscore</a>

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: LSP0Utils.sol

24: function getLSP1DelegateValue(
        mapping(bytes32 => bytes) storage erc725YStorage
    ) internal view returns (bytes memory) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0Utils.sol#L24

```solidity
File: LSP0Utils.sol

37: function getLSP1DelegateValueForTypeId(
        mapping(bytes32 => bytes) storage erc725YStorage,
        bytes32 typeId
    ) internal view returns (bytes memory) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0Utils.sol#L37

```solidity
File: LSP10Utils.sol

63: function generateReceivedVaultKeys(
        address receiver,
        address vault,
        bytes32 vaultMapKey
    ) internal view returns (bytes32[] memory keys, bytes[] memory values) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP10ReceivedVaults/LSP10Utils.sol#L63

```solidity
File: LSP10Utils.sol

115: function generateSentVaultKeys(
        address sender,
        bytes32 vaultMapKey,
        uint128 vaultIndex
    ) internal view returns (bytes32[] memory keys, bytes[] memory values) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP10ReceivedVaults/LSP10Utils.sol#L115

```solidity
File: LSP10Utils.sol

233: function getLSP10ReceivedVaultsCount(
        IERC725Y account
    ) internal view returns (bytes memory) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP10ReceivedVaults/LSP10Utils.sol#L233

```solidity
File: LSP17Utils.sol

13: function isExtension(
        uint256 parametersLengthWithOffset,
        uint256 msgDataLength
    ) internal pure returns (bool) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP17ContractExtension/LSP17Utils.sol#L13

```solidity
File: LSP1Utils.sol

22: function tryNotifyUniversalReceiver(
        address lsp1Implementation,
        bytes32 typeId,
        bytes memory data
    ) internal {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP1UniversalReceiver/LSP1Utils.sol#L22

```solidity
File: LSP1Utils.sol

40: function callUniversalReceiverWithCallerInfos(
        address universalReceiverDelegate,
        bytes32 typeId,
        bytes calldata receivedData,
        address msgSender,
        uint256 msgValue
    ) internal returns (bytes memory) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP1UniversalReceiver/LSP1Utils.sol#L40

```solidity
File: LSP1Utils.sol

72: function getTransferDetails(
        bytes32 typeId
    )
        internal
        pure
        returns (
            bool invalid,
            bytes10 mapPrefix,
            bytes4 interfaceId,
            bool isReceiving
        )
    {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP1UniversalReceiver/LSP1Utils.sol#L72

```solidity
File: LSP2Utils.sol

22: function generateSingletonKey(
        string memory keyName
    ) internal pure returns (bytes32) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol#L22

```solidity
File: LSP2Utils.sol

32: function generateArrayKey(
        string memory keyName
    ) internal pure returns (bytes32) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol#L32

```solidity
File: LSP2Utils.sol

52: function generateArrayElementKeyAtIndex(
        bytes32 arrayKey,
        uint128 index
    ) internal pure returns (bytes32) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol#L52

```solidity
File: LSP2Utils.sol

71: function generateMappingKey(
        string memory firstWord,
        string memory lastWord
    ) internal pure returns (bytes32) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol#L71

```solidity
File: LSP2Utils.sol

94: function generateMappingKey(
        string memory firstWord,
        address addr
    ) internal pure returns (bytes32) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol#L94

```solidity
File: LSP2Utils.sol

115: function generateMappingKey(
        bytes10 keyPrefix,
        bytes20 bytes20Value
    ) internal pure returns (bytes32) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol#L115

```solidity
File: LSP2Utils.sol

136: function generateMappingWithGroupingKey(
        string memory firstWord,
        string memory secondWord,
        address addr
    ) internal pure returns (bytes32) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol#L136

```solidity
File: LSP2Utils.sol

161: function generateMappingWithGroupingKey(
        bytes6 keyPrefix,
        bytes4 mapPrefix,
        bytes20 subMapKey
    ) internal pure returns (bytes32) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol#L161

```solidity
File: LSP2Utils.sol

181: function generateMappingWithGroupingKey(
        bytes10 keyPrefix,
        bytes20 bytes20Value
    ) internal pure returns (bytes32) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol#L181

```solidity
File: LSP2Utils.sol

199: function generateJSONURLValue(
        string memory hashFunction,
        string memory json,
        string memory url
    ) internal pure returns (bytes memory) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol#L199

```solidity
File: LSP2Utils.sol

216: function generateASSETURLValue(
        string memory hashFunction,
        string memory assetBytes,
        string memory url
    ) internal pure returns (bytes memory) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol#L216

```solidity
File: LSP2Utils.sol

231: function isEncodedArray(bytes memory data) internal pure returns (bool) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol#L231

```solidity
File: LSP2Utils.sol

251: function isEncodedArrayOfAddresses(
        bytes memory data
    ) internal pure returns (bool) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol#L251

```solidity
File: LSP2Utils.sol

283: function isBytes4EncodedArray(
        bytes memory data
    ) internal pure returns (bool) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol#L283

```solidity
File: LSP2Utils.sol

312: function isCompactBytesArray(
        bytes memory compactBytesArray
    ) internal pure returns (bool) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol#L312

```solidity
File: LSP5Utils.sol

64: function generateReceivedAssetKeys(
        address receiver,
        address asset,
        bytes32 assetMapKey,
        bytes4 interfaceID
    ) internal view returns (bytes32[] memory keys, bytes[] memory values) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP5ReceivedAssets/LSP5Utils.sol#L64

```solidity
File: LSP5Utils.sol

117: function generateSentAssetKeys(
        address sender,
        bytes32 assetMapKey,
        uint128 assetIndex
    ) internal view returns (bytes32[] memory keys, bytes[] memory values) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP5ReceivedAssets/LSP5Utils.sol#L117

```solidity
File: LSP5Utils.sol

244: function getLSP5ReceivedAssetsCount(
        IERC725Y account
    ) internal view returns (bytes memory) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP5ReceivedAssets/LSP5Utils.sol#L244

```solidity
File: LSP6Utils.sol

25: function getPermissionsFor(
        IERC725Y target,
        address caller
    ) internal view returns (bytes32) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Utils.sol#L25

```solidity
File: LSP6Utils.sol

39: function getAllowedCallsFor(
        IERC725Y target,
        address from
    ) internal view returns (bytes memory) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Utils.sol#L39

```solidity
File: LSP6Utils.sol

58: function getAllowedERC725YDataKeysFor(
        IERC725Y target,
        address caller
    ) internal view returns (bytes memory) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Utils.sol#L58

```solidity
File: LSP6Utils.sol

78: function hasPermission(
        bytes32 addressPermission,
        bytes32 permissionToCheck
    ) internal pure returns (bool) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Utils.sol#L78

```solidity
File: LSP6Utils.sol

91: function isCompactBytesArrayOfAllowedCalls(
        bytes memory allowedCallsCompacted
    ) internal pure returns (bool) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Utils.sol#L91

```solidity
File: LSP6Utils.sol

120: function isCompactBytesArrayOfAllowedERC725YDataKeys(
        bytes memory allowedERC725YDataKeysCompacted
    ) internal pure returns (bool) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Utils.sol#L120

```solidity
File: LSP6Utils.sol

150: function setDataViaKeyManager(
        address keyManagerAddress,
        bytes32[] memory keys,
        bytes[] memory values
    ) internal returns (bytes memory result) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Utils.sol#L150

```solidity
File: LSP6Utils.sol

169: function combinePermissions(
        bytes32[] memory permissions
    ) internal pure returns (bytes32) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Utils.sol#L169

```solidity
File: LSP6Utils.sol

194: function generateNewPermissionsKeys(
        IERC725Y account,
        address controller,
        bytes32 permissions
    ) internal view returns (bytes32[] memory keys, bytes[] memory values) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Utils.sol#L194

```solidity
File: LSP6Utils.sol

226: function getPermissionName(
        bytes32 permission
    ) internal pure returns (string memory errorMessage) {

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Utils.sol#L226



</details>





### <a href="#qa-summary">[NC&#x2011;7]</a><a name="NC&#x2011;7"> `override` function arguments that are unused should have the variable name removed or commented out to avoid compiler warnings

#### <ins>Proof Of Concept</ins>


```solidity
File: LSP0ERC725AccountCore.sol

851: function _getExtension: bytes4 functionSelector

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L851










### <a href="#qa-summary">[NC&#x2011;8]</a><a name="NC&#x2011;8"> Using underscore at the end of variable name

The use of underscore at the end of the variable name is uncommon and also suggests that the variable name was not completely changed. Consider refactoring `variableName_` to `variableName`.

#### <ins>Proof Of Concept</ins>


<details>

```solidity
File: LSP6KeyManager.sol

20: target_;

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManager.sol#L20

```solidity
File: LSP6KeyManagerInitAbstract.sol

22: target_;

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6KeyManagerInitAbstract.sol#L22

```solidity
File: LSP7DigitalAsset.sol

45: isNonDivisible_;

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/LSP7DigitalAsset.sol#L45

```solidity
File: LSP7DigitalAssetInitAbstract.sol

34: isNonDivisible_;

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/LSP7DigitalAssetInitAbstract.sol#L34

```solidity
File: LSP7CappedSupply.sol

28: tokenSupplyCap_;

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/extensions/LSP7CappedSupply.sol#L28

```solidity
File: LSP7CappedSupplyInitAbstract.sol

28: tokenSupplyCap_;

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP7DigitalAsset/extensions/LSP7CappedSupplyInitAbstract.sol#L28

```solidity
File: LSP8CappedSupply.sol

30: tokenSupplyCap_;

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CappedSupply.sol#L30

```solidity
File: LSP8CappedSupplyInitAbstract.sol

30: tokenSupplyCap_;

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CappedSupplyInitAbstract.sol#L30



</details>




### <a href="#qa-summary">[NC&#x2011;9]</a><a name="NC&#x2011;9"> Unusual loop variable

The normal name for loop variables is `i`, and when there is a nested loop, to use `j`. The code below chooses to use the variables differently, which may lead to confusion

#### <ins>Proof Of Concept</ins>

```solidity
File: LSP2Utils.sol

251: function isEncodedArrayOfAddresses(
        bytes memory data
    ) internal pure returns (bool) {
        if (!isEncodedArray(data)) return false;

        uint256 offset = uint256(bytes32(data));
        uint256 arrayLength = data.toUint256(offset);

        uint256 pointer = offset + 32;

        for (uint256 ii = 0; ii < arrayLength; ) {
            bytes32 key = data.toBytes32(pointer);

            
            
            if (bytes12(key) != bytes12(0)) return false;

            
            pointer += 32;

            unchecked {
                ++ii;
            }
        }

        return true;
    }

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol#L251

```solidity
File: LSP2Utils.sol

283: function isBytes4EncodedArray(
        bytes memory data
    ) internal pure returns (bool) {
        if (!isEncodedArray(data)) return false;

        uint256 offset = uint256(bytes32(data));
        uint256 arrayLength = data.toUint256(offset);
        uint256 pointer = offset + 32;

        for (uint256 ii = 0; ii < arrayLength; ) {
            bytes32 key = data.toBytes32(pointer);

            
            if (uint224(uint256(key)) != 0) return false;

            
            pointer += 32;

            unchecked {
                ++ii;
            }
        }

        return true;
    }

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP2ERC725YJSONSchema/LSP2Utils.sol#L283

```solidity
File: LSP6ExecuteModule.sol

245: function _verifyAllowedCall(
        address controlledContract,
        address controllerAddress,
        bytes calldata payload
    ) internal view virtual {
        (
            uint256 operationType,
            address to,
            uint256 value,
            bytes4 selector,
            bool isEmptyCall
        ) = _extractExecuteParameters(payload);

        
        bytes memory allowedCalls = ERC725Y(controlledContract)
            .getAllowedCallsFor(controllerAddress);

        if (allowedCalls.length == 0) {
            revert NoCallsAllowed(controllerAddress);
        }

        bytes4 requiredCallTypes = _extractCallType(
            operationType,
            value,
            isEmptyCall
        );

        for (uint256 ii = 0; ii < allowedCalls.length; ii += 34) {
            
            
            
            
            
            
            
            
            

            
            if (ii + 34 > allowedCalls.length) {
                revert InvalidEncodedAllowedCalls(allowedCalls);
            }

            
            bytes memory allowedCall = BytesLib.slice(allowedCalls, ii + 2, 32);

            
            
            
            if (
                bytes28(bytes32(allowedCall) << 32) ==
                bytes28(type(uint224).max)
            ) {
                revert InvalidWhitelistedCall(controllerAddress);
            }

            if (
                _isAllowedCallType(allowedCall, requiredCallTypes) &&
                _isAllowedAddress(allowedCall, to) &&
                _isAllowedStandard(allowedCall, to) &&
                _isAllowedFunction(allowedCall, selector)
            ) return;
        }

        revert NotAllowedCall(controllerAddress, to, selector);
    }

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP6KeyManager/LSP6Modules/LSP6ExecuteModule.sol#L245

```solidity
File: LSP8IdentifiableDigitalAssetCore.sol

258: function _clearOperators(
        address tokenOwner,
        bytes32 tokenId
    ) internal virtual {
        
        
        
        
        
        
        EnumerableSet.AddressSet storage operatorsForTokenId = _operators[
            tokenId
        ];

        uint256 operatorListLength = operatorsForTokenId.length();
        for (uint256 i = 0; i < operatorListLength; ) {
            
            address operator = operatorsForTokenId.at(0);
            _revokeOperator(operator, tokenOwner, tokenId);

            unchecked {
                ++i;
            }
        }
    }

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L258






### <a href="#qa-summary">[NC&#x2011;10]</a><a name="NC&#x2011;10"> Use named function calls

Code base has an extensive use of named function calls, but it somehow missed one instance where this would be appropriate.

It should use named function calls on function call, as such:
```solidity
    library.exampleFunction{value: _data.amount.value}({
        _id: _data.id,
        _amount: _data.amount.value,
        _token: _data.token,
        _example: "",
        _metadata: _data.metadata
    });
```

#### <ins>Proof Of Concept</ins>


```solidity
File: ERC725XCore.sol

186: (bool success, bytes memory returnData) = target.call{value: value}(
            data
        );

```

https://github.com/code-423n4/2023-06-lukso/tree/main/submodules/ERC725/implementations/contracts/ERC725XCore.sol#L186









### <a href="#qa-summary">[NC&#x2011;11]</a><a name="NC&#x2011;11"> Use of `_renounceOwnership` can lead to unexpected behaviors

Renouncing to the ownership of a contract can lead to unexpected behaviors. Method `setAddresses` can only be called once therefore, due to the call from the method to `_renounceOwnership`. Consider using a method to submit a proposal of new addresses, then another one to accept the proposal where `_renounceOwnership` is called at the end.

#### <ins>Proof Of Concept</ins>


```solidity
File: LSP0ERC725AccountCore.sol

664: return LSP14Ownable2Step._renounceOwnership();
672: LSP14Ownable2Step._renounceOwnership();

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L664

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP0ERC725Account/LSP0ERC725AccountCore.sol#L672



```solidity
File: LSP14Ownable2Step.sol

112: _renounceOwnership();

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol#L112

```solidity
File: LSP14Ownable2Step.sol

130: delete _renounceOwnershipStartedAt;
177: delete _renounceOwnershipStartedAt;

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol#L130

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol#L177



```solidity
File: LSP14Ownable2Step.sol

150: function _renounceOwnership() internal virtual {
152: uint256 confirmationPeriodStart = _renounceOwnershipStartedAt +
158: _renounceOwnershipStartedAt == 0
161: _renounceOwnershipStartedAt == 0
163: _renounceOwnershipStartedAt = currentBlock;

```

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol#L150

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol#L152

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol#L158

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol#L161

https://github.com/code-423n4/2023-06-lukso/tree/main/contracts/LSP14Ownable2Step/LSP14Ownable2Step.sol#L163










### <a href="#qa-summary">[NC&#x2011;12]</a><a name="NC&#x2011;12"> Use SMTChecker

The highest tier of smart contract behavior assurance is formal mathematical verification. All assertions that are made are guaranteed to be true across all inputs → The quality of your asserts is the quality of your verification

https://twitter.com/0xOwenThurm/status/1614359896350425088?t=dbG9gHFigBX85Rv29lOjIQ&s=19




