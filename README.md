pragma solidity ^0.8.0; 
contract SupplyChain { 
struct Item { 
uint id; 
address owner; 
} 
mapping(uint => Item) public items; 
bool private inProgress; // Flag to prevent re-entries 
modifier nonReentrant() { 
require(!inProgress, "Non-reentrancy lock"); 
inProgress = true; 
_; 
inProgress = false; 
}
function addItem(uint itemId, address initialOwner) public { 
require(items[itemId].owner == address(0), "Item ID already exists"); 
require(initialOwner != address(0), "Invalid owner address"); 
items[itemId] = Item({id: itemId, owner: initialOwner}); 
} 
function transferItem(uint itemId, address newOwner) public payable 
nonReentrant { 
// Check if the item exists 
require(items[itemId].owner != address(0), "Invalid Item ID"); 
require(items[itemId].owner == msg.sender, "Not the owner"); 
require(newOwner != address(0), "Invalid new owner"); 
require(newOwner != items[itemId].owner, "Cannot transfer to the same 
owner"); 
// Require a valid transaction value 
uint requiredValue = 1 ether; 
require(msg.value >= requiredValue, "Insufficient transaction value"); 
// Update ownership 
items[itemId].owner = newOwner; 
// Transfer Ether to the old owner 
(bool sent, ) = payable(msg.sender).call{value: msg.value}(""); 
require(sent, "Failed to transfer Ether to the previous owner"); 
} 
}
