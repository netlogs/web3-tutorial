# Solidity简明教程

本教程适用于有语言基础，速查或者快速入门。

## 版权许可标识

~~~ solidity
// SPDX-License-Identifier: MIT
~~~

一般放在文件开头

更多内容请查看[许可](https://learnblockchain.cn/docs/solidity/layout-of-source-files.html)

## 版本

~~~ solidity
pragma solidity ^0.5.2;
~~~

含义是既不允许低于 0.5.2 版本的编译器编译， 也不允许高于（包含） `0.6.0` 版本的编译器编译（第二个条件因使用 `^` 被添加）。

~~~ solidity
pragma solidity >=0.6.12 <0.9.0;
~~~

## 变量

状态变量

局部变量

全局变量

### 默认值

~~~ Solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.7;

contract DefaultValues {
    bool public b; //false
    uint public u; //0
    int public i; //0
    address public a; //0x0000000(一共40个0,20位16进制数字)
    bytes32 public b32; //0x00000 一共64个0,32位

}
~~~

### 常量

~~~ solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.7;

contract Constants {
    address public constant MY_ADDRESS = 0x2E35C375782713b7feedEd99B18C2F2728B2566D;
    uint public constant MY_UINT = 123;
}


contract Var {
    address public myaddr = 0x2E35C375782713b7feedEd99B18C2F2728B2566D;
}
~~~

**constant**定义常量

常量和变量消耗的gas不一样，deploy之后点击`MY_ADDRESS`可以查看gas费用是373

~~~
execution cost	373 gas (Cost only applies when called by a contract)
~~~

下面这是Var合约内读取`myaddr`消耗的gas

~~~
execution cost	2483 gas (Cost only applies when called by a contract)
~~~

## 结构控制

### 判断

~~~ solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.26;

contract IfElse {
    function example(uint x) external pure returns (uint) {
        if (x < 10) {
            return 1;
        } else if ( x < 20) {
            return 2;
        } else {
            return 3;
        }
    }
    function ternary(uint x) external pure returns (uint) {
        return x < 10 ? 1 : 2;
    }
}

~~~

### 循环

~~~ solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.26;

contract ForAndWhile {
    function loops() external pure {
        for (uint i = 0; i < 10; ++i) {
            //code
           if (i == 2)  continue;
            if (i == 4) {
                break;
            }
        }
        int b = 3;
        while(b > 0) {
            --b;
        }
    }
}

~~~

和C语言一样，同时要注意控制循环次数，太多的循环会导致gas费用的高昂

## Error

~~~ solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.26;

// require, revert, assert
// - gas refund, state updates are reverted
// 8.0 version 更新了，custom error, save gas
contract Error {
    function testRequire(uint _i) public pure {
        require(_i <= 10, "i > 10");
        //code
    }
    function testRevert(uint _i) public pure {
        if(_i > 10) {
            revert("i > 10");
        }
        //code
    }
    uint public num = 123;
    function testAssert() public view {
        assert(num == 123);
    }

    function foo(uint _i) public {
        num += 1;
        require(_i < 10); //如果i不符合条件，前面+1的num会被回滚，状态不会被更改，gas退还
    }

    
    function testCustomError1(uint _i) public pure {
        require(_i <= 10, "very long error message xxxxxxxxxxxxxxx");//使用require时，如果error信息很长，会消耗很多gas
    }

    error MyError();

    function testCustomError2(uint _i) public pure {
        if (_i > 10) revert MyError(); //通过这种方式触发自定义报错
    }
    //同时可以：
    error MyError2(address caller, uint i);
    function testCustomError3(uint _i) public view {
        if (_i > 10) revert MyError2(msg.sender, _i); //通过这种方式触发自定义报错
    }
}

~~~



## 函数



~~~ solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.7;

contract Counter {
    uint public cnt;
    function inc() external {
        cnt += 1;
    }

    function dec() external {
        cnt -= 1;
    }
}
~~~

**external**：外部可视，意思是在合约内部的其他函数不可以调用，只能被外部读取

**public**：公开可视，内部外部都可以调用

~~~ solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.26;

contract FUnctionOutputs {
    function returnMany() public pure returns (uint, bool) {
        return (1, true);
    }
    function returnManyWithname() public pure returns (uint x, bool b) {
        return (1, true);
    }
    //隐式赋值返回
    function assigned() public pure returns (uint x, bool b) {
        x = 1; 
        b = true;
    }

    //合约内调用
    function otherfunc() public pure {
        (uint a, bool b) = returnMany();
        (, bool _b) = returnManyWithname();
    }
}
~~~



### 函数修改器

~~~ solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.26;

contract FunctionModifier {
    bool public paused;
    uint public cnt;

    function setPasuse(bool _paused) external {
        paused = _paused;
    }

    function inc() external {
        require(!paused, "paused");
        cnt += 1;
    }
    function dec() external {
        require(!paused, "paused");
        cnt -= 1;
    }
    //当我们期望合约暂停时cnt不被修改，我们可以在修改的func中添加require，函数修改器支持更强大的操作，
    //有点类似于C语言中，把暂停的判断提炼到一个判断函数内，然后在修改的函数都提前调用一下判断函数
    //写法：
    modifier whenNotPaused() {
        require(!paused, "paused");
        _; //此处代表，使用此修改器的函数的其他的代码在哪里运行
    }

    function new_incOr_dec() external whenNotPaused {
        //cnt += 1; or cnt -= 1;
    }


    //带参数的函数修改器

    modifier cap(uint _x) {
        require(_x < 100, "x >= 100");
        _;
    }
    function incBy(uint _x) external whenNotPaused cap(_x) {
        cnt += _x;
    }

    //sanddwich写法
    //下面相当于把foo中对cnt的一部分操作提到了修改器中
    //cnt 先执行+=10，再返回foo执行+=1，再返回sandwich执行*=2
    modifier sandwich() {
        cnt += 10;
        _;
        cnt *= 2;
    }

    function foo() external sandwich {
        cnt += 1;
    }
}
~~~

### 构造函数

合约部署时被调用一次，之后再也不能被调用

~~~ solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.26;

contract Constructor {
    address public owner;
    uint public x;

    constructor(uint _x) {
        owner = msg.sender;
        x = _x;
    }
    //通过传参赋予x值，ctrl+s编译好之后，部署时，Deploy旁边会多一个输入框，代表构造函数的参数列表
}
~~~

## 数组

~~~ solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.26;

contract Array {
    uint[] public nums = [1, 2, 3];
    uint[10] public numsFixed= [4, 5, 6];

    function examples() external {
        nums.push(4); //[1, 2, 3, 4]; 适用于不定长数组
        nums[1] = 9; //下标，和C语言一样[1, 9, 3, 4]
        delete nums[1]; //[1, 0, 3, 4] ！不能减少数组长度
        nums.pop(); //[1, 0, 3] 类似于stl中的stack，减少了数组长度
        uint len = nums.length; //长度

        //create array in memory
        uint[] memory arr = new uint[](5);
        //在内存中的不允许pop、push
    }

    function returnArray() external view  returns (uint[] memory) {
        return nums;


    }
}
~~~

## 映射

~~~ solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.26;

contract Array {
    mapping(address => uint) public balances;
    mapping(address => mapping(address => bool)) public isFriend;

    function examples() external {
        balances[msg.sender] = 123;
        uint bal = balances[msg.sender];
        uint bal2 = balances[address(1)]; // 0
        balances[msg.sender] += 456;

        delete balances[msg.sender]; //0

        isFriend[msg.sender][address(this)] = true;

    }
}
~~~

## 结构体

~~~ solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.26;

//结合数组和mapping
contract Structs {
    struct Car {
        string model;
        uint year;
    }

    Car public car;
    Car[] public cars;

    mapping(address => Car[]) public carsByOwner;

    function examples() external {
        Car memory toyota = Car("Toyota", 1999);
        Car memory lambo = Car({year: 111, model: "Lamborghini"});
        Car memory zero;
        zero.model = "aaaa";
        zero.year = 1993;

        cars.push(zero);
        cars.push(toyota);
        cars.push(lambo);

        cars.push(Car("Tesla", 1991));

        Car memory temp = cars[0];
        Car storage real = cars[0];
        real.year = 0; //storage带有指针效果
        delete real.year; //恢复cars[0]的成员year的默认值
        delete cars[1]; //默认值全部恢复
    }
}
~~~

## 枚举

~~~ solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.26;

//结合数组和mapping
contract Enums {
    enum Status {
        None,
        Pending,
        Shipped,
        Completed,
        Rejected,
        Canceled
    }

    Status public status;

    struct Order {
        address buyer;
        Status status;
    }

    Order[] public orders;

    function get() external view returns (Status) {
        return status;
    }

    function set(Status _status) external {
        status = _status;
    }

    function ship() external {
        status = Status.Shipped;
    }

    function reset() external {
        delete status; //枚举类型的默认值是第一个字段，本例为"None"
    }
}
~~~



## 总结

管理员合约

~~~ solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.26;

contract Owner {
    address public owner;

    constructor() {
        owner = msg.sender;
    }

    modifier onlyOwner() {
        require(owner == msg.sender, "no owner");
        _;
    }

    function setOwner(address _newowner) external onlyOwner {
        require(_newowner != address(0), "no 0 address");
        owner = _newowner;
    }

    function ownerUse() external onlyOwner {
        //code
    }

    function anyoneUse() external {
        //code
    }
}
~~~

## 在线IDE

https://remix.ethereum.org/

