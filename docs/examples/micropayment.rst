********************
ערוץ מיקרו-תשלומים
********************

בחלק זה נלמד כיצד לבנות יישום לדוגמה
של ערוץ תשלום. הערוץ משתמש בחתימות קריפטוגרפיות כדי ליצור
העברות חוזרות ונשנות של איתר בין אותם הצדדים באופן מאובטח, מיידי
וללא עמלות טרנזקציה. לצורך הדוגמה, עלינו להבין כיצד
לחתום ולאמת חתימות ולהגדיר את ערוץ התשלום.

יצירה ואימות של חתימות
=================================

תארו לעצמכם שאליס רוצה לשלוח קצת איתר לבוב, כלומר,
אליס היא השולחת ובוב הוא הנמען.

אליס צריכה לשלוח רק הודעות חתומות קריפטוגרפית מחוץ לשרשרת
(למשל באמצעות דואר אלקטרוני) לבוב בדומה לכתיבת צ'קים.

אליס ובוב משתמשים בחתימות כדי לאשר טרזקציות, מה שאפשרי בחוזים חכמים באיתריום.
אליס תבנה חוזה חכם פשוט שיאפשר לה לשדר איתר, אבל במקום לקרוא לפונקציה בעצמה
כדי ליזום תשלום, היא תאפשר לבוב לעשות זאת, ולכן תשלם את עמלת הטרנזקציה.

החוזה יפעל באופן הבא:

    1. אליס מתקינה את החוזה ``ReceiverPays``, ומצרפת מספיק איתר כדי לכסות את התשלומים שיבוצעו.
 	2. אליס מאשרת תשלום על ידי חתימה על הודעה עם המפתח הפרטי שלה.
 	3. אליס שולחת לבוב  ההודעה חתומה קריפטוגרפית. אין צורך לשמור את ההודעה בסוד
    	(הסבר בהמשך), ומנגנון השליחה אינו משנה.
 	4. בוב תובע את התשלום שלו על ידי הצגת ההודעה החתומה לחוזה החכם, זה מאמת את
    	האותנטיות של ההודעה ולאחר מכן משחרר את הכספים.

יצירת החתימה
----------------------

אליס לא צריכה ליצור אינטראקציה עם רשת איתריום
כדי לחתום על העסקה, התהליך לא מקוון לחלוטין.
במדריך זה נחתום על הודעות בדפדפן
באמצעות `web3.js <https://github.com/web3/web3.js>`_
ו-`MetaMask <https://metamask.io>`_, תוך שימוש בשיטה המתוארת
ב-`EIP-712 <https://github.com/ethereum/EIPs/pull/712>`_,
מכיוון שהיא מספקת מספר יתרונות אבטחה נוספים.

.. code-block:: javascript

    /// Hashing first makes things easier
    var hash = web3.utils.sha3("message to sign");
    web3.eth.personal.sign(hash, web3.eth.defaultAccount, function () { console.log("Signed"); });

.. note::
  ``web3.eth.personal.sign`` מציין את אורך
   ההודעה לנתונים החתומים. מכיוון שדבר ראשון אנחנו מבצעים hash, ההודעה
   תמיד תהיה באורך של 32 בתים בדיוק, ולכן אורך
   הקידומת הזו תמיד זהה.

על מה לחתום
------------

עבור חוזה המבצע תשלומים, ההודעה החתומה חייבת לכלול:

     1. כתובת הנמען.
     2. הסכום להעברה.
     3. הגנה מפני התקפות חוזרות.

התקפה חוזרת היא כאשר נעשה שימוש חוזר בהודעה חתומה
לאישור פעולה נוספת. כדי להימנע מהתקפות חוזרות
אנחנו משתמשים באותה טכניקה כמו בטרנזקציות איתריום עצמן,
ה-nonce, שהוא מספר הטרנזקציות שנשלחו על ידי
חשבון. החוזה החכם בודק אם נעשה שימוש ב-nonce מספר פעמים.

סוג אחר של התקפה חוזרת יכול להתרחש כאשר הבעלים
מתקין חוזה חכם ``ReceiverPays``, עושה כמה
תשלומים, ולאחר מכן משמיד את החוזה. אחר כך בעל החוזה מחליט
להתקין שוב את החוזה החכם ``RecipientPays``, אבל
החוזה החדש אינו מכיר את ה-nonces ששימשו
בהתקנה הקודמת, כך שהתוקף יוכל להשתמש שוב בהודעות הישנות.

