# **The Importance of Strong Typing, Driven From Tests**

## **1. Introduction**  
Strong typing is a fundamental concept in programming languages that ensures variables, functions, and data structures are explicitly defined with their types, and type safety is enforced at compile-time or runtime. This white paper explores:  

- Why strong typing is important in programming.  
- The risks of weak or dynamic typing.  
- How Test-Driven Development (TDD) can help enforce strong typing in Java.  

We will use practical Java examples to demonstrate these concepts, drawing inspiration from structured repositories like [TDD Bank Account (Java)](https://github.com/xp-dojo/tdd-bank-account-java/tree/PeteSolution) and [MaximumTrainer White Papers](https://github.com/MaximumTrainer/white-papers/).  

---

## **2. What is Strong Typing?**  
Strong typing means that the programming language enforces strict type rules, preventing operations between incompatible types unless explicitly allowed (e.g., through casting).  

### **Example in Java (Strongly Typed)**  
```java
int number = 10;
String text = "Hello";

// Compile-time error: incompatible types
// String result = number + text; // Not allowed without explicit conversion
String result = Integer.toString(number) + text; // Correct: explicit conversion
```

### **Weak Typing (Hypothetical Scenario in Java)**  
If Java allowed weak typing (like JavaScript), the following would compile but cause unexpected behavior:  
```java
// Hypothetical weak typing in Java (not valid)
// int value = "10" + 5; // Could result in "105" instead of 15
```
**Problem:** Weak typing can lead to hidden bugs due to implicit type coercion.  

---

## **3. Why Strong Typing Matters**  

### **3.1. Prevents Runtime Errors**  
Strong typing catches type-related errors at compile-time rather than runtime.  

**Example:**  
```java
public class BankAccount {
    private double balance;

    public void deposit(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("Amount must be positive");
        }
        this.balance += amount;
    }
}

// Compiler ensures only 'double' is passed
BankAccount account = new BankAccount();
account.deposit(100.0); // OK
// account.deposit("100"); // Compile-time error
```

### **3.2. Improves Code Readability & Maintainability**  
Explicit types make code self-documenting.  

```java
// Strongly typed method signature
public BigDecimal calculateInterest(BigDecimal principal, BigDecimal rate) { ... }

// vs. weakly typed (if Java allowed it)
// public Object calculateInterest(Object principal, Object rate) { ... } // Dangerous!
```

### **3.3. Enables Better Tooling Support**  
IDEs can provide:  
- Auto-completion.  
- Refactoring support.  
- Early error detection.  

---

## **4. Risks of Weak or Dynamic Typing**  

Languages like JavaScript or Python (without type hints) allow:  
```javascript
// JavaScript Example (weak typing)
let value = "10" + 5; // "105" (string concatenation, not addition)
```
**Problems:**  
- Hard-to-debug issues.  
- No compiler checks.  
- More unit tests needed to catch type errors.  

---

## **5. Using TDD to Enforce Strong Typing in Java**  

Test-Driven Development (TDD) helps define correct types early by writing tests before implementation.  

### **5.1. Example: TDD for a Strongly Typed Bank Account**  

#### **Step 1: Write a Failing Test**  
```java
@Test
public void depositPositiveAmountIncreasesBalance() {
    BankAccount account = new BankAccount();
    account.deposit(100.0);
    assertEquals(100.0, account.getBalance(), 0.001);
}

@Test(expected = IllegalArgumentException.class)
public void depositNegativeAmountThrowsException() {
    BankAccount account = new BankAccount();
    account.deposit(-50.0); // Should throw an exception
}
```

#### **Step 2: Implement the Strongly Typed Class**  
```java
public class BankAccount {
    private double balance;

    public void deposit(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("Amount must be positive");
        }
        this.balance += amount;
    }

    public double getBalance() {
        return balance;
    }
}
```

#### **Step 3: Refactor with Stronger Types (e.g., BigDecimal for Money)**  
```java
public class BankAccount {
    private BigDecimal balance = BigDecimal.ZERO;

    public void deposit(BigDecimal amount) {
        if (amount.compareTo(BigDecimal.ZERO) <= 0) {
            throw new IllegalArgumentException("Amount must be positive");
        }
        this.balance = this.balance.add(amount);
    }
}
```

**Key Benefits of TDD + Strong Typing:**  
- Forces explicit type definitions.  
- Catches type misuse early.  
- Encourages domain-specific types (e.g., `BigDecimal` over `double` for money).  

---

## **6. Conclusion**  
Strong typing is essential for:  
✔ **Early error detection** (compile-time vs. runtime).  
✔ **Code clarity and maintainability**.  
✔ **Better tooling and refactoring support**.  

**TDD reinforces strong typing by:**  
✔ Defining expected types in tests first.  
✔ Preventing weakly typed "quick fixes."  
✔ Encouraging domain-driven design.  

---

# **Enforcing Strong Typing with Domain-Specific Types:  
From `double` to `BigDecimal` to a `Money` Type in Java**  

## **1. Introduction**  
In the previous section, we saw how **strong typing** improves safety and correctness in Java programs. We evolved a `BankAccount` class from using `double` (risky for financial calculations) to `BigDecimal` (more precise).  

However, we can go further by introducing a **domain-specific `Money` type**, which encapsulates both **amount** and **currency**. This ensures:  
✅ **Type safety** – Prevents mixing different currencies.  
✅ **Business logic encapsulation** – Centralizes currency conversion and validation.  
✅ **Clearer intent** – Makes the code self-documenting.  

We’ll use **Test-Driven Development (TDD)** to drive this design.  

---

## **2. Problem with `BigDecimal` Alone**  
While `BigDecimal` solves floating-point precision issues, it doesn’t:  
❌ Track **currency type** (USD, EUR, etc.).  
❌ Prevent accidental mixing of currencies.  
❌ Enforce monetary operations (e.g., disallow adding USD to EUR).  

### **Bad Example (Without `Money` Type)**  
```java
BigDecimal amountInUsd = new BigDecimal("100.00");
BigDecimal amountInEur = new BigDecimal("85.00");

// Compiles but makes no sense—mixing currencies without conversion!
BigDecimal total = amountInUsd.add(amountInEur); 
```

---

## **3. Solution: Introducing a `Money` Type**  

### **Step 1: Define the `Money` Class (TDD First)**  
We start by writing a failing test.  

#### **Test Case: Money Addition (Same Currency)**  
```java
@Test
public void addMoneyOfSameCurrency() {
    Money usd100 = new Money(new BigDecimal("100.00"), Currency.USD);
    Money usd50 = new Money(new BigDecimal("50.00"), Currency.USD);
    Money result = usd100.add(usd50);
    assertEquals(new Money(new BigDecimal("150.00"), Currency.USD), result);
}
```

#### **Test Case: Reject Mixed Currencies**  
```java
@Test(expected = IllegalArgumentException.class)
public void rejectAddingDifferentCurrencies() {
    Money usd100 = new Money(new BigDecimal("100.00"), Currency.USD);
    Money eur50 = new Money(new BigDecimal("50.00"), Currency.EUR);
    usd100.add(eur50); // Should throw an exception
}
```

#### **Implementing the `Money` Class**  
```java
public final class Money {
    private final BigDecimal amount;
    private final Currency currency;

    public Money(BigDecimal amount, Currency currency) {
        if (amount.compareTo(BigDecimal.ZERO) < 0) {
            throw new IllegalArgumentException("Amount cannot be negative");
        }
        this.amount = amount;
        this.currency = Objects.requireNonNull(currency);
    }

    public Money add(Money other) {
        if (!this.currency.equals(other.currency)) {
            throw new IllegalArgumentException("Cannot add different currencies");
        }
        return new Money(this.amount.add(other.amount), this.currency);
    }

    // equals(), hashCode(), toString() omitted for brevity
}
```

**Key Improvements:**  
✔ **Immutable** – Prevents accidental modification.  
✔ **Type-safe operations** – Ensures only compatible `Money` objects interact.  
✔ **Business rules enforced** – Rejects negative amounts and mixed currencies.  

---

## **4. Refactoring `BankAccount` to Use `Money`**  

### **Step 1: Update Tests to Use `Money`**  
```java
@Test
public void depositIncreasesBalance() {
    BankAccount account = new BankAccount(Currency.USD);
    account.deposit(new Money(new BigDecimal("100.00"), Currency.USD));
    assertEquals(new Money(new BigDecimal("100.00"), Currency.USD), account.getBalance());
}

@Test(expected = IllegalArgumentException.class)
public void rejectDepositWithWrongCurrency() {
    BankAccount account = new BankAccount(Currency.USD);
    account.deposit(new Money(new BigDecimal("100.00"), Currency.EUR)); // Should throw
}
```

### **Step 2: Modify `BankAccount` to Enforce Currency**  
```java
public final class BankAccount {
    private Money balance;

    public BankAccount(Currency currency) {
        this.balance = new Money(BigDecimal.ZERO, currency);
    }

    public void deposit(Money amount) {
        if (!amount.getCurrency().equals(balance.getCurrency())) {
            throw new IllegalArgumentException("Currency mismatch");
        }
        this.balance = this.balance.add(amount);
    }

    public Money getBalance() {
        return balance;
    }
}
```

**Benefits:**  
✔ **No more "naked" `BigDecimal`** – All monetary values are wrapped in `Money`.  
✔ **Currency safety** – The account rejects deposits in the wrong currency.  
✔ **Clear domain language** – Code expresses financial concepts explicitly.  

---

## **5. Extending `Money` with Advanced Operations**  

### **5.1. Currency Conversion**  
We can extend `Money` to support conversions using an exchange rate.  

#### **Test Case: Convert USD to EUR**  
```java
@Test
public void convertUsdToEur() {
    Money usd100 = new Money(new BigDecimal("100.00"), Currency.USD);
    BigDecimal exchangeRate = new BigDecimal("0.85"); // 1 USD = 0.85 EUR
    Money eurAmount = usd100.convertTo(Currency.EUR, exchangeRate);
    assertEquals(new Money(new BigDecimal("85.00"), Currency.EUR), eurAmount);
}
```

#### **Implementation in `Money`**  
```java
public Money convertTo(Currency targetCurrency, BigDecimal exchangeRate) {
    if (exchangeRate.compareTo(BigDecimal.ZERO) <= 0) {
        throw new IllegalArgumentException("Exchange rate must be positive");
    }
    return new Money(this.amount.multiply(exchangeRate), targetCurrency);
}
```

### **5.2. Withdrawal Validation**  
We can ensure withdrawals don’t overdraw the account.  

#### **Test Case: Reject Overdraft**  
```java
@Test(expected = IllegalArgumentException.class)
public void rejectOverdraft() {
    BankAccount account = new BankAccount(Currency.USD);
    account.deposit(new Money(new BigDecimal("100.00"), Currency.USD));
    account.withdraw(new Money(new BigDecimal("150.00"), Currency.USD)); // Should throw
}
```

#### **Implementation in `BankAccount`**  
```java
public void withdraw(Money amount) {
    if (!amount.getCurrency().equals(balance.getCurrency())) {
        throw new IllegalArgumentException("Currency mismatch");
    }
    if (amount.getAmount().compareTo(balance.getAmount()) > 0) {
        throw new IllegalArgumentException("Insufficient funds");
    }
    this.balance = new Money(
        balance.getAmount().subtract(amount.getAmount()),
        balance.getCurrency()
    );
}
```

---

## **6. Conclusion: The Power of Strong Typing & TDD**  

### **Key Takeaways**  
1. **Avoid Primitive Obsession**  
   - Replace `double`/`BigDecimal` with domain types like `Money`.  
2. **Enforce Business Rules at Compile Time**  
   - Prevent invalid operations (e.g., mixing currencies).  
3. **TDD Guides Better Design**  
   - Tests help define correct behavior before implementation.  
4. **Immutable Types Reduce Bugs**  
   - `Money` is immutable, preventing accidental modifications.  

### **Final Code Structure**  
```
src/
├── main/
│   ├── domain/
│   │   ├── Money.java       // Strongly typed monetary value
│   │   └── BankAccount.java // Uses Money instead of BigDecimal
│   └── Currency.java        // Enum (USD, EUR, etc.)
└── test/
    └── domain/
        ├── MoneyTest.java    // Tests for Money operations
        └── BankAccountTest.java // Tests for deposits/withdrawals
```

### **Further Improvements**  
- **Add `CurrencyConverter`** (for real-time exchange rates).  
- **Support Serialization** (e.g., JSON/DB mapping).  
- **Thread Safety** (if used in concurrent systems).  

By combining **strong typing**, **TDD**, and **domain-driven design**, we create **safer, more maintainable & understandable systems**.  

---

**References**  
- [Martin Fowler - Money Pattern](https://martinfowler.com/eaaCatalog/money.html)  
- [Effective Java - Item 17: Minimize Mutability](https://learning.oreilly.com/library/view/effective-java/9780134686097/)  
- [TDD Bank Account Example (Java)](https://github.com/xp-dojo/tdd-bank-account-java)  
- Martin Fowler, *Refactoring: Improving the Design of Existing Code*  
- Kent Beck, *Test-Driven Development: By Example*  

---
