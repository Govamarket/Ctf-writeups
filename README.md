# Ctf-writeups

## Client-Side Trust & Business Logic Vulnerability

### Web Security Lab Write-up


##  Overview

This project documents a critical **business logic vulnerability** discovered in a web application lab environment. The flaw allows an attacker to manipulate cart data, user balance, and product pricing by tampering with client-controlled parameters.

The vulnerability was identified during testing using an intercepting proxy tool.

---

##  Vulnerability Summary

* **Type:** Business Logic Flaw / Client-Side Trust Issue
* **Severity:** Critical
* **Attack Vector:** Parameter Tampering via Intercepted HTTP Requests
* **Affected Components:**

  * Cart API
  * Purchase API
  * Client-side validation logic

---

##  Affected Endpoints

```
POST /api/cart/add
GET  /api/cart
POST /api/purchase
```

---

##  Description

The application incorrectly trusts data provided by the client for sensitive operations such as:

* User balance validation
* Product pricing
* Cart item quantities
* Purchase calculations

Instead of enforcing validation on the server, the application relies on client-side logic, which can be bypassed using request interception.

This allows full manipulation of purchase logic, including pricing and balance values.

---

##  Tools Used

* Burp Suite
* Browser Developer Tools
* Manual HTTP request modification

---

## 🔬 Proof of Concept (PoC)

### 1. Add Item to Cart

```http
POST /api/cart/add
Content-Type: application/json

{
  "itemId": 1,
  "quantity": 1
}
```

---

### 2. Modify Cart Balance (Tampering)

```http
GET /api/cart

{
  "userBalance": 9999999999
}
```

 Result: Server accepted manipulated balance value.

---

### 3. Modify Purchase Request

```http
POST /api/purchase
Content-Type: application/json

{
  "cartItems": [
    {
      "item_id": 1,
      "quantity": 1,
      "price": 1000
    }
  ]
}
```

 Result: Server returned `200 OK` and processed purchase successfully despite modified pricing and balance values.

---

##  Impact

This vulnerability allows an attacker to:

* Purchase items at reduced or zero cost
* Manipulate product pricing during checkout
* Modify user balance arbitrarily
* Bypass client-side purchase restrictions
* Corrupt transaction integrity

---

##  Root Cause

The issue is caused by:

* Trusting client-side data for financial calculations
* Lack of server-side validation for:

  * price
  * balance
  * quantity
* Absence of authoritative backend recalculation

---

##  Recommended Fix

###  1. Server-side price enforcement

Never accept price from client:

```python
price = db.get_product_price(item_id)
```

---

###  2. Server-side balance validation

```python
balance = db.get_user_balance(user_id)
```

---

###  3. Recalculate totals on backend only

```python
total = sum(price * quantity for item in cart)
```

---

###  4. Validate input strictly

Reject:

* negative quantities
* null values
* excessively large numbers

---

###  5. Ignore client-supplied financial fields

Never trust:

* `price`
* `userBalance`
* computed totals from request body

---

##  Key Lessons Learned

* Client-side validation is not security
* All financial logic must be server-side
* Request interception reveals trust boundaries
* APIs must never accept authoritative values from clients

---

##  Conclusion

This vulnerability demonstrates a critical breakdown in trust boundaries between client and server. By relying on client-controlled data for financial operations, the system becomes fully exploitable via simple request manipulation.
<div>
  <img width="1366" height="688" alt="bypass" src="https://github.com/user-attachments/assets/70fb9e28-7c7f-490b-aa0c-ec37845ff701" />
<img width="1366" height="650" alt="repeater" src="https://github.com/user-attachments/assets/367789a4-5486-45fb-8378-e4172afef646" />
<img width="1366" height="646" alt="succef" src="https://github.com/user-attachments/assets/e38a8d68-3a4b-4387-8a12-335996ecbc62" />
<img width="1366" height="650" alt="Capturepif" src="https://github.com/user-attachments/assets/3ca46722-fa54-4c9c-ac7b-1d50f06afc02" />

</div>

*Written by [Possible (@SIEM Latency)](https://x.com/SIEM_Latency)) — The Cyber Lab Journal*
