.. index:: ! type;reference, ! reference type, storage, memory, location, array, struct

.. _reference-types:

סוגי הפנייה (Reference)
=========================

ניתן לשנות ערכים של סוג הפניה (Reference) באמצעות מספר שמות שונים.
זאת בניגוד לסוגי ערכים שבהם אתם מקבלים עותק בלתי תלוי בכל פעם
שבה נעשה שימוש במשתנה מסוג ערך (value type). לכן, יש לטפל בסוגי הפניות
בזהירות רבה יותר מאשר בסוגי ערכים. נכון לעכשיו, סוגי הפניות כוללים structs,
מערכים ומיפויים. אם אתם משתמשים בסוג הפניה, אתם תמיד צריכים לספק במפורש
את אזור הנתונים שבו מאוחסן הסוג: ``memory`` (שאורך חייו מוגבל
לקריאת לפונקציה חיצונית), ``storage`` (המיקום בו משתני המצב
מאוחסנים, כאשר משך החיים מוגבל למשך חיי החוזה)
או ``calldata`` (מיקום נתונים מיוחד המכיל את הארגומנטים של הפונקציה).

הקצאה או המרת סוג שמשנה את מיקום הנתונים תמיד תגרור פעולת העתקה אוטומטית,
בעוד שהמשימות בתוך אותו מיקום נתונים מועתקות רק במקרים מסוימים של סוגי אחסון.

.. _data-location:

מיקומי נתונים
-------------

לכל סוג הפניה יש תוספת "מיקום הנתונים", המציינת
את המיקום שבו הוא מאוחסן. ישנם שלושה מיקומי נתונים: ``memory``, ``storage``, ``calldata``.
המיקום calldata הוא מיקום לא קבוע שאינו ניתן לשינוי,
שבו מאוחסנים ארגומנטים של פונקציה, ומתנהג בעיקר כמו memory.

