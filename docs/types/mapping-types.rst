.. index:: !mapping
.. _mapping-types:

סוגי מיפוי
=============

סוגי משתני מיפוי משתמשים בתחביר
``mapping(KeyType KeyName? => ValueType ValueName?)``
ובמשתנים של סוג המיפוי המוצהר באמצעות התחביר
``mapping(KeyType KeyName? => ValueType ValueName?) VariableName``.
ה-``KeyType`` יכול להיות כל סוג מובנה של ערך, ``bytes``, ``string`` או כל
סוג חוזה או enum. סוגים אחרים המוגדרים על ידי המשתמש
או סוגים מורכבים, כגון מיפויים, structs או סוגי מערכים
לא מורשים. ``ValueType`` יכול להיות כל סוג, כולל מיפויים, מערכים ו-structs. ``KeyName``
ו-``ValueName`` הם אופציונליים (לכן ``mapping(KeyType => ValueType)`` עובד גם כן) ויכולים להיות כל מזהה חוקי שאינו סוג משתנה.

אתם יכולים לחשוב על מיפויים כעל
`טבלאות hash <https://en.wikipedia.org/wiki/Hash_table>`_,
שמאותחלות באופן וירטואלי, כך שכל מפתח אפשרי קיים וממופה לערך
שהייצוג שלו בבתים הוא כולו אפסים,
:ref:`ערך ברירת מחדל <default-value>` של סוג המשתנה.
הדמיון מסתיים שם. נתוני המפתח אינם מאוחסנים ב-mapping,
רק ה-hash ``keccak256`` שלו משמש כדי לחפש את הערך.

מכיוון שכך, למיפויים אין אורך או מושג של מפתח או
ערך מוגדר, ולכן לא ניתן למחוק אותם ללא מידע נוסף
לגבי המפתחות שהוקצו (ראו :ref:`clearing-mappings`).

מיפויים יכולים להיות מאוחסנים רק ב-``storage`` וכך
מאופשרים למשתני מצב, כסוגי התייחסות ל-storage
בפונקציות, או כפרמטרים לפונקציות של ספרייה.
לא ניתן להשתמש בהם כפרמטרים או לפרמטרים המוחזרים מפונקציות
חוזה גלויות לכולם.
הגבלות אלו נכונות גם עבור מערכים ו-structs המכילים מיפויים.

ניתן לסמן משתני מצב מסוג מיפוי כ-``public`` וסולידיטי יוצר
:ref:`getter <visibility-and-getters>` בשבילכם. ה- ``KeyType`` הופך לפרמטר
עם השם ``KeyName`` (אם צויין) עבור ה-getter.
אם ``ValueType`` הוא סוג ערך או struct, ה-getter מחזיר ``ValueType`` עם
השם ``ValueName`` (אם צויין).
אם ``ValueType`` הוא מערך או מיפוי, למקבל יש פרמטר אחד עבור
כל ``KeyType``, רקורסיבית.

בדוגמה למטה, החוזה ``MappingExample`` מגדיר סוג מיפוי ``balances`` ציבורי,
עם מפתח מסוג ``address``, וערך מסוג ``uint``, ממפה
כתובת איתריום לערך מספר שלם ללא סימן. מכיוון ש-``uint`` הוא סוג של ערך, ה-getter
מחזירה ערך התואם לסוג זה, אותו תוכלו לראות בחוזה ``MappingUser``
שמחזיר את הערך בכתובת שצוינה.

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.0 <0.9.0;

    contract MappingExample {
        mapping(address => uint) public balances;

        function update(uint newBalance) public {
            balances[msg.sender] = newBalance;
        }
    }

    contract MappingUser {
        function f() public returns (uint) {
            MappingExample m = new MappingExample();
            m.update(100);
            return m.balances(address(this));
        }
    }

הדוגמה שלהלן היא גרסה פשוטה של
`token ERC20 <https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/ERC20.sol>`_.
``_allowances`` היא דוגמה לסוג מיפוי בתוך סוג מיפוי אחר.

בדוגמה, ה-``KeyName`` וה-``ValueName`` האופצונליים מסופקים עבור המיפוי.
לדבר זה אין כל השפעה על הפונקציונליות של החוזה או של ה-bytecode. הוא רק מגדיר
את השדה ``name``
עבור הכניסות והיציאות ב-ABI עבור ה-getter של המיפוי.

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity ^0.8.18;

    contract MappingExampleWithNames {
        mapping(address user => uint balance) public balances;

        function update(uint newBalance) public {
            balances[msg.sender] = newBalance;
        }
    }


