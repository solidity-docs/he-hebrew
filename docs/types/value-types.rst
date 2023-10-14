.. index:: ! value type, ! type;value
.. _value-types:

סוגי ערך (Value Types)
===========

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
--------

``int`` / ``uint``: מספרים שלמים עם או בלי סימן בגדלים שונים. מילות המפתח ``uint8`` עד ``uint256`` בקפיצות של ``8`` (ללא סימן מ-8 עד 256 ביטים) ו-``int8`` עד ``int256``. בנוסף, ``uint`` ו-``int`` הם כינויים עבור ``uint256`` ו-``int256``, בהתאמה.

אופרטורים:

* השוואה: ``<=``, ``<``, ``==``, ``!=``, ``>=``, ``>`` (התוצאה היא ``bool``)
* אופרטורים בביטים: ``&``, ``|``, ``^`` (exclusive-or של ביטים), ``~`` (הערך ההופכי של הביטים)
* אופרטןרי הזזה: ``<<`` (הזזה לשמאל), ``>>`` (הזזה לימין)
* אופרטורים אריתמטיים: ``+``, ``-``, ``-`` אונארי (רק למספרים שלמים אם סימן), ``*``, ``/``, ``%`` (צםדולו), ``**`` (העלאה בחזקה)

עבור סוג מספר שלם ``X``, אתם יכולים להשתמש ב-``type(X).min`` ו-``type(X).max`` כדי
לגשת לערך המינימלי והמקסימלי של הסוג.

.. warning::

   מספרים שלמים בסולידיטי מוגבלים לטווח מסוים. לדוגמה, עם ``uint32``, הטווח הוא``0`` עד ``1 - 32**2``.
   ישנם שני מצבים שבהם מתבצע חשבון על סוגים אלה: מצב ה-"wrapping" או "unchecked"  (לא נבדקת) ומצב ה-"checked" (נבדקת).
   כברירת מחדל, אריתמטיקה תמיד "checked", כלומר אם התוצאה של פעולה נופלת מחוץ לטווח הערכים
   של הסוג, מתבצע revert לקריאה מוחזרת באמצעות :ref:`failing assertion<assert-and-require>`.
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
האופרנד הימני חייב להיות מסוג ללא סימן, ניסיון להזיז לפי סוג עם סימן יגרום לשגיאת קומפילציה.

ניתן "לדמות" הזזות באמצעות הכפל בחזקות של שתיים באופן הבא. שימו לב שהקיטום
לסוג האופרנד השמאלי מבוצע תמיד בסוף, אך לא מוזכר במפורש.

- ``x << y`` מוערך כביטוי האריתמטי ``x * 2**y``.
- ``x >> y`` is מוערך כביטוי האריתמטי ``x / 2**y``, מעוגל בכיוון של האינסוף השלילי.

.. warning::
    לפני גרסה ``0.5.0`` הזזה לימין ``x >> y`` ל-``x`` שלילי היתה מוערכת 
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

שימו לב שבניגוד לכך, שחלוקה ב-:ref:`ליטרלים רציונליים<rational_literals>` גורמת לערכים עם דיוק שרירותי.

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
  בכל מקרה, מומלץ לבצע בדיקות עלות גז ושימוש באופטימיזר.

.. note::
  שימו לב ש-``0**0`` מוגדר על-ידי ה-EVM כ-``1``.

.. index:: ! ufixed, ! fixed, ! fixed point number

מספרי נקודה קבועה
-------------------

.. warning::
    מספרי נקודה קבועה עדיין לא נתמכים במלואם על ידי סולידיטי. אפשר להגדיר אותם, אבל
    לא ניתן להשים אל או מהם.