.. note::
    אם אפשר, נסו להשתמש ב-``calldata`` כמיקום נתונים מכיוון שהוא ימנע העתקות
    והוא גם מוודא שאי אפשר לשנות את הנתונים. פונקציה יכולה להחזיר
    מערכים ו-structs  ב-מיקום של `calldata``, אבל אי אפשר
    להקצות סוגים אלו.

.. note::
    לפני גרסה 0.6.9 מיקום הנתונים עבור ארגומנטים מסוג הפניה הוגבל
    ל-``calldata`` בפונקציות חיצוניות, ``memory`` בפונקציות ציבוריות וכן
    ``memory`` או ``storage`` בפונקציות פנימיות או פרטיות.
    עכשיו ``memory`` וגם ``calldata`` מותרים בכל הפונקציות ללא קשר לנראות שלהם.

.. note::
    לפני גרסה 0.5.0 ניתן היה להשמיט את מיקום הנתונים, והיתה ברירת מחדל למיקומים שונים
    תלוי בסוג המשתנה, סוג הפונקציה וכולי, אבל עכשיו לכל הסוגים המורכבים חייבים להגדיר במפורש את מיקום הנתונים.

.. _data-location-assignment:

מיקום נתונים והתנהגות השמה (Assignment)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

מיקומי נתונים רלוונטיים לא רק להתמדה (persistency) של הנתונים, אלא גם לסמנטיקה של השמות (assignments):

* הקצאות בין ``storage`` ו-``memory`` (או מתוך ``calldata``)
  תמיד מייצר עותק עצמאי.
* הקצאות מ-``memory`` ל-``memory`` יוצרות הפניות בלבד. זאת אומרת
  ששינויים במשתנה memory אחד נראים גם בכל משתני memory אחרים
  המתייחסים לאותם נתונים.
* גם הקצאות מ-``storage`` למשתנה storage **מקומי**
  מקצות הפניות.
* כל שאר ההקצאות ל-``storage`` תמיד מעתיקות. דוגמאות לכך
  הן הקצאות למשתני מצב או איברים של
  משתנים מקומיים מסוג storage struct, גם אם המשתנה המקומי
  עצמו הוא רק הפנייה.

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.5.0 <0.9.0;

    contract C {
        // The data location of x is storage.
        // This is the only place where the
        // data location can be omitted.
        uint[] x;

        // The data location of memoryArray is memory.
        function f(uint[] memory memoryArray) public {
            x = memoryArray; // works, copies the whole array to storage
            uint[] storage y = x; // works, assigns a pointer, data location of y is storage
            y[7]; // fine, returns the 8th element
            y.pop(); // fine, modifies x through y
            delete x; // fine, clears the array, also modifies y
            // The following does not work; it would need to create a new temporary /
            // unnamed array in storage, but storage is "statically" allocated:
            // y = memoryArray;
            // Similarly, "delete y" is not valid, as assignments to local variables
            // referencing storage objects can only be made from existing storage objects.
            // It would "reset" the pointer, but there is no sensible location it could point to.
            // For more details see the documentation of the "delete" operator.
            // delete y;
            g(x); // calls g, handing over a reference to x
            h(x); // calls h and creates an independent, temporary copy in memory
        }

        function g(uint[] storage) internal pure {}
        function h(uint[] memory) public pure {}
    }

.. index:: ! array

.. _arrays:

מערכים
------

למערכים יכול להיות גודל קבוע או דינאמי בזמן קומפילציה.

מערך בגודל קבוע ``k`` וסוג רכיב ``T`` נכתב כ-``T[k]``,
ומערך בגודל דינמי כ-``T[]``.

לדוגמה, מערך של 5 מערכים דינמיים של ``uint`` נכתב בשם
``uint[][5]``. שימו לב שהסימון הפוך בהשוואה לשפות אחרות.
בסולידיטי, ``X[3]`` הוא תמיד מערך המכיל שלושה אלמנטים מסוג ``X``,
גם אם ``X`` הוא עצמו מערך. זה לא המקרה בשפות אחרות כגון C.

האינדקסים מתחילים מאפס, והגישה היא בכיוון ההפוך מההצהרה.

לדוגמה, אם יש לך משתנה ``uint[][5] memory x``, ניגשים ל-``uint``
השביעי במערך הדינאמי השלישי באמצעות ``x[2][6]``, וכדי לגשת
למערך הדינאמי השלישי, השתמשו ב-``x[2]``. שוב,
אם יש לכם מערך ``T[5] a`` עבור סוג ``T`` שיכול להיות גם מערך,
אז ל- ``a[2]`` תמיד יש את הסוג ``T``.

רכיבי מערך יכולים להיות מכל סוג, כולל מיפוי או strauct.
ההגבלות הכלליות על סוגי משתנים חלות גם לגבי מערכים
בכך שניתן לאחסן מיפויים רק ב-``storage``
ופונקציות public זקוקות לפרמטרים שהם :ref:`סוגי ABI <ABI>`.

אפשר לסמן מערכי משתני מצב ``public`` ולגרום לסולידיטי 
ליצור :ref:`getter <visibility-and-getters>`.
האינדקס המספרי הופך לפרמטר נדרש עבור ה-getter.

גישה למערך מעבר לקצה שלו גורמת לשגיאה.
ניתן להשתמש בשיטות ``()push.`` ו-``push(value).``
כדי להוסיף רכיב חדש בסוף מערך בגודל דינאמי,
כאשר ``()push.`` מוסיף אלמנט מאותחל באפס ומחזיר הפניה אליו.

.. note::
    ניתן לשנות גודל מערכים בגודל דינאמי רק ב-storage.
    ב-memory, מערכים כאלה יכולים להיות בגודל שרירותי, אך לא ניתן לשנות את הגודל לאחר הקצאת מערך.

.. index:: ! string, ! bytes

.. _strings:

.. _bytes:

``bytes`` ו-``string`` כמערכים
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

משתנים מסוג ``bytes (בתים)`` ו-``string`` (מחרוזת) הם מערכים מיוחדים.
הסוג ``bytes`` דומה ל-``[]bytes1``,
אבל הוא דחוס ב-calldata וב-memory. משתנה ``string`` שווה למשתנה ``bytes``
אך אינו מאפשר אורך או גישה לאינדקס.

לסולידיטי אין פונקציות למניפולציות של מחרוזות, אבל ישנן
ספריות מחרוזות צד-שלישי לבצוע פעולות כאלו.
אתם יכולים גם להשוות בין שתי מחרוזות לפי ה-keccak256-hash שלהן
``keccak256(abi.encodePacked(s1)) == keccak256(abi.encodePacked(s2))``
וכן לשרשר שתי מחרוזות באמצעות ``string.concat(s1, s2)``.

אתם צריכים להעדיף ``bytes`` על-פני ``[]bytes1`` כי זה זול יותר,
מכיוון ששימוש ב-``[]bytes1`` ב``memory`` מוסיף 31 בתים לריפוד בין האלמנטים.
שימו לב שב-``storage``, הריפוד לא מתבצע עקב דחיסת הנתונים,
ראו :ref:`bytes and string <bytes-and-string>`.
ככלל, השתמשו ב-``bytes`` עבור נתוני בתים גולמיים באורך שרירותי
וב-``string`` עבור נתוני מחרוזת (UTF-8) באורך שרירותי.
אם אתם יכולים להגביל את האורך למספר מסוים של בתים,
השתמשו תמיד באחד מסוגי הערכים ``bytes1`` עד ``bytes32``
כי הם הרבה יותר זולים.

.. note::
    אם אתם רוצים לגשת לייצוג בתים של מחרוזת ``s``, השתמשו ב-
    ``;'bytes(s).length`` / ``bytes(s)[7] = 'x``. 
    זכרו שאתם ניגשים ל-bytes ברמה נמוכה של ייצוג UTF-8,
    ולא לתווים האינדיבידואלים.

.. index:: ! bytes-concat, ! string-concat

.. _bytes-concat:
.. _string-concat:

הפונקציות ``bytes.concat`` ו-``string.concat``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

אתם יכולים לשרשר מספר שרירותי של ערכי ``string`` באמצעות ``string.concat``.
הפונקציה מחזירה מערך ``string memory`` יחיד המכיל את תוכן הארגומנטים ללא ריפוד.
אם אתם רוצים להשתמש בפרמטרים מסוגים אחרים שאינם ניתנים להמרה
באופן פנימי ל``string``, עליכם להמיר אותם תחילה  ל-``string``.

באופן דומה, הפונקציה ``bytes.concat`` יכולה לשרשר מספר שרירותי של ערכי ``bytes`` או ``bytes1 ... bytes32``.
הפונקציה מחזירה מערך ``bytes memory`` יחיד המכיל את תוכן הארגומנטים ללא ריפוד.
אם אתם רוצים להשתמש בפרמטרי string או סוגים אחרים שאינם ניתנים להמרה באופן פנימי ל-``bytes``, אתם צריכים להמיר אותם תחילה ל-``bytes`` או ל-``bytes1``/.../``bytes32`` .


.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity ^0.8.12;

    contract C {
        string s = "Storage";
        function f(bytes calldata bc, string memory sm, bytes16 b) public view {
            string memory concatString = string.concat(s, string(bc), "Literal", sm);
            assert((bytes(s).length + bc.length + 7 + bytes(sm).length) == bytes(concatString).length);

            bytes memory concatBytes = bytes.concat(bytes(s), bc, bc[:2], "Literal", bytes(sm), b);
            assert((bytes(s).length + bc.length + 2 + 7 + bytes(sm).length + b.length) == concatBytes.length);
        }
    }

אם אתם קוראים ל-``string.concat`` או ל-``bytes.concat`` ללא ארגומנטים הם מחזירים מערך ריק.

.. index:: ! array;allocating, new

הקצאת מערכים ב-Memory
^^^^^^^^^^^^^^^^^^^^^^^^

ניתן ליצור ב-memory מערכים בעלי אורך דינאמי באמצעות האופרטור ``new``.
בניגוד למערכים ב-storage, **לא** ניתן לשנות גודל של מערכים ב-memory (למשל
פונקציות ``push.`` לאיבר אינן זמינות).
אתם צריכים לחשב את הגודל הנדרש מראש
או ליצור מערך חדש ב-memory ולהעתיק כל איבר.

כמו כל המשתנים בסולידיטי, האלמנטים של מערכים חדשים שהוקצו מאותחלים תמיד
עם :ref:`ערך ברירת מחדל<default-value>`.

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.16 <0.9.0;

    contract C {
        function f(uint len) public pure {
            uint[] memory a = new uint[](7);
            bytes memory b = new bytes(len);
            assert(a.length == 7);
            assert(b.length == len);
            a[6] = 8;
        }
    }

.. index:: ! literal;array, ! inline;arrays

ליטרלים של מערכים
^^^^^^^^^^^^^^^^^^^^

ליטרלים של מערך היא רשימה מופרדת בפסיק של ביטוי אחד או יותר, מוקפת
בסוגריים מרובעים (``[...]``). לדוגמה ``[1, a, f(3)]``. סוג
הליטרל של המערך נקבע באופן הבא:

הוא תמיד מערך ב-memory בגודל סטטי שאורכו הוא מספר הביטויים.

סוג הבסיס של המערך הוא סוג הביטוי הראשון ברשימה כך
שניתן להמיר אליו את כל הביטויים האחרים באופן פנימי .
זו שגיאה אם אין ביטוי כזה במערך.

לא מספיק שיש סוג שאליו ניתן להמיר את כל האלמנטים. אחד המרכיבים
חייב להיות מהסוג הזה.

בדוגמה למטה, הסוג של ``[1, 2, 3]`` הוא
``uint8[3] memory``, מכיוון שהסוג של כל אחד מהקבועים הללו הוא ``uint8``.
אם אתם רוצים שהתוצאה תהיה מסוג ``uint[3] memory``, עליכם להמיר
את האלמנט הראשון ל-``uint``.

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.16 <0.9.0;

    contract C {
        function f() public pure {
            g([uint(1), 2, 3]);
        }
        function g(uint[3] memory) public pure {
            // ...
        }
    }

המערך הליטרלי ``[1, 1-]`` אינו חוקי בגלל שסוג הביטוי הראשון
הוא ``uint8`` בעוד הסוג של השני הוא ``int8`` והם לא יכולים להיות מומרים
אחד לשני באופן פנימי. כדי המערך יהיה חוקי, אתם יכולים
להשתמש ב-``[int8(1), 1-]``, למשל.

מכיוון שלא ניתן להמיר זה לזה מערכי memory בגודל קבוע אבל מסוג שונה
(גם אם סוגי הבסיס יכולים), אם אתם רוצים להשתמש במערך דו מימדי ליטרלי,
אתם תמיד צריכים לציין במפורש את סוג הבסיס:

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.16 <0.9.0;

    contract C {
        function f() public pure returns (uint24[2][4] memory) {
            uint24[2][4] memory x = [[uint24(0x1), 1], [0xffffff, 2], [uint24(0xff), 3], [uint24(0xffff), 4]];
            // The following does not work, because some of the inner arrays are not of the right type.
            // uint[2][4] memory x = [[0x1, 1], [0xffffff, 2], [0xff, 3], [0xffff, 4]];
            return x;
        }
    }

לא ניתן להציב מערכי memory בגודל קבוע במערכי memory בגודל דינאמי.
כלומר, הדבר הבא אינו אפשרי:

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.0 <0.9.0;

    // This will not compile.
    contract C {
        function f() public {
            // The next line creates a type error because uint[3] memory
            // cannot be converted to uint[] memory.
            uint[] memory x = [uint(1), 3, 4];
        }
    }

מתוכנן שמגבלה זו תוסר בעתיד, אבל הסרה כזו לא פשוטה
בגלל האופן שבו מערכים מועברים ב-ABI.

אם אתם רוצים לאתחל מערכים בגודל דינאמי, עליכם להציב ערכים בכל
אלמנט בנפרד:

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.16 <0.9.0;

    contract C {
        function f() public pure {
            uint[] memory x = new uint[](3);
            x[0] = 1;
            x[1] = 3;
            x[2] = 4;
        }
    }

.. index:: ! array;length, length, push, pop, !array;push, !array;pop

.. _array-members:

מרכיבים של מערך
^^^^^^^^^^^^^^^^^^

**length**:
 	למערכים יש איבר ``length`` המכיל את מספר האלמנטים שלהם.
 	אורך מערכי הזיכרון קבוע (אך דינאמי, כלומר יכול להיות תלוי
 	בפרמטרי זמן ריצה) לאחר יצירתם.
**()push**:
  	למערכי storage דינמיים ו-``bytes`` (לא ``string``) יש פונקצייה
  	שנקראת ``()push`` שאתם יכולים להשתמש בה כדי להוסיף אלמנט מאותחל אפס בסוף המערך.
  	הפונקציה מחזירה הפניה לאלמנט, כך שניתן יהיה להשתמש בו כמו
  	``x.push().t = 2`` או ``x.push() = b``.
**push(x)**:
  	למערכי storage דינמיים ו-``bytes`` (לא ``string``) יש פונקציה
  	שנקראת ``push(x)`` שתוכלו להשתמש בה כדי להוסיף אלמנט נתון בסוף המערך.
  	הפונקציה לא מחזירה כלום.
**()pop**:
  	למערכים דינאמיים של storage ו-``bytes`` (לא ``string``) יש
  	פונקציה בשם ``()pop`` שתוכלו להשתמש בה כדי להסיר אלמנט
  	מסוף המערך. פונקציה זו גם קוראת באופן פנימי
  	ל-:ref:`delete<delete>` לאלמנט שהוסר. הפונקציה לא מחזירה כלום.

.. note::
 	הגדלת אורך מערך storage על ידי קריאה ל-``()push``
 	הוא בעל עלויות גז קבועות מכיוון שה-storage  מאותחל באפס,
 	בעוד שלהקטנת האורך על ידי קריאת ``()pop`` יש
 	עלות שתלויה ב"גודל" האלמנט המוסר.
 	אם האלמנט הזה הוא מערך, דבר זה יכול להיות מאוד יקר מכיוון,
 	שהתהליך כולל את ניקוי האלמנטים שהוסרו
 	בדומה לקריאה ל- :ref:`delete<delete>` לגביהם.

.. note::
    כדי להשתמש במערכים של מערכים בפונקציות חיצוניות (במקום ציבוריות), אתם צריכים
    להפעיל ABI coder v2.

.. note::
    בגרסאות EVM לפני גרסת Byzantium, לא ניתן היה לגשת
    למערכים דינאמיים שהוחזרו מקריאות לפונקציות. אם אתם קוראים לפונקציות
    שמחזירים מערכים דינאמיים, הקפידו להשתמש ב-EVM שמוגדר
    למצב Byzantium.

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.6.0 <0.9.0;

    contract ArrayContract {
        uint[2**20] aLotOfIntegers;
        // Note that the following is not a pair of dynamic arrays but a
        // dynamic array of pairs (i.e. of fixed size arrays of length two).
        // In Solidity, T[k] and T[] are always arrays with elements of type T,
        // even if T itself is an array.
        // Because of that, bool[2][] is a dynamic array of elements
        // that are bool[2]. This is different from other languages, like C.
        // Data location for all state variables is storage.
        bool[2][] pairsOfFlags;

        // newPairs is stored in memory - the only possibility
        // for public contract function arguments
        function setAllFlagPairs(bool[2][] memory newPairs) public {
            // assignment to a storage array performs a copy of ``newPairs`` and
            // replaces the complete array ``pairsOfFlags``.
            pairsOfFlags = newPairs;
        }

        struct StructType {
            uint[] contents;
            uint moreInfo;
        }
        StructType s;

        function f(uint[] memory c) public {
            // stores a reference to ``s`` in ``g``
            StructType storage g = s;
            // also changes ``s.moreInfo``.
            g.moreInfo = 2;
            // assigns a copy because ``g.contents``
            // is not a local variable, but a member of
            // a local variable.
            g.contents = c;
        }

        function setFlagPair(uint index, bool flagA, bool flagB) public {
            // access to a non-existing index will throw an exception
            pairsOfFlags[index][0] = flagA;
            pairsOfFlags[index][1] = flagB;
        }

        function changeFlagArraySize(uint newSize) public {
            // using push and pop is the only way to change the
            // length of an array
            if (newSize < pairsOfFlags.length) {
                while (pairsOfFlags.length > newSize)
                    pairsOfFlags.pop();
            } else if (newSize > pairsOfFlags.length) {
                while (pairsOfFlags.length < newSize)
                    pairsOfFlags.push();
            }
        }

        function clear() public {
            // these clear the arrays completely
            delete pairsOfFlags;
            delete aLotOfIntegers;
            // identical effect here
            pairsOfFlags = new bool[2][](0);
        }

        bytes byteData;

        function byteArrays(bytes memory data) public {
            // byte arrays ("bytes") are different as they are stored without padding,
            // but can be treated identical to "uint8[]"
            byteData = data;
            for (uint i = 0; i < 7; i++)
                byteData.push();
            byteData[3] = 0x08;
            delete byteData[2];
        }

        function addFlag(bool[2] memory flag) public returns (uint) {
            pairsOfFlags.push(flag);
            return pairsOfFlags.length;
        }

        function createMemoryArray(uint size) public pure returns (bytes memory) {
            // Dynamic memory arrays are created using `new`:
            uint[2][] memory arrayOfPairs = new uint[2][](size);

            // Inline arrays are always statically-sized and if you only
            // use literals, you have to provide at least one type.
            arrayOfPairs[0] = [uint(1), 2];

            // Create a dynamic byte array:
            bytes memory b = new bytes(200);
            for (uint i = 0; i < b.length; i++)
                b[i] = bytes1(uint8(i));
            return b;
        }
    }

.. index:: ! array;dangling storage references

הפניות לא-יציבות (Dangling References) לרכיבי מערך Storage
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

כאשר עובדים עם מערכים ב-storage, עליכם להקפיד
להימנע מהפניות לא-יציבות (dangling references).
הפניה לא-יציבה היא התייחסות שמצביעה על משהו שכבר לא קיים או
שהועבר מבלי לעדכן את ההפניה. הפניה לא-יציבה יכולה להתרחש למשל, אם אתם
מציבים התייחסות לאלמנט מערך במשתנה מקומי ולאחר מכן
מבצעים  ``()pop.`` מהמערך המכיל:

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.8.0 <0.9.0;

    contract C {
        uint[][] s;

        function f() public {
            // Stores a pointer to the last array element of s.
            uint[] storage ptr = s[s.length - 1];
            // Removes the last array element of s.
            s.pop();
            // Writes to the array element that is no longer within the array.
            ptr.push(0x42);
            // Adding a new element to ``s`` now will not add an empty array, but
            // will result in an array of length 1 with ``0x42`` as element.
            s.push();
            assert(s[s.length - 1][0] == 0x42);
        }
    }

הכתיבה ב-``ptr.push(0x42)`` **לא** תבצע revert, למרות העובדה ש-``ptr`` כבר לא
מתייחס לרכיב חוקי של ``s``. מכיוון שהקומפיילר מניח ש-storage שלא בשימוש
תמיד מאופס, ``()s.push`` עוקב לא יכתוב במפורש אפסים ל-storage,
ולכן הרכיב האחרון של ``s`` אחרי ה-``()push`` יהיה באורך ``1`` ויכיל
``0x42`` כאלמנט הראשון שלו.

שימו לב שסולידיטי לא מאפשרת להגדיר הפניות לסוגי-ערכים ב-storage. הסוגים האלו
של הפניות בלתי-יציבות מוגבלות לסוגי הפניות מקוננים. עם זאת, הפניות הבלתי-יציבות
יכולות להתקיים באופן זמני בעת שימוש בביטויים מורכבים בהשמות tuple:

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.8.0 <0.9.0;

    contract C {
        uint[] s;
        uint[] t;
        constructor() {
            // Push some initial values to the storage arrays.
            s.push(0x07);
            t.push(0x03);
        }

        function g() internal returns (uint[] storage) {
            s.pop();
            return t;
        }

        function f() public returns (uint[] memory) {
            // The following will first evaluate ``s.push()`` to a reference to a new element
            // at index 1. Afterwards, the call to ``g`` pops this new element, resulting in
            // the left-most tuple element to become a dangling reference. The assignment still
            // takes place and will write outside the data area of ``s``.
            (s.push(), g()[0]) = (0x42, 0x17);
            // A subsequent push to ``s`` will reveal the value written by the previous
            // statement, i.e. the last element of ``s`` at the end of this function will have
            // the value ``0x42``.
            s.push();
            return s;
        }
    }

תמיד בטוח יותר להציב ב-storage רק פעם אחת בכל הצהרה (statement) ולהימנע
מביטויים מורכבים בצד שמאל של השמה.

עליכם לנקוט משנה זהירות כאשר אתם עוסקים בהתייחסויות לאלמנטים של
מערכי ``bytes``, מכיוון ש-``()push.`` במערך בתים עשוי
לעבור :ref:`מפריסה קצרה לארוכה ב-storage<bytes-and-string>`

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.8.0 <0.9.0;

    // This will report a warning
    contract C {
        bytes x = "012345678901234567890123456789";

        function test() external returns(uint) {
            (x.push(), x.push()) = (0x01, 0x02);
            return x.length;
        }
    }

כאן, כאשר ה-``()x.push`` הראשון מוערך, ``x`` עדיין מאוחסן 
במבנה קצר, וכך ``()x.push`` מחזירה הפניה לרכיב בסלוט הראשון של
``x``. עם זאת, ``()x.push`` השני מעביר את מערך הבתים למבנה גדול.
כעת האלמנט שאליו התייחס ה-``()x.push`` נמצא באזור הנתונים של המערך
בעוד ההפניה עדיין מצביעה על מיקומו המקורי, שהוא כעת חלק משדה האורך
וההשמה תעוות למעשה את האורך של ``x``.
ליתר ביטחון, הגדילו מערכי bytes רק ברכיב אחד לכל היותר במהלך השמה
בודדת ואל תפנו בו-זמנית למערך על-ידי אינדקס באותו משפט.

בעוד האמור לעיל מתאר את ההתנהגות של הפניות storage לא-יציבות
בגרסה הנוכחית של הקומפיילר, כל קוד עם הפניות לא-יציבות צריך
להיחשב כבעל *התנהגות לא מוגדרת*. בפרט, המשמעות היא
שכל גרסה עתידית של הקומפיילר עשויה לשנות את התנהגות הקוד
כולל הפניות לא-יציבות.

הקפידו להימנע מהפניות לא-יציבות בקוד שלכם!.

.. index:: ! array;slice

.. _array-slices:

פרוסות מערך (Array Slices)
-----------------------------


פרוסות מערך (Array Slices) הן תצוגה של חלק רציף ממערך.
הן נכתבות כ-``x[start:end]``, כאשר ``start`` ו-``end`` הם ביטויים
מסוג uint256 (או ניתן להמיר אליו באופן פנימי). המרכיב הראשון של
פרוסה הוא ``x[start]`` והרכיב האחרון הוא ``x[end - 1]``.

אם ``start`` גדול מ-``end`` או אם ``end`` גדול יותר
יותר מאורך המערך, נזרק exception.

גם ``start`` וגם ```end`` הם אופציונליים: ברירת המחדל של ``start``
היא ``0`` וברירת מחדל של ``end`` היא אורך המערך.