אליס יכולה להגן מפני התקפה זו על ידי הכללת
כתובת החוזה בהודעה, ורק הודעות שמכילות
את כתובת החוזה עצמו תתקבל. אתם יכולים למצוא
דוגמה לכך בשתי השורות הראשונות של הפונקציה ``claimPayment()``
של החוזה המלא בסוף סעיף זה.

אריזת ארגומנטים
-----------------

כעת, לאחר שזיהינו איזה מידע לכלול בהודעה החתומה,
אנחנו מוכנים לחבר את ההודעה, לבצע עליה hash ולחתום עליה. לצורך הפשטות,
אנחנו משרשרים את הנתונים. הספרייה `ethereumjs-abi <https://github.com/ethereumjs/ethereumjs-abi>`_
מספקת פונקציה בשם ``soliditySHA3`` המחקה את ההתנהגות של
הפונקציה ``keccak256`` של סולידיטי ומופעלת על ארגומנטים המקודדים באמצעות ``abi.encodePacked``.
להלן פונקציית JavaScript שיוצרת את החתימה המתאימה לדוגמא של ``ReceiverPays``:

.. code-block:: javascript

    // recipient is the address that should be paid.
    // amount, in wei, specifies how much ether should be sent.
    // nonce can be any unique number to prevent replay attacks
    // contractAddress is used to prevent cross-contract replay attacks
    function signPayment(recipient, amount, nonce, contractAddress, callback) {
        var hash = "0x" + abi.soliditySHA3(
            ["address", "uint256", "uint256", "address"],
            [recipient, amount, nonce, contractAddress]
        ).toString("hex");

        web3.eth.personal.sign(hash, web3.eth.defaultAccount, callback);
    }

שחזור חותם ההודעות בסולידיטי
-----------------------------------------

באופן כללי, חתימות ECDSA מורכבות משני פרמטרים,
``r`` ו-``s``. חתימות באיתריום כוללות
פרמטר שלישי בשם ``v``, שבו אתם יכולים להשתמש כדי לאמת איזה
מפתח פרטי של החשבון שימש לחתימה על ההודעה, וכן לאמת את
שולח הטרנזקציה. סולידיטי מספקת
function :ref:`ecrecover <פונקציות-מתמטיות-ו-קריפטוגרפיות>` מובנות
שמקבלות הודעה יחד עם הפרמטרים ``r``, ``s`` ו-``v``
ומחזירות את הכתובת ששימשה לחתימה על ההודעה.

חילוץ פרמטרי החתימה
-----------------------------------

Sחתימות המיוצרות על ידי web3.js הן שרשור של ``r``,
``s`` ו-``v``. לכן הצעד הראשון הוא לפצל את הפרמטרים הללו.
אתם יכולים לעשות זאת בצד הלקוח, אבל הביצוע בתוך
החוזה החכם היא שאתם צריכים לשלוח רק
פרמטר חתימה אחד ולא שלושה. פיצול של מערך בתים
לחלקים המרכיבים אותו הוא בעייתי, ולכן אנחנו משתמשים בפונקציה
:doc:`הרכבה מוטבעת <assembly>` כדי לבצע את העבודה ב-``splitSignature``
(הפונקציה השלישית בחוזה המלא בסוף סעיף זה).

חישוב ה-hash של ההודעה
--------------------------

החוזה החכם צריך לדעת בדיוק אילו פרמטרים נחתמו, ולכן הוא חייב
ליצור מחדש את ההודעה מהפרמטרים ולהשתמש בה לאימות חתימה.
הפונקציות ``prefixed`` ו``recoverSigner`` עושות זאת בפונקציה ``claimPayment``.

החוזה המלא
-----------------

