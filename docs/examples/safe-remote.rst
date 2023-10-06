.. index:: purchase, remote purchase, escrow

********************
רכישה בטוחה מרחוק
********************

רכישת מוצרים מרחוק דורשת כיום מספר גורמים שצריכים לסמוך זה על זה.
התצורה הפשוטה ביותר כוללת מוכר וקונה. הקונה רוצה לקבל
פריט מהמוכר והמוכר היה רוצה לקבל פיצוי כלשהו, למשל איתר,
בחזרה. החלק הבעייתי הוא המשלוח: אין דרך לקבוע בוודאות
שהפריט הגיע לקונה.

ישנן מספר דרכים לפתור בעיה זו, אך כולן נופלות בדרך זו או אחרת.
בדוגמה הבאה, שני הצדדים צריכים להכניס פי שניים מערכו של הפריט לתוך
חוזה כנאמנות. ברגע שזה קרה, האיתר יישאר נעול בתוך
החוזה עד שהקונה יאשר שקיבל את הפריט. אחרי זה,
הקונה מקבל את הסכום שנשאר מהפיקדון (מחצית מהפיקדון שלו) והמוכר מקבל פי שלוש
(ההפקדה שלו בתוספת ערך הפריט). הרעיון הוא
שלשני הצדדים יש תמריץ לפתור את המצב כי אחרת
האיתר שלהם נעול לנצח.

החוזה הזה כמובן לא פותר את הבעיה, אבל נותן סקירה של איך
אפשר להשתמש במבנים דמויי מכונת-מצבים בתוך חוזה.


.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity ^0.8.4;
    contract Purchase {
        uint public value;
        address payable public seller;
        address payable public buyer;

        enum State { Created, Locked, Release, Inactive }
        // The state variable has a default value of the first member, `State.created`
        State public state;

        modifier condition(bool condition_) {
            require(condition_);
            _;
        }

        /// Only the buyer can call this function.
        error OnlyBuyer();
        /// Only the seller can call this function.
        error OnlySeller();
        /// The function cannot be called at the current state.
        error InvalidState();
        /// The provided value has to be even.
        error ValueNotEven();

        modifier onlyBuyer() {
            if (msg.sender != buyer)
                revert OnlyBuyer();
            _;
        }

        modifier onlySeller() {
            if (msg.sender != seller)
                revert OnlySeller();
            _;
        }

        modifier inState(State state_) {
            if (state != state_)
                revert InvalidState();
            _;
        }

        event Aborted();
        event PurchaseConfirmed();
        event ItemReceived();
        event SellerRefunded();

        // Ensure that `msg.value` is an even number.
        // Division will truncate if it is an odd number.
        // Check via multiplication that it wasn't an odd number.
        constructor() payable {
            seller = payable(msg.sender);
            value = msg.value / 2;
            if ((2 * value) != msg.value)
                revert ValueNotEven();
        }

        /// Abort the purchase and reclaim the ether.
        /// Can only be called by the seller before
        /// the contract is locked.
        function abort()
            external
            onlySeller
            inState(State.Created)
        {
            emit Aborted();
            state = State.Inactive;
            // We use transfer here directly. It is
            // reentrancy-safe, because it is the
            // last call in this function and we
            // already changed the state.
            seller.transfer(address(this).balance);
        }

        /// Confirm the purchase as buyer.
        /// Transaction has to include `2 * value` ether.
        /// The ether will be locked until confirmReceived
        /// is called.
        function confirmPurchase()
            external
            inState(State.Created)
            condition(msg.value == (2 * value))
            payable
        {
            emit PurchaseConfirmed();
            buyer = payable(msg.sender);
            state = State.Locked;
        }

        /// Confirm that you (the buyer) received the item.
        /// This will release the locked ether.
        function confirmReceived()
            external
            onlyBuyer
            inState(State.Locked)
        {
            emit ItemReceived();
            // It is important to change the state first because
            // otherwise, the contracts called using `send` below
            // can call in again here.
            state = State.Release;

            buyer.transfer(value);
        }

        /// This function refunds the seller, i.e.
        /// pays back the locked funds of the seller.
        function refundSeller()
            external
            onlySeller
            inState(State.Release)
        {
            emit SellerRefunded();
            // It is important to change the state first because
            // otherwise, the contracts called using `send` below
            // can call in again here.
            state = State.Inactive;

            seller.transfer(3 * value);
        }
    }
