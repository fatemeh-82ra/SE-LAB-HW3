

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



### پرسش سوم: مشکلات نوشتن تست پس از کدنویسی

نوشتن تست پس از کدنویسی می‌تواند منجر به یک سوگیری ناخودآگاه شود که در آن، تست‌ها برای تأیید رفتار موجود کد نوشته می‌شوند، نه برای به چالش کشیدن الزامات آن. این موضوع اغلب باعث نادیده گرفتن موارد مرزی و خطاها می‌شود، همانطور که در مجموعه تست اولیه این آزمایشگاه مشاهده شد. همچنین، شناسایی تأثیر دقیق یک تغییر در کد را دشوارتر می‌کند، زیرا هیچ تست ناموفق اولیه‌ای برای تأیید وجود خطا وجود ندارد.


۱. **ایجاد حس امنیت کاذب:**  نوشتن تست پس از کدنویسی می‌تواند یک حس امنیت کاذب ایجاد کند؛ توسعه‌دهندگان ممکن است با دیدن پاس شدن تست‌های اولیه به اشتباه تصور کنند که برنامه بدون باگ است.

۲. **انطباق تست با کد به جای نیازمندی‌ها:**  توسعه‌دهندگان ممکن است به طور ناخودآگاه تست‌هایی بنویسند که با پیاده‌سازی ناقص و پراشکال کد مطابقت دارد، به جای اینکه آن را در برابر نیازمندی‌های واقعی بسنجند.

۳. **پیچیدگی تست‌نویسی:**  کدی که بدون فکر کردن به تست نوشته شده باشد، ممکن است بعداً تست‌نویسی را دشوار یا پیچیده کند و باعث شود بخش‌هایی از کد بدون تست باقی بمانند.
