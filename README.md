// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

/// @title VoteChori - simple voting contract for beginners
/// @notice Owner creates proposals, registers voters, starts/stops voting. Each registered voter can cast one vote.
contract VoteChori {
    address public owner;
    bool public votingActive;

    
qstruct Proposal {
        string name;
        uint256 voteCount;
    }

    // proposals array
    Proposal[] public proposals;

    // track registered voters and if they voted
    mapping(address => bool) public isRegistered;
    mapping(address => bool) public hasVoted;
    mapping(address => uint256) public votesGiven; // stores index of chosen proposal (optional)

    // events
    event ProposalAdded(uint256 indexed proposalId, string name);
    event VoterRegistered(address indexed voter);
    event VotingStarted();
    event VotingEnded();
    event Voted(address indexed voter, uint256 indexed proposalId);

    modifier onlyOwner() {
        require(msg.sender == owner, "Only owner");
        _;
    }

    modifier onlyWhileActive() {
        require(votingActive, "Voting is not active");
        _;
    }

    constructor() {
        owner = msg.sender;
        votingActive = false;
    }

    /// @notice Owner adds a proposal before voting starts
    /// @param _name Name/description of the proposal
    function addProposal(string calldata _name) external onlyOwner {
        require(!votingActive, "Cannot add proposals while voting");
        proposals.push(Proposal({name: _name, voteCount: 0}));
        emit ProposalAdded(proposals.length - 1, _name);
    }

    /// @notice Owner can register voters (whitelist). If you want open voting, skip registering any voters
    /// @param _voter address of voter
    function registerVoter(address _voter) external onlyOwner {
        require(!votingActive, "Cannot register while voting");
        require(!isRegistered[_voter], "Already registered");
        isRegistered[_voter] = true;
        emit VoterRegistered(_voter);
    }

    /// @notice Owner starts the voting phase
    function startVoting() external onlyOwner {
        require(!votingActive, "Voting already active");
        require(proposals.length > 0, "Need at least one proposal");
        votingActive = true;
        emit VotingStarted();
    }

    /// @notice Owner ends the voting phase
    function endVoting() external onlyOwner onlyWhileActive {
        votingActive = false;
        emit VotingEnded();
    }

    /// @notice Cast a vote for a proposal index. Voter must be registered and not have voted.
    /// @param _proposalId index in proposals array (0-based)
    function vote(uint256 _proposalId) external onlyWhileActive {
        require(_proposalId < proposals.length, "Invalid proposal");
        require(isRegistered[msg.sender], "Not registered to vote");
        require(!hasVoted[msg.sender], "Already voted");

        hasVoted[msg.sender] = true;
        votesGiven[msg.sender] = _proposalId;
        proposals[_proposalId].voteCount += 1;

        emit Voted(msg.sender, _proposalId);
    }

    /// @notice Read-only: get number of proposals
    function getProposalCount() external view returns (uint256) {
        return proposals.length;
    }

    /// @notice Read-only: get proposal details
    function getProposal(uint256 _id) external view returns (string memory name, uint256 voteCount) {
        require(_id < proposals.length, "Invalid proposal");
        Proposal storage p = proposals[_id];
        return (p.name, p.voteCount);
    }

    /// @notice Compute winner (index) by highest votes. If tie, returns first highest found.
    /// @dev This iterates through proposals (OK for small lists). Don't call on-chain in large lists.
    function winningProposal() public view returns (uint256 winnerId, string memory winnerName, uint256 winnerVotes) {
        uint256 highest = 0;
        uint256 winningIndex = 0;
        for (uint256 i = 0; i < proposals.length; i++) {
            if (proposals[i].voteCount > highest) {
                highest = proposals[i].voteCount;
                winningIndex = i;
            }
        }
        return (winningIndex, proposals[winningIndex].name, proposals[winningIndex].voteCount);
    }

    /// @notice Owner can change owner (transfer admin)
    function transferOwnership(address _newOwner) external onlyOwner {
        require(_newOwner != address(0), "Zero address");
        owner = _newOwner;
    }
}
