// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import {ReentrancyGuard} from "@openzeppelin/contracts/utils/ReentrancyGuard.sol";
import {Math} from "@openzeppelin/contracts/utils/math/Math.sol";

/// @title PARUFYX Goal
/// @notice Fan-Token mit linearer Bonding Curve:
///         price(s) = basePrice + slope * s / 1e18
///         Kurvenparameter sind unveraenderlich (kein Owner, kein setCurve).
///         Kauf rundet auf, Verkauf rundet ab.
///         Die Verkaufsgebuehr bleibt in der Reserve.
contract PARUFYXGoal is ERC20, ReentrancyGuard {
    uint256 public constant MAX_SUPPLY = 1_000_000_000 ether;
    uint256 private constant SCALE = 1 ether;

    uint256 public constant SELL_FEE_BPS = 100; // 1 %
    uint256 private constant BPS = 10_000;
    uint256 public constant MAX_SLOPE = 1e18;

    uint256 public constant MAX_USERNAME_BYTES = 32;
    uint256 public constant MAX_CLUB_BYTES = 48;

    uint256 public immutable basePrice;
    uint256 public immutable slope;

    /// @notice Aktuell verkaufte Menge (eigene Buchfuehrung)
    uint256 public sold;

    struct Profile {
        string username;
        string favoriteClub;
    }
    mapping(address => Profile) public profiles;

    event ProfileSet(address indexed user, string username, string favoriteClub);
    event Bought(address indexed buyer, uint256 amount, uint256 cost);
    event Sold(address indexed seller, uint256 amount, uint256 payout, uint256 fee);

    constructor(uint256 initialBasePrice, uint256 initialSlope)
        ERC20("PARUFYX Goal", "PFXG")
    {
        require(initialBasePrice > 0, "Invalid base price");
        require(initialSlope <= MAX_SLOPE, "Slope too high");
        basePrice = initialBasePrice;
        slope = initialSlope;
        _mint(address(this), MAX_SUPPLY);
    }

    function setProfile(string calldata username, string calldata favoriteClub) external {
        require(bytes(username).length <= MAX_USERNAME_BYTES, "Username too long");
        require(bytes(favoriteClub).length <= MAX_CLUB_BYTES, "Club name too long");
        profiles[msg.sender] = Profile({username: username, favoriteClub: favoriteClub});
        emit ProfileSet(msg.sender, username, favoriteClub);
    }

    function currentPrice() public view returns (uint256) {
        return basePrice + Math.mulDiv(slope, sold, SCALE);
    }

    function _integral(uint256 s0, uint256 amount, Math.Rounding rounding)
        private
        view
        returns (uint256)
    {
        uint256 linearPart = Math.mulDiv(basePrice, amount, SCALE, rounding);
        uint256 curvePart = Math.mulDiv(slope * amount, 2 * s0 + amount, 2 * SCALE * SCALE, rounding);
        return linearPart + curvePart;
    }

    function costFor(uint256 amount) public view returns (uint256) {
        require(sold + amount <= MAX_SUPPLY, "Not enough tokens left");
        return _integral(sold, amount, Math.Rounding.Ceil);
    }

    function payoutFor(uint256 amount) public view returns (uint256 net, uint256 fee) {
        require(amount <= sold, "Amount exceeds circulating supply");
        uint256 gross = _integral(sold - amount, amount, Math.Rounding.Floor);
        fee = Math.mulDiv(gross, SELL_FEE_BPS, BPS, Math.Rounding.Ceil);
        net = gross - fee;
    }

    /// @notice msg.value ist zugleich die Preisobergrenze, Ueberschuss geht zurueck.
    function buy(uint256 amount) external payable nonReentrant {
        require(amount > 0, "Amount must be > 0");
        uint256 cost = costFor(amount);
        require(msg.value >= cost, "Insufficient payment");

        sold += amount;
        _transfer(address(this), msg.sender, amount);
        emit Bought(msg.sender, amount, cost);

        uint256 excess = msg.value - cost;
        if (excess > 0) {
            (bool refunded, ) = msg.sender.call{value: excess}("");
            require(refunded, "Refund failed");
        }
    }

    function sell(uint256 amount, uint256 minPayout) external nonReentrant {
        require(amount > 0, "Amount must be > 0");
        (uint256 net, uint256 fee) = payoutFor(amount);
        require(net >= minPayout, "Slippage");
        require(address(this).balance >= net, "Insufficient reserve");

        sold -= amount;
        _transfer(msg.sender, address(this), amount);
        emit Sold(msg.sender, amount, net, fee);

        (bool sent, ) = msg.sender.call{value: net}("");
        require(sent, "Payout failed");
    }
}