.. code-block:: solidity
    :force:

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.7.0 <0.9.0;
    // This will report a warning due to deprecated selfdestruct
    contract ReceiverPays {
        address owner = msg.sender;

        mapping(uint256 => bool) usedNonces;

        constructor() payable {}

        function claimPayment(uint256 amount, uint256 nonce, bytes memory signature) external {
            require(!usedNonces[nonce]);
            usedNonces[nonce] = true;

            // this recreates the message that was signed on the client
            bytes32 message = prefixed(keccak256(abi.encodePacked(msg.sender, amount, nonce, this)));

            require(recoverSigner(message, signature) == owner);

            payable(msg.sender).transfer(amount);
        }

        /// destroy the contract and reclaim the leftover funds.
        function shutdown() external {
            require(msg.sender == owner);
            selfdestruct(payable(msg.sender));
        }

        /// signature methods.
        function splitSignature(bytes memory sig)
            internal
            pure
            returns (uint8 v, bytes32 r, bytes32 s)
        {
            require(sig.length == 65);

            assembly {
                // first 32 bytes, after the length prefix.
                r := mload(add(sig, 32))
                // second 32 bytes.
                s := mload(add(sig, 64))
                // final byte (first byte of the next 32 bytes).
                v := byte(0, mload(add(sig, 96)))
            }

            return (v, r, s);
        }

        function recoverSigner(bytes32 message, bytes memory sig)
            internal
            pure
            returns (address)
        {
            (uint8 v, bytes32 r, bytes32 s) = splitSignature(sig);

            return ecrecover(message, v, r, s);
        }

        /// builds a prefixed hash to mimic the behavior of eth_sign.
        function prefixed(bytes32 hash) internal pure returns (bytes32) {
            return keccak256(abi.encodePacked("\x19Ethereum Signed Message:\n32", hash));
        }
    }


כתיבת ערוץ תשלומים פשוט
================================

אליס בונה כעת יישום פשוט אך מלא של ערוץ תשלום.
ערוצי תשלום עושים שימוש בחתימות קריפטוגרפיות
להעברות חוזרות ונשנות של איתר בצורה מאובטחת, מיידית וללא עמלות עסקה.

מהו ערוץ תשלומים?
--------------------------

ערוצי תשלום מאפשרים למשתתפים לבצע העברות חוזרות ונשנות של איתר
ללא שימוש בטרנזקציות. זאת אומרת שאתם יכולים למנוע את העיכובים
והעמלות הקשורות לטרנזקציות. כאן נחקור ערוץ תשלום
חד כיווני פשוט בין שני צדדים (אליס ובוב). התהליך בערוץ כולל שלושה שלבים:

 	1. אליס מממנת חוזה חכם עם איתר. פעולה זו "פותחת" את ערוץ התשלום.
 	2. אליס חותמת על הודעות שמפרטות את התשלושם מיועד לנמען. שלב זה חוזר על עצמו עבור כל תשלום.
 	3. בוב "סוגר" את ערוץ התשלום, מושך את חלקו מהאיתר ושולח את השארית בחזרה לאליס.

.. note::
   רק שלבים 1 ו-3 דורשים טרנזקציות באיתריום, בשלב 2 השולחת
   משדרת הודעה חתומה קריפטוגרפית לנמענן דרך שיטות שהן
   מחוץ לרשת (למשל אימייל). המשמעות היא שרק שתי טרנזקציות
   נדרשות לתמיכה במספר כלשהו של העברות.

מובטח לבוב שיקבל את הכספים שלו כי החוזה החכם מפקח על
האיתר ומכבד הודעות חתומות תקפות. החוזה החכם גם אוכף
פסקי זמן, כך שלאליס מובטח שבסופו של דבר שתוכל לשחזר את הכסף שלה גם אם
הנמען מסרב לסגור את הערוץ. משתתפי ערוץ התשלום הם שמחליטים
כמה זמן לשמור אותו פתוח. לעסקה קצרת מועד,
כגון תשלום לרשת קפה עבור כל דקה של גישה לרשת,
הערוץ עשוי להישמר פתוח לזמן מוגבל. מצד שני, עבור
תשלום חוזר, כמו תשלום שכר שעתי לעובד,
ניתן לשמור את ערוץ התשלום פתוח מספר חודשים או שנים.

פתיחת ערוץ התשלום
---------------------------

כדי לפתוח את ערוץ התשלום, אליס מתקינה את החוזה החכם, מצרפת את
האיתר להפקדה וקובעת את הנמען המיועד ואת
משך הזמן המקסימלי לקיום הערוץ. זו הפונקציה
``SimplePaymentChannel`` בחוזה, בסוף סעיף זה.

ביצוע תשלומים
---------------

Aאליס מבצעת תשלומים על ידי שליחת הודעות חתומות לבוב.
שלב זה מתבצע כולו מחוץ לרשת איתריום.
ההודעות נחתמות בצורה קריפטוגרפית על ידי השולח ולאחר מכן מועברות ישירות לנמען.

כל הודעה כוללת את המידע הבא:

 	* כתובת החוזה החכם, המשמשת למניעת התקפות שידור חוזר בין חוזים.
 	* הסכום הכולל של האיתר שחייבים לנמען עד כה.

