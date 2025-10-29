// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

/// @title VoteChori - simple voting contract for beginners
/// @notice Owner creates proposals, registers voters, starts/stops voting. Each registered voter can cast one vote.
contract VoteChori {
    address public owner;
    bool public votingActive;

    
