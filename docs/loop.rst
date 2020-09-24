====
循环
====
.. code-block:: solidity

    loop (maxLoopCount)
        loopBody


因为安全原因，比特币脚本没有提供循环结构。
sCrypt通过重复循环体 ``maxLoopCount`` 次来实现循环。
例如，下面的循环

.. code-block:: solidity

    loop (10) {
        x = x * 2;
    }

相当于展开成如下形式

.. code-block:: solidity

    x = x * 2;
    x = x * 2;
    x = x * 2;
    x = x * 2;
    x = x * 2;
    x = x * 2;
    x = x * 2;
    x = x * 2;
    x = x * 2;
    x = x * 2;


因为 `循环展开 <https://en.wikipedia.org/wiki/Loop_unrolling>`_ 是在编译时进行的，编译器需要知道 ``maxLoopCount`` 的值，所以 ``maxLoopCount`` 必须是一个常数。


如果 ``maxLoopCount`` 设置得太小，合约可能无法正常运行。如果 ``maxLoopCount`` 设置得太大，那么生成的脚本大小会不必要地膨胀，增加执行成本。
有一些方法可以把 ``maxLoopCount`` 的值设置得更合理。
其中一个方法是在链下模拟运行合约以找到合理的循环次数。
另一种方法是利用循环本身的特性。比如，如果一个循环需要遍历 ``sha256`` 哈希值的每个比特，那么 ``maxLoopCount`` 就应该设置为 ``256`` 。

访问循环变量
=================
.. code-block:: solidity

    int i = 0;
    loop (3) {
        // i 为循环变量
        i = i + 1;
        x = x * 2;
    }

有条件的循环
================
.. code-block:: solidity

    loop (3) {
        // 这里放置循环条件
        if (x < 8) {
            x = x * 2;
        }
    }

break
=====
.. code-block:: solidity

    bool done = false;
    loop (3) {
        if (!done) {
            x = x * 2;
            if (x >= 8) {
                done = true;
            }
        }
    }