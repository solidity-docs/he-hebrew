.. index:: ! installing

.. _installing-solidity:

################################
התקנת הקומפיילר של סולידיטי
################################

גרסאות
==========

מספרי הגרסות של סולידיטי נקבעות על-פי `גירסאות סמנטיות <https://semver.org>`_. 
בנוסף, גרסאות תיקון (patch) עם גרסה עיקרית 0 (כלומר 0.x.y) לא
מכילות שינויי התנהגות. ז"א שניתן להניח שקוד שמתקמפל עם גרסה 0.x.y
יתקמפל גם עם גרסה 0.x.z שבה z > y.

בנוסף לגרסאות, אנו מספקים **בניות nightly לפתוח** כדי להקל
על מפתחים לנסות תכונות עתידיות
ולספק משוב מוקדם. שימו לב עם זאת שלמרות שבדרך כלל בניות ה-nightly
יציבות מאוד, הן מכילות קוד שהוא עדיין בפיתוח ולכן
לא מובטח שיעבדו תמיד. למרות מאמצינו, הן עשויות
להכיל שינויים לא מתועדים ו/או שמשנים את ההתנהגות של הקוד הקיים, שינויים
שלא יהפכו לחלק משחרור בפועל. גרסאות ה-nightly לא מיועדות לשימוש בייצור.

בעת הטמעת חוזים חדשים לשימוש, עליכם להשתמש בגרסה האחרונה של סולידיטי ששוחררה.
זאת מכיוון ששינויים שמשנים התנהגות, כמו גם תכונות חדשות ותיקוני באגים, מתווספים לסולידיטי באופן קבוע.
אנחנו משתמשים כעת במספר גרסה 0.x `כדי לציין את קצב השינוי המהיר הזה <https://semver.org/#spec-item-4>`_.

רמיקס
======

*אנחנו ממליצים על רמיקס (Remix) עבור חוזים קטנים וללמידה מהירה של סולידיטי.*

`לשימוש מקוון ברמיקס <https://remix.ethereum.org/>`_, אין צורך להתקין דבר. אם אתם רוצים להשתמש בו ללא חיבור לאינטרנט, ראו https://github.com/ethereum/remix-live/tree/gh-pages#readme ופעלו לפי ההוראות בדף זה. רמיקס הוא גם אפשרות נוחה לבדיקת בניית nightly מבלי להתקין מספר גרסאות סולידיטי שונות.

אפשרויות נוספות בדף זה מפרטות את אופן ההתקנה של הקומפיילר של סולידיטי שתומך בשורות-פקודה (command-line). בחרו קומפיילר שתומך בשורות-פקודה אם אתם עובדים על חוזה גדול יותר או אם אתם זקוקים לאפשרויות קומפילציה נוספות.

.. _solcjs:

npm / Node.js
=============

השתמשו ב-``npm`` כדרך נוחה וניידת להתקנת ``solcjs``, הקומפיילר של סולידיטי.
לתוכנת `solcjs` יש פחות תכונות מהדרכים לגישה לקומפיילר שיתוארו
בהמשך בדף זה.
תיעוד :ref:`commandline-compiler` מניח שאתם משתמשים
בקומפיילר עם התכונות המלאות, ``solc``. השימוש ב- ``solcjs`` מתועד בתוך
`הרפוזיטורי <https://github.com/ethereum/solc-js>`_ שלו.

הערה: פרויקט solc-js נגזר מה-C++
`solc` באמצעות Emscripten, כלומר שניהם משתמשים באותו קוד מקור של הקומפיילר.
ניתן להשתמש ב-solc-js בפרוייקטים של JavaScript ישירות (כגון רמיקס).
בבקשה הסתכלו במאגר solc-js לקבלת הוראות.

.. code-block:: bash

    npm install -g solc

.. note::

    קובץ ההפעלה של שורות-הפקודה נקרא ``solcjs``.

 	אפשרויות שורות-הפקודה של ``solcjs`` אינן תואמות ל``solc`` ולכלים (כגון ``geth``)
 	כך שההתנהגות של ``solc`` לא תעבוד עם ``solcjs``.

Docker
======

