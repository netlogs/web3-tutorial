# Solidy简明教程2

 ## 存储位置

**memory** 内存的存储类型，**局部变量**，生命周期随着作用域结束而结束

**storage** **状态变量**的存储类型，有点像引用或者指针

**calldata** 只能用在参数中，如果使用calldata，会节约gas

~~~ solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.2;

contract DataLocations {
    struct Mystruct {
        uint foo;
        string text;
    }

    mapping(address => Mystruct) public mys;

    function examples(uint[] calldata y, string calldata s) external returns (uint[] memory) {
        mys[msg.sender] = Mystruct({foo: 123, text: "bar"});
         Mystruct storage p = mys[msg.sender]; //此处也可以直接mys[msg.sender].text进行修改，当修改量少时消耗较少的gas，修改项多时使用storage
         p.text = "new_text"; //mys[msg.sender].text is: new_text

         Mystruct memory readonly = mys[msg.sender];
         readonly.foo = 444; //but ..mys[msg.sender].foo is: 123

         _internal(y); 
         //1. 如果_internal函数的参数列表中的y是memory类型，那么这里调用的时候会进行一次拷贝，消耗gas
         //如果使用calldata那么可以直接传递过去，不会发生拷贝

         uint[] memory memArr = new uint[](5);
         memArr[0] = 1;
         return memArr;
    }

    function _internal(uint[] calldata y) private {
        uint x = y[0];
    }

}
~~~

## 事件

记录当前contract状态的方法，不会记录在状态变量之中，而是体现在区块链浏览器上或者交易记录中的log

~~~ solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.2;

contract Event {
    event Log(string message, uint val);
    event IndexLog(address indexed sender, uint val, uint a, uint b); //添加indexed 链外可以搜索查询

    

    function examples() external {
        emit Log("hahaha", 1234);
        emit IndexLog(msg.sender, 22, 22, 22); //在链外通过工具可以查询该地址的一些事件
        
    }

    event Message(address indexed _from, address indexed _to, string message);

    function sendMessage(address _to, string calldata message) external {
        emit Message(msg.sender, _to, message);
    }
}
~~~

## 继承那些事

~~~ solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.2;

contract A {
    function foo() public pure virtual returns (string memory) {
        return "A";
    }
    function bar() public pure virtual returns (string memory) {
        return "A";
    }
    function baz() public pure returns (string memory) {
        return "A";
    }
}

contract B is A { //继承写法
    function foo() public pure override returns (string memory) {
        return "B";
    }
    function bar() public pure virtual override returns (string memory) {
        return "A";
    }
}

contract C is B {
    function bar() public pure override returns (string memory) {
        return "C";
    }
}
~~~

### 多线继承

优先写比较基类的继承关系，

`X基类，Y is X`

`Z is X, Y`

~~~ solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.2;

contract X {
    function foo() public pure virtual returns (string memory) {
        return "X";
    }
    function bar() public pure virtual returns (string memory) {
        return "X";
    }
    function x() public pure returns (string memory) {
        return "X";
    }
}

contract Y is X { //继承写法
    function foo() public pure virtual override returns (string memory) {
        return "Y";
    }
    function bar() public pure virtual override returns (string memory) {
        return "Y";
    }
    function y() public pure returns (string memory) {
        return "Y";
    }
}

contract Z is X, Y {
    function foo() public pure override(X, Y) returns (string memory) {
        return "Z";
    }

    function bar() public pure override(X, Y) returns (string memory) {
        return "Z";
    }
}
~~~

### 调用父级合约的构造函数

~~~ solidity
contract X {
    function foo() public pure virtual returns (string memory) {
        return "X";
    }
    function bar() public pure virtual returns (string memory) {
        return "X";
    }
    function x() public pure returns (string memory) {
        return "X";
    }
}

contract Y is X {
	X.foo(); //1
	super.foo(); //2 多重继承时，父类的func都会被执行
}
~~~

## 可视范围

- private 内部可见
- internal 内部inside和继承child范围可见
- public 内部inside和外部可见
- external  仅仅外部可见

## 不可变常量

关键词：`immutable`

第一次定义后，就不可被改变

一般用于你不知道其初始化，但是又是常量的情况。第一次赋值后变为常量。