``fixed`` / ``ufixed``: מספרי נקודה קבועה עם או בלי סימן בגדלים שונים. מילות המפתח ``ufixedMxN`` ו-``fixedMxN``, כאשר ``M`` מייצג את מספר הביטים שבהם הסוג משתמש
ו-``N`` מייצג את מספר הנקודות העשרוניות הזמינות. ``M`` חייב להיות ניתן לחלוקה ב-8 -
בין 8 ל-256 ביטים. ``N`` חייב להיות בין 0 ל-80, כולל.
``ufixed`` ו-``fixed`` הם כינויים עבור ``ufixed128x18`` ו-``fixed128x18``, בהתאמה.

אופרטורים:

* השוואות: ``<=``, ``<``, ``==``, ``!=``, ``>=``, ``>`` (התןצאה היא מסוג ``bool``)
* אופרטורים אריתמטיים: ``+``, ``-``, unary ``-``, ``*``, ``/``, ``%`` (מודולו)

.. note::
    ההבדל העיקרי בין נקודה צפה (``float`` ו``double`` בשפות תכנות רבות, ליתר דיוק מספרי IEEE 754) למספרי נקודה קבועה הוא
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
payable`` על-ידי המרות חיצוניות ``payable(...)``. בשביל סוג של חוזה, המרה זו מותרת
רק אם החוזה יכול לקבל איתר, זאת אומרת שלחוזה יש פונקציית :ref:`receive
<receive-ether-function>` או פונקציית payable fallback. שימו לב ש-``payable(0)`` הוא חוקי
והוא יוצא מן הכלל לכלל הזה.

.. note::
    אם אתם צריכים משתנה מסוג ``address`` ומתכוונים לשלוח אליו איתר, צריך
    להגדיר את הסוג כ-``address payable`` כדי להפוך את הדרישה הזו לגלויה.
    נסו גם לעשות את ההבחנה או ההמרה הזו מוקדם ככל האפשר.

    ההבחנה בין ``address`` ו``address payable`` הוצגה עם גרסה 0.5.0.
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
    מספרים הקסדצימליים מעורבים באותיות גדולות וקטנות התואמים ל-`EIP-55 <https://github.com/ethereum/EIPs/blob/master/EIPS/eip-55.md>`_ מטופלים באופן אוטומטי כליטרלים מסוג ``address``. ראו :ref:`Address Literals<address_literals>`.

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
    אם ``x`` הוא כתובת חוזה, הקוד שלו (ליתר דיוק: :ref:`receive-ether-function` שלו, אם קיים, או אחרת :ref:`fallback-function` שלו, אם קיים) יתבצע יחד עם הקריאה ל-``transfer`` (זוהי תכונה של ה-EVM ולא ניתן למנוע אותה). אם  הביצוע נכשל בכל דרך או שנגמר לו הגז,
    העברת האיתר תבוטל (יתבצע revert) והחוזה הנוכחי יעצר עם חריגה.

* ``send``

``send`` הוא המקבילה ברמה נמוכה של ``transfer``. אם הביצוע נכשל, החוזה הנוכחי לא יעצר עם חריגה, אבל ``send`` יחזיר ``false``.

