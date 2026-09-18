// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import {Ownable} from "@openzeppelin/contracts/access/Ownable.sol";

/// @title PARUFYX Goal
/// @notice Einfacher Fan-Token mit klassischer linearer Bonding Curve:
///         price(s) = basePrice + slope * s, wobei s die Menge bereits
///         verkaufter Token ist. Je mehr Token im Umlauf sind, desto
///         teurer wird der naechste Kauf - und umgekehrt beim Verkauf.
///         Jede Adresse kann zusaetzlich ein Profil (Username +
///         Lieblingsverein) hinterlegen.
contract PARUFYXGoal is ERC20, Ownable {
    uint256 private constant SCALE = 1 ether;
    1_000_000_000 ether;
    uint256 private constant SCALE = 1 ether;

    /// @notice Startpreis pro Token in Wei, bei s = 0.
    uint256 public basePrice;
    /// @notice Preisanstieg pro verkauftem Token in Wei (Steigung der Kurve).
    uint256 public slope;

    struct Profile {
        string username;
        string favoriteClub;
    }
    mapping(address => Profile) public profiles;

    event ProfileSet(address indexed user, string username, string favoriteClub);
    event Bought(address indexed buyer, uint256 amount, uint256 cost);
    event Sold(address indexed seller, uint256 amount, uint256 payout);
    event CurveUpdated(uint256 basePrice, uint256 slope);

    constructor(address initialHolder, uint256 initialBasePrice, uint256 initialSlope)
        ERC20("PARUFYX Goal", "PFXG")
        Ownable(initialHolder)
    {
        require(initialHolder != address(0), "Invalid holder");
        require(initialBasePrice > 0, "Invalid base price");
        basePrice = initialBasePrice;
        slope = initialSlope;
        _mint(address(this), MAX_SUPPLY);
    }

    function setProfile(string calldata username, string calldata favoriteClub) external {
        profiles[msg.sender] = Profile({username: username, favoriteClub: favoriteClub});
        emit ProfileSet(msg.sender, username, favoriteClub);
    }

    function soldSupply() public view returns (uint256) {
        return MAX_SUPPLY - balanceOf(address(this));
    }

    function costFor(uint256 amount) public view returns (uint256) {
        uint256 s0 = soldSupply();
        uint256 linearPart = (basePrice * amount) / SCALE;
        uint256 curvePart = (slope * amount / SCALE) * (2 * s0 + amount) / (2 * SCALE);
        return linearPart + curvePart;
    }

    function payoutFor(uint256 amount) public view returns (uint256) {
        uint256 s0 = soldSupply();
        require(s0 >= amount, "Amount exceeds circulating supply");
        uint256 s1 = s0 - amount;
        uint256 linearPart = (basePrice * amount) / SCALE;
        uint256 curvePart = (slope * amount / SCALE) * (2 * s1 + amount) / (2 * SCALE);
        return linearPart + curvePart;
    }

    function buy(uint256 amount) external payable {
        require(amount > 0, "Amount must be > 0");
        require(balanceOf(address(this)) >= amount, "Not enough tokens left");
        uint256 cost = costFor(amount);
        require(msg.value >= cost, "Insufficient payment");
        _transfer(address(this), msg.sender, amount);
        if (msg.value > cost) {
            (bool refunded, ) = msg.sender.call{value: msg.value - cost}("");
            require(refunded, "Refund failed");
        }
        emit Bought(msg.sender, amount, cost);
    }

    function sell(uint256 amount) external {
        require(amount > 0, "Amount must be > 0");
        uint256 payout = payoutFor(amount);
        require(address(this).balance >= payout, "Insufficient reserve");
        _transfer(msg.sender, address(this), amount);
        (bool sent, ) = msg.sender.call{value: payout}("");
        require(sent, "Payout failed");
        emit Sold(msg.sender, amount, payout);
    }

    function setCurve(uint256 newBasePrice, uint256 newSlope) external onlyOwner {
        require(newBasePrice > 0, "Invalid base price");
        basePrice = newBasePrice;
        slope = newSlope;
        emit CurveUpdated(newBasePrice, newSlope);
    } 
    # PARUFYX Goal (PFXG)
