===============
内联汇编
===============
脚本是一种低级语言，可以看作是 `比特币虚拟机 <https://medium.com/@xiaohuiliu/introduction-to-bitcoin-smart-contracts-9c0ea37dc757>`_ 的汇编语言。
通常，开发人员不用直接使用它，而是用像sCrypt这样的高级语言。但有些情况下用脚本语言更合适。
例如，直接写出更优化的脚本，比用sCrypt生成的更高效。
或者脚本是用外部工具（如 `MiniForth <https://powping.com/posts/95e53a7305ad9d333d072575946d0cfd0d6321f40af40f9c66c70955ada94e58>`_ ）生成的，想集成到sCrypt中。

用汇编表示法可以直接把脚本嵌入到sCrypt源代码中。可以用脚本编写sCrypt函数，并像正常的sCrypt函数一样被调用。

调用规范
==================
用脚本写的函数，整个函数体要用 ``asm`` 模式括起来。函数参数位于栈顶，与定义顺序相反。
比如，当进入 ``asm`` 模式时， 函数 ``function foo(int p0, bytes p1, bool p2): int`` 中， ``p2`` 位于栈顶， ``p1`` 是从栈顶开始的第二个元素， ``p0`` 是第三个。
当退出 ``asm`` 模式时，需要弹出栈里的所有参数，并把返回值放到栈顶。
栈里其他元素保持不变。

下面是两个汇编函数。

.. code-block:: solidity

    // 计算 "b" 的字节数
    function len(bytes b): int {
        asm {
            op_size
            op_nip
        }
    }

    function lenFail(bytes b): int {
        // 这个实现是错的，因为在返回时，栈里有两个元素。
        asm {
            op_size
        }
    }

汇编变量
=================
在 ``asm`` 模式下，可以用 ``$`` 前缀来使用变量。其他脚本都是逐字复制到最终脚本输出中的，而汇编变量不是。
变量以它的作用域为前缀，这样可以避免重名，由变量所在的合约和函数进行唯一标识。
例如，在合约 ``contractFoo`` 中的函数 ``func`` 中有个变量 ``$var`` ，在最终脚本输出中会被转换成 ``$contractFoo.func.var``。

下面展示一个例子。

.. code-block:: solidity

    function p2pkh(Sig sig, PubKey pubKey): bool {
        asm {
            op_dup
            op_hash160
            $pkh
            op_equalverify
            op_checksig
        }
    }

注意
=====
内联汇编绕过了sCrypt的许多功能，例如类型检查等。所以使用这个高级功能时要格外小心。
此外，该功能在与外部工具的兼容性上是不区分大小写的。