.. warning::
    ישנן כמה סכנות בשימוש ב``send``: ההעברה נכשלת אם עומק מחסנית השיחה הוא 1024
    (הקורא תמיד יכול לאלץ מצב זה) וההברה תכשל גם אם לנמען נגמר הגז. אז כדי
    לבצע העברות איתר בטוחות, תמיד צריך לבדוק את ערך ההחזרה של ``send``, השתמשו ב-`transfer`` או אפילו טוב יותר:
    השתמשו בתבנית שבה הנמען מושך את האיתר.

* ``call``, ``delegatecall`` ו-``staticcall``

על מנת להתממשק עם חוזים שאינם עומדים ב-ABI,
או לקבל שליטה ישירה יותר על הקידוד,
קיימות הפונקציות `` call``, ``delegatecall`` ו``staticcall``.
כולן מקבלות פרמטר אחד של ``bytes memory`` ו
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
    לחוזה שלכם. לכן צריך להיות מוכנים שלינויים במשתני מצב
    כאשר הקריאה חוזרת. הדרך הרגילה לאינטראקציה עם חוזים אחרים
    היא לקרוא לפונקציה על אובייקט חוזה (``x.f()``).

.. note::
    גרסאות קודמות של סולידיטי אפשרו לפונקציות אלו לקבל
    ארגומנטים שרירותיים וגם טיפלו בארגומנט ראשון מסוג
    ``bytes4`` אחרת. מקרי קצה אלו הוסרו בגרסה 0.5.0.

אפשר לכוונן את הגז המסופק עם ה-``gas`` modifier:

.. code-block:: solidity

    address(nameReg).call{gas: 1000000}(abi.encodeWithSignature("register(string)", "MyName"));

באופן דומה, ניתן לשלוט גם על ערך האיתר שיסופק:

.. code-block:: solidity

    address(nameReg).call{value: 1 ether}(abi.encodeWithSignature("register(string)", "MyName"));

לבסוף, ניתן לשלב את השינויים הללו. הסדר שלהם לא משנה:

.. code-block:: solidity

    address(nameReg).call{gas: 1000000, value: 1 ether}(abi.encodeWithSignature("register(string)", "MyName"));

באופן דומה, ניתן להשתמש בפונקציה ``delegacall``: ההבדל הוא שרק הקוד של הכתובת הנתונה יהיה בשימוש. כל שאר ההיבטים (אחסון, יתרה,...) נלקחים מהחוזה הנוכחי. מטרת ``delegacall`` היא להשתמש בקוד ספרייה המאוחסן בחוזה אחר. על המשתמש לוודא שפריסת ה-storage בשני החוזים מתאימה לשימוש ב-delegatecall.

.. note::
    לפני גרסת homestead של הקומפיילר, רק גרסה מוגבלת בשם ``callcode`` הייתה זמינה  והיאלא סיפקה גישה לערכי ``msg.sender`` ו-``msg.value`` המקוריים. פונקציה זו הוסרה בגרסה 0.5.0.

מאז גרסת byzantium
 של הקומפיילר ניתן להשתמש גם ב-``staticcall``. לקריאה זו פונקציונליות זהה ל-``call``, אבל היא תבצע revert אם הפונקציה שנקראה תשנה את משתני המצב בכל דרך שהיא.

כל שלוש הפונקציות ``call``, ``delegatecall`` ו-``staticcall`` הן פונקציות ברמה נמוכה מאוד ויש להשתמש בהן רק כמוצא אחרון מכיוון שהן פוגעות בבטיחות הסוג של סולידיטי.

האפשרות ``gas`` זמינה בכל שלוש השיטות, בעוד האפשרות ``value`` זמינה רק
ב``call``.

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
--------------

Every :ref:`contract<contracts>` defines its own type.
You can implicitly convert contracts to contracts they inherit from.
Contracts can be explicitly converted to and from the ``address`` type.

Explicit conversion to and from the ``address payable`` type is only possible
if the contract type has a receive or payable fallback function.  The conversion is still
performed using ``address(x)``. If the contract type does not have a receive or payable
fallback function, the conversion to ``address payable`` can be done using
``payable(address(x))``.
You can find more information in the section about
the :ref:`address type<address>`.

.. note::
    Before version 0.5.0, contracts directly derived from the address type
    and there was no distinction between ``address`` and ``address payable``.

If you declare a local variable of contract type (``MyContract c``), you can call
functions on that contract. Take care to assign it from somewhere that is the
same contract type.

You can also instantiate contracts (which means they are newly created). You
can find more details in the :ref:`'Contracts via new'<creating-contracts>`
section.

The data representation of a contract is identical to that of the ``address``
type and this type is also used in the :ref:`ABI<ABI>`.

Contracts do not support any operators.

The members of contract types are the external functions of the contract
including any state variables marked as ``public``.

For a contract ``C`` you can use ``type(C)`` to access
:ref:`type information<meta-type>` about the contract.

.. index:: byte array, bytes32

Fixed-size byte arrays
----------------------

The value types ``bytes1``, ``bytes2``, ``bytes3``, ..., ``bytes32``
hold a sequence of bytes from one to up to 32.

Operators:

* Comparisons: ``<=``, ``<``, ``==``, ``!=``, ``>=``, ``>`` (evaluate to ``bool``)
* Bit operators: ``&``, ``|``, ``^`` (bitwise exclusive or), ``~`` (bitwise negation)
* Shift operators: ``<<`` (left shift), ``>>`` (right shift)
* Index access: If ``x`` is of type ``bytesI``, then ``x[k]`` for ``0 <= k < I`` returns the ``k`` th byte (read-only).

The shifting operator works with unsigned integer type as right operand (but
returns the type of the left operand), which denotes the number of bits to shift by.
Shifting by a signed type will produce a compilation error.

Members:

* ``.length`` yields the fixed length of the byte array (read-only).

.. note::
    The type ``bytes1[]`` is an array of bytes, but due to padding rules, it wastes
    31 bytes of space for each element (except in storage). It is better to use the ``bytes``
    type instead.

.. note::
    Prior to version 0.8.0, ``byte`` used to be an alias for ``bytes1``.

Dynamically-sized byte array
----------------------------

``bytes``:
    Dynamically-sized byte array, see :ref:`arrays`. Not a value-type!
``string``:
    Dynamically-sized UTF-8-encoded string, see :ref:`arrays`. Not a value-type!

.. index:: address, ! literal;address

.. _address_literals:

Address Literals
----------------

Hexadecimal literals that pass the address checksum test, for example
``0xdCad3a6d3569DF655070DEd06cb7A1b2Ccd1D3AF`` are of ``address`` type.
Hexadecimal literals that are between 39 and 41 digits
long and do not pass the checksum test produce
an error. You can prepend (for integer types) or append (for bytesNN types) zeros to remove the error.

.. note::
    The mixed-case address checksum format is defined in `EIP-55 <https://github.com/ethereum/EIPs/blob/master/EIPS/eip-55.md>`_.