ערוץ תשלום נסגר רק פעם אחת, בתום סדרת ההעברות.
לכן, רק אחת מההודעות שנשלחו נפדת. זו הסיבה לכך
שכל הודעה מציינת סכום כולל מצטבר של איתר שחייבים, במקום את
סכום המיקרו-תשלום הבודד. לכן, באופן טבעי הנמען יבחר
לממש את ההודעה האחרונה כי זו ההודעה עם הסכום הגבוה ביותר.
אין צורך יותר ב-nonce לכל הודעה, כי החוזה החכם
מכבד הודעה אחת בלבד. הכתובת של החוזה החכם עדיין בשימוש
כדי למנוע שימוש בהודעה המיועדת לערוץ תשלום אחד בערוץ אחר.

הנה קוד ה-JavaScript שהשתנה כדי לחתום באופן קריפטוגרפי על הודעה מהסעיף הקודם:

.. code-block:: javascript

    function constructPaymentMessage(contractAddress, amount) {
        return abi.soliditySHA3(
            ["address", "uint256"],
            [contractAddress, amount]
        );
    }

    function signMessage(message, callback) {
        web3.eth.personal.sign(
            "0x" + message.toString("hex"),
            web3.eth.defaultAccount,
            callback
        );
    }

    // contractAddress is used to prevent cross-contract replay attacks.
    // amount, in wei, specifies how much Ether should be sent.

    function signPayment(contractAddress, amount, callback) {
        var message = constructPaymentMessage(contractAddress, amount);
        signMessage(message, callback);
    }


סגירת ערץ התשלום
---------------------------

כאשר בוב מוכן לקבל את הכספים שלו, זהו הזמן
לסגור את ערוץ התשלום על ידי קריאה לפונקציה ``close`` בחוזה החכם.
סגירת הערוץ משלמת לנמען את האיתר שחייבים לו,
משמידה את החוזה, ושולחת את כל האיתר שנותר בחזרה לאליס.
לסגירת הערוץ, בוב צריך לספק הודעה חתומה על ידי אליס.

על החוזה החכם לוודא שההודעה מכילה חתימה תקפה מהשולח.
התהליך לביצוע אימות זה זהה לתהליך בו משתמש הנמען.
פונקציות Solidity ``isValidSignature`` ו-``recoverSigner`` פועלות בדיוק כמו
המקבילות שלהן ב-JavaScript בסעיף הקודם, כאשר הפונקציה האחרונה נלקחה מהחוזה ``ReceiverPays``.

רק נמען ערוץ התשלום יכול להתקשר לפונקציית ``close``,
שמעבירה באופן טבעי את הודעת התשלום העדכנית ביותר, מכיוון שהודעה הזו
מכילה את סך החוב הגבוה ביותר. אם השולחים היו מורשים לקרוא לפונקציה זו,
הם היו יכולים לספק הודעה עם סכום נמוך יותר ולרמות את הנמען.

הפונקציה מאמתת שההודעה החתומה תואמת את הפרמטרים הנתונים.
אם הכל מסתדר, נשלח לנמען חלקו באיתר,
והשולח מקבל את השאר באמצעות ``השמדה עצמית - selfdestruct``.
אתם יכולים לראות את הפונקציה ``close`` בחוזה המלא.

פקיעת תוקף ערוץ
-------------------

בוב יכול לסגור את ערוץ התשלום בכל זמן, אך אם לא יעשה זאת,
אליס צריכה דרך לגבות את כספי הנאמנות שלה. נקבע זמן *פקיעת תוקף*
בזמן התקנת החוזה. ברגע שמגיע הזמן הזה, אליס יכולה להתקשר
``claimTimeout`` כדי להחזיר את הכספים שלה. אתם יכולים לראות את הפונקציה ``claimTimeout`` בחוזה המלא.

לאחר הקריאה לפונקציה זו, בוב לא יכול יותר לקבל איתר,
לכן חשוב שבוב יסגור את הערוץ לפני פקיעת התוקף.

החוזה המלא
-----------------

