.. index:: contract, state variable, function, event, struct, enum, function;modifier

.. _contract_structure:

***********************
מבנה של חוזה
***********************

חוזים בסולידיטי דומים למחלקות (classes) בשפות מונחות עצמים.
כל חוזה יכול להכיל הצהרות של :ref:`structure-state-variables`, :ref:`structure-functions`,
:ref:`structure-function-modifiers`, :ref:`structure-events`, :ref:`structure-errors`, :ref:`structure-struct-types` ו-:ref:`structure-enum-types`.
בנוסף על כך, חוזים יכולים לרשת מחוזים אחרים.

ישנם גם סוגים מיוחדים של חוזים הנקראים :ref:`ספריות<libraries>` ו-:ref:`ממשקים<interfaces>`.

הסעיף על :ref:`חוזים<contracts>` מכיל יותר פרטים מאשר סעיף זה,
אשר נותן רק סקירה מהירה.

.. _structure-state-variables:

משתני מצב
===============

משתני מצב הם משתנים שהערכים שלהם מאוחסנים באופן קבוע
ב-storage של חוזה.

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.0 <0.9.0;

    contract SimpleStorage {
        uint storedData; // State variable
        // ...
    }

עיין בסעיף :ref:`types` עבור סוגי משתני מצב חוקיים ו-
:ref:`visibility-and-getters` עבור אפשרויות של נראות (visibility).

.. _structure-functions:

פונקציות
=========

פונקציות הן יחידות ההפעלה של הקוד. פונקציות בדרך כלל
מוגדרות בתוך חוזה, אך ניתן להגדיר אותן גם מחוץ לחוזים.

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.7.1 <0.9.0;

    contract SimpleAuction {
        function bid() public payable { // Function
            // ...
        }
    }

    // Helper function defined outside of a contract
    function helper(uint x) pure returns (uint) {
        return x * 2;
    }

:ref:`קריאות לפונקציות` יכולות להתבצע פנימית או חיצונית
וברמות שונות של :ref:`נראות<visibility-and-getters>`
כלפי חוזים אחרים. :ref:`פונקציות<functions>` מקבלות את :ref:`הפרמטרים ומשתנים מוחזרים<function-parameters-return-variables>` כדי להעביר פרמטרים
וערכים ביניהם.

.. _structure-function-modifiers:

משני פונקציות
==================

ניתן להשתמש במשני פונקציות (function modifiers) כדי לתקן את הסמנטיקה של פונקציות בצורה הצהרתית
(ראו :ref:`modifiers` בסעיף החוזים).

הגדרת אותו שם משנה (modifier) אבל עם פרמטרים שונים (overloading),
בלתי אפשרית.

כמו פונקציות, ניתן לבצע :ref:`override <modifier-overriding>` של משנים.

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.22 <0.9.0;

    contract Purchase {
        address public seller;

        modifier onlySeller() { // Modifier
            require(
                msg.sender == seller,
                "Only seller can call this."
            );
            _;
        }

        function abort() public view onlySeller { // Modifier usage
            // ...
        }
    }

.. _structure-events:

אירועים (Events)
=================

אירועים (Events) הם ממשקים נוחים עם שרותי הלוגים של EVM.

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.21 <0.9.0;

    contract SimpleAuction {
        event HighestBidIncreased(address bidder, uint amount); // Event

        function bid() public payable {
            // ...
            emit HighestBidIncreased(msg.sender, msg.value); // Triggering event
        }
    }

ראו :ref:`events` בסעיף חוזים למידע על אופן ההגדרה על אירועים
ואיך ניתן להשתמש בהם מתוך dapp.

.. _structure-errors:

שגיאות
======

שגיאות מאפשרות לכם להגדיר שמות תיאור ונתונים עבור מצבי כשל.
ניתן להשתמש בשגיאות ב-:ref:`פקודות revert <revert-statement>`.
בהשוואה לתיאורי מחרוזת, שגיאות זולות הרבה יותר ומאפשרות לכם
לקודד נתונים נוספים. אתם יכולים להשתמש ב-NatSpec כדי לתאר את השגיאות
למשתמש.

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity ^0.8.4;

    /// Not enough funds for transfer. Requested `requested`,
    /// but only `available` available.
    error NotEnoughFunds(uint requested, uint available);

    contract Token {
        mapping(address => uint) balances;
        function transfer(address to, uint amount) public {
            uint balance = balances[msg.sender];
            if (balance < amount)
                revert NotEnoughFunds(amount, balance);
            balances[msg.sender] -= amount;
            balances[to] += amount;
            // ...
        }
    }

למידע נוסף ראו :ref:`errors` בסעיף החוזים.

.. _structure-struct-types:

סוגי Struct
=============

Structs הם סוגי משתנים המוגדרים בהתאמה אישית שיכולים לקבץ מספר משתנים (ראה
:ref:`structs` בסעיף types).

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.0 <0.9.0;

    contract Ballot {
        struct Voter { // Struct
            uint weight;
            bool voted;
            address delegate;
            uint vote;
        }
    }

.. _structure-enum-types:

סוגי Enum
==========

ניתן להשתמש ב-Enums ליצירת סוגי משתנים מותאמים אישית
עם קבוצה סופית של 'ערכים קבועים'
(ראו :ref:`enums` בקטע types).

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.0 <0.9.0;

    contract Purchase {
        enum State { Created, Locked, Inactive } // Enum
    }
