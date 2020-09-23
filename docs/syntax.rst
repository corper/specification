====================
语法规范
====================

形式规范
====================
.. math::

    \begin{align*}
    program &::= [importDirective]^*\ [contract]^+\\
    importDirective &::= \mathrm{import}\ "\mathrm{ID}";\\
    contract &::= \mathrm{contract}\ \mathrm{ID}\ \{\ [var]^*\ [constructor]\ [function]^+\ \}\\
    var &::= formal;\\
    formal &::= \mathrm{TYPE}\ \mathrm{ID}\\
    constructor &::= \mathrm{constructor}([formal[,\ formal]^*])\ \{\ [stmt]^*\ \}\\
    function &::= \mathrm{[public|static]}\ \mathrm{function}\ \mathrm{ID}([formal[,\ formal]^*])\ \mathrm{[returns}\ (\mathrm{TYPE]})\ \{\ [stmt]^*\ \mathrm{[return}\ expr;]\ \}\\
    stmt &::= \mathrm{TYPE}\ \mathrm{ID} = expr;\\
            &\ \ \ |\ \ \mathrm{ID}\ \mathrm{ID} = \mathrm{new}\ \mathrm{ID}(expr^*);\\
            &\ \ \ |\ \ \mathrm{ID} = expr;\\
            &\ \ \ |\ \ \mathrm{require}(expr);\\
            &\ \ \ |\ \ \mathrm{exit}(expr);\\
            &\ \ \ |\ \ \mathrm{if}\ (expr)\ stmt\ [\mathrm{else}\ stmt]\\
            &\ \ \ |\ \ \mathrm{loop}\ (intConst)\ stmt\\
            &\ \ \ |\ \ \{\ [stmt]^*\ \}\\
            &\ \ \ |\ \ \mathrm{CODESEPARATOR}\\
    expr &::= \mathsf{UnaryOp}\ expr\\
            &\ \ \ |\ \ expr\ \mathsf{BinaryOp}\ expr\\
            &\ \ \ |\ \ \mathrm{ID}(expr[,\ expr]^*)\\
            &\ \ \ |\ \ \mathrm{ID}.\mathrm{ID}\\
            &\ \ \ |\ \ \mathrm{ID}.\mathrm{ID}(expr[,\ expr]^*)\\
            &\ \ \ |\ \ \mathrm{ID}\mathbf{[}expr:expr\mathbf{]}\\
            &\ \ \ |\ \ (expr)\\
            &\ \ \ |\ \ \mathrm{ID}\\
            &\ \ \ |\ \ boolConst \\
            &\ \ \ |\ \ intConst \\
            &\ \ \ |\ \ bytesConst \\
    \end{align*}

大部分语法含义都是显而易见的。sCrypt特有的语法会在后面介绍。

行注释用 ``//`` 开头，在 ``/*`` 和 ``*/`` 之间的是块注释。

类型
=====
基本类型
-----------

* **bool** - 布尔类型，值为 ``true`` 或 ``false`` 。
* **int** - 有符号的任意长度整数类型，有十进制和十六进制两种格式。

    .. code-block:: solidity

        int a1 = 42;
        int a2 = -4242424242424242;
        int a3 = 55066263022277343669578718895168534326250603453777594175500187360389116729240;
        int a4 = 0xFF8C;

* **bytes** - 可变长度的字节数组，以 ``b`` 开头，后面跟单引号引起来的十六进制数。

    .. code-block:: solidity

        bytes b1 = b'ffee1234';
        bytes b2 = b'414136d08c5ed2bf3ba048afe6dcaebafeffffffffffffffffffffffffffffff00';

用函数 ``unpack`` 可以把 ``bytes`` 类型转换为 ``int`` 类型。采用小端 `符号-值 表示法 <https://www.tutorialspoint.com/sign-magnitude-notation>`_ ，
最高位比特表示符号（ ``0`` 为正， ``1`` 为负）。 用函数 ``pack`` 可以把 ``int`` 类型转换为 ``bytes`` 类型。

    .. code-block:: solidity

        int a1 = unpack(b'36');    // 54 十进制
        int a2 = unpack(b'b6');    // -54
        int a3 = unpack(b'e803');  // 1000
        int a4 = unpack(b'e883');  // -1000
        bytes b = pack(a4);        // b'e883'

* **auto** 关键字 - ``auto`` 关键字表示变量的类型由变量的初始值来决定。

    .. code-block:: solidity

        auto a1 = b'36';      // bytes a1 = b'36';
        auto a2 = 1 + 5 * 3;  // int a2 = 1 + 5 * 3;

``bytes`` 的子类型
---------------------

这些子类型是 ``bytes`` 类型的特定版本，用于进一步提高类型安全性。
要把 ``bytes`` 类型强制转换成某个子类型，必须显式调用与该子类型同名的函数。

* **PubKey** - 公钥类型。

    .. code-block:: solidity

        PubKey pubKey = PubKey(b'0200112233445566778899aabbccddeeffffeeddccbbaa99887766554433221100');