## 接受ETH

关键词：`payable`

~~~ solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.2;

contract Payable {
    address payable public owner; //可发送ETH主币

    constructor() {
        owner = payable(msg.sender); //使用时要给带有payable属性
    }

    function deposit() external payable { //函数可接受ETH主币的传入

    }

    function getBalance() external view returns (uint) {
        return address(this).balance;
    }
}
~~~

## 回退函数

当你调用合约中不存在的函数时，或者向合约发送主币的时候，都会调用回退函数

当你期望发送主币的时候触发fallback，那么你需要给fallback方法加上payable属性，

solidity8.0以上，新增**receive**用来只接受主币

**触发逻辑：**当合约收到主币时，判断是否调用了数据，也就是msg.data是否为空，如果没有调用数据，执行**fallback**。如果为空，判断**receive**是否存在，存在调用**receive**，调用**receive**，如果不为空，不存在调用**fallback**

~~~ solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.2;

contract Fallback {
    event Log(string func, address sender, uint value, bytes data);
    fallback() external payable {
        emit Log("fallback", msg.sender, msg.value, msg.data);
    }
    receive() external payable {
        emit Log("receive", msg.sender, msg.value, "");
     }
}
~~~

## 发送ETH

`transfer `消耗2300gas，gas耗费完或者其他异常情况，revert

`send `消耗2300gas，返回bool值

`call` 会把剩余的gas都发送过去

~~~ solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.2;
//transfer  2300gas revert
//send    2300 gas, return bool
//call   all gas, returns bool and data
contract SendETH {
    constructor() payable {

    }
    receive() external payable {

    }

    //三种发送ETH主币的方法
    function Sendbytransfer(address payable _to) external payable {
        _to.transfer(123);
    }
    function Sendbysend(address payable _to) external payable {
        bool sent = _to.send(123);
        require(sent, "send failed");
    }
    function Sendbycall(address payable _to) external payable returns (bytes memory) {
        (bool success, bytes memory data) = _to.call{value: 123}("");
        require(success, "send failed");
        return data;
    }
    
}

//接受ETH的合约，即上面的发送合约发送ETH到该地址

contract Ethreceive {
    event Log(uint amount, uint gas);

    receive() external payable {
        emit Log(msg.value, gasleft());
    }
}
~~~

## 小结：钱包合约

~~~ solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.2;

contract EthWallet {
    address payable public owner;

    constructor() payable {
        owner = payable(msg.sender);
    }

    receive() external payable { }

    function withdraw(uint _amount) external {
        require(msg.sender == owner, "no owner");

        owner.transfer(_amount); //两种效果一样，但是msg.sender在内存，节约gas
        //payable(msg.sender).transfer(_amount);
    }
    function getBalance() external view returns (uint) {
        return address(this).balance;
    }
    function getOwnBalance() external view returns (uint) {
        return msg.sender.balance;
    }
}
~~~



## 调用其他合约

区分两种不同的调用方式，以及可以在调用的时候发送主币

~~~ solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.2;

contract CallTest {
    function setOtherContract_X(address _test, uint _x) external {
        Test(_test).setX(_x);
    }
    
    function getOtherContract_X(Test _test) external view returns (uint) {
        return _test.getX(); //直接将另一个合约名字作为类型传参，然后调用也可
    }

    function setOtherContract_XandETH(address _test, uint _x) external payable {
        Test(_test).setXandReceiveETH{value: msg.value}(_x);
    }

    function getOtherContract_XandETH(Test _test) external view returns (uint, uint) {
        return _test.getXandETH(); 
    }
    

}
contract Test {
    uint public x;
    uint public value;

    function setX(uint _x) external {
        x = _x;
    }
    function getX() external view returns (uint) {
        return x;
    }

    function setXandReceiveETH(uint _x) external payable {
        x = _x;
        value = msg.value;
    }
    function getXandETH() external view returns (uint, uint) {
        return (x, value);
    }

}
~~~

## 接口合约

当我们不知道另一个合约的源码，或者另一个合约源码很长，可以通过接口方法来调用

~~~ solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.2;

