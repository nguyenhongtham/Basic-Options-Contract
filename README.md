# Basic-Options-Contract
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.26;

import "@openzeppelin/contracts/access/Ownable.sol";

contract BaseOptions is Ownable {
    struct Option {
        address buyer;
        uint256 strikePrice;
        uint256 expiry;
        uint256 amount;
        bool exercised;
    }

    mapping(uint256 => Option) public options;
    uint256 public nextOptionId;

    event OptionCreated(uint256 id, address buyer, uint256 strikePrice, uint256 expiry);

    function createOption(uint256 strikePrice, uint256 duration) external payable {
        uint256 id = nextOptionId++;
        options[id] = Option({
            buyer: msg.sender,
            strikePrice: strikePrice,
            expiry: block.timestamp + duration,
            amount: msg.value,
            exercised: false
        });
        emit OptionCreated(id, msg.sender, strikePrice, block.timestamp + duration);
    }

    function exercise(uint256 optionId) external {
        Option storage opt = options[optionId];
        require(msg.sender == opt.buyer, "Not buyer");
        require(block.timestamp <= opt.expiry, "Expired");
        require(!opt.exercised, "Already exercised");

        opt.exercised = true;
        payable(msg.sender).transfer(opt.amount); // Simplified settlement
    }
}
