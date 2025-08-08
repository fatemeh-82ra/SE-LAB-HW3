

# گزارش آزمایشگاه TDD: مدیریت حساب

این گزارش فرآیند یافتن و رفع یک خطا در برنامه مدیریت حساب بانکی و پیاده‌سازی یک قابلیت جدید با استفاده از روش توسعه مبتنی بر آزمون (TDD) را مستند می‌کند.

-----

## بخش اول: کشف و رفع خطا

این بخش فرآیند شناسایی خطایی که توسط مجموعه تست‌های اولیه نادیده گرفته شده بود، نوشتن یک تست جدید برای آشکارسازی آن و اصلاح کد را پوشش می‌دهد.

### پرسش اول: خطا و نحوه تشخیص آن

خطا این است که متد `calculateBalance` به اشتباه اجازه می‌دهد یک برداشت منجر به موجودی منفی حساب شود که این موضوع الزامات برنامه را نقض می‌کند. این خطا به این دلیل تشخیص داده نشده بود که مجموعه تست اولیه فقط سناریوهایی را پوشش می‌داد که در آن‌ها حساب موجودی کافی داشت و نتوانسته بود مورد مرزی (edge case) یعنی برداشت مبلغی بیش از موجودی فعلی را آزمایش کند.

### پرسش دوم: آشکارسازی و رفع خطا

#### مورد آزمون جدید برای آشکارسازی خطا (مرحله «قرمز»)

پس از یافتن خطا، اولین گام نوشتن تستی بود که به طور مشخص به دلیل وجود همین خطا ناموفق شود. تست زیر به فایل `AccountBalanceCalculatorTest.java` اضافه شد. این تست بررسی می‌کند که آیا برداشت مبلغی بزرگتر از موجودی فعلی به درستی مدیریت (یعنی نادیده گرفته) می‌شود یا خیر.


```java
@Test
void testWithdrawalExceedingBalanceShouldBeIgnored() {
    // Arrange: A list of transactions where withdrawal is more than the balance
    List<Transaction> transactions = Arrays.asList(
            new Transaction(TransactionType.DEPOSIT, 100),
            new Transaction(TransactionType.WITHDRAWAL, 150)
    );

    // Act: Calculate the balance
    int balance = AccountBalanceCalculator.calculateBalance(transactions);

    // Assert: The balance should remain 100, as the invalid withdrawal is ignored.
    assertEquals(100, balance);
}
```

هنگامی که این تست بر روی کد اصلی اجرا می‌شود، همانطور که انتظار می‌رود، ناموفق است.
<img width="1920" height="993" alt="Screenshot (2513)" src="https://github.com/user-attachments/assets/8283ef44-4940-455d-987f-1d77d8d3acd1" />

#### رفع خطا (مرحله «سبز»)

برای رفع خطا، متد `calculateBalance` در فایل `AccountBalanceCalculator.java` طوری تغییر داده شد که قبل از پردازش یک برداشت، یک بررسی انجام دهد.


```java
// Method to calculate balance based on transactions
public static int calculateBalance(List<Transaction> transactions) {
    int balance = 0;
    for (Transaction t : transactions) {
        if (t.getType() == TransactionType.DEPOSIT) {
            balance += t.getAmount();
        } else if (t.getType() == TransactionType.WITHDRAWAL) {
            // FIX: Only subtract if the balance is sufficient
            if (balance >= t.getAmount()) {
                balance -= t.getAmount();
            }
        }
    }
    return balance;
}
```

پس از اعمال این اصلاح، تمام تست‌ها، از جمله تست جدید، با موفقیت پاس می‌شوند.
<img width="1920" height="1000" alt="Screenshot (2514)" src="https://github.com/user-attachments/assets/dce7c674-915c-4f17-b4f0-316330e55541" />