"תמונות דוקר" (Docker images)
של בניית סולידיטי זמינות באמצעות תמונת ``solc`` מארגון ``ethereum``.
השתמשו בתג ``stable`` עבור הגרסה האחרונה שפורסמה, וב-``nightly`` לשינויים שעלולים להיות לא יציבים.

תמונת דוקר מריצה את קובץ ההפעלה של הקומפיילר כך שתוכלו להעביר אליו את כל ארגומנטי המהדר.
לדוגמה, הפקודה למטה מושכת את הגרסה היציבה של תמונת ``solc`` (אם היא עדיין לא מותקנת אצלכם),
ומריצה אותה בקונטיינר חדש, עם העברת הארגומנט ``--help``.

.. code-block:: bash

    docker run ethereum/solc:stable --help

לדוגמה, אתם יכולים לציין גרסאות בניית גרסה ע"י תג עבור גרסה 0.5.4.

.. code-block:: bash

    docker run ethereum/solc:0.5.4 --help

כדי להשתמש בתמונת דוקר כדי לקמפל קבצי סולידיטי במחשב המארח, התקינו
תיקיה מקומית עבור קלט ופלט, וציינו את החוזה לקימפול. לדוגמה:

.. code-block:: bash

    docker run -v /local/path:/sources ethereum/solc:stable -o /sources/output --abi --bin /sources/Contract.sol

אתם יכולים גם להשתמש בממשק JSON הסטנדרטי (מומלץ בעת שימוש בקומפיילר עם כלי עבודה).
בעת שימוש בממשק זה, אין צורך להעלות ספריות כל עוד קלט ה- JSON הוא
עצמאי (כלומר, אין התייחסות לקבצים חיצוניים כלשהם שצריכים
:ref:`להטען על ידי ה-import callback
<initial-vfs-content-standard-json-with-import-callback>`).

.. code-block:: bash

    docker run ethereum/solc:stable --standard-json < input.json > output.json

חבילות לינוקס
==============

חבילות בינאריות של סולידיטי זמינות ב-
`solidity/releases <https://github.com/ethereum/solidity/releases>`_.

יש לנו גם PPAs עבור אובונטו. אתם יכולים לקבל את הגרסה היציבה האחרונה
באמצעות הפקודות הבאות:

.. code-block:: bash

    sudo add-apt-repository ppa:ethereum/ethereum
    sudo apt-get update
    sudo apt-get install solc

ניתן להתקין את גרסת ה-nightly באמצעות הפקודות הבאות:

.. code-block:: bash

    sudo add-apt-repository ppa:ethereum/ethereum
    sudo add-apt-repository ppa:ethereum/ethereum-dev
    sudo apt-get update
    sudo apt-get install solc

יתר על כן, חלק מההפצות של לינוקס מספקות חבילות משלהן. חבילות אלו אינן
מתוחזקות ישירות על ידינו אך בדרך כלל מעודכנות על ידי מנהלי החבילות בהתאמה.

לדוגמה, ל- Arch Linux יש חבילות לגרסת הפיתוח העדכנית ביותר כחבילות AUR: `סולידיטי <https://aur.archlinux.org/packages/solidity>`_
ו-`solidity-bin <https://aur.archlinux.org/packages/solidity-bin>`_.

.. note::

    שימו לב שחבילות `AUR <https://wiki.archlinux.org/title/Arch_User_Repository>`_
 	הן תוכן המיוצר על ידי המשתמש וחבילות לא רשמיות. היזהרו בעת השימוש בהן.

קיימת גם `חבילת snap <https://snapcraft.io/solc>`_, עם זאת, היא **כרגע לא מתוחזקת**.
החבילה ניתנת להתקנה בכל `ההפצות הנתמכות של לינוקס <https://snapcraft.io/docs/core/install>`_.
להתקנת הגרסה היציבה האחרונה של solc:

.. code-block:: bash

    sudo snap install solc

אם אתם רוצים לעזור בבדיקת גרסת הפיתוח העדכנית של סולידיטי
שכוללת את השינויים האחרונים, אנא השתמשו ב:

.. code-block:: bash

    sudo snap install solc --edge