.. index:: integer, rational number, ! literal;rational

.. _rational_literals:

Rational and Integer Literals
-----------------------------

Integer literals are formed from a sequence of digits in the range 0-9.
They are interpreted as decimals. For example, ``69`` means sixty nine.
Octal literals do not exist in Solidity and leading zeros are invalid.

Decimal fractional literals are formed by a ``.`` with at least one number after the decimal point.
Examples include ``.1`` and ``1.3`` (but not ``1.``).

Scientific notation in the form of ``2e10`` is also supported, where the
mantissa can be fractional but the exponent has to be an integer.
The literal ``MeE`` is equivalent to ``M * 10**E``.
Examples include ``2e10``, ``-2e10``, ``2e-10``, ``2.5e1``.

Underscores can be used to separate the digits of a numeric literal to aid readability.
For example, decimal ``123_000``, hexadecimal ``0x2eff_abde``, scientific decimal notation ``1_2e345_678`` are all valid.
Underscores are only allowed between two digits and only one consecutive underscore is allowed.
There is no additional semantic meaning added to a number literal containing underscores,
the underscores are ignored.

Number literal expressions retain arbitrary precision until they are converted to a non-literal type (i.e. by
using them together with anything other than a number literal expression (like boolean literals) or by explicit conversion).
This means that computations do not overflow and divisions do not truncate
in number literal expressions.

For example, ``(2**800 + 1) - 2**800`` results in the constant ``1`` (of type ``uint8``)
although intermediate results would not even fit the machine word size. Furthermore, ``.5 * 8`` results
in the integer ``4`` (although non-integers were used in between).