הדוגמה שלהלן משתמשת ב-``allowances_`` כדי לרשום את הסכום שמישהו אחר רשאי למשוך מחשבונך.

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.22 <0.9.0;

    contract MappingExample {

        mapping(address => uint256) private _balances;
        mapping(address => mapping(address => uint256)) private _allowances;

        event Transfer(address indexed from, address indexed to, uint256 value);
        event Approval(address indexed owner, address indexed spender, uint256 value);

        function allowance(address owner, address spender) public view returns (uint256) {
            return _allowances[owner][spender];
        }

        function transferFrom(address sender, address recipient, uint256 amount) public returns (bool) {
            require(_allowances[sender][msg.sender] >= amount, "ERC20: Allowance not high enough.");
            _allowances[sender][msg.sender] -= amount;
            _transfer(sender, recipient, amount);
            return true;
        }

        function approve(address spender, uint256 amount) public returns (bool) {
            require(spender != address(0), "ERC20: approve to the zero address");

            _allowances[msg.sender][spender] = amount;
            emit Approval(msg.sender, spender, amount);
            return true;
        }

        function _transfer(address sender, address recipient, uint256 amount) internal {
            require(sender != address(0), "ERC20: transfer from the zero address");
            require(recipient != address(0), "ERC20: transfer to the zero address");
            require(_balances[sender] >= amount, "ERC20: Not enough funds.");

            _balances[sender] -= amount;
            _balances[recipient] += amount;
            emit Transfer(sender, recipient, amount);
        }
    }


.. index:: !iterable mappings
.. _iterable-mappings:

מיפוי איטרטיבי
-----------------

אתם לא יכולים לבצע מיפויים איטרטיביים, כלומר אתם
לא יכולים לבצע enumerate למפתחות שלהם.
עם זאת, ניתן ליישם מבנה נתונים מעליהם
ולבצע איטרציות עליו. לדוגמה, הקוד שלהלן מיישם
ספריית ``IterableMapping`` שחוזה ``User`` מוסיף לה נתונים,
והפונקציה ``sum`` מבצעת איטרציות כדי לסכם את כל הערכים.

.. code-block:: solidity
    :force:

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity ^0.8.8;

    struct IndexValue { uint keyIndex; uint value; }
    struct KeyFlag { uint key; bool deleted; }

    struct itmap {
        mapping(uint => IndexValue) data;
        KeyFlag[] keys;
        uint size;
    }

    type Iterator is uint;

    library IterableMapping {
        function insert(itmap storage self, uint key, uint value) internal returns (bool replaced) {
            uint keyIndex = self.data[key].keyIndex;
            self.data[key].value = value;
            if (keyIndex > 0)
                return true;
            else {
                keyIndex = self.keys.length;
                self.keys.push();
                self.data[key].keyIndex = keyIndex + 1;
                self.keys[keyIndex].key = key;
                self.size++;
                return false;
            }
        }

        function remove(itmap storage self, uint key) internal returns (bool success) {
            uint keyIndex = self.data[key].keyIndex;
            if (keyIndex == 0)
                return false;
            delete self.data[key];
            self.keys[keyIndex - 1].deleted = true;
            self.size --;
        }

        function contains(itmap storage self, uint key) internal view returns (bool) {
            return self.data[key].keyIndex > 0;
        }

        function iterateStart(itmap storage self) internal view returns (Iterator) {
            return iteratorSkipDeleted(self, 0);
        }

        function iterateValid(itmap storage self, Iterator iterator) internal view returns (bool) {
            return Iterator.unwrap(iterator) < self.keys.length;
        }

        function iterateNext(itmap storage self, Iterator iterator) internal view returns (Iterator) {
            return iteratorSkipDeleted(self, Iterator.unwrap(iterator) + 1);
        }

        function iterateGet(itmap storage self, Iterator iterator) internal view returns (uint key, uint value) {
            uint keyIndex = Iterator.unwrap(iterator);
            key = self.keys[keyIndex].key;
            value = self.data[key].value;
        }

        function iteratorSkipDeleted(itmap storage self, uint keyIndex) private view returns (Iterator) {
            while (keyIndex < self.keys.length && self.keys[keyIndex].deleted)
                keyIndex++;
            return Iterator.wrap(keyIndex);
        }
    }

    // How to use it
    contract User {
        // Just a struct holding our data.
        itmap data;
        // Apply library functions to the data type.
        using IterableMapping for itmap;

        // Insert something
        function insert(uint k, uint v) public returns (uint size) {
            // This calls IterableMapping.insert(data, k, v)
            data.insert(k, v);
            // We can still access members of the struct,
            // but we should take care not to mess with them.
            return data.size;
        }

        // Computes the sum of all stored data.
        function sum() public view returns (uint s) {
            for (
                Iterator i = data.iterateStart();
                data.iterateValid(i);
                i = data.iterateNext(i)
            ) {
                (, uint value) = data.iterateGet(i);
                s += value;
            }
        }
    }