.. note::

    ה-snap של ``solc`` משתמשת בסגירות קפדנית (strict confinement). זהו המצב המאובטח ביותר עבור חבילות snap
 	אבל דבר זה מגיע עם מגבלות, כמו גישה רק לקבצים בספריות ``/home`` ו``/media`` שלכם.
 	למידע נוסף, עבור אל `הסרת המסתורין מ-Snap Confinement <https://snapcraft.io/blog/demystifying-snap-confinement>`_.


חבילות macOS
==============

אנחנו מפיצים את הקומפיילר של סולידיטי דרך Homebrew
כגרסת-בנייה-מהמקור. bottles שנבנו מראש
כרגע לא נתמכים.

.. code-block:: bash

    brew update
    brew upgrade
    brew tap ethereum/ethereum
    brew install solidity

כדי להתקין את גרסת 0.4.x / 0.5.x העדכנית ביותר של סולידיטי, אתם יכולים להשתמש גם ב-``brew install solidity@4``
ו-``brew install solidity@5``, בהתאמה.

אם אתם צריכים גרסה ספציפית של סולידיטי, אפשר להתקין
נוסחת Homebrew ישירות מ-Github.

הסתכלו ב-
`solidity.rb commits on Github <https://github.com/ethereum/homebrew-ethereum/commits/master/solidity.rb>`_.

העתיקו את ה-commit hash של הגרסה הרצויה ובדקו אותה במחשב שלכם.

.. code-block:: bash

    git clone https://github.com/ethereum/homebrew-ethereum.git
    cd homebrew-ethereum
    git checkout <your-hash-goes-here>

התקינו אותה באמצעות ``brew``:
.. code-block:: bash

    brew unlink solidity
    # eg. Install 0.4.8
    brew install solidity.rb

קבצים בינאריים סטטיים
=======================

אנחנו מתחזקים רפוזיטורי המכיל בנייות סטטיות של גרסאות קומפיילר קודמות ונוכחיות לכל
הפלטפורמות הנתמכות ב- `solc-bin`_. זהו גם המקום שבו תוכלו למצוא את בניות ה-nightly.

הרפוזיטורי הוא לא רק דרך מהירה וקלה עבור משתמשי קצה להשיג קבצים בינאריים לשימוש
אלא גם נועד להיות ידידותי לכלים של צד שלישי:

- התוכן משוקף אל https://binaries.soliditylang.org שם ניתן להוריד אותו בקלות
   דרך HTTPS ללא כל אימות, הגבלת קצב או צורך להשתמש ב-git.
- התוכן מוגש עם כותרות 'Content-Type' נכונות ותצורת CORS מקלה כך שהוא
   ניתן לטעינה ישירות על ידי כלים הפועלים בדפדפן.
- קבצים בינאריים אינם דורשים התקנה או פרוק (unpack - למעט רכיבים ישנים יותר של Windows
   שמצורפים להם קובצי DLL נחוצים).
- אנו שואפים לרמה גבוהה של תאימות לאחור. קבצים, לאחר שנוספו, אינם מוסרים או מועברים
   מבלי לספק הפניה מהמיקום הישן. הם גם לעולם לא משתנים
   וצריכים תמיד להתאים ל-checksum המקורי. החריג היחיד יהיו
   קבצים בעייתיים או בלתי שמישים עם פוטנציאל לגרימת יותר נזק מתועלת אם יישארו כפי שהם.
