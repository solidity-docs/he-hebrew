.. index:: ! value type, ! type;value
.. _value-types:

סוגי ערך (Value Types)
=======================

סוגי המשתנים להלן נקראים סוגי ערך מכיוון שהמשתנים שלהם תמיד יועברו לפי ערך, כלומר הם תמיד מועתקים כאשר הם משמשים כארגומנטים של פונקציה או בהשמות.

.. index:: ! bool, ! true, ! false

בולאנים
--------

``bool``: הערכים האפשריים הם הקבועים ``true`` ו-``false``.

אופרטורים:

*  ``!`` (שלילה לוגית)
*  ``&&`` (צירוף לוגי, "and")
*  ``||`` (ניתוק לוגי, "or")
*  ``==`` (שוויון)
*  ``!=`` (אי-שוויון)

האופרטורים ``||`` ו-``&&`` מיישמים את כללי ה"קיצור" הנפוצים. זאת אומרת שבביטוי ``(f(x) || g(y``, אם ``(f(x`` מוערך כ-``true``, אז ``(g(x`` לא יוערך גם אם עשויות להיות לכך תופעות לוואי.

.. index:: ! uint, ! int, ! integer
.. _integers:

מספרים שלמים
--------------

``int`` / ``uint``: מספרים שלמים עם או בלי סימן בגדלים שונים. מילות המפתח ``uint8`` עד ``uint256`` בקפיצות של ``8`` (ללא סימן מ-8 עד 256 ביטים) ו-``int8`` עד ``int256``. בנוסף, ``uint`` ו-``int`` הם כינויים עבור ``uint256`` ו-``int256``, בהתאמה.

אופרטורים:

* השוואות: ``<=``, ``<``, ``==``, ``!=``, ``>=``, ``>`` (התוצאה היא ``bool``)
* אופרטורים בביטים: ``&``, ``|``, ``^`` (exclusive-or של ביטים), ``~`` (הערך ההופכי של הביטים)
* אופרטןרי הזזה: ``<<`` (הזזה לשמאל), ``>>`` (הזזה לימין)
* אופרטורים אריתמטיים: ``+``, ``-``, ``-`` אונארי (רק למספרים שלמים עם סימן), ``*``, ``/``, ``%`` (מודולו), ``**`` (העלאה בחזקה)

עבור סוג מספר שלם ``X``, אפשר להשתמש ב-``type(X).min`` ו-``type(X).max`` כדי
לגשת לערך המינימלי והמקסימלי של הסוג.

.. warning::

   מספרים שלמים בסולידיטי מוגבלים לטווח מסוים. לדוגמה, עם ``uint32``, הטווח הוא``0`` עד ``1 - 32**2``.
   ישנם שני מצבים שבהם מתבצע חשבון על סוגים אלה: מצב ה-"wrapping" או "unchecked"  (ללא בדיקה) ומצב ה-"checked" (עם בדיקה).
   כברירת מחדל, אריתמטיקה תמיד נבדקת - "checked", כלומר אם התוצאה של פעולה נופלת מחוץ לטווח הערכים
   של הסוג, מתבצע revert לקריאה המוחזרת באמצעות :ref:`failing assertion<assert-and-require>`.
   אתם יכולים לעבור למצב "unchecked"  באמצעות ``{ ... } unchecked``.
   פרטים נוספים ניתן למצוא בסעיף אודות :ref:`unchecked <unchecked>`.



השוואות
^^^^^^^^^^^

הערך של השוואה מתקבל מהשוואת המספרים.

פעולות בביטים
^^^^^^^^^^^^^^

פעולות בביטים מבוצעות על-ידי ייצוג המשלים ל-2 של המספר.
המשמעות היא, למשל:

.. code-block:: solidity

    ~int256(0) == int256(-1)

הזזות
^^^^^^

סוג התוצאה של פעולת הזזה הוא הסוג של האופרנד השמאלי, תוך קיטום התוצאה כך שתתאים לסוג.
האופרנד הימני חייב להיות מסוג ללא סימן. ניסיון להזיז לפי סוג עם סימן יגרום לשגיאת קומפילציה.

ניתן "לדמות" הזזות באמצעות כפל בחזקות של שתיים באופן הבא. שימו לב שהקיטום
לסוג האופרנד השמאלי מבוצע תמיד בסוף, אך לא מוזכר במפורש.

- ``x << y`` מוערך כביטוי האריתמטי ``x * 2**y``.
- ``x >> y`` מוערך כביטוי האריתמטי ``x / 2**y``, מעוגל בכיוון של האינסוף השלילי.

.. warning::
    לפני גרסה ``0.5.0`` הזזה לימין ``x >> y`` עם ``x`` שלילי היתה מוערכת 
    כביטוי המתמטי ``x / 2**y`` מעוגל לכיוון אפס,
    למשל, שימוש בהזזה ימינה עם עיגול כלפי מעלה (לכיוון אפס) במקום עיגול כלפי מטה (לכיוון אינסוף שלילי).

.. note::
    בדיקות גלישה לעולם אינן מבוצעות עבור פעולות הזזה כפי שהן נעשות עבור פעולות אריתמטיות.
    במקום זאת, התוצאה תמיד מקוצצת.

חיבור, חיסור וכפל
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

לחיבור, חיסור וכפל יש את הסמנטיקה הרגילה, עם שני
מצבים שונים לגבי גלישה (overflow) וגלישה-מלמטה (underflow):

כברירת מחדל, בכל חישוב אריתמטי נבדק אם התרחשה גלישה, אך ניתן לבטל זאת
על-ידי שימוש בבלוק :ref:`unchecked<unchecked>`, וכתוצאה מכך תתבצע
אריתמטיקה מעגלית (wrapping arithmetic). פרטים נוספים
ניתן למצוא בסעיף הזה.

הביטוי ``x-`` שווה ערך ל``(T(0) - x)`` שבו
``T`` הוא הסוג של ``x``. ניתן להחיל אותו רק על סוגים בעלי סימן.
הערך של ``x-`` יכול להיות
חיובי אם ``x`` הוא שלילי. קיימת גם בעיה שצריך להיזהר ממנה בגלל
הייצוג המשלים ל-2:

אם השתמשתם בקוד ``;int x = type(int).min``, אז ``x-`` אינו מתאים לטווח החיובי.
פירוש הדבר הוא ש-``unchecked { assert(-x == x); }`` עובד, והביטוי ``x-``
במצב checked גורם ל-assert להכשל.

חלוקה
^^^^^^^^

מכיוון שסוג התוצאה של פעולה הוא תמיד סוג של אחד
האופרנדים, תוצאת חלוקה של מספרים שלמים היא תמיד מספר שלם.
בסולידיטי, עיגול של חלוקה מתבצע לכיוון האפס. משמעות הדבר היא
ש-``int256(-5) / int256(2) == int256(-2)``.

שימו לב שבניגוד לכך, חלוקה ב-:ref:`ליטרלים רציונליים<rational_literals>` גורמת לערכים עם דיוק שרירותי.

.. note::
  חלוקה באפס גורמת ל- :ref:`Panic error<assert-and-require>`. אין אפשרות לבטל בדיקה זו על-ידי ``{ ... } unchecked``.

.. note::
  הביטוי ``type(int).min / (-1)`` הוא המקרה היחיד שבו החלוקה גורמת לגלישה.
  במצב אריתמטי עם בדיקה (checked), דבר זה יגרום ל-assertion כושל, בעוד
  שבמצב של אריתמטיקה מעגלית (wrapping), הערך יהיה ``type(int).min``.

מודולו
^^^^^^

פעולת המודולו ``a % n`` מניבה שארית ``r`` אחרי החלוקה של האופרנד ``a``
באופרנד ``n``, כאשר ``q = int(a / n)`` ו-``r = a - (n * q)``. משתמע מכך שתוצאת מודולו
היא עם אותו סימן של האופרנד השמאלי (או אפס) ו-``a % n == -(-a % n)`` מתקיים לערכים שליליים של ``a``:

* ``int256(5) % int256(2) == int256(1)``
* ``int256(5) % int256(-2) == int256(1)``
* ``int256(-5) % int256(2) == int256(-1)``
* ``int256(-5) % int256(-2) == int256(-1)``

.. note::
  מודולו אפס גורם ל-:ref:`Panic error<assert-and-require>`. אין אפשרות לבטל בדיקה זו על-ידי ``{ ... } unchecked``.

העלאה בחזקה
^^^^^^^^^^^^^^

העלאה בחזקה אפשרית רק עבור סוגים ללא סימן במעריך. הסוג המתקבל
של העלאה בחזקה שווה תמיד לסוג הבסיס. בבקשה תדאגו לכך שהמשתנה יהיה
גדול מספיק כדי להחזיק את התוצאה והתכוננו לכשלים פוטנציאליים ב-assertion
או בהתנהגות האריתמטיקה המעגלית.

.. note::
  במצב checked, ההעלאה בחזקה משתמשת רק באופקוד ``exp`` הזול יחסית עבור בסיסים קטנים.
  במקרים כמו ``x**3``, הביטוי ``x*x*x`` עשוי להיות זול יותר.
  בכל מקרה, מומלץ לבצע בדיקות עלות גז ושימוש באופטימייזר.

.. note::
  שימו לב ש-``0**0`` מוגדר על-ידי ה-EVM כ-``1``.

.. index:: ! ufixed, ! fixed, ! fixed point number

מספרי נקודה קבועה
-------------------

.. warning::
    מספרי נקודה קבועה עדיין לא נתמכים במלואם על ידי סולידיטי. אפשר להגדיר אותם, אבל
    לא ניתן להשים אליהם או מהם.

``fixed`` / ``ufixed``: מספרי נקודה קבועה עם או בלי סימן בגדלים שונים. מילות המפתח ``ufixedMxN`` ו-``fixedMxN``, כאשר ``M`` מייצג את מספר הביטים שבהם הסוג משתמש
ו-``N`` מייצג את מספר הנקודות העשרוניות הזמינות. ``M`` חייב להיות ניתן לחלוקה ב-8 -
בין 8 ל-256 ביטים. ``N`` חייב להיות בין 0 ל-80, כולל.
``ufixed`` ו-``fixed`` הם כינויים עבור ``ufixed128x18`` ו-``fixed128x18``, בהתאמה.

אופרטורים:

* השוואות: ``<=``, ``<``, ``==``, ``!=``, ``>=``, ``>`` (התוצאה היא מסוג ``bool``)
* אופרטורים אריתמטיים: ``+``, ``-``, unary ``-``, ``*``, ``/``, ``%`` (מודולו)

.. note::
    ההבדל העיקרי בין נקודה צפה (``float`` ו-``double`` בשפות תכנות רבות, ליתר דיוק מספרי IEEE 754) למספרי נקודה קבועה הוא
    שמספר הביטים המשמשים למספר השלם ולחלק של ההשבר (החלק שאחרי הנקודה העשרונית) גמיש בראשון, בעוד שהוא קבוע
    באחרון. בדרך כלל, בנקודה צפה כמעט כל הביטים משמשים לייצוג המספר, בעוד שרק מספר קטן של ביטים מגדירים
    את מיקום הנקודה העשרונית.

.. index:: address, balance, send, call, delegatecall, staticcall, transfer

.. _address:

כתובת
-------

הסוג כתובת מגיע בשני טעמים זהים במידה רבה:

- ``address``: מכיל ערך ב-20 בתים (גודל של כתובת איתריום).
- ``address payable``: כמו ``address``, אבל עם מרכיבים נוספים ``transfer`` ו-``send``.

הרעיון מאחורי ההבחנה הזו הוא ש-``address payable`` (כתובת לתשלום) היא כתובת שאליה ניתן לשלוח איתר,
בזמן שאתם לא אמורים לשלוח איתר לכתובת רגילה, למשל מכיוון שזה עשוי להיות חוזה חכם
שלא נבנה לקבל את האיתר.

המרה בין סוגים:

המרות פנימיות מ-``address payable`` ל-``address`` מותרות, כאשר המרה מ-``address`` ל-``address payable``
חייבת להיות חיצונית על-ידי ``payable(<address>)``.

המרות חיצוניות מ\\אל ``address`` מותרות ל-``uint160``, ליטרלים של מספרים שלמים,
``bytes20`` וסוגי חוזה.

רק ביטויים מסוג ``address`` וסוגי חוזה יכולים להיות מומרים ל-``address
payable`` על-ידי המרות חיצוניות ``(...)payable``. בשביל סוג של חוזה, המרה זו מותרת
רק אם החוזה יכול לקבל איתר, זאת אומרת שלחוזה יש פונקציית :ref:`receive
<receive-ether-function>` או פונקציית payable fallback. שימו לב ש-``payable(0)`` הוא חוקי
והוא יוצא מן הכלל לכלל הזה.

.. note::
    אם אתם צריכים משתנה מסוג ``address`` ומתכוונים לשלוח אליו איתר, צריך
    להגדיר את הסוג כ-``address payable`` כדי להפוך את הדרישה הזו לגלויה.
    נסו גם לעשות את ההבחנה או ההמרה הזו מוקדם ככל האפשר.

    ההבחנה בין ``address`` ו-``address payable`` הוצגה עם גרסה 0.5.0.
    כמו כן, החל מגרסה זו, חוזים אינם ניתנים להמרה באופן פנימי לסוג ``address``, אך עדיין ניתן להמיר אותם במפורש
    ל-``address`` או ל-``address payable``, אם יש להם פונקציית receive או פונקציית payable fallback.


אופרטורים:

* ``<=``, ``<``, ``==``, ``!=``, ``>=`` ו-``>``

.. warning::
    אם אתם ממירים סוג שמשתמש בגודל בתים גדול יותר מ-``address``, למשל ``bytes32``, אז ה-``address`` נקטם.
    כדי להפחית את עמימות ההמרה, החל מגרסה 0.4.24, הקומפיילר יאלץ אותכם להפוך את הקיטוע למפורש בהמרה.
    קחו לדוגמה את הערך של 32 בתים ``0x1111222223333444455556666777788889999AAAABBBBCCCCDDDDEEEEFFFFCCCC``.

    אתם יכולים להשתמש ב-``address(uint160(bytes20(b)))``,  עם התוצאה ``0x111122223333444455556666777788889999aAaa``,
    או אתם יכולים להשתמש ב-``address(uint160(uint256(b)))``, עם התוצאה ``0x777788889999AaAAbBbbCcccddDdeeeEfFFfCcCc``.

.. note::
    מספרים הקסדצימליים מעורבים באותיות גדולות וקטנות התואמים ל-`EIP-55 <https://github.com/ethereum/EIPs/blob/master/EIPS/eip-55.md>`_ מטופלים באופן אוטומטי כליטרלים מסוג ``address``. ראו :ref:`ליטרלים של כתובת<address_literals>`.

.. _members-of-addresses:

מרכיבים של כתובות
^^^^^^^^^^^^^^^^^^^^

לעיון מהיר בכל מרכיבי הכתובת, ראו :ref:`address_related`.

* ``balance`` ו-``transfer``

אפשר לבדוק מה היתרה של כתובת באמצעות המאפיין ``balance``
ולשלוח את איתר (ביחידות של wei) לכתובת payable באמצעות פונקציית ``transfer``:

.. code-block:: solidity
    :force:

    address payable x = payable(0x123);
    address myAddress = address(this);
    if (x.balance < 10 && myAddress.balance >= 10) x.transfer(10);

פונקציית ``transfer`` נכשלת אם יתרת החוזה הנוכחית אינה גדולה מספיק
או אם העברת האיתר נדחתה על ידי החשבון המקבל. בכישלון, פונקציית ``transfer``
מבצעת revert.

.. note::
    אם ``x`` הוא כתובת חוזה, הקוד שלו (ליתר דיוק: :ref:`receive-ether-function` שלו, אם קיימת, או אחרת :ref:`fallback-function` שלו, אם קיימת) יתבצע יחד עם הקריאה ל-``transfer`` (זוהי תכונה של ה-EVM ולא ניתן למנוע אותה). אם  הביצוע נכשל בכל דרך או שנגמר לו הגז,
    העברת האיתר תבוטל (יתבצע revert) והחוזה הנוכחי יעצר עם חריגה.

* ``send``

``send`` היא המקבילה ברמה נמוכה של ``transfer``. אם הביצוע נכשל, החוזה הנוכחי לא יעצר עם חריגה, אבל ``send`` תחחזיר ``false``.

.. warning::
    ישנן כמה סכנות בשימוש ב-``send``: ההעברה נכשלת אם עומק מחסנית הקריאה הוא 1024
    (הקורא תמיד יכול לאלץ מצב זה) וההעברה תכשל גם אם לנמען נגמר הגז. לכן כדי
    לבצע העברות איתר בטוחות, תמיד צריך לבדוק את ערך ההחזרה של ``send``, השתמשו ב-``transfer`` או אפילו טוב יותר:
    השתמשו בתבנית שבה הנמען מושך את האיתר.

* ``call``, ``delegatecall`` ו-``staticcall``

על מנת להתממשק עם חוזים שאינם עומדים ב-ABI,
או לקבל שליטה ישירה יותר על הקידוד,
קיימות הפונקציות ``call``, ``delegatecall`` ו-``staticcall``.
כולן מקבלות פרמטר אחד של ``bytes memory``
ומחזירות את תנאי ההצלחה (כ-``bool``) ואת הנתונים המוחזרים
(``bytes memory``).
הפונקציות ``abi.encode``, ``abi.encodePacked``, ``abi.encodeWithSelector``
ו-``abi.encodeWithSignature`` יכולות לשמש לקידוד נתונים מובנים (structured data).

דוגמה:

.. code-block:: solidity

    bytes memory payload = abi.encodeWithSignature("register(string)", "MyName");
    (bool success, bytes memory returnData) = address(nameReg).call(payload);
    require(success);

.. warning::
    כל הפונקציות הללו הן פונקציות ברמה נמוכה ויש להשתמש בהן בזהירות.
    באופן ספציפי, כל חוזה לא ידוע עלול להיות זדוני ואם תקראו לו, אתם
    מעבירים את השליטה לחוזה הזה שיכול בתורו להתקשר בחזרה
    לחוזה שלכם. לכן צריך להיות מוכנים לשינויים במשתני מצב
    כאשר הקריאה חוזרת. הדרך הרגילה לאינטראקציה עם חוזים אחרים
    היא לקרוא לפונקציה על אובייקט חוזה (``()x.f``).

.. note::
    גרסאות קודמות של סולידיטי אפשרו לפונקציות אלו לקבל
    ארגומנטים שרירותיים וגם טיפלו בארגומנט ראשון מסוג
    ``bytes4`` אחרת. מקרי קצה אלו הוסרו בגרסה 0.5.0.

אפשר לכוונן את הגז המסופק עם המשנה ``gas``:

.. code-block:: solidity

    address(nameReg).call{gas: 1000000}(abi.encodeWithSignature("register(string)", "MyName"));

באופן דומה, ניתן לשלוט גם על ערך האיתר שיסופק:

.. code-block:: solidity

    address(nameReg).call{value: 1 ether}(abi.encodeWithSignature("register(string)", "MyName"));

לבסוף, ניתן לשלב את המשנים האלו. כאשר הסדר שלהם לא משנה:

.. code-block:: solidity

    address(nameReg).call{gas: 1000000, value: 1 ether}(abi.encodeWithSignature("register(string)", "MyName"));

באופן דומה, ניתן להשתמש בפונקציה ``delegacall``: ההבדל הוא שרק הקוד של הכתובת הנתונה יהיה בשימוש. כל שאר ההיבטים (אחסון, יתרה,...) נלקחים מהחוזה הנוכחי. מטרת ``delegacall`` היא להשתמש בקוד ספרייה המאוחסן בחוזה אחר. על המשתמש לוודא שפריסת ה-storage בשני החוזים מתאימה לשימוש ב-delegatecall.

.. note::
    לפני גרסת homestead של הקומפיילר, רק גרסה מוגבלת בשם ``callcode`` הייתה זמינה  והיא לא סיפקה גישה לערכי ``msg.sender`` ו-``msg.value`` המקוריים. פונקציה זו הוסרה בגרסה 0.5.0.

מאז גרסת byzantium
של הקומפיילר ניתן להשתמש גם ב-``staticcall``. לקריאה זו פונקציונליות זהה ל-``call``, אבל היא תבצע revert אם הפונקציה שנקראה תשנה את משתני המצב בכל דרך שהיא.

כל שלוש הפונקציות ``call``, ``delegatecall`` ו-``staticcall`` הן פונקציות ברמה נמוכה מאוד ויש להשתמש בהן רק כמוצא אחרון מכיוון שהן פוגעות בבטיחות הסוג של סולידיטי.

האפשרות ``gas`` זמינה בכל שלוש השיטות, בעוד האפשרות ``value`` זמינה רק
ב-``call``.

.. note::
    עדיף להימנע מהסתמכות על ערכי גז מקודדים בקוד החוזה החכם שלכם,
    ללא קשר לשאלה אם משתני מצב נקראים או נכתבים בחוזה, מכיוון שיכולים להיות לכך הרבה מכשולים.
    כמו כן, הגישה לגז עשויה להשתנות בעתיד.

* ``code`` ו-``codehash``

אתם יכולים לתשאל את הקוד שנפרס עבור כל חוזה חכם. השתמשו ב-``code.`` כדי לקבל את ה-EVM bytecode בתור
``bytes memory``, שעשוי להיות ריק. השתמשו ב-``codehash.`` כדי לקבל את ה-hash Keccak-256 של הקוד הזה
(כמו ``bytes32``). שימו לב ש-``addr.codehash`` זול יותר משימוש ב-``keccak256(addr.code)``.

.. note::
    ניתן להמיר את כל החוזים לסוג ``address``, כך שניתן לבדוק את היתרה של
    החוזה הנוכחי באמצעות ``address(this).balance``.

.. index:: ! contract type, ! type; contract

.. _contract_types:

סוגי חוזה
-----------

כל :ref:`חוזה<contracts>` מגדיר סוג משלו.
אתם יכולים להמיר חוזים באופן פנימי לחוזים שהם יורשים מהם.
ניתן להמיר חוזים במפורש לסוג ``address`` וממנו.

המרה מפורשת מ\\אל מסוג ``address payable`` אפשרית רק
אם לסוג החוזה יש פונקציית receive או פונקציית payable fallback. ההמרה עדיין
מבוצעת באמצעות ``address(x)``. אם לסוג החוזה אין פונקציית receive או פונקציית payable fallback, ההמרה ל-``address payable`` יכולה להתבצע באמצעות
``payable(address(x))``.
תוכלו למצוא מידע נוסף בסעיף על
ה-:ref:`סוג כתובת<address>`.

.. note::
    לפני גרסה 0.5.0, חוזים נגזרו ישירות מסוג הכתובת
    ולא הייתה הבחנה בין ``address`` ו-``address payable``.

אם אתם מצהירים על משתנה מקומי מסוג חוזה (``MyContract c``), אתם יכולים לקרוא
לפונקציות בחוזה זה. יש לדאוג לתת ערך למשתנה ממקום שהוא
אותו סוג חוזה.

אתם גם יכולים ליצור חוזים (מה שאומר שהם נוצרו לאחרונה). אתם
יכוליםל מצוא פרטים נוספים בסעיף :ref:`'Contracts via new'<creating-contracts>`.

ייצוג הנתונים של חוזה זהה לזה של סוג ``address``
וסוג זה משמש גם ב-:ref:`ABI<ABI>`.

חוזים אינם תומכים באף אופרטור.

המרכיבים בסוגי חוזה הן הפונקציות החיצוניות של החוזה,
כולל כל משתני המצב המסומנים כ-``public``.

עבור חוזה ``C`` אפשר להשתמש ב-``type(C)`` כדי לגשת
ל-:ref:`אינפורמציה על סוג<meta-type>` בחוזה.

.. index:: byte array, bytes32

מערכי byte בגודל קבוע
----------------------

סוגי הערך ``bytes1``, ``bytes2``, ``bytes3``, ..., ``bytes32``
מכילים רצף של אחד עד 32 בתים.

אופרטורים:

* השוואות: ``<=``, ``<``, ``==``, ``!=``, ``>=``, ``>`` (התוצאה היא ``bool``)
* פעולות בביטים: ``&``, ``|``, ``^`` (bitwise exclusive or בביטים), ``~`` (שלילת ביטים)
* פעולות הזזה: ``<<`` (הזזה לשמאל), ``>>`` (הזזה לימין)
* גישה לאינדקס: אם ``x`` הוא מסוג ``bytesI``, אז ``x[k]`` בשביל ``0 <= k < I`` מחזיר את הבית ה-``k`` (קריאה בלבד).

אופרטור ההזזה עובד עם סוג מספר שלם ללא סימן כאופרנד ימני (אבל
מחזיר את סוג האופרנד השמאלי), המציין את מספר הביטים שיש להזיז.
הזזה לפי סוג עם סימן תיצור שגיאת קומפילציה..

מרכיבים:

* ``length.`` האורך הקבוע של מערך הבתים (לקריאה בלבד).

.. note::
    הסוג ``[]bytes1`` הוא מערך של בתים, אך עקב כללי ריפוד, הוא מבזבז
    31 בתים לכל אלמנט (למעט ב-storage). עדיף להשתמש בסוג ``bytes``
    במקום זאת.

.. note::
    לפני גרסה 0.8.0, ``byte`` היה כינוי עבור ``bytes1``.

מערכי byte בגודל דינאמי
----------------------------

``bytes``:
    מערכי byte בגודל דינאמי, ראו :ref:`arrays`. לא סוג-ערך!
``string``:
    מחרוזות UTF-8-encoded בגודל דינאמי, ראו :ref:`arrays`. לא סוג-ערך!

.. index:: address, ! literal;address

.. _address_literals:

ליטרלים של כתובת
----------------

מילים הקסדצימליות שעוברות את מבחן בדיקת ה-checksum של כתובות, למשל
``0xdCad3a6d3569DF655070DEd06cb7A1b2Ccd1D3AF``, הם מסוג ``address``.
מילים הקסדצימליות באורך שהוא בין 39 ל-41 ספרות
שלא עוברות את הבדיקה
גורמות לשגיאה. אפשר להוסיף בהתחלה (עבור סוגי מספרים שלמים) או להוסיף בסוף (עבור סוגי bytesNN) אפסים כדי לבטל את השגיאה.

.. note::
    הפורמט של checksum לכתובת עם אותיות גדולות וקטנות מוגדר ב-`EIP-55 <https://github.com/ethereum/EIPs/blob/master/EIPS/eip-55.md>`_.

.. index:: integer, rational number, ! literal;rational

.. _rational_literals:

ליטרלים של מספרים רציונאליים ושלמים
--------------------------------------

ליטרלים של מספרים שלמים נוצרים מרצף של ספרות בטווח 0-9,
שמתפרשות כעשרוניות. לדוגמה, ``69`` פירושו שישים ותשע.
ליטרים אוקטליים לא קיימים בסולידיטי ואפסים מובילים אינם חוקיים.

ליטרלים של שבר עשרוני נוצרים על ידי ``.`` עם ספרה אחת לפחות אחרי הנקודה העשרונית.
דוגמאות כוללות ``1.`` ו-``1.3`` (אבל לא ``.1``).

סימון מדעי בצורה של ``2e10`` נתמך גם כן, כאשר
המנטיסה יכולה להיות חלקית אבל המעריך חייב להיות מספר שלם.
הליטרל``MeE`` שווה ערך ל-``M * 10**E``.
דוגמאות: ``2e10``, ``-2e10``, ``2e-10``, ``2.5e1``.

ניתן להשתמש בקו תחתון כדי להפריד בין הספרות של ליטרלים מספריים כדי לסייע בקריאה.
לדוגמה, כל הבטויים להלן תקפים: עשרוני ``123_000``, הקסדצימלי ``0x2eff_abde``, סימון עשרוני מדעי ``1_2e345_678``.
ניתן להשתמש בקו תחתון רק בין שתי ספרות ומותר רק קו תחתון אחד ברצף.
לא מתווספת משמעות סמנטית נוספת למספר ליטרלי המכיל קווים תחתונים,
מעשית מתעלמים מהקוים התחתונים.

ביטויים ליטרליים של מספרים שומרים על דיוק שרירותי עד שהם מומרים לסוג לא ליטרלי (כלומר על ידי
שימוש בהם יחד עם כל דבר אחר מלבד ביטויי ליטרל מספרי  (כמו ליטרלים בוליאנים) או על ידי המרה מפורשת).
המשמעות היא שהחישובים אינם גולשים והחלוקות אינן נקטמות
בביטויים ליטרליים מספריים.

לדוגמה, תוצאת ``(2**800 + 1) - 2**800`` הוא הקבוע ``1`` (מהסוג ``uint8``),
למרות שתוצאות הביניים אפילו לא יתאימו לגודל מילת המכונה. יתר על כן, תוצאת ``5. * 8``
היא המספר השלם ``4`` (למרות שלא נעשה שימוש במספרים שלמים ביניהם).

.. warning::
    בעוד שרוב האופרטורים מייצרים ביטוי ליטרלי כשהם מיושמים על ליטרלים, ישנם אופרטורים מסוימים שאינם עוקבים אחר הדפוס הזה:

    - אופרטור טרנרי (``... : ... ? ...``),
    - מציין במערך (``array>[<index>]>``).

    אתם יכולים לצפות שבטויים כמו ``(true ? 1 : 0) + 255`` או ``[0][1, 2, 3] + 255`` יהיו זהים לשמוש בליטרל 256
    באופן ישיר, אבל למעשה הם מחושבים בסוג ``uint8`` ויכולים לצור גלישה.

כל אופרטור שניתן להחיל על מספרים שלמים יכול להיות מיושם גם על ביטויים ליטרליים של מספרים
כל עוד האופרנדים הם מספרים שלמים. אם אחד מהשניים הוא שבר, פעולות על ביטים אינן מותרות
וההעלאה בחזקה אסורה אם המעריך הוא שבר (כי התוצאה עלולה להיות מספר לא רציונלי).

הזזות והעלאה בחזקה עם מספרים ליטרליים כסוגי אופרנד שמאלי (או בסיס) ומספרים שלמים
כמו האופרנד הימני (מעריך) מבוצעים תמיד
בסוג ``uint256`` (עבור ליטרלים לא שליליים) או ``int256`` (עבור ליטרלים שליליים),
ללא קשר לסוג האופרנד הימני (מעריך).

.. warning::
    חלוקה של ליטרלים שלמים היתה קוטמת בסולידיטי לפני גרסה 0.4.0, אך כעת היא הופכת למספר רציונלי, כלומר, ``5 / 2`` אינו שווה ל-``2``, אלא ל-``2.5``.

.. note::
    לסולידיטי יש סוג ליטרלי של מספר לכל מספר רציונלי.
    ליטרלים שלמים וליטרלים של מספר רציונלי שייכים לסוגים ליטרליים של מספרים.
    יתר על כן, כל הביטויים הליטרליים של המספרים (כלומר הביטויים
    שמכילים רק מספר ליטרלי ואופרטורים) שייכים לסוגי מספר ליטרלי.
    לכן הביטויים הליטרליים של המספר ``1 + 2`` ו-``2 + 1`` שניהם
    שייכים לאותו סוג ליטרלי של מספר עבור המספר הרציונלי שלוש.

.. note::
    ביטויים ליטרלייים של מספרים מומרים לסוג לא ליטרלי מיד כאשר הם משמשים עם ביטויים לא ליטרליים.
    בהתעלם מהסוגים, הערך של הביטוי שהוקצה ל-``b``
    להלן מוערך למספר שלם. מכיוון ש-``a`` הוא מסוג ``uint128``,
    הביטוי ``2.5 + a`` חייב להיות בעל סוג מתאים. מכיוון שאין סוג משותף
    עבור הסוגים של ``2.5`` ו-``uint128``, הקומפיילר של סולידיטי לא מקבל
    את הקוד הזה.

.. code-block:: solidity

    uint128 a = 1;
    uint128 b = 2.5 + a + 0.5;

.. index:: ! literal;string, string
.. _string_literals:

סוגים וליטרלים של מחרוזות
-------------------------

ליטרלים של מחרוזות נכתבים עם מרכאות כפולות או בודדות (``"foo"`` או ``'bar'``), וניתן גם לפצל אותם למספר חלקים עוקבים (``"foo" "bar"`` שווה ל-``"foobar"``), דבר שיכול להועיל כאשר מתמודדים עם מחרוזות ארוכות. לא נוסף אפס בסוף המחרוזת כמו ב-C; לכן, ``"foo"`` מייצג שלושה בתים, לא ארבעה כמו ב-C. כמו עם ליטרלים שלמים, הסוג שלהם יכול להשתנות, אבל הם ניתנים להמרה באופן פנימי ל-``bytes1``, ..., ``bytes32``, אם הם מתאימים, ל-``bytes`` ול-``string``.

לדוגמה, עם ``"bytes32 samevar = "stringliteral`` המחרוזת הליטרלית מתפרשת בצורת הבתים הגולמית שלה כאשר היא מיושמת לסוג ``bytes32``.

מחרוזת ליטרלים יכולה להכיל רק תווי ASCII להדפסה, כלומר התווים 0x20 .. 0x7E.

בנוסף, מחרוזות ליטרים תומכות גם בתווי escape הבאים:

- ``<newline>\`` (escape ל-newline אמיתי)
- ``\\`` (תו backslash)
- ``'\`` (גרש בודד)
- ``"\`` (מרכאה בודדת)
- ``n\`` (תו newline)
- ``r\`` (תו carriage return)
- ``t\`` (תו tab)
- ``xNN\`` (תו hex escape, ראו למטה)
- ``uNNNN\`` (תו unicode escape, ראו למטה)

``xNN\`` לוקח ערך hex ומכניס את הבית המתאים, בעוד ש-``uNNNN\`` לוקח Unicode codepoint ומכניס רצף UTF-8.

.. note::

    עד גרסה 0.8.0 היו שלושה רצפי escape נוספים: ``b\``, ``f\`` ו-``v\``.
    הם זמינים בדרך כלל בשפות אחרות אך לעתים נדירות נחוצים בפועל.
    אם אתם צריכים אותם, עדיין ניתן להכניס אותם באמצעות escapes הקסדצימליים, כלומר ``x08\``, ``x0c\``
    ו-``x0b\``, בהתאמה, בדיוק כמו כל תו ASCII אחר.

למחרוזת בדוגמה הבאה יש אורך של עשרה בתים.
היא מתחילה ב-newline byte, ואחריו מרכאה בודדת, גרש, תו נטוי אחורי ולאחר מכן (ללא הפרדה)
רצף התווים ``abcdef``.

.. code-block:: solidity
    :force:

    "\n\"\'\\abc\
    def"

כל מסיים שורה יוניקוד שאינו newline (כלומר LF, VF, FF, CR, NEL, LS, PS) נחשב
כמסיים מחרוזת ליטרלית. newline מסיים את המחרוזת הליטרלית רק אם אין לפניו ``\``.

.. index:: ! literal;unicode

ליטרלים של יוניקוד
---------------------

בעוד שמחרוזת ליטרלים רגילה יכולה להכיל רק ASCII, ליטרלי יוניקוד - עם קידומת מילת המפתח "unicode" - יכולים להכיל כל רצף UTF-8 חוקי.
הם גם תומכים באותם רצפי escape ממש כמו מחרוזות ליטרלים רגילות.

.. code-block:: solidity

    string memory a = unicode"Hello 😃";

.. index:: ! literal;hexadecimal, bytes

ליטרלים הקסדצימליים
--------------------

ליטרלים הקסדצימליים מופיעים עם קידומת של מילת המפתח ``hex`` והם מוקפים בגרשים
או במרכאות (``'hex"001122FF"``, ``hex'0011_22_FF``). התוכן שלהם חייב להיות
ספרות הקסדצימליות שיכולות להשתמש בקו תחתון בודד כמפריד ביניהן
בגבולות של בתים. הערך של הליטרל יהיה הייצוג הבינארי
של הרצף ההקסדצימלי.

מחרוזות ליטרלים הקסדצימליים מרובות המופרדות על-ידי רווח משורשרות למחרוזת ליטרלים אחת:
``hex"00112233" hex"44556677"`` שווה ערך ל-``hex"0011223344556677``.

במובנים מסוימים, ליטרלים הקסדצימליים מתנהגים כמו :ref:`string literals <string_literals>` אך אינם
ניתנים להמרה באופן פנימי לסוג ``string``.

.. index:: enum

.. _enums:

Enums
-----

Enums הם דרך אחת ליצור סוג מוגדר על ידי משתמש בסולידיטי. הם ניתנים להמרה במפורש
אל ומכל סוגי המספרים השלמים אך המרה פנימית אסורה. ההמרה המפורשת
ממספר שלם בודקת בזמן ריצה שהערך נמצא בטווח של ה-enum ואחרת גורמת
ל-:ref:`שגיאת פאניקה<assert-and-require>`.
Enums דורשים לפחות איבר אחד, וערך ברירת המחדל הוא האיבר הראשון.
ל-Enums אין יותר מ-256 איברים.

ייצוג הנתונים זהה ל-enums ב-C: האפשרויות מיוצגות על ידי
ערכי מספרים שלמים ללא סימן החל מ-``0``.

באמצעות ``type(NameOfEnum).min`` ו-``type(NameOfEnum).max`` אפשר לקבל את
הערך הקטן והגדול ביותר בהתאמה של ה-enum הנתון.


.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity ^0.8.8;

    contract test {
        enum ActionChoices { GoLeft, GoRight, GoStraight, SitStill }
        ActionChoices choice;
        ActionChoices constant defaultChoice = ActionChoices.GoStraight;

        function setGoStraight() public {
            choice = ActionChoices.GoStraight;
        }

        // Since enum types are not part of the ABI, the signature of "getChoice"
        // will automatically be changed to "getChoice() returns (uint8)"
        // for all matters external to Solidity.
        function getChoice() public view returns (ActionChoices) {
            return choice;
        }

        function getDefaultChoice() public pure returns (uint) {
            return uint(defaultChoice);
        }

        function getLargestValue() public pure returns (ActionChoices) {
            return type(ActionChoices).max;
        }

        function getSmallestValue() public pure returns (ActionChoices) {
            return type(ActionChoices).min;
        }
    }

.. note::
    ניתן להצהיר על Enums גם ברמת הקובץ, מחוץ להגדרות החוזה או הספרייה.

.. index:: ! user defined value type, custom type

.. _user-defined-value-types:

סוגי-ערך המוגדרים על-ידי המשתמש
-----------------------------------

סוג ערך שמוגדר על ידי משתמש מאפשר יצירת הפשטת עם עלות אפסית על פני סוג ערך אלמנטרי.
בדומה לכינוי, אבל עם דרישות סוג מחמירות יותר.

סוג ערך המוגדר על ידי משתמש מוגדר באמצעות ``type C is V``, כאשר ``C`` הוא השם של החדש
ו-``V`` הוא סוג הערך המובנה ("הסוג הבסיסי"). הפונקציה
``C.wrap`` משמשת להמרה מהסוג הבסיסי לסוג המשתמש. באופן דומה,
הפונקציה ``C.unwrap`` משמשת להמרה מסוג המשתמש לסוג הבסיסי.

לסוג ``C`` אין אופרטורים או פונקציות מצורפות. בפרט, אפילו
האופרטור ``==`` אינו מוגדר. המרות מפורשות ופנימיות לסוגים אחרים ומהם אסורות.

ייצוג הנתונים של ערכים מסוגים כאלה עובר בירושה מהסוג הבסיסי
והסוג הבסיסי משמש גם ב-ABI.

הדוגמה הבאה ממחישה סוג מותאם אישית ``UFixed256x18`` המייצג נקודה קבועה עשרונית;
סוג עם 18 ספרות עשרוניות וספרייה מינימלית לביצוע פעולות אריתמטיות על הסוג.


.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity ^0.8.8;

    // Represent a 18 decimal, 256 bit wide fixed point type using a user-defined value type.
    type UFixed256x18 is uint256;

    /// A minimal library to do fixed point operations on UFixed256x18.
    library FixedMath {
        uint constant multiplier = 10**18;

        /// Adds two UFixed256x18 numbers. Reverts on overflow, relying on checked
        /// arithmetic on uint256.
        function add(UFixed256x18 a, UFixed256x18 b) internal pure returns (UFixed256x18) {
            return UFixed256x18.wrap(UFixed256x18.unwrap(a) + UFixed256x18.unwrap(b));
        }
        /// Multiplies UFixed256x18 and uint256. Reverts on overflow, relying on checked
        /// arithmetic on uint256.
        function mul(UFixed256x18 a, uint256 b) internal pure returns (UFixed256x18) {
            return UFixed256x18.wrap(UFixed256x18.unwrap(a) * b);
        }
        /// Take the floor of a UFixed256x18 number.
        /// @return the largest integer that does not exceed `a`.
        function floor(UFixed256x18 a) internal pure returns (uint256) {
            return UFixed256x18.unwrap(a) / multiplier;
        }
        /// Turns a uint256 into a UFixed256x18 of the same value.
        /// Reverts if the integer is too large.
        function toUFixed256x18(uint256 a) internal pure returns (UFixed256x18) {
            return UFixed256x18.wrap(a * multiplier);
        }
    }

שימו לב איך ל- ``UFixed256x18.wrap`` ו-``FixedMath.toUFixed256x18`` יש אותה חתימה אבל
הן מבצעות פעולות שונות מאד: הפונקצייה ``UFixed256x18.wrap`` מחזירה ``UFixed256x18``
שיש לו ייצוג נתונים זהה לקלט, בעוד ש-``toUFixed256x18`` מחזירה
``UFixed256x18`` שיש לו אותו ערך נומרי.

.. index:: ! function type, ! type; function

.. _function_types:

סוגי פונקציות
--------------

סוגי פונקציות הם סוגי משתנה מסוג פונקציה. משתנים מסוג פונקציה
ניתן להקצות מפונקציות ופרמטרי פונקציה מסוג פונקציה
יכולים לשמש להעברת פונקציות והחזרת פונקציות מקריאות לפונקציות.
סוגי פונקציות מגיעים בשני טעמים - פונקציות *internal (פנימיות)* ו-*external (חיצוניות)*:

ניתן לקרוא לפונקציות פנימיות רק בתוך החוזה הנוכחי (ליתר דיוק,
בתוך יחידת הקוד הנוכחית, הכוללת גם פונקציות של ספרייה פנימית
ופונקציות שעברו בירושה) מכיוון שלא ניתן להפעיל אותן מחוץ
להקשר של החוזה הנוכחי. קריאה לפונקציה פנימית מתממשת
על ידי קפיצה לכניסה שלה, בדיוק כמו בעת קריאה פנימית לפונקציה של
החוזה הנוכחי.

פונקציות חיצוניות מורכבות מכתובת ומחתימת פונקציה ואפשר
להעביר או להחזיר אותן בקריאות לפונקציות חיצוניות.

סוגי הפונקציות מסומנים כדלקמן:

.. code-block:: solidity
    :force:

    function (<parameter types>) {internal|external} [pure|view|payable] [returns (<return types>)]

בניגוד לסוגי הפרמטרים, סוגי ההחזרה אינם יכולים להיות ריקים - אם
סוג הפונקציה לא אמור להחזיר כלום, יש להשמיט את כל
החלק של ``returns (<return types>)``.

כברירת מחדל, סוגי פונקציות הם פנימיים, כך שאפשר להשמיט
את מילת המפתח ``internal``.
שימו לב שדבר זה חל רק על סוגי פונקציות.
נראות יש לציין במפורש עבור פונקציות המוגדרות בחוזים, מכיוון
שאין להם ברירת מחדל.

המרות:

סוג פונקציה ``A`` ניתן להמרה באופן פנימי לסוג פונקציה ``B`` אם ורק אם
סוגי הפרמטרים שלהם זהים, סוגי ההחזרות זהים,
המאפיין הפנימי/חיצוני שלהן זהה והאפשרות לשנות את מצב של ``A``
מוגבלת יותר מהאפשרות לשנות את מצב ``B``. באופן ספציפי:

- פונקציה ``pure`` יכולה להיות מומרת לפונקציית ``view`` ו-``non-payable``
- פונקציה ``view`` יכולה להיות מומרת לפונקציית ``non-payable``
- פונקציה ``payable`` יכולה להיות מומרת לפונקציית ``non-payable``

אין אפשרויות אחרות להמרות בין סוגי פונקציות.

הכלל לגבי ``payable`` (קבלת תשלום אפשרית)
ו-``non-payable`` (אין אפשרות לקבל תשלום) עשוי להיות מעט
מבלבל, אבל בעצם, פונקציה שהיא ``payable``
מקבלת גם תשלום של אפס איתר, כך שהיא גם ``non-payable``.
מצד שני, פונקציית ``non-payable`` תדחה את האיתר שנשלח אליה,
כך שלא ניתן להמיר פונקציות ``non-payable`` לפונקציות ``payable``.
כדי להבהיר, דחיית האיתר מגבילה יותר מאשר אי דחיית האיתר.
לכן אפשר לעקוף פונקציה payable עם non-payable אך לא להפך.

בנוסף, כאשר מגדירים מצביע לפונקציה ``non-payable``,
הקומפיילר לא אוכף שהפונקציה המוצבעת תדחה את האיתר.
במקום זאת, הוא אוכף שמצביע הפונקציה לעולם אינו משמש לשליחת איתר.
דבר זה מאפשר השמה של מצביע פונקציה ``payable`` למצביע פונקציה ``non-payable``,
וכך מובטח ששני הסוגים מתנהגים באותו אופן, כלומר בשניהם לא ניתן להשתמש
לשלוח איתר.

אם משתנה מסוג function לא מאותחל, התוצאה של קריאה לפונקציה היא
:ref:`שגיאת פאניקה<assert-and-require>`. אותו דבר קורה אם קוראים לפונקציה
לאחר``delete`` שלה.

אם נעשה שימוש בסוגי פונקציות חיצוניות מחוץ להקשר של סולידיטי,
הם מטופלים כסוג ``function``, המקודד את הכתובת
ואחריו מזהה הפונקציה ביחד בסוג ``bytes24`` יחיד.

שימו לב שניתן להשתמש בפונקציות public של החוזה הנוכחי בתור
פונקציות internal או external. כדי להשתמש ב-``f`` כפונקציה internal ,
השתמשו רק ב-``f``. אם ברצונכם להשתמש בצורה ה-external שלה, השתמשו ב-``this.f``.

ניתן להקצות פונקציה מסוג internal למשתנה מסוג internal function ללא קשר
למקום שבו הוא מוגדר.
דבר זה כולל פונקציות private, internal ו-public  של חוזים וספריות, כמו גם
פונקציות חופשיות.
סוגי פונקציות external, לעומת זאת, תואמים רק לפונקציות של חוזים public ו-external.

.. note::
    פונקציות external עם פרמטרים של ``calldata`` אינן תואמות לסוגי פונקציות external עם פרמטרים ``calldata``.
    הן תואמות לסוגים המקבילים עם פרמטרי ``memory`` במקום זאת.
    לדוגמה, אין פונקציה שניתן להצביע עליה על ידי ערך
    מסוג ``function (string calldata) external`` בעוד
    ``function (string memory) external`` יכולה להצביע גם
    על ``{} function f(string memory) external`` וגם על
    ``{} function g(string calldata) external``.
    הסיבה לכך היא שלשני המיקומים הארגומנטים מועברים לפונקציה באותו אופן.
    הקורא לא יכול להעביר את ה-calldata שלה ישירות לפונקציה חיצונית,
    ותמיד מקודד את הארגומנטים לזיכרון על-ידי ABI.
    סימון הפרמטרים בתור ``calldata`` משפיע רק על יישום הפונקציה החיצונית והוא
    חסר משמעות למצביע פונקציה בצד הקורא.

ספריות אינן נכללות מכיוון שהן דורשות ``delegatecall`` ומשתמשות ב-:ref:`מוסכמות ABI אחרות
עבור הבוררים שלהן <library-selectors>`.
לפונקציות המוצהרות בממשקים אין הגדרות ולכן גם הצבעה עליהן אינה הגיונית.

מרכיבים:

לפונקציות external (או public) יש את המרכיבים הבאים:

* ``address.`` מחזירה את כתובת החוזה של הפונקציה.
* ``selector.`` מחזירה את ה-:ref:`בורר הפונקציות של ה-ABI <abi_function_selector>`.

.. note::
  פונקציות external (או public) נהגו לכלול את המרכיבים הנוספים
  ``gas(uint).`` ו-``value(uint).``. אלה הוצאו משימוש בסולידיטי  0.6.2
  והוסרה בסולידיטי  0.7.0. במקום זאת, השתמשו ב-``{... :gas}`` וב-``{... :value}``
  כדי לציין את כמות הגז או כמות ה-wei שנשלחתה לפונקציה,
  בהתאמה. ראו :ref:`קריאות לפונקציות חיצוניות <external-function-calls>` עבור
  מידע נוסף.

דוגמה שמראה כיצד להשתמש במרכיבים:

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.6.4 <0.9.0;

    contract Example {
        function f() public payable returns (bytes4) {
            assert(this.f.address == address(this));
            return this.f.selector;
        }

        function g() public {
            this.f{gas: 10, value: 800}();
        }
    }

דוגמה שמראה כיצד להשתמש בסוגי פונקציות פנימיות:

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.16 <0.9.0;

    library ArrayUtils {
        // internal functions can be used in internal library functions because
        // they will be part of the same code context
        function map(uint[] memory self, function (uint) pure returns (uint) f)
            internal
            pure
            returns (uint[] memory r)
        {
            r = new uint[](self.length);
            for (uint i = 0; i < self.length; i++) {
                r[i] = f(self[i]);
            }
        }

        function reduce(
            uint[] memory self,
            function (uint, uint) pure returns (uint) f
        )
            internal
            pure
            returns (uint r)
        {
            r = self[0];
            for (uint i = 1; i < self.length; i++) {
                r = f(r, self[i]);
            }
        }

        function range(uint length) internal pure returns (uint[] memory r) {
            r = new uint[](length);
            for (uint i = 0; i < r.length; i++) {
                r[i] = i;
            }
        }
    }


    contract Pyramid {
        using ArrayUtils for *;

        function pyramid(uint l) public pure returns (uint) {
            return ArrayUtils.range(l).map(square).reduce(sum);
        }

        function square(uint x) internal pure returns (uint) {
            return x * x;
        }

        function sum(uint x, uint y) internal pure returns (uint) {
            return x + y;
        }
    }

דוגמה נוספת המשתמשת בסוגי פונקציות חיצוניות:

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.22 <0.9.0;


    contract Oracle {
        struct Request {
            bytes data;
            function(uint) external callback;
        }

        Request[] private requests;
        event NewRequest(uint);

        function query(bytes memory data, function(uint) external callback) public {
            requests.push(Request(data, callback));
            emit NewRequest(requests.length - 1);
        }

        function reply(uint requestID, uint response) public {
            // Here goes the check that the reply comes from a trusted source
            requests[requestID].callback(response);
        }
    }


    contract OracleUser {
        Oracle constant private ORACLE_CONST = Oracle(address(0x00000000219ab540356cBB839Cbe05303d7705Fa)); // known contract
        uint private exchangeRate;

        function buySomething() public {
            ORACLE_CONST.query("USD", this.oracleResponse);
        }

        function oracleResponse(uint response) public {
            require(
                msg.sender == address(ORACLE_CONST),
                "Only oracle can call this."
            );
            exchangeRate = response;
        }
    }

.. note::
    פונקציות למבדה או inline מתוכננות אך עדיין אינן נתמכות.
