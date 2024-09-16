pragma solidity ^0.8.0;

contract Marketplace {
    
    struct Item {
        string name;
        string description;
        uint256 price;
        address payable seller;
        bool available;
    }
    
    mapping(uint256 => Item) public items;
    uint256 public itemCount;
    
    event ItemListed(uint256 itemId, string name, uint256 price, address seller);
    event OrderPlaced(uint256 itemId, address buyer);
    event TransactionCompleted(uint256 itemId, address buyer, address seller, uint256 price);
    
    constructor() {
        itemCount = 0;
    }
    
    function listItem(string memory _name, string memory _description, uint256 _price) public {
        itemCount++;
        Item storage newItem = items[itemCount];
        newItem.name = _name;
        newItem.description = _description;
        newItem.price = _price;
        newItem.seller = payable(msg.sender);
        newItem.available = true;
        
        emit ItemListed(itemCount, _name, _price, msg.sender);
    }
    
    function placeOrder(uint256 _itemId) public payable {
        require(_itemId > 0 && _itemId <= itemCount, "Invalid item ID");
        Item storage item = items[_itemId];
        require(item.available, "Item is not available");
        require(msg.value == item.price, "Incorrect payment amount");
        
        item.available = false;
        
        emit OrderPlaced(_itemId, msg.sender);
    }
    
    function completeTransaction(uint256 _itemId) public {
        require(_itemId > 0 && _itemId <= itemCount, "Invalid item ID");
        Item storage item = items[_itemId];
        require(!item.available, "Item is still available");
        require(msg.sender != item.seller, "Seller cannot complete the transaction");
        
        item.seller.transfer(item.price);
        
        emit TransactionCompleted(_itemId, msg.sender, item.seller, item.price);
    }
    
    function getItem(uint256 _itemId) public view returns (string memory, string memory, uint256, address, bool) {
        require(_itemId > 0 && _itemId <= itemCount, "Invalid item ID");
        Item storage item = items[_itemId];
        
        return (item.name, item.description, item.price, item.seller, item.available);
    }
}