- קבצים מועברים גם ב-HTTP וגם ב-HTTPS. כל עוד אתם משיגים את רשימת הקבצים בצורה מאובטחת
   (באמצעות git, HTTPS, IPFS וכו') ומאמתים את ה-hash של הקבצים הבינאריים
   לאחר הורדתם, אינכם צריכים להשתמש ב-HTTPS עבור הקבצים הבינאריים עצמם.

אותם קבצים בינאריים זמינים ברוב המקרים ב'עמוד השחרור של סולידיטי ב-Github'_.
ההבדל הוא שאנחנו בדרך כלל לא מעדכנים מהדורות ישנות בדף ההפצה של Github. זאת אומרת שבדרך  כלל
לא נשנה את שמם אם אופן קביעת השמות משתנה ולא נוסיף בניות לפלטפורמות
שלא נתמכו בזמן השחרור. זה קורה רק ב-``solc-bin``.

מאגר ``solc-bin`` מכיל מספר ספריות ברמה העליונה, כל אחת מייצגת פלטפורמה אחת.
כל אחת מהן כוללת קובץ ``list.json`` המפרט את הקבצים הבינאריים הזמינים. למשל ב
``emscripten-wasm32/list.json`` תמצא את המידע הבא לגבי גרסה 0.7.4:

.. code-block:: json

    {
      "path": "solc-emscripten-wasm32-v0.7.4+commit.3f05b770.js",
      "version": "0.7.4",
      "build": "commit.3f05b770",
      "longVersion": "0.7.4+commit.3f05b770",
      "keccak256": "0x300330ecd127756b824aa13e843cb1f43c473cb22eaf3750d5fb9c99279af8c3",
      "sha256": "0x2b55ed5fec4d9625b6c7b3ab1abd2b7fb7dd2a9c68543bf0323db2c7e2d55af2",
      "urls": [
        "bzzr://16c5f09109c793db99fe35f037c6092b061bd39260ee7a677c8a97f18c955ab1",
        "dweb:/ipfs/QmTLs5MuLEWXQkths41HiACoXDiH8zxyqBHGFDRSzVE5CS"
      ]
    }

זאת אומרת ש:

- אתם יכולים למצוא את הקובץ הבינארי באותה ספרייה תחת השם
   `solc-emscripten-wasm32-v0.7.4+commit.3f05b770.js <https://github.com/ethereum/solc-bin/blob/gh-pages/emscripten-wasm32/solc-emscripten-wasm32-v0.7.4 +commit.3f05b770.js>`_.
   שימו לב שהקובץ עשוי להיות קישור סימבולי, ותצטרכו לפתור אותו בעצמכם אם אינכם משתמשים ב-git כדי להוריד אותו או שמערכת הקבצים שלכם לא תומכת בקישורים סימבוליים.
- עותק של הקובץ הבינארי נמצא גם ב-https://binaries.soliditylang.org/emscripten-wasm32/solc-emscripten-wasm32-v0.7.4+commit.3f05b770.js.
   במקרה זה אין צורך ב-git וקישורים סימבוליים נפתרים בשקיפות, או על ידי העברת עותק
   של הקובץ או החזרת הפניית HTTP redirect.
- הקובץ זמין גם ב-IPFS בכתובת `QmTLs5MuLEWXQkths41HiACoXDiH8zxyqBHGFDRSzVE5CS`_.
- ייתכן שהקובץ יהיה זמין בעתיד ב-Swarm בכתובת `16c5f09109c793db99fe35f037c6092b061bd39260ee7a677c8a97f18c955ab1`_.
- אתם יכולים לאמת את תקינות הקובץ הבינארי על ידי השוואת ה-keccak256 hash שלו ל-
   ``0x300330ecd127756b824aa13e843cb1f43c473cb22eaf3750d5fb9c99279af8c3``. ניתן לחשב את ה-hash
   בשורת הפקודה באמצעות כלי השירות ``keccak256sum`` המסופק על ידי הפונקציה `sha3sum`_ או `keccak256()
   מ-ethereumjs-util`_ ב-JavaScript.
- אתם יכולים גם לאמת את תקינות הקובץ הבינארי על ידי השוואת ה-sha256 שלו ל-sha256
   ``0x2b55ed5fec4d9625b6c7b3ab1abd2b7fb7dd2a9c68543bf0323db2c7e2d55af2``.

.. warning::

   בשל דרישת התאימות-לאחור החזקה, הרפוזיטורי מכיל כמה אלמנטים מדור קודם
	אך עליכם להימנע משימוש בהם בעת כתיבת כלים חדשים:

   - השתמשו ב-``emscripten-wasm32/`` (עם חלופה (fallback) ל-``emscripten-asmjs/``) במקום ``bin/`` אם
  	אתם רוצה את הביצועים הטובים ביותר. עד גרסה 0.6.1 סיפקנו רק קבצים בינאריים של asm.js.
  	החל מ-0.6.2 עברנו ל- `WebAssembly builds`_ עם ביצועים טובים בהרבה. יש לנו
  	בניה מחדש של הגרסאות הישנות יותר עבור wasm אבל קבצי asm.js המקוריים נשארים ב- ``bin/``.
  	את הקבצים החדשים היה צריך למקם בספרייה נפרדת כדי למנוע התנגשויות בשמות.
	- השתמשו ב-``emscripten-asmjs/`` וב-``emscripten-wasm32/`` במקום בספריות ``bin/`` ו-``wasm/``
  	אם אתה רוצים להיות בטוחים שאתה מורידים wasm או asm.js בינארי
	- השתמשו ב-``list.json`` במקום ``list.js`` וב-``list.txt``. פורמט רשימת JSON מכילה את הכל
  	המידע מהישנים ועוד.
	- השתמשו ב-https://binaries.soliditylang.org במקום https://solc-bin.ethereum.org. כדי לשמור על
  	פשטות העברנו כמעט את כל מה שקשור לקומפיילר תחת תחום ``soliditylang.org`` החדש
  	וזה חל גם על ``solc-bin``. בעוד שהדומיין החדש מומלץ, הישן
  	עדיין נתמך באופן מלא ומובטח שיצביע על אותו מיקום.

.. warning::

    הקבצים הבינאריים זמינים גם בכתובת https://ethereum.github.io/solc-bin/ אבל דף זה
 	הפסיק להתעדכן מיד לאחר שחרורה של גרסה 0.7.2. דף זה לא יקבל מהדורות חדשות
 	או בניות nightly לכל הפלטפורמות ואינו משרת את מבנה הספריות החדש, כולל
 	בניה ללא emscripten.

 	אם אתם משתמשים בו, אנא עברו ל-https://binaries.soliditylang.org. דבר זה מאפשר לנו לבצע שינויים באירוח הבסיסי בצורה שקופה
 	ולמזער את ההפרעות. בניגוד לתחום ``ethereum.github.io``, שאין לנו כל שליטה עליו,
 	מובטח ש-``binaries.soliditylang.org`` יעבוד וישמור על אותו מבנה כתובת URL
 	בטווח הרחוק.

.. _IPFS: https://ipfs.io
.. _Swarm: https://swarm-gateways.net/bzz:/swarm.eth
.. _solc-bin: https://github.com/ethereum/solc-bin/
.. _Solidity release page on github: https://github.com/ethereum/solidity/releases
.. _sha3sum: https://github.com/maandree/sha3sum
.. _keccak256() function from ethereumjs-util: https://github.com/ethereumjs/ethereumjs-util/blob/master/docs/modules/_hash_.md#const-keccak256
.. _WebAssembly builds: https://emscripten.org/docs/compiling/WebAssembly.html
.. _QmTLs5MuLEWXQkths41HiACoXDiH8zxyqBHGFDRSzVE5CS: https://gateway.ipfs.io/ipfs/QmTLs5MuLEWXQkths41HiACoXDiH8zxyqBHGFDRSzVE5CS
.. _16c5f09109c793db99fe35f037c6092b061bd39260ee7a677c8a97f18c955ab1: https://swarm-gateways.net/bzz:/16c5f09109c793db99fe35f037c6092b061bd39260ee7a677c8a97f18c955ab1/

.. _building-from-source:

בנייה מקוד המקור
====================
דרישות קדם - כל מערכות ההפעלה
-------------------------------------

להלן התלות עבור כל הבניות של סולידיטי:

+-----------------------------------+-------------------------------------------------------+
| תוכנה                            | הערות                                                 |
+===================================+=======================================================+
| `CMake`_ (version 3.21.3+ on      | מחולל קבצי בנייה חוצה פלטפורמות.                  |
| Windows, 3.13+ otherwise)         |                                                       |
+-----------------------------------+-------------------------------------------------------+
| `Boost`_ (version 1.77 on         | ספריות C++.                                          |
| Windows, 1.65+ otherwise)         |                                                       |
+-----------------------------------+-------------------------------------------------------+
| `Git`_                            | כלי שורוכלי שורות-פקודה לאחזור קוד מקור.           |
+-----------------------------------+-------------------------------------------------------+
| `z3`_ (version 4.8.16+, Optional) | לשימוש עם בודק SMT.                                  |
+-----------------------------------+-------------------------------------------------------+
| `cvc4`_ (Optional)                | לשימוש עם בודק SMT.                                  |
+-----------------------------------+-------------------------------------------------------+

.. _cvc4: https://cvc4.cs.stanford.edu/web/
.. _Git: https://git-scm.com/download
.. _Boost: https://www.boost.org
.. _CMake: https://cmake.org/download/
.. _z3: https://github.com/Z3Prover/z3

.. note::
    גרסאות סולידיטי לפני 0.5.10 יכולות להיכשל בקישור מול גרסאות Boost 1.70+.
 	פתרון אפשרי הוא שינוי שם זמני של ``<Boost Install path>/lib/cmake/Boost-1.70.0``
 	לפני הפעלת הפקודה cmake כדי להגדיר את סולידיטי.

 	החל מ-0.5.10 קישור מול Boost 1.70+ אמור לעבוד ללא התערבות ידנית.

.. note::
    תצורת הבנייה המוגדרת כברירת מחדל דורשת גרסה ספציפית של Z3 (האחרונה ביותר בזמן
 	שהקוד עודכן לאחרונה). שינויים שהוכנסו בין מהדורות Z3 מביאות לעתים קרובות לתוצאות מעט שונות
 	(אך עדיין תקפות). מבחני ה-SMT שלנו אינם מביאים בחשבון את ההבדלים הללו
 	וככל הנראה ייכשלו עם גרסה שונה מזו שהם נכתבו עבורה. זה לא אומר
 	שבנייה שמשתמשת בגרסה אחרת היא פגומה. אם תעבירו את האופציה ``-DSTRICT_Z3_VERSION=OFF``
 	ל- CMake, תוכלו לבנות עם כל גרסה שעומדת בדרישה שניתנה בטבלה למעלה.
 	עם זאת, אם תעשו זאת, בבקשה זכרו להעביר את האופציה ``--no-smt`` ל``scripts/tests.sh``
 	כדי לדלג על מבחני SMT.

.. note::
    כברירת מחדל, הבנייה מתבצעת ב*מצב פדנטי* שמאפשרת אזהרות נוספות ומגדיר לקומפיילר להתייחס לכל האזהרות כשגיאות.
 	דבר זה מאלץ מפתחים לתקן אזהרות כשהן מתעוררות, כך שהן לא יצטברו "כדי לתקן מאוחר יותר".
 	אם אתם מעוניינים רק ביצירת גרסה לשחרור ואינכם מתכוונים לשנות את קוד המקור
 	כדי להתמודד עם אזהרות כאלה, אתם יכולים להעביר את האופציה ``-DPEDANTIC=OFF`` אל CMake כדי להשבית את המצב הזה.
 	פעולה זו אינה מומלצת לשימוש כללי אך עשויה להיות נחוצה בעת שימוש בשרשרת כלים שאנחנו
 	לא בודקים או כאשר מנסים לבנות גרסה ישנה עם כלים חדשים יותר.
 	אם אתם נתקלים באזהרות כאלה, אנא שקלו
 	`לדווח עליהם <https://github.com/ethereum/solidity/issues/new>`_.

גרסת קומפיילר מינימלית
^^^^^^^^^^^^^^^^^^^^^^^^^

הקומפיילרים הבאים של C++ וגרסאות המינימום שלהם יכולים לבנות את בסיס הקוד של סולידיטי:

- `GCC <https://gcc.gnu.org>`_, version 8+
- `Clang <https://clang.llvm.org/>`_, version 7+
- `MSVC <https://visualstudio.microsoft.com/vs/>`_, version 2019+

דרישות קדם - macOS
---------------------

כדי לבנות macOS, ודאו שיש לכם את הגרסה העדכנית ביותר של
`Xcode מותקנת <https://developer.apple.com/xcode/resources/>`_.
התקנה זו מכילה את `Clang C++ קומפייילר <https://en.wikipedia.org/wiki/Clang>`_,
`Xcode IDE <https://en.wikipedia.org/wiki/Xcode>`_ וכלי פיתוח אחרים של Apple
הדרושים לבניית יישומי C++ ב-OS X.
אם אתם מתקינים Xcode בפעם הראשונה, או שזה עתה התקנתם
גרסה חדשה אז תצטרכו להסכים לרישיון לפני שתוכלו להפעיל
את פקודת הבנייה:

.. code-block:: bash

    sudo xcodebuild -license accept

סקריפט הבנייה של OS X שלנו משתמש במנהל החבילות 'the Homebrew <https://brew.sh>`_
להתקנת תלויות חיצוניות.
להלן אופן הסרת ההתקנה של Homebrew
<https://docs.brew.sh/FAQ#how-do-i-uninstall-homebrew>`_,
אם אי פעם תרצו להתחיל מחדש מאפס.

דרישות קדם - Windows
-----------------------

עליכם להתקין את התלויות הבאות עבור בניות ל-Windows של סולידיטי:

+-----------------------------------+-------------------------------------------------------+
| תוכנה                            | הערות                                                 |
+===================================+=======================================================+
| `Visual Studio 2019 Build Tools`_ | C++ compiler                                          |
+-----------------------------------+-------------------------------------------------------+
| `Visual Studio 2019`_  (Optional) | C++ compiler and dev environment.                     |
+-----------------------------------+-------------------------------------------------------+
| `Boost`_ (version 1.77)           | C++ libraries.                                        |
+-----------------------------------+-------------------------------------------------------+

אם כבר יש לכם IDE אחד ואתם צריכים רק את הקומפיילר והספריות,
אתם יכולים להתקין את Visual Studio 2019 Build Tools.

Visual Studio 2019 מספק גם IDE וגם קומפיילר וספריות נחוצות.
לכן, אם אין לכם IDE ואתם מעדיפים לפתח בסולידיטי, Visual Studio 2019
עשויה להיות בחירה טובה עבורכם כדי להתקין הכל בקלות.

להלן רשימת הרכיבים שיש להתקין
ב-Visual Studio 2019 Build Tools או Visual Studio 2019:

* תכונות ליבה של Visual Studio C++
* ערכת כלים VC++ 2019 v141 (x86,x64)
* Windows Universal CRT SDK
* Windows 8.1 SDK
* תמיכה ב-C++/CLI

.. _Visual Studio 2019: https://www.visualstudio.com/vs/
.. _Visual Studio 2019 Build Tools: https://visualstudio.microsoft.com/vs/older-downloads/#visual-studio-2019-and-other-products

יש לנו סקריפט עזר שבו אתם יכולים להשתמש כדי להתקין את כל התלויות החיצונית הנדרשות:

.. code-block:: bat

    scripts\install_deps.ps1

סקריפט זה יתקין את ``boost`` ו``cmake`` בספריית המשנה ``deps``.

שכפול הרפוזיטורי
--------------------

כדי לשכפל את קוד המקור, בצעו את הפקודה הבאה:

.. code-block:: bash

    git clone --recursive https://github.com/ethereum/solidity.git
    cd solidity

אם אתם רוצים לעזור לפיתוח סולידיטי,
אתם צריכים לשכפל (fork) את סולידיטי ולהוסיף את העותק האישי שלכם כ-remote שני:

.. code-block:: bash

    git remote add personal git@github.com:[username]/solidity.git

.. note::
    שיטה זו תגרום לבניית גרסת טרום שחרור שתגרום למשל
 	להגדרת דגל בכל bytecode המיוצר על ידי קומפיילר כזה.
 	אם אתם רוצים לבנות מחדש קומפיילר סולידיטי שישוחרר,
 	אנא השתמשו ב-tarball המקורי בדף השחרור של github:

 	https://github.com/ethereum/solidity/releases/download/v0.X.Y/solidity_0.X.Y.tar.gz

 	(לא "קוד המקור" שסופק על ידי github).

בנייה ע"י שורת-פקודה
-----------------------

**הקפידו להתקין תלויות חיצוניות (ראה למעלה) לפני הבנייה.**

פרויקט סולידיטי משתמש ב- CMake כדי להגדיר את הבנייה.
אולי תרצו להשתמש ב- `cache`_ כדי להאיץ בנייה חוזרת.
CMake ישתמש בו אוטומטית.
בניית סולידיטי דומה למדי ב-Linux, macOS ו-Unices אחרים:

.. _ccache: https://ccache.dev/

.. code-block:: bash

    mkdir build
    cd build
    cmake .. && make

או אפילו יותר קל - ב-Linux וב-macOS אתם יכולים להריץ:

.. code-block:: bash

    #note: this will install binaries solc and soltest at usr/local/bin
    ./scripts/build.sh

.. warning::

    בניית גרסה ל-BSD אמורה לעבוד, אך לא נבדקה על ידי צוות סולידיטי.

ול-Windows:

.. code-block:: bash

    mkdir build
    cd build
    cmake -G "Visual Studio 16 2019" ..

במקרה שאתם רוצים להשתמש בגרסת ה-boost המותקנת על ידי ``scripts\install_deps.ps1``,
תצטרכו להעביר בנוסף את ``-DBoost_DIR="deps\boost\lib\cmake\Boost-*"`` ואת ``-DCMAKE_MSVC_RUNTIME_LIBRARY=MultiThreaded``
כארגומנטים לקריאה ל-``cmake``.

דבר זה אמור לגרום ליצירת **solidity.sln** בספריית הבנייה הזו.
לחיצה כפולה על הקובץ אמורה לגרום להפעלת Visual Studio. אנחנו מציעים לבנות
תצורת **שחרור**, אבל כל האפשרויות האחרות גם יעבדו.

לחלופין, אתם יכולים לבנות בשורת הפקודה עבור Windows כך:

.. code-block:: bash

    cmake --build . --config Release

אפשרויות CMake
================

אם אתם מעוניינים לדעת אילו אפשרויות זמינות של CMake, הריצו את ``cmake .. -LH``.

.. _smt_solvers_build:

פותרי SMT
-----------
ניתן לבנות סולידיטי עם שימוש בפותרי SMT, ודבר זה נעשה כברירת מחדל אם
הם נמצאים במערכת. ניתן להשבית כל פותר על ידי אופציית ``cmake``.

*הערה: במקרים מסוימים, דבר זה יכול להיות גם פתרון אפשרי לכשלים בבנייה.*


בתוך תיקיית הבנייה אפשר להשבית את הפותרים, מכיוון שהם מופעלים כברירת מחדל:

.. code-block:: bash

    # disables only Z3 SMT Solver.
    cmake .. -DUSE_Z3=OFF

    # disables only CVC4 SMT Solver.
    cmake .. -DUSE_CVC4=OFF

    # disables both Z3 and CVC4
    cmake .. -DUSE_CVC4=OFF -DUSE_Z3=OFF

מחרוזת הגרסה בפירוט
============================

מחרוזת גרסת סולידיטי מכילה ארבעה חלקים:

- מספר הגרסה
- תג טרום-הפצה, בדרך כלל מוגדר ל-``develop.YYYY.MM.DD`` או ``nightly.YYYY.MM.DD``
- commit בפורמט של ``commit.GITHASH``
- פלטפורמה, בעלת מספר שרירותי של פריטים, המכילה פרטים על הפלטפורמה והקומפיילר

אם יש שינויים מקומיים, ה-commit יתוקן לאחר מכן עם ``.mod``.

חלקים אלה משולבים כנדרש על ידי SemVer, כאשר התג של סולידיטי לפני השחרור שווה לתג קדם השחרור של SemVer,
ושילוב של ה-commit והפלטפורמה של Solidity מרכיבים את ה-metadata של בניית ה-SemVer.

דוגמה לגרסה: ``0.4.8+commit.60cc1668.Emscripten.clang``.

דוגמה לפני שחרור: ``0.4.9-nightly.2017.1.17+commit.6ecb4aa3.Emscripten.clang``

מידע חשוב על גירסאות
======================================

לאחר ביצוע שחרור גרסה, רמת מספר גרסת התיקונים מקודמת (bump), מכיוון שאנו מניחים שרק
שינויים ברמת התיקון יבואו בהמשך. כאשר השינויים מתמזגים, יש לקדם את מספר הגרסה בהתאם
ל-SemVer וחומרת השינוי. לבסוף, תמיד נוצרת גרסה עם בניית גרסה
של ה-nightly הנוכחי, אך ללא מפרט ``prerelease``.


דוגמה:

1. הגרסה 0.4.0 נוצרת.
2. לבניית nightly יש גרסה של 0.4.1 מעתה ואילך.
3. מוכנסים שינויים בלתי פוסקים --> אין שינוי בגרסה.
4. הוצג שינוי שמשנה התנהגות --> הגרסה מקודמת ל-0.5.0.
5. הגרסה 0.5.0 נוצרת.

התנהגות זו עובדת היטב עם :ref:`version pragma <version_pragma>`.