.. warning::
    While most operators produce a literal expression when applied to literals, there are certain operators that do not follow this pattern:

    - Ternary operator (``... ? ... : ...``),
    - Array subscript (``<array>[<index>]``).

    You might expect expressions like ``255 + (true ? 1 : 0)`` or ``255 + [1, 2, 3][0]`` to be equivalent to using the literal 256
    directly, but in fact they are computed within the type ``uint8`` and can overflow.

Any operator that can be applied to integers can also be applied to number literal expressions as
long as the operands are integers. If any of the two is fractional, bit operations are disallowed
and exponentiation is disallowed if the exponent is fractional (because that might result in
a non-rational number).

Shifts and exponentiation with literal numbers as left (or base) operand and integer types
as the right (exponent) operand are always performed
in the ``uint256`` (for non-negative literals) or ``int256`` (for a negative literals) type,
regardless of the type of the right (exponent) operand.

.. warning::
    Division on integer literals used to truncate in Solidity prior to version 0.4.0, but it now converts into a rational number, i.e. ``5 / 2`` is not equal to ``2``, but to ``2.5``.

.. note::
    Solidity has a number literal type for each rational number.
    Integer literals and rational number literals belong to number literal types.
    Moreover, all number literal expressions (i.e. the expressions that
    contain only number literals and operators) belong to number literal
    types.  So the number literal expressions ``1 + 2`` and ``2 + 1`` both
    belong to the same number literal type for the rational number three.


.. note::
    Number literal expressions are converted into a non-literal type as soon as they are used with non-literal
    expressions. Disregarding types, the value of the expression assigned to ``b``
    below evaluates to an integer. Because ``a`` is of type ``uint128``, the
    expression ``2.5 + a`` has to have a proper type, though. Since there is no common type
    for the type of ``2.5`` and ``uint128``, the Solidity compiler does not accept
    this code.

.. code-block:: solidity

    uint128 a = 1;
    uint128 b = 2.5 + a + 0.5;

.. index:: ! literal;string, string
.. _string_literals:

String Literals and Types
-------------------------

String literals are written with either double or single-quotes (``"foo"`` or ``'bar'``), and they can also be split into multiple consecutive parts (``"foo" "bar"`` is equivalent to ``"foobar"``) which can be helpful when dealing with long strings.  They do not imply trailing zeroes as in C; ``"foo"`` represents three bytes, not four.  As with integer literals, their type can vary, but they are implicitly convertible to ``bytes1``, ..., ``bytes32``, if they fit, to ``bytes`` and to ``string``.

For example, with ``bytes32 samevar = "stringliteral"`` the string literal is interpreted in its raw byte form when assigned to a ``bytes32`` type.

String literals can only contain printable ASCII characters, which means the characters between and including 0x20 .. 0x7E.

Additionally, string literals also support the following escape characters:

- ``\<newline>`` (escapes an actual newline)
- ``\\`` (backslash)
- ``\'`` (single quote)
- ``\"`` (double quote)
- ``\n`` (newline)
- ``\r`` (carriage return)
- ``\t`` (tab)
- ``\xNN`` (hex escape, see below)
- ``\uNNNN`` (unicode escape, see below)

``\xNN`` takes a hex value and inserts the appropriate byte, while ``\uNNNN`` takes a Unicode codepoint and inserts an UTF-8 sequence.

.. note::

    Until version 0.8.0 there were three additional escape sequences: ``\b``, ``\f`` and ``\v``.
    They are commonly available in other languages but rarely needed in practice.
    If you do need them, they can still be inserted via hexadecimal escapes, i.e. ``\x08``, ``\x0c``
    and ``\x0b``, respectively, just as any other ASCII character.

The string in the following example has a length of ten bytes.
It starts with a newline byte, followed by a double quote, a single
quote a backslash character and then (without separator) the
character sequence ``abcdef``.

.. code-block:: solidity
    :force:

    "\n\"\'\\abc\
    def"

Any Unicode line terminator which is not a newline (i.e. LF, VF, FF, CR, NEL, LS, PS) is considered to
terminate the string literal. Newline only terminates the string literal if it is not preceded by a ``\``.

