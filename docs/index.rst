=============================
sCrypt语言参考
=============================
.. image::  _static/images/logo.png
    :width: 120px
    :alt: sCrypt logo
    :align: center

sCrypt (发音为 “ess crypt”) 是Bitcoin SV的智能合约高级语言。
Bitcoin通过基于栈的类Forth脚本语言来支持智能合约功能。
但是，用原生脚本语言写智能合约既麻烦又容易出错。
随着合约规模和复杂度的上升，很快就变得不可行了。

sCrypt会让开发链上运行的智能合约变得轻松。

* 容易学习。语法上，sCrypt与Solidity类似，这让现有的智能合约开发人员更容易接受。但是，这种相似仅限于语法上，因为编译器会把sCrypt编译成Bitcoin脚本，而不是EVM字节码。
* 静态类型。类型检查可以在编译时就帮助检查出很多错误。


.. warning:: sCrypt仍处于试验阶段，仅用于小规模使用。

.. toctree::
   :maxdepth: 2
   :caption: 介绍

   intro
   ide

.. toctree::
   :maxdepth: 2
   :caption: 语言规范

   syntax
   loop
   functions
   contracts

.. toctree::
   :maxdepth: 1
   :caption: sCrypt示例

   p2pkh
   rpuzzle
   ackermann
   rabin
   multipartyhashpuzzles

.. toctree::
   :maxdepth: 1
   :caption: 高级用法

   asm

联系方式
---------
`scrypt.io <https://scrypt.io>`_

`Slack <https://join.slack.com/t/scryptworkspace/shared_invite/enQtNzQ1OTMyNDk1ODU3LTJmYjE5MGNmNDZhYmYxZWM4ZGY2MTczM2NiNTIxYmFhNTVjNjE5MGYwY2UwNDYxMTQyNGU2NmFkNTY5MmI1MWM>`_

`Telegram <https://t.me/joinchat/GwaRAxKT16JjXyHt5PuhHw>`_

贡献
------------
有关我们需要什么以及如何上手的更多信息，请参见我们 `GitHub <https://github.com/scrypt-sv>`_ 上的 `CONTRIBUTING.md <https://github.com/scrypt-sv/specification/blob/master/docs/CONTRIBUTING.md>`_ 。