#### مورد آزمون جدید برای آشکارسازی خطا (مرحله «قرمز»)
خطای بعدی در تاریخچه تراکنش ها دیده میشه که تست زیر اون رو بررسی می کنه، درواقع این تست بررسی می‌کند که وقتی چند تراکنش واریز انجام می‌ شود، متد محاسبه موجودی باید آن‌ها را درست و کامل در تاریخچه ذخیره کند.
```java
@Test
   void testTransactionHistoryAfterDeposits() {
       // Perform deposits
       List<Transaction> transactions = Arrays.asList(
               new Transaction(TransactionType.DEPOSIT, 100),
               new Transaction(TransactionType.DEPOSIT, 200)
       );

       // Calculate balance, which will also add transactions to the history
       AccountBalanceCalculator.calculateBalance(transactions);

       // Ensure the transaction history contains the correct transactions
       List<Transaction> history = AccountBalanceCalculator.getTransactionHistory();
       assertEquals(2, history.size(), "Transaction history should contain 2 transactions");

       // Check if the transactions are correctly recorded
       assertTrue(history.containsAll(transactions), "Transaction history should contain both deposit transactions");
   }
```
هنگامی که این تست بر روی کد اصلی اجرا می‌شود، همانطور که انتظار می‌رود، ناموفق است.
<img width="1652" height="736" alt="image" src="https://github.com/user-attachments/assets/2fd1a7d0-ec48-411a-8eb2-a2c08b1675dc" />
#### رفع خطا (مرحله «سبز»)
برای رفع خطا، متد `calculateBalance` در فایل `AccountBalanceCalculator.java` طوری تغییر داده شد که لیست تراکنش ها را به روزرسانی کند.
```java
public static int calculateBalance(List<Transaction> transactions) {

        // FIX: Ensure transaction history is updated with the new transactions
        transactionHistory.addAll(transactions);

        int balance = 0;
        for (Transaction t : transactions) {
            if (t.getType() == TransactionType.DEPOSIT) {
                balance += t.getAmount();
            } else if (t.getType() == TransactionType.WITHDRAWAL) {
                // FIX: Only subtract if the balance is sufficient
                if (balance >= t.getAmount()) {
                    balance -= t.getAmount();
                }
            }
        }
        return balance;
    }
```
بنابراین شاهد پاس شدن تست مذکور هستیم:
<img width="1432" height="659" alt="image" src="https://github.com/user-attachments/assets/5f5e238d-a8f4-4029-af1c-78cf469d10eb" />

پس از انجام تغییرات بالا مشاهده می شود که تست بعدی که تاثیر نوع تراکنش بر ثبت آن در تاریخچه را بررسی می کند نیز پاس می شود
<img width="1458" height="608" alt="image" src="https://github.com/user-attachments/assets/8ddb6f23-4af9-4163-8383-1437f767a144" />


#### مورد آزمون سوم:
```java
@Test
    void testTransactionHistoryShouldContainOnlyLastCalculationTransactions() {
//        // Perform first calculation with some transactions
        List<Transaction> firstTransactions = Arrays.asList(
                new Transaction(TransactionType.DEPOSIT, 100),
                new Transaction(TransactionType.WITHDRAWAL, 50)
        );

        AccountBalanceCalculator.calculateBalance(firstTransactions);
//
//        // Ensure the transaction history contains the correct transactions from the first calculation
       List<Transaction> historyAfterFirstCalc = AccountBalanceCalculator.getTransactionHistory();
        assertEquals(2, historyAfterFirstCalc.size(), "Transaction history should contain 2 transactions after the first calculation");
        assertTrue(historyAfterFirstCalc.containsAll(firstTransactions), "Transaction history should contain the first set of transactions");
//
//        // Perform second calculation with different transactions
        List<Transaction> secondTransactions = Arrays.asList(
                new Transaction(TransactionType.DEPOSIT, 200),
                new Transaction(TransactionType.WITHDRAWAL, 150)
        );
//
        AccountBalanceCalculator.calculateBalance(secondTransactions);
//
//        // Ensure the transaction history only contains transactions from the second calculation
        List<Transaction> historyAfterSecondCalc = AccountBalanceCalculator.getTransactionHistory();
        assertEquals(2, historyAfterSecondCalc.size(), "Transaction history should contain 2 transactions after the second calculation");
        assertTrue(historyAfterSecondCalc.containsAll(secondTransactions), "Transaction history should contain the second set of transactions");
        assertFalse(historyAfterSecondCalc.containsAll(firstTransactions), "Transaction history should not contain the first set of transactions after the second calculation");
    }
}
```
هنگامی که این تست بر روی کد اصلی اجرا می‌شود، همانطور که انتظار می‌رود، ناموفق است.