.. index:: ! literal;unicode

Unicode Literals
----------------

While regular string literals can only contain ASCII, Unicode literals – prefixed with the keyword ``unicode`` – can contain any valid UTF-8 sequence.
They also support the very same escape sequences as regular string literals.

.. code-block:: solidity

    string memory a = unicode"Hello 😃";

.. index:: ! literal;hexadecimal, bytes

Hexadecimal Literals
--------------------

Hexadecimal literals are prefixed with the keyword ``hex`` and are enclosed in double
or single-quotes (``hex"001122FF"``, ``hex'0011_22_FF'``). Their content must be
hexadecimal digits which can optionally use a single underscore as separator between
byte boundaries. The value of the literal will be the binary representation
of the hexadecimal sequence.

Multiple hexadecimal literals separated by whitespace are concatenated into a single literal:
``hex"00112233" hex"44556677"`` is equivalent to ``hex"0011223344556677"``

Hexadecimal literals in some ways behave like :ref:`string literals <string_literals>` but are not
implicitly convertible to the ``string`` type.

.. index:: enum

.. _enums:

Enums
-----

Enums are one way to create a user-defined type in Solidity. They are explicitly convertible
to and from all integer types but implicit conversion is not allowed.  The explicit conversion
from integer checks at runtime that the value lies inside the range of the enum and causes a
:ref:`Panic error<assert-and-require>` otherwise.
Enums require at least one member, and its default value when declared is the first member.
Enums cannot have more than 256 members.

The data representation is the same as for enums in C: The options are represented by
subsequent unsigned integer values starting from ``0``.

Using ``type(NameOfEnum).min`` and ``type(NameOfEnum).max`` you can get the
smallest and respectively largest value of the given enum.


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
    Enums can also be declared on the file level, outside of contract or library definitions.

.. index:: ! user defined value type, custom type

.. _user-defined-value-types:

User-defined Value Types
------------------------

A user-defined value type allows creating a zero cost abstraction over an elementary value type.
This is similar to an alias, but with stricter type requirements.

A user-defined value type is defined using ``type C is V``, where ``C`` is the name of the newly
introduced type and ``V`` has to be a built-in value type (the "underlying type"). The function
``C.wrap`` is used to convert from the underlying type to the custom type. Similarly, the
function ``C.unwrap`` is used to convert from the custom type to the underlying type.

The type ``C`` does not have any operators or attached member functions. In particular, even the
operator ``==`` is not defined. Explicit and implicit conversions to and from other types are
disallowed.

The data-representation of values of such types are inherited from the underlying type
and the underlying type is also used in the ABI.

The following example illustrates a custom type ``UFixed256x18`` representing a decimal fixed point
type with 18 decimals and a minimal library to do arithmetic operations on the type.


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

Notice how ``UFixed256x18.wrap`` and ``FixedMath.toUFixed256x18`` have the same signature but
perform two very different operations: The ``UFixed256x18.wrap`` function returns a ``UFixed256x18``
that has the same data representation as the input, whereas ``toUFixed256x18`` returns a
``UFixed256x18`` that has the same numerical value.

.. index:: ! function type, ! type; function

.. _function_types:

Function Types
--------------

Function types are the types of functions. Variables of function type
can be assigned from functions and function parameters of function type
can be used to pass functions to and return functions from function calls.
Function types come in two flavours - *internal* and *external* functions:

Internal functions can only be called inside the current contract (more specifically,
inside the current code unit, which also includes internal library functions
and inherited functions) because they cannot be executed outside of the
context of the current contract. Calling an internal function is realized
by jumping to its entry label, just like when calling a function of the current
contract internally.

External functions consist of an address and a function signature and they can
be passed via and returned from external function calls.

Function types are notated as follows:

.. code-block:: solidity
    :force:

    function (<parameter types>) {internal|external} [pure|view|payable] [returns (<return types>)]

