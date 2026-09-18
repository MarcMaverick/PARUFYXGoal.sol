l// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import {ERC20Permit} from "@openzeppelin/contracts/token/ERC20/extensions/ERC20Permit.sol";
import {ERC20Votes} from "@openzeppelin/contracts/token/ERC20/extensions/ERC20Votes.sol";
import {Nonces} from "@openzeppelin/contracts/utils/Nonces.sol";

contract PARUFYX is ERC20, ERC20Permit, ERC20Votes {
    uint256 public constant MAX_SUPPLY = 1_000_000_000 ether;

    constructor(address initialHolder)
        ERC20("PARUFYX", "PARUFYX")
        ERC20Permit("PARUFYX")
    {
        require(initialHolder != address(0), "Invalid holder");
        _mint(initialHolder, MAX_SUPPLY);
    }

    function _update(
        address from,
        address to,
        uint256 amount
    ) internal override(ERC20, ERC20Votes) {
        super._update(from, to, amount);
    }

    function nonces(
        address owner
    )
        public
        view
        override(ERC20Permit, Nonces)
        returns (uint256)
    {
        return super.nonces(owner);
    }

    // --- FIX ---
    // ERC20Votes nutzt standardmaessig einen block-basierten Clock-Mode.
    // Der Governor interpretiert votingDelay()/votingPeriod() ("1 days", "7 days")
    // dann faelschlich als Blockanzahl statt als Sekunden.
    // Durch Umstellung auf timestamp-basierten Clock-Mode werden diese Werte
    // korrekt als tatsaechliche Zeit (Sekunden) interpretiert.
    function clock() public view override returns (uint48) {
        return uint48(block.timestamp);
    }

    function CLOCK_MODE() public pure override returns (string memory) {
        return "mode=timestamp";
    }   }
