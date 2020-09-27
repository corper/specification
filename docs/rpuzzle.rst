========
R-Puzzle
========

在R-puzzle中，临时私钥 ``k`` 不会被公开，而对应的公钥的横坐标 ``r`` 会被公开。通过 ``r`` 和签名，就可以用 ``checkSig`` 检验是否知道 ``k`` 。

R-Puzzle的一个关键步骤就是从 `DER`_ 编码的签名中提取 ``r`` 。用如下代码，比在 `R-Puzzle`_ 的讨论中展示的方法容易多了。

.. code-block:: solidity

    contract RPuzzle {
        Sig s;  // s = b'3045022100948c67a95f856ae875a48a2d104df9d232189897a811178a715617d4b090a7e90220616f6ced5ab219fe1bfcf9802994b3ce72afbb2db0c4b653a74c9f03fb99323f01'

        function getSigR(Sig sig): bytes {
            bytes lenBytes = sig[3:4];
            int len = unpack(lenBytes);
            bytes r = sig[4:4+len];
            return r;
        }

        // r = b'00948c67a95f856ae875a48a2d104df9d232189897a811178a715617d4b090a7e9'
        public function unlock(bytes r) {
            require(r == this.getSigR(this.s));
        }
    }


.. _DER: https://docs.moneybutton.com/docs/bsv-signature.html
.. _R-Puzzle: https://streamanity.com/video/2AZUShrYn34XrG?ref=632cb174-4e88-4a6c-91a6-14a25d6b4f58&t=1376