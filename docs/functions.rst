=========
函数
=========

用户自定义函数
======================
sCrypt允许开发者定义自己的函数，如下所示：

.. code-block:: solidity

    function sum(int a, int b) returns (int) {
        return a + b;
    }

这些函数只在合约内可见，类似Solidity里的 ``private`` 函数。

公有函数
---------------
如果公有函数可以运行完，则返回 ``true`` ，否则返回 ``false`` 。
公有函数没有 ``return`` ，返回语句是隐式包含其中的。也就是说，

.. code-block:: solidity

    public function sum(int a) {
        require(a == 0);
    }

功能上等同于

.. code-block:: solidity

    public function sum(int a) returns (bool) {
        require(a == 0);
        return true;
    }

静态函数和静态属性
----------------------------
可以通过合约名来直接引用静态函数/方法，而不必创建合约实例，就像Javascript或C++中的静态函数/方法一样。

.. code-block:: solidity

    contract Foo {
        int i;
        static int N = 0;

        static function incByN(int a) returns (int) {
                return a + Foo.N;
        }

        function double(int x) returns (int) {
            return Foo.incByN(x) + this.i;
        }
    }

    contract Bar {
        public function unlock(int y) {
            require(y == Foo.sum(1, 2));
        }
    }

``return``
----------
由于比特币脚本缺乏对 ``return`` 语义的支持，所以函数必须以 ``return`` 语句结尾，并且 ``return`` 语句只能放在函数末尾，不能放在其他位置。
将来可能会放松这个限制。一般来说这不是问题，可以用如下方式避免在其他位置返回：

.. code-block:: solidity

    function abs(int a) returns (int) {
        if (a > 0) {
            return a;
        } else {
            return -a;
        }
    }

可以改写成

.. code-block:: solidity

    function abs(int a) returns (int) {
        int ret = 0;

        if (a > 0) {
            ret = a;
        } else {
            ret = -a;
        }
        return ret;
    }

递归
---------
不允许递归。函数体内不允许调用自己。

.. Warning:: 当前还没有实现对间接递归的检测。如果函数A调用了函数B，函数B又调用了函数A，那么编译进程会被挂起。需要小心以避免引起间接递归。


库函数
=================
sCrypt实现了如下库函数，在全局可见。

数学
----
* ``int abs(int a)``
* ``int min(int a, int b)``
* ``int max(int a, int b)``
* ``bool within(int x, int min, int max)``

哈希
-------
* ``Ripemd160 ripemd160(bytes b)``
* ``Sha1 sha1(bytes b)``
* ``Sha256 sha256(bytes b)``
* ``Ripemd160 hash160(bytes b)``

  ripemd160(sha256(b))

* ``Sha256 hash256(bytes b)``

  sha256(sha256(b))

签名校验
----------------------
* ``bool checkSig(Sig sig, PubKey pk)``
* ``bool checkMultiSig(Sig[] sigs, PubKey[] pks)``

``字节数组`` 操作
-----------------
* 与 ``int`` 互相转换

用函数 ``unpack`` 可以把 ``bytes`` 类型转换为 ``int`` 类型。
采用小端格式的 `符号-值 表示法 <https://www.tutorialspoint.com/sign-magnitude-notation>`_ ，
最高位比特表示符号（ ``0`` 为正， ``1`` 为负）。 
用函数 ``pack`` 可以把 ``int`` 类型转换为 ``bytes`` 类型。

    .. code-block:: solidity

        int a1 = unpack(b'36');    // 54 十进制
        int a2 = unpack(b'b6');    // -54
        int a3 = unpack(b'e803');  // 1000
        int a4 = unpack(b'e883');  // -1000
        bytes b = pack(a4);        // b'e883'

* ``bytes num2bin(int num, int size)``

  把数字 ``num`` 转换为字节数为 ``size`` 的字节数组，包括符号比特。如果字节数组无法容纳被转换的数字，则会转换失败。

* ``reverseBytes20(bytes b)`` ``reverseBytes32(bytes b)``

  反转长度为20或32的字节数组。当对数字进行大小端转换时很有用。

.. code-block:: solidity

        // returns b'6cfeea2d7a1d51249f0624ee98151bfa259d095642e253d8e2dce1e79df33f79'
        reverseBytes32(b'793ff39de7e1dce2d853e24256099d25fa1b1598ee24069f24511d7a2deafe6c')