![IMG_7445 png](https://github.com/user-attachments/assets/9fc9f389-e725-4b09-a1eb-40257c6bb0df)

#### رفع خطا:

برای رفع خطا، باید قبل از خط
```java
// FIX: Ensure transaction history is updated with the new transactionstransactionHistory.addAll(transactions);
```
این کد را بگذاریم:
```java
clearTransactionHistory();
```
پس از اعمال این تغییر، شاهد پاس شدن تست هستیم:
![IMG_7447 png](https://github.com/user-attachments/assets/5871c179-9594-4cfe-8cc1-fbd2f56afd7c)

### پرسش سوم: مشکلات نوشتن تست پس از کدنویسی

نوشتن تست پس از کدنویسی می‌تواند منجر به یک سوگیری ناخودآگاه شود که در آن، تست‌ها برای تأیید رفتار موجود کد نوشته می‌شوند، نه برای به چالش کشیدن الزامات آن. این موضوع اغلب باعث نادیده گرفتن موارد مرزی و خطاها می‌شود، همانطور که در مجموعه تست اولیه این آزمایشگاه مشاهده شد. همچنین، شناسایی تأثیر دقیق یک تغییر در کد را دشوارتر می‌کند، زیرا هیچ تست ناموفق اولیه‌ای برای تأیید وجود خطا وجود ندارد.


۱. **ایجاد حس امنیت کاذب:**  نوشتن تست پس از کدنویسی می‌تواند یک حس امنیت کاذب ایجاد کند؛ توسعه‌دهندگان ممکن است با دیدن پاس شدن تست‌های اولیه به اشتباه تصور کنند که برنامه بدون باگ است.

۲. **انطباق تست با کد به جای نیازمندی‌ها:**  توسعه‌دهندگان ممکن است به طور ناخودآگاه تست‌هایی بنویسند که با پیاده‌سازی ناقص و پراشکال کد مطابقت دارد، به جای اینکه آن را در برابر نیازمندی‌های واقعی بسنجند.

۳. **پیچیدگی تست‌نویسی:**  کدی که بدون فکر کردن به تست نوشته شده باشد، ممکن است بعداً تست‌نویسی را دشوار یا پیچیده کند و باعث شود بخش‌هایی از کد بدون تست باقی بمانند.

#### پرسش چهارم: مزایای نوشتن موارد آزمون پیش از کدنویسی

۱. **شفاف‌سازی نیازمندی‌ها پیش از شروع**  با نوشتن تست‌ها قبل از کدنویسی، توسعه‌دهنده مجبور می‌شود دقیقاً بفهمد که چه چیزی باید ساخته شود. این باعث می‌شود نیازمندی‌ها بهتر تحلیل شوند و ابهام‌ها زودتر شناسایی شوند.
⸻
۲. **طراحی بهتر و ماژولارتر**  نوشتن تست‌ها پیش از پیاده‌سازی باعث می‌شود توسعه‌دهنده به طراحی ماژولار و قابل تست فکر کند. این منجر به کدهایی می‌شود که کوپلینگ کمتر و مسئولیت‌های واضح‌تر دارند.
⸻
۳. **پیشگیری از باگ‌ها به جای رفع آن‌ها**  با مشخص بودن آزمون‌ها از ابتدا، توسعه‌دهنده دقیقاً می‌داند کد باید چه رفتاری داشته باشد و از بروز بسیاری از خطاها در همان ابتدا جلوگیری می‌شود.
⸻
۴. **بازخورد سریع و اعتماد به تغییر**  هنگامی که تست‌ها از قبل وجود دارند، هر تغییری در کد به سرعت می‌تواند بررسی شود. این باعث افزایش اعتماد به ریفکتورینگ و توسعه‌های آینده می‌شود.
⸻
۵. **مستندسازی خودکار**  موارد آزمون مانند یک مستند زنده برای رفتار سیستم هستند. دیگران با نگاه به تست‌ها می‌توانند بفهمند یک تابع یا ماژول قرار است چه کاری انجام دهد.
⸻
۶. **کاهش هزینه کلی توسعه**  اگرچه در ابتدا زمان بیشتری برای نوشتن تست‌ها صرف می‌شود، اما به دلیل کاهش خطاها، زمان کمتر در اشکال‌زدایی، و افزایش کیفیت، هزینه کلی پروژه کاهش می‌یابد.
