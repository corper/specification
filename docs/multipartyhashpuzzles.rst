=========================================
多方哈希谜题（Multiparty Hash Puzzles）
=========================================

在哈希谜题（hash puzzle）合约中，花费方必须提供一个原像（preimage） ``x`` ，让 ``x`` 的哈希值等于预先定义好的值 ``y`` ，才可以解锁UTXO。
这种合约可以扩展到多方，提供多个原像来满足 ``y1 = H(x1)`` ， ``y2 = H(x2)`` ， ...， ``yN = H(xN)`` [#]_ 。下面展示了一个三方的例子。

.. code-block:: solidity

    contract MultiPartyHashPuzzles {
        Sha256 hash1;   // hash1 = b'136523B9FEA2B7321817B28E254A81A683D319D715CEE2360D051360A272DD4C'
        Sha256 hash2;   // hash2 = b'E222E30CF5C982E5F6251D755B0B16F608ACE631EB3BA9BDAF624FF1651ABF98'
        Sha256 hash3;   // hash3 = b'2A79F5D9F8B3770A59F91E0E9B4C379F7C7A32353AA6450065E43A8616EF5722'
        
        // preimage1: e.g., "bsv" -> b'627376'
        // preimage2: e.g., "sCrypt" -> b'734372797074'
        // preimage3: e.g., "IDE" -> b'494445'
        public function unlock(bytes preimage1, bytes preimage2, bytes preimage3) {
            require(sha256(preimage1) == this.hash1);
            require(sha256(preimage2) == this.hash2);
            require(sha256(preimage3) == this.hash3);
        }
    }


上面的方案在当 ``N`` 比较大时会有问题，因为在锁定脚本里要把 ``N`` 个哈希值都包含进去，这会让交易变大。
有个替代方案，我们可以把所有的 ``y`` 值放到一个里面： ``y = H(H(y1 || y2) || y3)`` [#]_ ，如下所示。

.. code-block:: solidity

    contract MultiPartyHashPuzzlesCompact {
        // 锁定脚本里只需要一个哈希值，节省空间
        Sha256 combinedHash; // combinedHash = b'C9392767AB23CEFF09D207B9223C0C26F01A7F81F8C187A821A4266F8020064D'

        // preimage1: e.g., "bsv" -> b'627376'
        // preimage2: e.g., "sCrypt" -> b'734372797074'
        // preimage3: e.g., "IDE" -> b'494445'
        public function unlock(bytes preimage1, bytes preimage2, bytes preimage3) {
            Sha256 hash1 = sha256(preimage1);
            Sha256 hash2 = sha256(preimage2);
            Sha256 hash3 = sha256(preimage3);
            Sha256 hash12 = sha256(hash1 + hash2);
            Sha256 hash123 = sha256(hash12 + hash3);

            require(hash123 == this.combinedHash);
        }
    }


.. [#] ``H`` 哈希函数。 `这里 <https://www.pelock.com/products/hash-calculator>`_ 有个在线的哈希计算器。
.. [#] ``||`` 表示连接。