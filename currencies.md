## 1. Introduction

### 1.1 Purpose and Scope

This document standardizes currency representation in the L402 protocol for implementers of L402-compatible servers and clients. It focuses specifically on:

1. How to express monetary values in offers (unit of account)
2. The format for currency codes and amount values
3. Integer-based representation using smallest currency units
4. Best practices for currency handling

In the context of automated transactions between clients and servers, consistent currency representation ensures accurate pricing, reduces errors, and facilitates interoperability among diverse systems implementing the L402 protocol.

This specification does NOT cover:
- Payment method implementation details
- Currency conversion mechanisms
- Payment processor integration specifics

## 2. Protocol Elements

### 2.1 402 Response Structure

A 402 response MUST include a JSON body with the available offers the client can get a payment request for. Each offer MUST have two relevant fields relative to pricing: `amount` and `currency`.

These fields provide the foundation for the unit of account, establishing the value that must be paid to access the resource. Here's a simplified example:

```json
{
  "offers": [
    {
      "id": "offer_001",
      "amount": 500,
      "currency": "USD",
      "description": "Access to premium content",
      "payment_methods": ["credit_card", "lightning"]
    }
  ]
}
```

### 2.2 Currency Codes

Each offer returned by an L402-compatible server MUST include a `currency` field. This field defines the unit of account used to express the offer's amount. The L402 protocol supports both fiat and cryptocurrency codes.

#### 2.2.1 Fiat Currencies