לפרוסות מערך אין איברים. באופן פנימי הן ניתנות
להמרה למערכים מהסוג הבסיסי שלהן
והן תומכות בגישה לפי אינדקס. הגישה לפי אינדקס אינה
לפי המערך הבסיסי, אלא יחסית להתחלה של הפרוסה.

לפרוסות מערך אין שם סוג, לכן
לאף משתנה לא יכול להיות סוג של פרוסות מערך.
הם קיימים רק בביטויי ביניים.

.. note::
    נכון לעכשיו, פרוסות מערך מיושמות רק עבור מערכים ב-calldata.

פרוסות מערך שימושיות לפענוח ABI של נתונים משניים המועברים בפרמטרים של פונקציה:

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.8.5 <0.9.0;
    contract Proxy {
        /// @dev Address of the client contract managed by proxy i.e., this contract
        address client;

        constructor(address client_) {
            client = client_;
        }

        /// Forward call to "setOwner(address)" that is implemented by client
        /// after doing basic validation on the address argument.
        function forward(bytes calldata payload) external {
            bytes4 sig = bytes4(payload[:4]);
            // Due to truncating behavior, bytes4(payload) performs identically.
            // bytes4 sig = bytes4(payload);
            if (sig == bytes4(keccak256("setOwner(address)"))) {
                address owner = abi.decode(payload[4:], (address));
                require(owner != address(0), "Address of owner cannot be zero.");
            }
            (bool status,) = client.delegatecall(payload);
            require(status, "Forwarded call failed.");
        }
    }