In contrast to the parameter types, the return types cannot be empty - if the
function type should not return anything, the whole ``returns (<return types>)``
part has to be omitted.

By default, function types are internal, so the ``internal`` keyword can be
omitted. Note that this only applies to function types. Visibility has
to be specified explicitly for functions defined in contracts, they
do not have a default.

Conversions:

A function type ``A`` is implicitly convertible to a function type ``B`` if and only if
their parameter types are identical, their return types are identical,
their internal/external property is identical and the state mutability of ``A``
is more restrictive than the state mutability of ``B``. In particular:

- ``pure`` functions can be converted to ``view`` and ``non-payable`` functions
- ``view`` functions can be converted to ``non-payable`` functions
- ``payable`` functions can be converted to ``non-payable`` functions

No other conversions between function types are possible.

The rule about ``payable`` and ``non-payable`` might be a little
confusing, but in essence, if a function is ``payable``, this means that it
also accepts a payment of zero Ether, so it also is ``non-payable``.
On the other hand, a ``non-payable`` function will reject Ether sent to it,
so ``non-payable`` functions cannot be converted to ``payable`` functions.
To clarify, rejecting ether is more restrictive than not rejecting ether.
This means you can override a payable function with a non-payable but not the
other way around.

Additionally, When you define a ``non-payable`` function pointer,
the compiler does not enforce that the pointed function will actually reject ether.
Instead, it enforces that the function pointer is never used to send ether.
Which makes it possible to assign a ``payable`` function pointer to a ``non-payable``
function pointer ensuring both types behave the same way, i.e, both cannot be used
to send ether.

If a function type variable is not initialised, calling it results
in a :ref:`Panic error<assert-and-require>`. The same happens if you call a function after using ``delete``
on it.

If external function types are used outside of the context of Solidity,
they are treated as the ``function`` type, which encodes the address
followed by the function identifier together in a single ``bytes24`` type.

Note that public functions of the current contract can be used both as an
internal and as an external function. To use ``f`` as an internal function,
just use ``f``, if you want to use its external form, use ``this.f``.

A function of an internal type can be assigned to a variable of an internal function type regardless
of where it is defined.
This includes private, internal and public functions of both contracts and libraries as well as free
functions.
External function types, on the other hand, are only compatible with public and external contract
functions.

.. note::
    External functions with ``calldata`` parameters are incompatible with external function types with ``calldata`` parameters.
    They are compatible with the corresponding types with ``memory`` parameters instead.
    For example, there is no function that can be pointed at by a value of type ``function (string calldata) external`` while
    ``function (string memory) external`` can point at both ``function f(string memory) external {}`` and
    ``function g(string calldata) external {}``.
    This is because for both locations the arguments are passed to the function in the same way.
    The caller cannot pass its calldata directly to an external function and always ABI-encodes the arguments into memory.
    Marking the parameters as ``calldata`` only affects the implementation of the external function and is
    meaningless in a function pointer on the caller's side.

Libraries are excluded because they require a ``delegatecall`` and use :ref:`a different ABI
convention for their selectors <library-selectors>`.
Functions declared in interfaces do not have definitions so pointing at them does not make sense either.

Members:

External (or public) functions have the following members:

* ``.address`` returns the address of the contract of the function.
* ``.selector`` returns the :ref:`ABI function selector <abi_function_selector>`

.. note::
  External (or public) functions used to have the additional members
  ``.gas(uint)`` and ``.value(uint)``. These were deprecated in Solidity 0.6.2
  and removed in Solidity 0.7.0. Instead use ``{gas: ...}`` and ``{value: ...}``
  to specify the amount of gas or the amount of wei sent to a function,
  respectively. See :ref:`External Function Calls <external-function-calls>` for
  more information.

Example that shows how to use the members:

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

Example that shows how to use internal function types:

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

Another example that uses external function types:

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
    Lambda or inline functions are planned but not yet supported.
