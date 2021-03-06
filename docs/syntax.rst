====================
Syntax Specification
====================

Formal Specification
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

Most of the syntax is self explanatory. Syntax unique to sCrypt will be covered later.

Line comment starts with ``//`` and block comment comes between ``/*`` and ``*/``.

Types
=====
Basic Types
-----------

* **bool** - a boolean value ``true`` or ``false``.
* **int** - a signed integer of arbitrary length, whose literals are in decimal or hexadecimal format.

    .. code-block:: solidity

        int a1 = 42;
        int a2 = -4242424242424242;
        int a3 = 55066263022277343669578718895168534326250603453777594175500187360389116729240;
        int a4 = 0xFF8C;

* **bytes** - a variable length array of bytes, whose literals are in quoted hexadecimal format prefixed by ``b``.

    .. code-block:: solidity

        bytes b1 = b'ffee1234';
        bytes b2 = b'414136d08c5ed2bf3ba048afe6dcaebafeffffffffffffffffffffffffffffff00';

``bytes`` can be converted to ``int`` using function ``unpack``. Little-endian `sign-magnitude representation <https://www.tutorialspoint.com/sign-magnitude-notation>`_ is used, 
where the most significant bit indicates the sign (``0`` for positive, ``1`` for negative). ``int`` can be converted to ``bytes`` with ``pack``.

    .. code-block:: solidity

        int a1 = unpack(b'36');    // 54 decimal
        int a2 = unpack(b'b6');    // -54
        int a3 = unpack(b'e803');  // 1000
        int a4 = unpack(b'e883');  // -1000
        bytes b = pack(a4);        // b'e883'


Subtypes of ``bytes``
---------------------

These subtypes are more specific versions of ``bytes``, used to further improve type safety.
To cast a supertype ``bytes`` to them, a function of the type name must be explicitly called.

* **PubKey** - a public key type.

    .. code-block:: solidity

        PubKey pubKey = PubKey(b'0200112233445566778899aabbccddeeffffeeddccbbaa99887766554433221100');

* **Sig** - a signature type in `DER <https://docs.moneybutton.com/docs/bsv-signature.html>`_ format, including `signature hash type <https://github.com/libbitcoin/libbitcoin-system/wiki/Sighash-and-TX-Signing>`_, which is ``SIGHASH_ALL | SIGHASH_FORKID`` (``0x41``) in the below example.

    .. code-block:: solidity

        Sig sig = Sig(b'3045022100b71be3f1dc001e0a1ad65ed84e7a5a0bfe48325f2146ca1d677cf15e96e8b80302206d74605e8234eae3d4980fcd7b2fdc1c5b9374f0ce71dea38707fccdbd28cf7e41');

* **Ripemd160** - a RIPEMD-160 hash type.

    .. code-block:: solidity

        Ripemd160 r = Ripemd160(b'0011223344556677889999887766554433221100');

* **Sha1** - a SHA-1 hash type.

    .. code-block:: solidity

        Sha1 s = Sha1(b'0011223344556677889999887766554433221100');

* **Sha256** - a SHA-256 hash type.

    .. code-block:: solidity

        Sha256 s = Sha256(b'00112233445566778899aabbccddeeffffeeddccbbaa99887766554433221100');

Subtypes of ``int``
---------------------

* **PrivKey** - a private key type.

    .. code-block:: solidity

        PrivKey privKey = PrivKey(0x00112233445566778899aabbccddeeffffeeddccbbaa99887766554433221100);

``if`` statement
================
``if`` condition can be of type ``int`` and ``bytes``, besides ``bool``. They are implicitly converted to ``bool`` as in C and Javascript.
An ``int`` expression is evaluated to ``false`` if and only if it is ``0`` (including negative ``0``).
A ``bytes`` expression is evaluated to ``false`` if and only if every of its byte is ``b'00'`` (including empty ``bytes`` ``b''``).

    .. code-block:: solidity

      int cond = 25; // true
      int cond = 0;  // false
      int cond = unpack(b'80') // false since it is negative 0
      int cond = unpack(b'000080') // false since it is negative 0
      if (cond) {} // equivalent to if (cond != 0) {}
      
      bytes cond = b'00'; // false
      bytes cond = b''; // false
      bytes cond = b'80'; // true. Note b'80' is treated as false if converted to int
      bytes cond = b'10' & b'73'; // true since it evaluates to b'10'
      if (cond) {}


exit()
======
``exit(bool status);`` statement terminates contract execution. If ``status`` is ``true``, contract succeeds; otherwise, it fails.

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


Code Separator
==============
Three or more ``*`` in a line inserts an `OP_CODESEPARATOR <https://en.bitcoin.it/wiki/OP_CHECKSIG#How_it_works>`_. It is used to exclude what comes before (and including itself) it from being part of the signature.
Note there is no ``;`` at the end.

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


Operators
=========

.. list-table::
    :header-rows: 1
    :widths: 20 20 20

    * - Precedence 
      - Operator
      - Associativity 

    * - 1
      - ``- !``
      - right-associative

    * - 2
      - ``* / %``
      - left-associative

    * - 3
      - ``+ -``
      - left-associative

    * - 4
      - ``++``
      - left-associative

    * - 5
      - ``<< >>``
      - left-associative

    * - 6
      - ``< <= > >=``
      - left-associative

    * - 7
      - ``== !=``
      - left-associative

    * - 8
      - ``&``
      - left-associative

    * - 9
      - ``^``
      - left-associative

    * - 10
      - ``|``
      - left-associative

    * - 11
      - ``&&``
      - left-associative

    * - 12
      - ``||``
      - left-associative
..
    explain &&,|| evaluates both sides regardless


Scoping
=======
Scoping in sCrypt follows the prevailing scoping rules of C99 and Solidity.
Outer scope variable is shadowed by the inner scope variable of the same name.