//此为外部其他的合约，我们不知道其源码，但是我们知道它实现了Icounter接口
contract Count {
    uint public count;

    function inc() external {
        count += 1;
    }
    function dec() external {
        count -= 1;
    }

}

interface Icounter {
    function count() external view returns (uint); //外部合约Count中有count变量，所以这也可以是一个接口
    function inc() external;
}

contract CallInterface {
    uint public count;
    function example(address _cnt) external {
        Icounter(_cnt).inc();
        count = Icounter(_cnt).count();
    }
}
~~~

## call调用合约

在**call**合约中使用：

`_test.call{value: 111}(abi.encodeWithSignature());`来调用另一个合约的**函数**，依靠`abi.encodeWithSignature`来实现。

其中`value: 111`表示在调用是发送主币，也可以带有gas`value: 111, gas:5000`但是要确保gas够用。

当Test合约的fallback不存在时，使用call调用不存在的函数时，即下文`callnoexist`会失败，原理见：**回退函数**



~~~ solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.2;

contract Test {
    string public message;
    uint public x;

    event Log(string message);

    
    fallback() external payable {
        emit Log("fallback was called");
    }
    

    function foo(string memory _message, uint _x) external payable returns (bool, uint) {
        message = _message;
        x = _x;
        return (true, 11);
    }
}

contract Call {
    bytes public data;
    function callFoo(address _test) external payable {
        (bool success, bytes memory _data) = _test.call{value: 111}(abi.encodeWithSignature(
            "foo(string, uint256", "call foo", 123
        ));
        require(success, "call failed");
        data = _data;
    }

    function callnoexist(address  _test) external {
        (bool success, ) = _test.call(abi.encodeWithSignature("noexist"));
        require(success, "call failed");
    }
}
~~~

## 委托调用

传统调用：

~~~ 
A call B, send 100wei
B call C, send 50wei

在C的视角
msg.sender = B;
msg.value = 50;
可能发生的状态变量也是在C上，ETH主币也会留在C中
~~~

而委托调用：delegatecall

~~~ 
A call B, send 100wei
B delegatecall C

此时，在C的视角
msg.sender = A;
msg.value = 100;
100个ETH主币保存在B，状态也是B中的状态变量发生改变。
~~~

**人话：** 合约X委托调用合约Y的函数，相当于使用了Y的函数作用于自身，在下面的函数中，DelegateCall委托调用了Test中的setVars方法，作用于DelegateCall合约自身。发生改变的也是DelegateCall的状态变量。

**tip：两个合约的状态变量等布局要保持一致，不然会发生奇怪的错误**，本质上像是内存布局的调用，假设DelegateCall的状态变量和Test的前面的状态变量一样，Test后面新增几个新的状态变量，就不会发生错误。

~~~ solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.2;

contract Test {
    uint public num;
    address public sender;
    uint public value;

    function setVars(uint _num) external payable {
        num = _num*2;
        sender = msg.sender;
        value = msg.value;
    }
}

contract DelegateCall {
    uint public num;
    address public sender;
    uint public value;

    function setVars(address _test, uint _num) external payable {
        //_test.delegatecall (
        //    abi.encodeWithSignature("setVars(uint256)", _num)
        //);

        //两种方法效果一样

        (bool success, bytes memory _data) = _test.delegatecall (
            abi.encodeWithSelector(Test.setVars.selector, _num)
        );
        require(success, "delegatecall failed");
    }


}
~~~

## 工厂合约

可以使用一个合约，创建另一个合约，同时也是可以添加payable方法，使用`value`来给合约发送ETH主币。

在ide测试使用的时候，deploy工厂之后，生成了一个account地址，可以通过deploy按钮下面的`At address`来直接生成对应的Account合约，然后就可以查看其内容了。

~~~ solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.2;

contract Account {
    address public bank;
    address public owner;

    constructor(address _owner) payable {
        bank = msg.sender;
        owner = _owner;
    }
}

contract AccountFactory {
    Account[] public accounts;
    function createAccount(address _owner) external payable {
        Account account = new Account{value: 123}(_owner);
        accounts.push(account);
    }
}
~~~

## 库合约

~~~ solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.2;

library Math {
    function max(uint x, uint y) internal pure returns (uint) {
        return x >= y ? x : y;
    }
}

