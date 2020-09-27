=======================
一个简单的智能合约示例
=======================

sCrypt中的合约（contract）在概念上类似于面向对象编程中的类（class）。
每个contract都为特定类型的合约（如：P2PKH或多重签名）提供了模板，可被实例化为可运行的合约对象。

.. code-block:: solidity
    
    contract Test {
        int x;

        constructor(int x) {
            this.x = x;
        }

        public function equal(int y) {
            require(y == this.x);
        }
    }

构造函数（constructor）
=======================
每个合约最多只能有一个构造函数。用于初始化合约的成员变量。
例如，初始化P2PKH合约的公钥哈希，或者哈希谜题（hash puzzle）合约的secret hash。

默认构造函数
-------------------
当没有构造函数时，编译器会生成一个默认的构造函数，按照声明顺序初始化每一个成员变量。
例如：

.. code-block:: solidity
    
    contract Test {
        int x1;
        bytes x2;
        bool x3;

        public function equal(int y) {}
    }

在功能上等同于

.. code-block:: solidity
    
    contract Test {
        int x1;
        bytes x2;
        bool x3;

        constructor(int x1, bytes x2, bool x3) {
            this.x1 = x1;
            this.x2 = x2;
            this.x3 = x3;
        }

        public function equal(int y) {}
    }

require()
=========
``require()`` 函数指定合约的限制条款/条件。它的参数是一个布尔条件表达式。
如果条件表达式的值为假，合约将终止执行并失败。否则，将继续执行。

公有函数
===========================
每个合约至少要有一个公有函数。用 ``public`` 关键字修饰，并且不返回任何值。函数体对应锁定脚本（通常被称为 ``scriptPubKey``），函数参数为解锁脚本（也被称为 ``scriptSig``）。
公有函数在合约外部是可见的，是合约的入口（就像 ``C`` 和 ``Java`` 的 ``main`` 函数）。

公有函数的最后一个语句必须是 ``require()`` 。 ``require()`` 也可以出现在公有函数的其他地方。只有在被调用的公有函数执行完毕，并且所有 ``require()`` 中的条件全部被满足时，合约才算执行成功。
在上面的例子中，只有 ``scriptSig`` （即 ``y`` ）等于 ``this.x`` 时合约才能执行成功。

多个公有函数
-------------------------
一个合约可以有多个公有函数，表示有多种方式可以满足合约。但一次只能调用其中一个。在这种情况下， ``scriptSig`` 的最后一个操作符必须是被调用的公有函数的索引，公有函数索引按照定义顺序从 ``0`` 开始。
举例如下，要调用公有函数 ``larger`` ， ``scriptSig`` 要用 ``y 2`` ，其中 ``2`` 是被调用的公有函数的索引。

.. code-block:: solidity

    contract Test {
        int x;

        public function equal(int y) {
            require(y == this.x);
        }

        public function smaller(int y) {
            require(y < this.x);
        }

        public function larger(int y) {
            require(y > this.x);
        }
    }