.. code-block:: solidity
    :force:

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.7.0 <0.9.0;
    // This will report a warning due to deprecated selfdestruct
    contract SimplePaymentChannel {
        address payable public sender;      // The account sending payments.
        address payable public recipient;   // The account receiving the payments.
        uint256 public expiration;  // Timeout in case the recipient never closes.

        constructor (address payable recipientAddress, uint256 duration)
            payable
        {
            sender = payable(msg.sender);
            recipient = recipientAddress;
            expiration = block.timestamp + duration;
        }

        /// the recipient can close the channel at any time by presenting a
        /// signed amount from the sender. the recipient will be sent that amount,
        /// and the remainder will go back to the sender
        function close(uint256 amount, bytes memory signature) external {
            require(msg.sender == recipient);
            require(isValidSignature(amount, signature));

            recipient.transfer(amount);
            selfdestruct(sender);
        }

        /// the sender can extend the expiration at any time
        function extend(uint256 newExpiration) external {
            require(msg.sender == sender);
            require(newExpiration > expiration);

            expiration = newExpiration;
        }

        /// if the timeout is reached without the recipient closing the channel,
        /// then the Ether is released back to the sender.
        function claimTimeout() external {
            require(block.timestamp >= expiration);
            selfdestruct(sender);
        }

        function isValidSignature(uint256 amount, bytes memory signature)
            internal
            view
            returns (bool)
        {
            bytes32 message = prefixed(keccak256(abi.encodePacked(this, amount)));

            // check that the signature is from the payment sender
            return recoverSigner(message, signature) == sender;
        }

        /// All functions below this are just taken from the chapter
        /// 'creating and verifying signatures' chapter.

        function splitSignature(bytes memory sig)
            internal
            pure
            returns (uint8 v, bytes32 r, bytes32 s)
        {
            require(sig.length == 65);

            assembly {
                // first 32 bytes, after the length prefix
                r := mload(add(sig, 32))
                // second 32 bytes
                s := mload(add(sig, 64))
                // final byte (first byte of the next 32 bytes)
                v := byte(0, mload(add(sig, 96)))
            }

            return (v, r, s);
        }

        function recoverSigner(bytes32 message, bytes memory sig)
            internal
            pure
            returns (address)
        {
            (uint8 v, bytes32 r, bytes32 s) = splitSignature(sig);

            return ecrecover(message, v, r, s);
        }

        /// builds a prefixed hash to mimic the behavior of eth_sign.
        function prefixed(bytes32 hash) internal pure returns (bytes32) {
            return keccak256(abi.encodePacked("\x19Ethereum Signed Message:\n32", hash));
        }
    }


.. note::
   הפונקציה ``splitSignature`` לא משתמשת בכל בדיקות האבטחה.
   יישום אמיתי צריך להשתמש בספרייה שנבדקה בקפדנות יותר,
   כגון `גרסת openzepplin <https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/cryptography/ECDSA.sol>`_ של קוד זה.

אימות תשלום
------------------

שלא כמו בסעיף הקודם, הודעות בערוץ תשלום אינן
נפדות מיד. הנמענים עוקבים אחר ההודעה האחרונה
ומממשים אותה כשמגיע הזמן לסגור את ערוץ התשלום. לכן
קריטי שהנמענים יבצעו אימות משלהם של כל הודעה.
אחרת אין ערובה שהם יוכלו לקבל תשלום
בסוף.

על הנמענים לאמת כל הודעה באמצעות התהליך הבא:

 	1. וידוא שכתובת החוזה בהודעה תואמת לערוץ התשלום.
 	2. וידוא שהסך הכולל החדש הוא הסכום הצפוי.
 	3. וידוא שהסכום החדש אינו עולה על סכום האיתר שהופקד.
 	4. וידוא שהחתימה תקפה ומגיעה משולח ערוץ התשלום.

נשתמש בספרייה `ethereumjs-util <https://github.com/ethereumjs/ethereumjs-util>`_
כדי לכתוב את האימות הזה. השלב האחרון יכול להיעשות במספר דרכים,
ואנחנו משתמשים ב-JavaScript. הקוד הבא משתמש בפונקציה ``constructPaymentMessage`` מ-**קוד ה-JavaScript** לחתימה למעלה:

.. code-block:: javascript

    // this mimics the prefixing behavior of the eth_sign JSON-RPC method.
    function prefixed(hash) {
        return ethereumjs.ABI.soliditySHA3(
            ["string", "bytes32"],
            ["\x19Ethereum Signed Message:\n32", hash]
        );
    }

    function recoverSigner(message, signature) {
        var split = ethereumjs.Util.fromRpcSig(signature);
        var publicKey = ethereumjs.Util.ecrecover(message, split.v, split.r, split.s);
        var signer = ethereumjs.Util.pubToAddress(publicKey).toString("hex");
        return signer;
    }

    function isValidSignature(contractAddress, amount, signature, expectedSigner) {
        var message = prefixed(constructPaymentMessage(contractAddress, amount));
        var signer = recoverSigner(message, signature);
        return signer.toLowerCase() ==
            ethereumjs.Util.stripHexPrefix(expectedSigner).toLowerCase();
    }