contract Test {
    function testMax(uint x, uint y) external pure returns (uint) {
        return Math.max(x, y);
    }
}

library ArrayLib {
    function find(uint[] storage arr, uint x) internal view returns (uint) {
        for(uint i = 0; i < arr.length; i++) {
            if (arr[i] == x) {
                return i;
            }
        }
        revert("not found");
    }
}

contract TestArray {
    uint[] public arr = [3, 2, 1];

    function testFind() external view returns (uint i) {
        return ArrayLib.find(arr, 2);
    }
}

//更方便的写法
contract TestArray2 {
    using ArrayLib for uint[];
    uint[] public arr = [3, 2, 1];

    function testFind() external view returns (uint i) {
        return arr.find(2);
    }
}
~~~

## hash运算

`keccak256`

**tip：在做hash运算进行编码时，使用abi.encodexxxx编码时，不同的编码方式出来的结果不一致**

![image-20240630194319168](C:\Users\liar\AppData\Roaming\Typora\typora-user-images\image-20240630194319168.png)

可以看到一个会补0，另一个不会补0。

那么使用`encodePacked`对`"AAAA", "BBB"` `"AAA", "ABBB"`编码，出来的结果会是一样的。如果在这个的基础上再次进行hash运算，就会导致hash碰撞。所以编码时比较好的方式是采用`abi.encode`，或者在要编码的字符串之前添加uint等方式来隔开。

~~~ solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.2;

contract HashFunc {
    function hash(string memory text, uint num, address addr) external pure returns (bytes32) {
        return keccak256(abi.encodePacked(text, num, addr));
    }

    function encode(string memory text1, string memory text2) external pure returns (bytes memory) {
        return abi.encode(text1, text2);
    }
    function encodePacked(string memory text1, string memory text2) external pure returns (bytes memory) {
        return abi.encodePacked(text1, text2);
    }
}
~~~

## 验证签名

对一个消息签名分为四步

1. 将消息签名，message to sign
2. hash(message)
3. sign(hash(message), private key) | offchain  把消息和私钥签名，在链下完成
4. 恢复签名 ecrecover(hash(message), signature) == signer ，然后验证signer和你期望的是否一致

~~~ solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.2;

contract Verifysign {
    function verify(address _signer, string memory _message, bytes memory _sign) external pure returns (bool) {
        bytes32 messageHash = getMessageHash(_message);
        bytes32 ethSignedMessageHash = getEthSignedMessageHash(messageHash);
        return recover(ethSignedMessageHash, _sign) == _signer;
    }
    function getMessageHash(string memory _message) public pure returns (bytes32) {
        return keccak256(abi.encodePacked(_message));
    }
    function getEthSignedMessageHash(bytes32 _messageHash) public pure returns (bytes32) {
        return keccak256(abi.encodePacked(
            "\x19Ethereum Signed Message:\n32",
            _messageHash)); //进行两次hash的原因可能是在数学界一次hash已经有被破解的可能性了
    }

    function recover(bytes32 _ethSignedMessageHash, bytes memory _sign) public pure returns (address) {
        (bytes32 r, bytes32 s, uint8 v) = _split(_sign);
        return ecrecover(_ethSignedMessageHash, v, r, s);
    }

    function _split(bytes memory _sign) internal pure returns (bytes32 r, bytes32 s, uint8 v) {
        require(_sign.length == 65, "invalid signature length");

        assembly {
            r := mload(add(_sign, 32))
            s := mload(add(_sign, 64))
            v := byte(0, mload(add(_sign, 96)))
        }
        
    }
    
}
~~~

## 小结：权限控制合约

## 自毁合约

~~~ solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.2;

contract Killer {
    constructor() payable {}

    function kill() external {
        selfdestruct(payable(msg.sender)); //像该地址强制发送ETH，即使你是不接受ETH主币的合约，也会给你
    }

    function testCall() external pure returns (uint) {
        return 123;
    }
}

contract Helper {
    function getBalance() external view returns (uint) {
        return address(this).balance;
    }

    function kill(Killer _kill) external {
        _kill.kill();
    }
}
~~~

## ERCP20标准合约

ERC20标准包含了一组接口IERC20，只要你的代码实现了全部接口，就代表你满足了ERC20标准



