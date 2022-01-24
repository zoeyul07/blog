---
emoji: ''
title: 'Ethereum Smart Contract: Solidity'
date: '2022-01-20 22:00:00'
author: seoyul
tags: blockchain
categories: BlockChain
---

## solidity
*이 글은 freeCodeCamp의 [Solidity Tutorial](https://www.youtube.com/watch?v=ipwxYa-F1uY)와 Dapp University의 [Master Solidity](https://www.youtube.com/watch?v=v92oqiSKJQ8&list=PLS5SEs8ZftgVnWHv2_mkvJjn5HBOkde3g&index=5) 내용을 정리한 글입니다.

### Data types

```javascript
// 어떤 버전을 사용할건지
pragma solidity ^0.6.0;

contract DataTypes{
    // statically typed language이기 때문에 타입을 지정해줘야한다.
    /* 이 variable은 contract에 속한 것으로 
    function 안에서 선언하는 local variable 과는 다르게 blockchain 에 store 된다.*/
    string value;

    // constructor를 사용하지 않고 다음과 같이 바로 사용할 수 있다.
    // string public value = "myValue";
    
    bool public myBool = true;
    bytes32 public myByte43 = "hello world";
    //int는 negative 가 가능하지만 unint는 불가능하다.
    int public myInt = -1;
    // default는 256
    uint public myUint = 1;
    uint8 public myUint8 = 8;
    uint256 public myUint256 = 99999;

    /*every user that has an connection with blockchain has an address, 
    every contract on the blockchain has an address.*/
    address public myAddress = 0x5Ab54....;
    
    // enumerated list, allow us to keep track of set list of things in contract
    // Waiting =0, Ready = 1, Active = 2
    enum State { Watiting, Ready, Active }
    
    State public state;

    // constant keyword를 이용하면 아래에서와 같이 set으로 값을 변경할 수 없게 지정할 수 있다.
    // string public constant value = "myValue";

    // deploy 혹은 generate 될 때 value 값 지정
    constructor() public {
        value = "myValue";
        state = State.Waiting;
    }

    // public keyword를 사용하면 blockchain 상의 이 contract에 접근할 수 있는 사람이라면 누구나 사용할 수 있다.
    // data location must be "memory" for return parameter in function.
    
    /* string public value; 와 같이 public keyword를 이용해서 상단에서 선언하게 되면 
    get() function 없이도 value에 접근할 수 있다.*/
    
    // value값을 바꾸지 않고 그대로 return 하므로 view를 사용한다. (views value, not change value)
    function get() public view returns(string memory){
        return value;
    }

    function set(string memory _value) public {
        value = _value;
    }

    function activate() public {
        //sate가 Wating이면 0. Ready면 1, Active이면 2
        state = State.Active;
    }

    function isActive() public view returns(bool) {
        return state == State.Active;
    }
    
 }
 ```

 ### Arrays

 ```javascript
pragma solidity ^0.6.0;

contract Arrays {
    //Person list data type을 갖고있다.

    /* solidity는 array의 size를 모르기 때문에 전체 array를 return하지 않는다.
    그렇기 때문에 person 을 id로 reference 해줘야한다. ( array의 index와 같은 의미 )*/
    Person [] public people;

    uint256 public peopleCount;

    //struct: define your own data structures inside solidity
    struct Person {
        string _firstName;
        string _lastName;
    }

    function addPerson(string memory _firstName, string memory _lastName) public {
        people.push(Person(_firstName, _lastName));
        peopleCount +=1;
    }
    
    function valueCount() public view returns (uint){
        return people.length;
    }

}
```

### Mappings

``` javascript
contract Mappings {
    uint256 public peopleCount;

    //mapping: associative array
    //uint는 key(unsigned integer로, id와 같은 역할을 하게된다.), person은 value를 나타내게된다.
    mapping(uint => Person) public people;
		mapping(uint => string) public names;
		mapping(address => mapping(uint => Book)) public myMapping;
		
    struct Person {
        uint _id;
        string _firstName;
        string _lastName;
    }
    
     struct Book {
        string title;
        string author;
    }
    
      constructor() public {
        names[1] = "Adam";
        names[2] = "Bruce";
        names[3] = "Carl";
    }

    function addPerson(string memory _firstName, string memory _lastName) public {
        peopleCount +=1;

        //reference people number 1
        people[peopleCount] = Person(peopleCount, _firstName, _lastName);
    }
    
      function addMyBook(uint _id, string memory _title, string memory  _author) public {
        myBooks[msg.sender][_id] = Book(_title, _author);
    }


}
```

### Modifier

#### 특정 owner만 실행 가능

```javascript
pragma solidity ^0.6.0;

contract Modifier{
    uint256 public peopleCount;
    mapping(uint => Person) public people;

    address owner;

    modifier onlyOwner() {
        //mgs: global keyword, function meta data passed in
        //mgs.sender: tells us account, address who calls the function

        //require: triggers error and revert trasaction if false( no gas fee )
        require(msg.sender == owner);   
        _; 
    }

    struct Person {
        uint _id;
        string _firstName;
        string _lastName;
    }

    constructor() public {
        //msg.sender is account that deploys the smart contract
        owner = msg.sender;
    }
    //modifier, only the owner can call function(any other count is impossible)
    function addPerson(string memory _firstName, string memory _lastName) public onlyOwner {
        incrementCount();

        //reference people number 1
        people[peopleCount] = Person(peopleCount, _firstName, _lastName);
    }

    //internal: private function
    //not exposed outside of contract
    function incrementCount () internal{
        peopleCount += 1;
    }

}
```

#### 특정 시간이 지나면 실행
```javascript
contract Modifier{
    uint256 public peopleCount;
    mapping(uint => Person) public people;

    //time is expressed in second in solidity
    uint256 openingTime = 1642668800;

    modifier onlyWhileOpen() {
        //to get now, get current block timestamp
        require(block.timestamp >= openingTime);   
        _; 
    }

    struct Person {
        uint _id;
        string _firstName;
        string _lastName;
    }

    //only call function if time has passed
    function addPerson(string memory _firstName, string memory _lastName) public onlyWhileOpen {
        incrementCount();
        people[peopleCount] = Person(peopleCount, _firstName, _lastName);
    }

    function incrementCount () internal{
        peopleCount += 1;
    }

}
```


### Contract

```javascript
pragma solidity ^0.5.1;

contract MyContract{
    //how can we send ether when we call function

    mapping(address => uint256) public balances;

    /*solidity requires explicitness 
    whenever you are declaring an address that can accept ether 
    and set up smart contract*/
    address payable wallet;

    //can get event stream for event
    event Purchase (address _buyer, uint256 _amount);

    constructor(address payable _wallet) public{
        wallet = _wallet;
    }

    //pubic can be called inside and outside
    //external can only be called  outside
    function() external payable {
        buyToken(); 
    }

    function buyToken() public payable {
        //simulate what happen
        //buy token and send ether to the token
        balances[msg.sender] += 1;

        //mgs.value: sent value
        //transfer ether to wallet
        wallet.transfer(msg.value);

        //trigger event
        emit Purchase(msg.sender, 1);
    }
}
```

#### calling contract function in other function

```javascript
pragma solidity ^0.5.1;

contract ERC20Token {
    string public name;
    mapping (address => uint256) public balances;
     
    function mint() public {
        /*calling contract function in another function 
        passes mgs.sender as contract address, 
        not who sent initial transaction*/

        /*if you use like this(below) ERC20Token balances won't increase. use tx.origin
        balances[msg.sender] ++;*/
        
        /*tx.origin: alwalys the person who originated the transaction 
        even if the smart contract is calling the function, */
        balances[tx.origin] ++;
    }
}

//to use ERC20Token contract, need to know ERC20Token contract address
contract MyContract{
    address payable wallet;
    address public token;

    constructor(address payable _wallet, address _token) public{
        wallet = _wallet;
        token = _token;
    }

    function() external payable {
        buyToken(); 
    }

    function buyToken() public payable {
        ERC20Token _token = ERC20Token(address(token));

        //if you use like this ERC20Token balances won't increase
        _token.mint();
        wallet.transfer(msg.value);
    }
}
```
### Inherit

```javascript
pragma solidity ^0.5.1;

contract ERC20Token {
    string public name;
    mapping (address => uint256) public balances;

    constructor(string memory _name) public {
        name = _name;
    }
     
    function mint() public {
        balances[tx.origin] ++;
    }
}

//inherit
contract MyToken is ERC20Token{
    string public symbol;
    address[] public owners;
    uint256 ownerCount;

    //overriding constructor, inherit name from ERC20Token
    constructor(string memory _name, string memory _symbol) ERC20Token(_name) public {
        symbol = _symbol;
    }

    function mint() public{
        super.mint();
        ownerCount ++;
        owners.push(msg.sender);
    }


}
```

```javascript
pragma solidity ^0.6.0;

contract Ownable {
    address owner;   

    constructor () public {
        owner = msg.sender;
    } 

    modifier onlyOwner() {
        require(msg.sender == owner, "must be owner");
        _;
    }
}

contract SecretVault {
    string secret;

    constructor (string memory _secret) public {
        secret = _secret;
    }

    function getSecret() public view returns(string memory) {
        return secret;
    } 
    
}

contract Inherit is Ownable {
    address secretVault;

    constructor (string memory _secret) public {
        SecretVault _secretVault = new SecretVault(_secret);
        secretVault = address(_secretVault);
        super;
    }

    function getSecret() public view onlyOwner returns(string memory) {
        SecretVault _secretVault = SecretVault(secretVault);
        return _secretVault.getSecret();
    }   
}
```

### Library

```javascript
pragma solidity ^0.5.1;


//find functions or store variables ...etc

/*don't have full behavior of smart contract, don't have storages, can not inherit. 
It is meant to be used in smart contract*/

/*safer way to do division inside of solidity 
preventing error (e.g if b is 0, can not divide)*/ 
library Math {
    function divide(uint256 a, uint256 b) internal pure returns (uint256) {
        require( b > 0);
        uint256 c = a / b;
        
        return c;
    }
}
```
```javascript
pragma solidity ^0.5.1;

import "./Math.sol"

contract MyContract {
    uint256 public value;

    function calculate(uint _value1, uint _value2) public {
        //if you deploy contract, library will be deployed automatically.
        value = Math.divide(_value1, _value2);
    }
}
```

```javascript
//using math library
import "./SafeMath.sol";

 contract MyContract {
    using SafeMath for uint256;

    function calculate(uint _value1, uint _value2) public {
        value = _value1.div(_value2);
    }
}
```

### Conditional & Loop

```javascript
pragma solidity ^0.6.0;

/*the keyword view is introduced for functions (it replaces constant). 
Calling a view cannot alter the behaviour of future interactions with any contract. 
This means such functions cannot use STORE, cannot send or receive ether 
and can only call other view or pure functions.*/

/*the keyword pure is introduced for functions,
they are view functions with the additional restriction 
that their value only depends on the function arguments. 
This means they cannot use STORE, LOAD, cannot send or receive ether, 
cannot use msg or block and can only call other pure functions*/

contract Loop {
    uint[] public numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

    address public owner;

    constructor() public {
        owner = msg.sender;
    }

    function countEvenNumbers() public view returns (uint){
        uint count = 0;

        for(uint i = 0; i < numbers.length; i++) {
            if(isEvenNumber(numbers[i])){
                count ++;
            }
        }
        return count;
    }

    /*With view you cannot modify the contract storage, 
    but you can access the storage. 
    With pure you cannot access the contract storage. 
    For example contract getters are views, 
    and utility libraries that do not access conctract storage 
    but only its parameters can be declared as pure.*/
    function isEvenNumber(uint _number) public pure returns(bool) {
        //5 / 2 == 2 R1
        //4 / 2 == 2 R0
        if ( _number % 2 == 0 ) {
            return true;
        } else {
            return false;
        }
    }

    function isOwner() public view returns(bool) {
        return (msg.sender == owner);
    }
}
```
### Payment

```javascript
pragma solidity ^0.6.0;

contract HotelRoom {
    enum Status { Vacant, Occupied }

    Status currentStatus;
    address payable public owner;

    event Occupy(address _occupant, uint _value);

    constructor () public {
        //user who calls this function
        owner = msg.sender;
        currentStatus = Status.Vacant;
    }

    modifier onlyWhileVacant {
         require(currentStatus == Status.Vacant, "Currently occupied");
         _;
    }

    modifier costs (uint _amount) {
        require( msg.value >= _amount, "Not enough Ether provided");
        _;
    }

    /*receive(): will be triggered whenever you pay to this smart contract.
     send ether to the address, then this function will be called*/
    receive() external payable onlyWhileVacant costs(2 ether) {
        currentStatus = Status.Occupied;
        owner.transfer(msg.value);
        emit Occupy(msg.sender, msg.value);
    }
}
```
***

[Ref]
- [풀 영상 링크](https://www.youtube.com/watch?v=ipwxYa-F1uY)
- [시리즈 영상 링크](https://www.youtube.com/watch?v=wJnXuCFVGFA&list=PLS5SEs8ZftgVnWHv2_mkvJjn5HBOkde3g&index=4)


```toc

```