.. index:: ! struct, ! type;struct

.. _structs:

Structs
-------

סולידיטי מספקת דרך להגדיר טיפוסים חדשים בצורה של structs, כפי
שמוצג בדוגמה הבאה:

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.6.0 <0.9.0;

    // Defines a new type with two fields.
    // Declaring a struct outside of a contract allows
    // it to be shared by multiple contracts.
    // Here, this is not really needed.
    struct Funder {
        address addr;
        uint amount;
    }

    contract CrowdFunding {
        // Structs can also be defined inside contracts, which makes them
        // visible only there and in derived contracts.
        struct Campaign {
            address payable beneficiary;
            uint fundingGoal;
            uint numFunders;
            uint amount;
            mapping(uint => Funder) funders;
        }

        uint numCampaigns;
        mapping(uint => Campaign) campaigns;

        function newCampaign(address payable beneficiary, uint goal) public returns (uint campaignID) {
            campaignID = numCampaigns++; // campaignID is return variable
            // We cannot use "campaigns[campaignID] = Campaign(beneficiary, goal, 0, 0)"
            // because the right hand side creates a memory-struct "Campaign" that contains a mapping.
            Campaign storage c = campaigns[campaignID];
            c.beneficiary = beneficiary;
            c.fundingGoal = goal;
        }

        function contribute(uint campaignID) public payable {
            Campaign storage c = campaigns[campaignID];
            // Creates a new temporary memory struct, initialised with the given values
            // and copies it over to storage.
            // Note that you can also use Funder(msg.sender, msg.value) to initialise.
            c.funders[c.numFunders++] = Funder({addr: msg.sender, amount: msg.value});
            c.amount += msg.value;
        }

        function checkGoalReached(uint campaignID) public returns (bool reached) {
            Campaign storage c = campaigns[campaignID];
            if (c.amount < c.fundingGoal)
                return false;
            uint amount = c.amount;
            c.amount = 0;
            c.beneficiary.transfer(amount);
            return true;
        }
    }

