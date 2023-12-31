סולידיטי
========

סולידיטי היא שפת תכנות עילית מובנת עצמים, המשמשת להטמעה של ״חוזים חכמים״.
חוזים חכמים הן תוכניות שנועדו לנהל התנהגות של חשבונות בתוך מצב האתריום (Ethereum State).

סולידיטי היא
`שפת סוגריים-מסולסלים <https://en.wikipedia.org/wiki/List_of_programming_languages_by_type#Curly-bracket_languages>`_
התוכננה לטרגט את המכונה הוירטואלית של אתריום. השפה מושפעת על ידי C++, פייתון, וג׳אבאסקריפט.

ניתן למצוא מידע נוסף על השפות מהן הושפעה סולידיטי ב :doc:`language influences <language-influences>`

סולידיטי נכתבת בצורה סטטית, תומכת ירושה, ספריות, וסוגי משתמש מסובכים מוגדרים מראש, בין התכונות האחרות.

עם סולידיטי, ניתן ליצור חוזים לשימושים כגון הצבעה, גיוס תרומות, מכרזים עיוורים, וארנקים מרובי חתימה.

כאשר מתקינים חוזה, יש להשתמש בגרסה הציבורית האחרונה של סולידיטי.
(מתקינים = deploying)
מלבד מקרים יוצאי דופן, רק הגרסה האחרונה מקבלת.
`תיקוני אבטחה <https://github.com/ethereum/solidity/security/policy#supported-versions>`_.
.. `security fixes <https://github.com/ethereum/solidity/security/policy#supported-versions>`_.
בנוסף, שינויים ״שוברים״ (שוברי גרסא) ותכונות חדשות מוצגות באופן קבוע.
כדי לייצג את הקצב המהיר של השינויים, אנחנו כרגע משתמשים במספר גרסה 0.y.z
`כדי לציין את הקצב המהיר של השינויים <https://semver.org/#spec-item-4>`_.
.. We currently use a 0.y.z version number `to indicate this fast pace of change <https://semver.org/#spec-item-4>`_.

.. warning::
   סולידיטי לאחרונה שחררה את גרסא 0.8x, שהכילה שינויים רבים ששוברו תאימות עם גרסאות קודמות.
  ודאו שקראתם את :doc:`רשימת השינויים המלאה <080-breaking-changes>`.


Ideas for improving Solidity or this documentation are always welcome,
read our :doc:`contributors guide <contributing>` for more details.

.. Hint::

  You can download this documentation as PDF, HTML or Epub
  by clicking on the versions flyout menu in the bottom-left corner and selecting the preferred download format.


Getting Started
---------------

**1. Understand the Smart Contract Basics**

If you are new to the concept of smart contracts, we recommend you to get started by digging into the "Introduction to Smart Contracts" section, which covers the following:

* :ref:`A simple example smart contract <simple-smart-contract>` written in Solidity.
* :ref:`Blockchain Basics <blockchain-basics>`.
* :ref:`The Ethereum Virtual Machine <the-ethereum-virtual-machine>`.

**2. Get to Know Solidity**

Once you are accustomed to the basics, we recommend you read the :doc:`"Solidity by Example" <solidity-by-example>`
and “Language Description” sections to understand the core concepts of the language.

**3. Install the Solidity Compiler**

There are various ways to install the Solidity compiler,
simply choose your preferred option and follow the steps outlined on the :ref:`installation page <installing-solidity>`.

.. hint::
  You can try out code examples directly in your browser with the
  `Remix IDE <https://remix.ethereum.org>`_.
  Remix is a web browser-based IDE that allows you to write, deploy and administer Solidity smart contracts,
  without the need to install Solidity locally.

.. warning::
    As humans write software, it can have bugs.
    Therefore, you should follow established software development best practices when writing your smart contracts.
    This includes code review, testing, audits, and correctness proofs.
    Smart contract users are sometimes more confident with code than their authors,
    and blockchains and smart contracts have their own unique issues to watch out for,
    so before working on production code, make sure you read the :ref:`security_considerations` section.

**4. Learn More**

If you want to learn more about building decentralized applications on Ethereum,
the `Ethereum Developer Resources <https://ethereum.org/en/developers/>`_ can help you with further general documentation around Ethereum,
and a wide selection of tutorials, tools, and development frameworks.

If you have any questions, you can try searching for answers or asking on the
`Ethereum StackExchange <https://ethereum.stackexchange.com/>`_,
or our `Gitter channel <https://gitter.im/ethereum/solidity>`_.

.. _translations:

Translations
------------

Community contributors help translate this documentation into several languages.
Note that they have varying degrees of completeness and up-to-dateness.
The English version stands as a reference.

You can switch between languages by clicking on the flyout menu in the bottom-left corner
and selecting the preferred language.

* `Chinese <https://docs.soliditylang.org/zh/latest/>`_
* `French <https://docs.soliditylang.org/fr/latest/>`_
* `Indonesian <https://github.com/solidity-docs/id-indonesian>`_
* `Japanese <https://github.com/solidity-docs/ja-japanese>`_
* `Korean <https://github.com/solidity-docs/ko-korean>`_
* `Persian <https://github.com/solidity-docs/fa-persian>`_
* `Russian <https://github.com/solidity-docs/ru-russian>`_
* `Spanish <https://github.com/solidity-docs/es-spanish>`_
* `Turkish <https://docs.soliditylang.org/tr/latest/>`_

.. note::

   We set up a GitHub organization and translation workflow to help streamline the community efforts.
   Please refer to the translation guide in the `solidity-docs org <https://github.com/solidity-docs>`_
   for information on how to start a new language or contribute to the community translations.

Contents
========

:ref:`Keyword Index <genindex>`, :ref:`Search Page <search>`

.. toctree::
   :maxdepth: 2
   :caption: Basics

   introduction-to-smart-contracts.rst
   solidity-by-example.rst
   installing-solidity.rst

.. toctree::
   :maxdepth: 2
   :caption: Language Description

   layout-of-source-files.rst
   structure-of-a-contract.rst
   types.rst
   units-and-global-variables.rst
   control-structures.rst
   contracts.rst
   assembly.rst
   cheatsheet.rst
   grammar.rst

.. toctree::
   :maxdepth: 2
   :caption: Compiler

   using-the-compiler.rst
   analysing-compilation-output.rst
   ir-breaking-changes.rst

.. toctree::
   :maxdepth: 2
   :caption: Internals

   internals/layout_in_storage.rst
   internals/layout_in_memory.rst
   internals/layout_in_calldata.rst
   internals/variable_cleanup.rst
   internals/source_mappings.rst
   internals/optimizer.rst
   metadata.rst
   abi-spec.rst

.. toctree::
   :maxdepth: 2
   :caption: Advisory content

   security-considerations.rst
   bugs.rst
   050-breaking-changes.rst
   060-breaking-changes.rst
   070-breaking-changes.rst
   080-breaking-changes.rst

.. toctree::
   :maxdepth: 2
   :caption: Additional Material

   natspec-format.rst
   smtchecker.rst
   yul.rst
   path-resolution.rst

.. toctree::
   :maxdepth: 2
   :caption: Resources

   style-guide.rst
   common-patterns.rst
   resources.rst
   contributing.rst
   language-influences.rst
   brand-guide.rst