For fiat currencies, L402 uses the standard [ISO 4217](https://en.wikipedia.org/wiki/ISO_4217) three-letter codes such as:
- `USD` for United States Dollar
- `EUR` for Euro
- `JPY` for Japanese Yen

The amount MUST be expressed in the **smallest indivisible** unit of the given currency. For example:

| Currency | Code (L402) | Smallest Unit | 1 Unit Equals |
|----------|------|--------------|---------------|
| US Dollar | USD | cent | 100 cents = $1.00 |
| Euro | EUR | cent | 100 cents = €1.00 |
| Japanese Yen | JPY | yen | 1 yen = ¥1.00 (no smaller unit) |

Currencies without subdivisions (like JPY or IDR) MUST still use the smallest whole unit as the base for integer representation.

#### 2.2.2 Cryptocurrencies

Cryptocurrency codes follow widely used ticker conventions. L402 does not require formal ISO recognition, but some cryptocurrencies have ISO entries that clients SHOULD recognize. For example, Bitcoin is usually expressed as BTC while XBT is the (non)-official ISO 4217 nomenclature.

Examples of supported cryptocurrency codes and their units:

| Cryptocurrency | Code (L402) | Smallest Unit | 1 Unit Equals |
|----------------|------|--------------|---------------|
| Bitcoin | BTC / XBT | satoshi | 100,000,000 sats = 1 BTC |
| Ethereum | ETH | wei | 1,000,000,000,000,000,000 wei = 1 ETH |
| Solana | SOL | lamport | 1,000,000,000 lamports = 1 SOL |


### 2.3 Integer Representation of Amounts

All amount fields in L402 offers MUST be represented as integers, corresponding to the smallest indivisible unit of the specified currency.

Using integers ensures consistent behavior across systems and avoids the precision issues and rounding errors commonly associated with floating point arithmetic. This is especially important for financial systems, where even small discrepancies can lead to significant bugs or security vulnerabilities.

For typed languages, it's safe to assume the integer will fit in a `uint64`. Some languages like Python can handle big integers out of the box. Others require special handling:

- **JavaScript**: Use `BigInt` for values that might exceed `Number.MAX_SAFE_INTEGER` (2^53 - 1)
  ```javascript
  // Correct way to handle large amounts in JavaScript
  const weiAmount = BigInt("1000000000000000000"); // 1 ETH in wei
  ```

Floating point numbers (floats) MUST NOT be used anywhere in the protocol to represent currency amounts. Implementations MUST reject offers that contain non-integer values in the amount field.

#### 2.3.1 Example Representations

| Currency | Amount (L402) | Human-readable |
|----------|---------------|----------------|
| USD | 150 | $1.50 (150 cents) |
| EUR | 999 | €9.99 (999 cents) |
| BTC | 10000 | 10,000 sats = 0.0001 BTC |
| ETH | 1000000000000000000 | 1 ETH (in wei) |

## 3. Currency Concepts

### 3.1 Units of Account vs. Media of Exchange

L402 clearly separates the concept of how much something costs (the unit of account) from how it is paid (the medium of payment).

#### 3.1.1 Unit of Account

When a server responds with a 402 and provides an offer, it MUST specify the price using an amount and a currency. This is the unit of account, the pricing reference used to describe the value of the offer. For example, 100 USD cents:

```json
{
  "amount": 100,
  "currency": "USD"
}
```

#### 3.1.2 Media of Exchange

The actual payment MAY be fulfilled using a different currency or payment method. This is possible because the conversion occurs at the time of payment, not when the offer is presented. The client is responsible for selecting a compatible payment method and fulfilling the equivalent value, as determined by the server.

Each offer specifies acceptable `payment_methods`, indicating the media of exchange through which the client can fulfill the payment. While currency conversion is outside the scope of this specification, clients must ensure that the payment amount corresponds to the value specified in the unit of account, considering any exchange rates at the time of payment.

An offer priced in USD can be paid using:
- A credit card that charges USD
- A Lightning invoice denominated in satoshis
- A cryptocurrency wallet that converts from another asset to the required value

#### 3.1.3 Stablecoins as Payment Media

Stablecoins like USDC are media of payment, not pricing units. An offer MAY be priced in USD and paid using USDC, but USDC is not listed as a currency code in the offer itself. This avoids confusion and reinforces the clear pricing model across fiat and crypto.

For example, an offer might specify:
```json
{
  "amount": 500,
  "currency": "USD",
  "payment_methods": ["credit_card", "lightning", "onchain"]
}
```

If the client selects "onchain" with Base as chain and USDC as the asset, the payment process would involve:
1. Converting the USD amount to the equivalent USDC amount (typically 1:1)
2. The server generating a payment request with an on-chain address
3. The client sending the USDC to that address
4. The server verifying the transaction before granting access

This separation enables consistent pricing while supporting diverse payment methods.

### 3.2 Fractional Amount Values

At present, the amount field in L402 offers MUST be an integer, representing the smallest indivisible unit of the specified currency (e.g., cents, satoshis, wei). However, in some scenarios, especially involving digital goods or high-frequency API access, sub-unit pricing (e.g., fractions of a cent or satoshi) may become necessary.

In the future, L402 MAY support expressing amounts as decimal strings to enable micro-payments without relying on floats. For example, a 10th of a USD cent:

```json
{
  "amount": "0.001", 
  "currency": "USD"
}
```

If adopted, these strings MUST be parsed using arbitrary-precision decimal types (e.g., Decimal in Python), and MUST NOT be interpreted as floating-point numbers. This will preserve accuracy and avoid rounding errors across implementations.

Until such support is formalized, all amount values MUST remain integers.

## 4. Implementation Guidelines

### 4.1 Best Practices

To ensure consistency, accuracy, and interoperability across L402 implementations, the following best practices are recommended:

#### 4.1.1 Use the Smallest Unit

Always represent amounts in the smallest indivisible unit of the currency:
- USD → cents
- EUR → cents
- BTC → satoshis
- ETH → wei
- SOL → lamports

#### 4.1.2 Avoid Floating Point Numbers

NEVER use floating-point numbers to represent currency amounts. Floats introduce rounding errors and inconsistencies, especially across different programming languages and platforms. Always use integers or, when micro-payments become supported, stringified decimals.

In the future precise decimal types will be use instead of floats:
- Python: `decimal.Decimal`
  ```python
  from decimal import Decimal
  # When parsing a potential future decimal amount
  amount = Decimal("0.001")
  ```
- Java: `java.math.BigDecimal`
  ```java
  import java.math.BigDecimal;
  // When parsing a potential future decimal amount
  BigDecimal amount = new BigDecimal("0.001");
  ```
- JavaScript: Libraries like `decimal.js` or `big.js`
  ```javascript
  import Decimal from 'decimal.js';
  // When parsing a potential future decimal amount
  const amount = new Decimal("0.001");
  ```

#### 4.1.3 Presentation Layer Conversion

When showing prices to users, convert internal amounts to a readable format only at the presentation layer:
- 150 cents → $1.50
- 10000 satoshis → 0.0001 BTC

Consider localization needs when formatting for display:
- Use locale-appropriate decimal separators (period in US, comma in Europe)
- Apply appropriate currency symbols and placement (prefix or suffix)
- Follow regional conventions for digit grouping

## 5. Glossary of Terms

- **Unit of Account**: The currency and denomination in which prices are expressed (e.g., USD)
- **Medium of Exchange**: The payment method or currency used to complete a transaction
- **Smallest Indivisible Unit**: The minimal denomination of a currency (e.g., cent, satoshi, wei)
- **Presentation Format**: How currency amounts are displayed to users in a human-readable form
- **Integer Representation**: Expressing currency amounts as whole numbers of the smallest unit
- **ISO 4217**: International standard for currency codes, using three-letter abbreviations
- **Arbitrary-precision Arithmetic**: Mathematical operations that handle numbers with unlimited precision
- **Stablecoin**: A cryptocurrency designed to maintain a stable value relative to a reference asset
