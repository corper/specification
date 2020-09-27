==================================
阿克曼函数（Ackermann Function）
==================================

阿克曼函数是递归函数的经典示例，特别值得注意的是它并不是一个原始递归函数。
它的值增长得非常快，调用树也增长得非常快。阿克曼函数通常定义如下：

.. math::

    A(m, n) =
    \begin{cases}
    n+1 & \mbox{if } m = 0 \\
    A(m-1, 1) & \mbox{if } m > 0 \mbox{ and } n = 0 \\
    A(m-1, A(m, n-1)) & \mbox{if } m > 0 \mbox{ and } n > 0.
    \end{cases}


nCrypt设计了一种使用 `原生脚本`_ 计算阿克曼函数值的方法，该方法是非常复杂的。下面我们给出了一个简单得多的版本。

.. code-block:: solidity

    contract Ackermann {
        int a;  // a = 2
        int b;  // b = 1

        function ackermann(int m, int n): int {
            bytes stk = num2bin(m, 1);

            // 在链下运行这个函数，来得到循环次数，并设置到下面的循环中
            // 比如， (2, 1) 需要14次循环，(3, 5)需要42438次循环
            loop (14) {
                if (length(stk) > 0) {
                    bytes top = stk[0:1];
                    m = unpack(top);

                    // pop
                    stk = stk[1:length(stk)];

                    if (m == 0) {
                        n = n + m + 1;
                    } else if (n == 0) {
                        n = n + 1;
                        m = m - 1;
                        // push
                        stk = num2bin(m, 1) + stk;
                    } else {
                        stk = num2bin(m - 1, 1) + stk;
                        stk = num2bin(m, 1) + stk;
                        n = n - 1;
                    }
                }
            }

            return n;
        }

        // y = 5
        public function unlock(int y) {
            require(y == this.ackermann(this.a, this.b));
        }
    }


.. _原生脚本: https://onedrive.live.com/?authkey=%21AMkX_N43zpZknj4&cid=68E98EDCE5760610&id=68E98EDCE5760610%2181946&parId=68E98EDCE5760610%2116494&o=OneUp
