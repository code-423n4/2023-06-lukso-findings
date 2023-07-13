<h1>Use calldata instead of memory for function arguments that do not get mutated</h1>

ERC725XCore.sol
 function execute(
        uint256 operationType,
        address target,
        uint256 value,
        bytes memory data
    ) public payable virtual override onlyOwner returns (bytes memory) {

https://github.com/ERC725Alliance/ERC725/blob/7171a0e25e83cfe4c4dec6262bb62b4422c0478f/implementations/contracts/ERC725XCore.sol#L40C4-L45C73

function executeBatch(
        uint256[] memory operationsType,
        address[] memory targets,
        uint256[] memory values,
        bytes[] memory datas
    ) public payable virtual override onlyOwner returns (bytes[] memory) 
https://github.com/ERC725Alliance/ERC725/blob/7171a0e25e83cfe4c4dec6262bb62b4422c0478f/implementations/contracts/ERC725XCore.sol#L52C5-L57C74

function _execute(
        uint256 operationType,
        address target,
        uint256 value,
        bytes memory data
    ) internal virtual returns (bytes memory) {
https://github.com/ERC725Alliance/ERC725/blob/7171a0e25e83cfe4c4dec6262bb62b4422c0478f/implementations/contracts/ERC725XCore.sol#L74C5-L79C48

function _executeBatch(
        uint256[] memory operationsType,
        address[] memory targets,
        uint256[] memory values,
        bytes[] memory datas
    ) internal virtual returns (bytes[] memory) {
https://github.com/ERC725Alliance/ERC725/blob/7171a0e25e83cfe4c4dec6262bb62b4422c0478f/implementations/contracts/ERC725XCore.sol#L127C5-L132C50

 function _executeCall(
        address target,
        uint256 value,
        bytes memory data
    ) internal virtual returns (bytes memory result) {
https://github.com/ERC725Alliance/ERC725/blob/7171a0e25e83cfe4c4dec6262bb62b4422c0478f/implementations/contracts/ERC725XCore.sol#L165C4-L169C55

  function _executeStaticCall(
        address target,
        bytes memory data
    ) internal virtual returns (bytes memory result) {
https://github.com/ERC725Alliance/ERC725/blob/7171a0e25e83cfe4c4dec6262bb62b4422c0478f/implementations/contracts/ERC725XCore.sol#L187C3-L190C55

  function _executeDelegateCall(
        address target,
        bytes memory data
    ) internal virtual returns (bytes memory result) {
https://github.com/ERC725Alliance/ERC725/blob/7171a0e25e83cfe4c4dec6262bb62b4422c0478f/implementations/contracts/ERC725XCore.sol#L204C3-L207C55

    function _deployCreate(
        uint256 value,
        bytes memory creationCode
    ) internal virtual returns (bytes memory newContract) {
https://github.com/ERC725Alliance/ERC725/blob/7171a0e25e83cfe4c4dec6262bb62b4422c0478f/implementations/contracts/ERC725XCore.sol#L221C1-L224C60

ERC725YCore.sol

  function getDataBatch(
        bytes32[] memory dataKeys
    ) public view virtual override returns (bytes[] memory dataValues) {
https://github.com/ERC725Alliance/ERC725/blob/7171a0e25e83cfe4c4dec6262bb62b4422c0478f/implementations/contracts/ERC725YCore.sol#L42C3-L44C73

   
    function setData(
        bytes32 dataKey,
        bytes memory dataValue
https://github.com/ERC725Alliance/ERC725/blob/7171a0e25e83cfe4c4dec6262bb62b4422c0478f/implementations/contracts/ERC725YCore.sol#L61C3-L64C31

 function setDataBatch(
        bytes32[] memory dataKeys,
        bytes[] memory dataValues
    ) public payable virtual override onlyOwner {
https://github.com/ERC725Alliance/ERC725/blob/7171a0e25e83cfe4c4dec6262bb62b4422c0478f/implementations/contracts/ERC725YCore.sol#L73C4-L76C50

LSP8Mintable.sol

 function mint(
        address to,
        bytes32 tokenId,
        bool allowNonLSP1Recipient,
        bytes memory data
https://github.com/code-423n4/2023-06-lukso/blob/9dbc96410b3052fc0fd9d423249d1fa42958cae8/contracts/LSP8IdentifiableDigitalAsset/presets/LSP8Mintable.sol#L32C4-L36C26

LSP8CompatibleERC721Mintable.sol


    function mint(
        address to,
        bytes32 tokenId,
        bool allowNonLSP1Recipient,
        bytes memory data
    ) public onlyOwner {
https://github.com/code-423n4/2023-06-lukso/blob/9dbc96410b3052fc0fd9d423249d1fa42958cae8/contracts/LSP8IdentifiableDigitalAsset/presets/LSP8CompatibleERC721Mintable.sol#L12C1-L18C25

LSP8CompatibleERC721MintableInit.sol
  function initialize(
        string memory name_,
        string memory symbol_,
        address newOwner_
    ) external virtual initializer {
https://github.com/code-423n4/2023-06-lukso/blob/9dbc96410b3052fc0fd9d423249d1fa42958cae8/contracts/LSP8IdentifiableDigitalAsset/presets/LSP8CompatibleERC721MintableInit.sol#L25C3-L29C37

LSP8MintableInit.sol
 function initialize(
        string memory name_,
        string memory symbol_,
   https://github.com/code-423n4/2023-06-lukso/blob/9dbc96410b3052fc0fd9d423249d1fa42958cae8/contracts/LSP8IdentifiableDigitalAsset/presets/LSP8MintableInit.sol#L25C4-L28C4

LSP20CallVerification.sol

 function _verifyCallResult(
        address logicVerifier,
        bytes memory callResult
    ) internal virtual {
https://github.com/code-423n4/2023-06-lukso/blob/9dbc96410b3052fc0fd9d423249d1fa42958cae8/contracts/LSP20CallVerification/LSP20CallVerification.sol#L49C4-L52C25

  function _validateCall(
        bool postCall,
        bool success,
        bytes memory returnedData
    ) internal pure {
https://github.com/code-423n4/2023-06-lukso/blob/9dbc96410b3052fc0fd9d423249d1fa42958cae8/contracts/LSP20CallVerification/LSP20CallVerification.sol#L69C3-L73C22

  function _revert(bool postCall, bytes memory returnedData) internal pure {
https://github.com/code-423n4/2023-06-lukso/blob/9dbc96410b3052fc0fd9d423249d1fa42958cae8/contracts/LSP20CallVerification/LSP20CallVerification.sol#L84C3-L84C79

LSP8IdentifiableDigitalAssetCore.sol
  function transfer(
        address from,
        address to,
        bytes32 tokenId,
        bool allowNonLSP1Recipient,
        bytes memory data
https://github.com/code-423n4/2023-06-lukso/blob/9dbc96410b3052fc0fd9d423249d1fa42958cae8/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L191C2-L196C26

  function transferBatch(
        address[] memory from,
        address[] memory to,
        bytes32[] memory tokenId,
        bool[] memory allowNonLSP1Recipient,
        bytes[] memory data
    ) public virtual {
https://github.com/code-423n4/2023-06-lukso/blob/9dbc96410b3052fc0fd9d423249d1fa42958cae8/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L210C3-L216C23

 function _mint(
        address to,
        bytes32 tokenId,
        bool allowNonLSP1Recipient,
        bytes memory data
https://github.com/code-423n4/2023-06-lukso/blob/9dbc96410b3052fc0fd9d423249d1fa42958cae8/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L313C4-L317C26

  function _burn(bytes32 tokenId, bytes memory data) internal virtual {
https://github.com/code-423n4/2023-06-lukso/blob/9dbc96410b3052fc0fd9d423249d1fa42958cae8/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L359C3-L359C74

  function _transfer(
        address from,
        address to,
        bytes32 tokenId,
        bool allowNonLSP1Recipient,
        bytes memory data
https://github.com/code-423n4/2023-06-lukso/blob/9dbc96410b3052fc0fd9d423249d1fa42958cae8/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L394C3-L399C26

 function _notifyTokenReceiver(
        address to,
        bool allowNonLSP1Recipient,
        bytes memory lsp1Data
https://github.com/code-423n4/2023-06-lukso/blob/9dbc96410b3052fc0fd9d423249d1fa42958cae8/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAssetCore.sol#L477C4-L480C30

 function safeTransferFrom(
        address from,
        address to,
        uint256 tokenId,
        bytes memory data
    ) public virtual {
        _safeTransfer(from, to, tokenId, data);
    }
https://github.com/code-423n4/2023-06-lukso/blob/9dbc96410b3052fc0fd9d423249d1fa42958cae8/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol#L197C3-L204C6

   function _transfer(
        address from,
        address to,
        bytes32 tokenId,
        bool allowNonLSP1Recipient,
        bytes memory data
    ) internal virtual override {
https://github.com/code-423n4/2023-06-lukso/blob/9dbc96410b3052fc0fd9d423249d1fa42958cae8/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol#L226C2-L232C34

  function _safeTransfer(
        address from,
        address to,
        uint256 tokenId,
        bytes memory data
    ) internal virtual {
https://github.com/code-423n4/2023-06-lukso/blob/9dbc96410b3052fc0fd9d423249d1fa42958cae8/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol#L246C3-L251C25


    function _mint(
        address to,
        bytes32 tokenId,
        bool allowNonLSP1Recipient,
        bytes memory data
https://github.com/code-423n4/2023-06-lukso/blob/9dbc96410b3052fc0fd9d423249d1fa42958cae8/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol#L258C1-L263C26

  function _burn(
        bytes32 tokenId,
        bytes memory data
    ) internal virtual override 
https://github.com/code-423n4/2023-06-lukso/blob/9dbc96410b3052fc0fd9d423249d1fa42958cae8/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol#L269C3-L272C33

function _checkOnERC721Received(
        address from,
        address to,
        uint256 tokenId,
        bytes memory data
    ) private returns (bool)
https://github.com/code-423n4/2023-06-lukso/blob/9dbc96410b3052fc0fd9d423249d1fa42958cae8/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol#L307C5-L312C29

 function _setData(
        bytes32 key,
        bytes memory value
    )
        internal
        virtual
        override(LSP4DigitalAssetMetadataInitAbstract, ERC725YCore)
    {
        super._setData(key, value);
    }
https://github.com/code-423n4/2023-06-lukso/blob/9dbc96410b3052fc0fd9d423249d1fa42958cae8/contracts/LSP8IdentifiableDigitalAsset/extensions/LSP8CompatibleERC721InitAbstract.sol#L341C4-L350C6