==================
标准合约
==================

多个合约
==================
一个文件中可以定义多个合约。在这种情况下，最后一个合约是主合约，这个合约会被编译。
其他合约都被主合约依赖。

在下面这个例子中，标准的P2PKH合约被改写为两个其他合约：一个用来检查公钥和公钥哈希是否匹配的哈希谜题（hash puzzle）合约，还有一个检查签名和公钥是否匹配的Pay-to-PubKey（P2PK）合约。

.. code-block:: solidity

    contract HashPuzzle {
        Ripemd160 hash;

        public function spend(bytes preimage) {
            require(hash160(preimage) == this.hash);
        }
    }

    contract Pay2PubKey {
        PubKey pubKey;

        public function spend(Sig sig) {
            require(checkSig(sig, this.pubKey));
        }
    }

    contract Pay2PubKeyHash {
        Ripemd160 pubKeyHash;

        public function spend(Sig sig, PubKey pubKey) {
            HashPuzzle hp = new HashPuzzle(this.pubKeyHash);
            require(hp.spend(pubKey));

            Pay2PubKey p2pk = new Pay2PubKey(pubKey);
            require(p2pk.spend(sig));
        }
    }


导入（import）
==============
或者，可以将上述合约分到三个文件中。 ``Pay2PubKeyHash`` 合约 ``import`` 其他两个合约作为依赖。
这就可以重用其他人写的合约，成为构建合约库的基础。

可以通过 ``new`` 来实例化一个合约。 ``require`` 函数的参数是条件表达式， 在条件表达式里可以调用合约的 ``public`` 函数。

.. code-block:: solidity

    import "./hashPuzzle.scrypt";
    import "./p2pk.scrypt";

    contract Pay2PubKeyHash {
        Ripemd160 pubKeyHash;

        public function spend(Sig sig, PubKey pubKey) {
            HashPuzzle hp = new HashPuzzle(this.pubKeyHash);
            require(hp.spend(pubKey));

            Pay2PubKey p2pk = new Pay2PubKey(pubKey);
            require(p2pk.spend(sig));
        }
    }


标准合约
==================
sCrypt自带标准库，里面定义了许多常用的合约。标准库是默认就导入的，不需要写 ``import`` 语句。

如下例子展示了对标准合约 ``P2PKH`` 的使用。

.. code-block:: solidity

    contract P2PKHStdDemo {
        Ripemd160 pubKeyHash;

        public function unlock(Sig sig, PubKey pubKey) {
            P2PKH p2pkh = new P2PKH(this.pubKeyHash);
            require(p2pkh.spend(sig, pubKey));
        }
    }


``OP_PUSH_TX`` 合约
-----------------------
对比特币脚本的一个严重误解是，脚本只能访问锁定脚本以及对应的解锁脚本中提供的数据。
因此，脚本的范围和能力被大大低估了。

sCrypt提供了一个强大的合约叫做 ``Tx``，它允许合约访问合约所在的 **整个交易** ，包括锁定脚本和解锁脚本。
我们把这种方法当成一个伪操作码 ``OP_PUSH_TX`` ，它可以把当前交易压到栈里，这样就可以在运行时访问了。
更准确地说，可以访问的是在签名校验时用到的原像（preimage）， 在 `BIP143`_ 中有原像的详细定义。
原像的数据格式如下：

    1. 交易的版本号（nVersion of the transaction）（4字节小端）
    2. 输入的输出点哈希（hashPrevouts） （32字节哈希值）
    3. 序列号哈希（hashSequence） （32字节哈希值）
    4. 此输入的输出点（outpoint） （32字节哈希值 + 4字节小端） 
    5. 此输入的锁定脚本（scriptCode of the input）（在CTxOuts中序列化为脚本）
    6. 此输入对应的输出中包含的聪数（value of the output spent by this input） (8字节小端)
    7. 此输入的序列号（nSequence of the input） （4字节小端）
    8. 输出的哈希（hashOutputs）（32字节哈希值）
    9. 交易的nLocktime（nLocktime of the transaction）（4字节小端）
    10. 交易的签名哈希类型（sighash type of the signature）（4字节小端）

举个例子，合约 ``CheckLockTimeVerify`` 确保合约中的币是时间锁定的，在 ``matureTime`` 这个时间点之前不能被花掉。其功能类似 `OP_CLTV`_。

.. code-block:: solidity

    contract CheckLockTimeVerify {
        int matureTime;

        public function spend(bytes sighashPreimage) {
            // this ensures the preimage is for the current tx
            require(Tx.checkPreimage(sighashPreimage));
            
            // parse nLocktime
            int len = length(sighashPreimage);
            int nLocktime = this.fromLEUnsigned(sighashPreimage[len - 8 : len - 4]);

            require(nLocktime >= this.matureTime);
        }
        
        function fromLEUnsigned(bytes b) returns (int) {
            // append positive sign byte. This does not hurt even when sign bit is already positive
            return unpack(b + b'00');
        }
    }

更多细节参见 `这篇文章 <https://blog.csdn.net/freedomhero/article/details/107306604>`_ 。
为了可以定制ECDSA签名，比如选择临时密钥，可以使用一个更加通用的版本 `TxAdvanced <https://blog.csdn.net/freedomhero/article/details/107333738>`_。

完整列表
---------

.. list-table::
    :header-rows: 1
    :widths: 20 20 20

    * - 合约
      - 构造参数
      - 公有函数
    
    * - P2PKH
      - Ripemd160 pubKeyHash
      - spend(Sig sig, PubKey pubKey)

    * - P2PK
      - PubKey pubKey
      - spend(Sig sig)
    
    * - HashPuzzleX [#]_
      - Y [#]_ hash
      - spend(bytes preimage)

    * - Tx
      - None
      - checkPreimage(bytes sighashPreimage)

.. [#] ``X`` 是哈希函数，可以是Ripemd160/Sha1/Sha256/Hash160
.. [#] ``Y`` 是哈希函数的返回值类型，可以是Ripemd160/Sha1/Sha256/Ripemd160

.. _BIP143: https://github.com/bitcoin-sv/bitcoin-sv/blob/master/doc/abc/replay-protected-sighash.md
.. _OP_CLTV: https://en.bitcoin.it/wiki/Timelock#CheckLockTimeVerify