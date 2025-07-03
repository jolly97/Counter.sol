here my privacy token Code:
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.30;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

/**
 * @title Jolly Token (JLY)
 * @notice A privacy-oriented ERC20 token with transfer restrictions and admin control
 */
contract JollyToken is ERC20, Ownable {
    mapping(address => bool) private _whitelist;
    bool public privacyModeEnabled = false;

    constructor(uint256 initialSupply) ERC20("Jolly", "JLY") {
        _mint(msg.sender, initialSupply * 10 ** decimals());
        _whitelist[msg.sender] = true; // deployer is whitelisted
    }

    /**
     * @notice Enable or disable privacy mode (restricts transfers to only whitelisted users)
     */
    function setPrivacyMode(bool enabled) external onlyOwner {
        privacyModeEnabled = enabled;
    }

    /**
     * @notice Add an address to the whitelist
     */
    function addToWhitelist(address user) external onlyOwner {
        _whitelist[user] = true;
    }

    /**
     * @notice Remove an address from the whitelist
     */
    function removeFromWhitelist(address user) external onlyOwner {
        _whitelist[user] = false;
    }

    /**
     * @notice Check if an address is whitelisted
     */
    function isWhitelisted(address user) public view returns (bool) {
        return _whitelist[user];
    }

    /**
     * @dev Override ERC20 transfer functions to enforce privacy rules if enabled
     */
    function _beforeTokenTransfer(
        address from,
        address to,
        uint256 amount
    ) internal override {
        if (privacyModeEnabled && from != address(0)) {
            require(_whitelist[from] && _whitelist[to], "Privacy mode: transfer not allowed");
        }
        super._beforeTokenTransfer(from, to, amount);
    }
}
