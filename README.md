# betting-managament-contract
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/access/Ownable.sol";

contract BettingManagement is Ownable {
    struct Bet {
        address user;
        uint256 amount;
        uint8 outcome;
        bool settled;
    }

    mapping(uint256 => Bet[]) public eventBets;
    mapping(address => uint256) public balances;

    event BetPlaced(address indexed user, uint256 eventId, uint256 amount, uint8 outcome);
    event BetSettled(address indexed user, uint256 eventId, uint256 amount, bool won);
    event Withdrawn(address indexed user, uint256 amount);

    function placeBet(uint256 eventId, uint8 outcome) external payable {
        require(msg.value > 0, "Bet amount must be greater than zero");
        eventBets[eventId].push(Bet(msg.sender, msg.value, outcome, false));
        emit BetPlaced(msg.sender, eventId, msg.value, outcome);
    }

    function settleBets(uint256 eventId, uint8 winningOutcome) external onlyOwner {
        for (uint256 i = 0; i < eventBets[eventId].length; i++) {
            Bet storage bet = eventBets[eventId][i];
            if (!bet.settled) {
                bet.settled = true;
                if (bet.outcome == winningOutcome) {
                    uint256 winnings = bet.amount * 2; // Simple 2x payout
                    balances[bet.user] += winnings;
                    emit BetSettled(bet.user, eventId, winnings, true);
                } else {
                    emit BetSettled(bet.user, eventId, bet.amount, false);
                }
            }
        }
    }

    function withdraw() external {
        uint256 amount = balances[msg.sender];
        require(amount > 0, "No funds to withdraw");
        balances[msg.sender] = 0;
        payable(msg.sender).transfer(amount);
        emit Withdrawn(msg.sender, amount);
    }

    receive() external payable {}
}
13579