* **Sig** - `DER <https://docs.moneybutton.com/docs/bsv-signature.html>`_ 格式的签名类型。 包含 `签名哈希类型 <https://github.com/libbitcoin/libbitcoin-system/wiki/Sighash-and-TX-Signing>`_ ，如下例子中的签名哈希类型是 ``SIGHASH_ALL | SIGHASH_FORKID`` (``0x41``)。

    .. code-block:: solidity

        Sig sig = Sig(b'3045022100b71be3f1dc001e0a1ad65ed84e7a5a0bfe48325f2146ca1d677cf15e96e8b80302206d74605e8234eae3d4980fcd7b2fdc1c5b9374f0ce71dea38707fccdbd28cf7e41');

* **Ripemd160** - RIPEMD-160哈希类型.

    .. code-block:: solidity

        Ripemd160 r = Ripemd160(b'0011223344556677889999887766554433221100');

* **Sha1** - SHA-1哈希类型。

    .. code-block:: solidity

        Sha1 s = Sha1(b'0011223344556677889999887766554433221100');

* **Sha256** - SHA-256哈希类型。

    .. code-block:: solidity

        Sha256 s = Sha256(b'00112233445566778899aabbccddeeffffeeddccbbaa99887766554433221100');

* **SigHashType** - 签名哈希类型

    .. code-block:: solidity

        SigHashType s = SigHashType(b'01');
        SigHashType s = SigHash.ALL | SigHash.ANYONECANPAY;

* **OpCodeType** - 操作码类型

    .. code-block:: solidity

        OpCodeType s = OpCode.OP_DUP + OpCode.OP_ADD;

``int`` 的子类型
---------------------

* **PrivKey** - 私钥类型

    .. code-block:: solidity

        PrivKey privKey = PrivKey(0x00112233445566778899aabbccddeeffffeeddccbbaa99887766554433221100);

``if`` 语句
================
除了 ``bool`` 类型， ``if`` 条件还可以是 ``int`` 和 ``bytes`` 。这些类型会被隐式转换为 ``bool`` 类型，与 C 和 Javascript 语言中的处理方式一样。
当且仅当 ``int`` 为 ``0`` （包括负 ``0`` ）时，为 ``false`` 。
当且仅当 ``bytes`` 的每个字节都是 ``b'00'`` （包括空 ``bytes`` ``b''`` ）时，为 ``false`` 。

    .. code-block:: solidity

      int cond = 25; // true
      int cond = 0;  // false
      int cond = unpack(b'80') // false 因为值为负0
      int cond = unpack(b'000080') // false 因为值为负0
      if (cond) {} // 等同于 if (cond != 0) {}
      
      bytes cond = b'00'; // false
      bytes cond = b''; // false
      bytes cond = b'80'; // true. 注意如果把 b'80' 转换成 int 的话， 会是 false
      bytes cond = b'10' & b'73'; // true 因为表达式的运算结果为 b'10'
      if (cond) {}


exit()
======
``exit(bool status);`` 语句用于结束合约的执行。 如果 ``status`` 参数是 ``true``， 合约执行成功； 否则执行失败。

    .. code-block:: solidity

      contract TestPositiveEqual {
          int x;

          constructor(int x) {
              this.x = x;
          }

          public function equal(int y) {
              if (y <= 0) {
                exit(true);
              }
              require(y == this.x);
          }
      }


代码分隔符
==============
一行中三个或更多 ``*`` 表示插入一个 `OP_CODESEPARATOR <https://en.bitcoin.it/wiki/OP_CHECKSIG#How_it_works>`_ 。 该操作符之前的内容（包括操作符本身）不参与签名计算。
注意，该语句行末没有 ``;`` 。

    .. code-block:: solidity

      contract TestSeparator {
          public function equal(int y) {
              int a = 0;
              // separator 1
              ***
              int b = 2;
              // separator 2
              *****
              require(y > 0);
          }
      }


操作符
=========

.. list-table::
    :header-rows: 1
    :widths: 20 20 20

    * - 优先级 
      - 操作符
      - 关联性 

    * - 1
      - ``- ! ~``
      - 右关联

    * - 2
      - ``* / %``
      - 左关联

    * - 3
      - ``+ -``
      - 左关联

    * - 4
      - ``<< >>``
      - 左关联

    * - 5
      - ``< <= > >=``
      - 左关联

    * - 6
      - ``== !=``
      - 左关联

    * - 7
      - ``&``
      - 左关联

    * - 8
      - ``^``
      - 左关联

    * - 9
      - ``|``
      - 左关联

    * - 10
      - ``&&``
      - 左关联

    * - 11
      - ``||``
      - 左关联
..
    explain &&,|| evaluates both sides regardless


作用域
=======
sCrypt的作用域遵循C99和Solidity的现行作用域规则。
外部作用域的变量会被内部作用域的同名变量覆盖。