החוזה אינו מספק את הפונקציונליות המלאה של חוזה מימון המונים,
אבל הוא מכיל את המושגים הבסיסיים הדרושים להבנת structs.
ניתן להשתמש בסוגי structs בתוך מיפויים ומערכים והם יכולים בעצמם
להכיל מיפויים ומערכים.

לא ייתכן ש-struct יכיל איבר מהסוג שלו,
למרות שה-struct עצמו יכול להיות סוג הערך של איבר מיפוי
או שהוא יכול להכיל מערך בגודל דינאמי מסוגו.
הגבלה זו הכרחית, מכיוון שגודל ה-struct חייב להיות סופי.

שימו לב כיצד בכל הפונקציות, סוג struct מוקצה למשתנה מקומי
עם מיקום נתונים ``storage``.
כתוצאה מכך, בהשמה למשתנה מקומי ה-struct לא מועתק אלא רק מאוחסנת הפניה ל-struct
במשתנה המקומי, וכך השמה למשתנה המקומי כותבת למעשה למשתנה מצב (state).

כמובן, אתם יכולים גם לגשת ישירות לחלקי ה-struct בלי
השמה למשתנה מקומי, כמו ב-``campaigs[campaignID].amount = 0``.

.. note::
    עד גרסת סולידיטי  0.7.0, memory struct המכילים מרכיבים מסוג storage בלבד (למשל מיפויים)
    היו מותרים ופקודה כמו ``campaigns[campaignID] = Campaign(beneficiary, goal, 0, 0)``
    בדוגמה למעלה תעבוד ופשוט תדלג בשקט על מרכיבים אלו.
