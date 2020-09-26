============================================
支付到公钥哈希（Pay to Public Key Hash）
============================================
支付到公钥哈希（Pay-to-PubKey-Hash） （`P2PKH <https://learnmeabitcoin.com/guide/p2pkh>`_） 合约被用来把比特币发送到比特币地址。是比特币网络中最常见的合约。
这种合约可以用两个参数来解锁：公钥和对应私钥创建的签名。

.. code-block:: solidity

    contract P2PKH {
        Ripemd160 pubKeyHash;

        public function unlock(Sig sig, PubKey pubKey) {
            require(hash160(pubKey) == this.pubKeyHash);
            require(checkSig(sig, pubKey));
        }
    }


