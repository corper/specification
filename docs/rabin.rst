===============================
拉宾签名（Rabin Signature）
===============================

`拉宾签名 <https://blog.csdn.net/freedomhero/article/details/107237537>`_ 是比特币中使用的ECDSA数字签名的一种替代形式。

.. code-block:: solidity

    contract RabinSignature {
        public function verifySig(int sig, bytes msg, bytes padding, int n) {
            int h = this.fromLEUnsigned(this.hash(msg + padding));
            require((sig * sig) % n == h % n);
        }

        function hash(bytes x): bytes {
            // 扩展成512比特的哈希值
            bytes hx = sha256(x);
            int idx = len(hx) / 2;
            return sha256(hx[:idx]) + sha256(hx[idx:]);
        }

        function fromLEUnsigned(bytes b): int {
            // 附加上符号字节，保证值不会为负数。即使符号比特已经是正的了，也不会有什么副作用。
            return unpack(b + b'00');
        }
    }