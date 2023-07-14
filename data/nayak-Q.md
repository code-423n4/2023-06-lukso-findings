
Use of floating pragma:
The contract should not use floating pragma, e.g. (*0.6.0 or >=0.4.0 *0.6.0), which allows a range of compiler versions. It is important to lock the pragma (for example, not using ^ in pragma solidity 0.8.10) to prevent contracts from being accidentally deployed using an older compiler with unfixed bugs.


As in "contracts/LSP0ERC725Account/ILSP0ERC725Account.sol", we can see ^0.8.4 is used, rather a version of 0.8.4 should be locked.

code in the contract:
pragma solidity ^0.8.4;

corrected code:
pragma solidity 0.8.4;