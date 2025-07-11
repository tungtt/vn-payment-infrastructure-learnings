# Vietnam Payment Infrastructure (July 2025)
The Vietnam Payment Infrastructure repository provides a clear and detailed guide to the country’s payment systems, focusing on the most essential flows and concepts for engineers and technologists. It covers a wide range of payment methods, from interbank transfers (high-value RTGS via NIEPS and low-value NRT ACH via NAPAS) to mobile-based payments (e.g., VietQR, e-wallets like Momo), card payments, and emerging methods like cryptocurrency and Buy Now Pay Later (BNPL).

The content is written in concise, direct language to help seasoned technologists break into payments, finance, and banking. It reflects the latest advancements and regulations, such as Circular No. 08/2024/TT-NHNN and the adoption of ISO 20022 messaging standards, ensuring the knowledge is up-to-date as of July 2025.

While the repository focuses on key concepts drawn from the author’s learning process, it also provides links to comprehensive external resources for a deeper understanding of the payments landscape.

## Payment flows
These are the payment flows that will be discussed in this repository.

### 1. Interbank Electronic Payment System (IBPS)
- [High-value or urgent or foreign-currency transfers (RTGS via NIEPS)](docs/Domestic%20High-Value%20Interbank%20Credit%20Transfers%20(RTGS).markdown)
- [Low-value transfers (NRT ACH via NAPAS)](docs/Domestic%20Low-Value%20Interbank%20Credit%20Transfers%20(Near%20Real-Time%20ACH).markdown)
- [Batch credit transfers or push transactions (DNS via NAPAS)](docs/Domestic%20Batch%20Interbank%20Credit%20Transfers%20(DNS).markdown)
- [Batch debit transfers or pull transactions (DNS via NAPAS)](docs/Domestic%20Batch%20Interbank%20Debit%20Transfers%20(Pull%20Transactions).markdown)
- [QR Code Payments: Consumer-to-Consumer (C2C) & Consumer-to-Merchant (C2M)](docs/Domestic%20QR%20Code%20Payment%20-%20Consumer-to-Consumer%20-%20Consumer-to-Merchant.markdown)

### 2. Cross-border Payments
- SWIFT transfers
- International payment services (e.g., Paypal, Stripe)
- Other correspondent banking networks
- Regional payments

### 3. Card Payments
- Domestic cards (NAPAS)
- International cards (Visa, Mastercard, JCB, etc.)
- Consumer-to-Consumer vs. Consumer-to-Merchant Payments

### 4. Digital Wallet Payment (e.g., Momo, ZaloPay, VNPay)

### 5. Mobile Money (via Mobile Money Operators)

### 6. Money Service Business (e.g., Western Union, MoneyGram)

### 7. Cryptocurrency Payments
- Via exchanges (e.g., Binance, Remitano)
- Via payment gateways (e.g., NOWPayments)

### 8. Buy Now Pay Later (BNPL)

## Key concepts

### Payment process
- Authorization
- Clearing
- Settlement
- Reconciliation

### Key actors
- Sender / Payer
- Recipient / Beneficiary / Payee
- Issuing / Originating Bank
- Receiving / Acquiring Bank
- Clearing House
- Settlement Bank
- State Treasury
- Card Network
- Payment Processor
- Payment Gateway

### Settlement types and timing

| Settlement Timing/Type | Gross Settlement | Net Settlement |
|------------------------|------------------|----------------|
| Real-Time | Real-Time Gross Settlement (RTGS) | *Real-Time Net Settlement (RTNS)* |
| Near Real-Time | Near Real-Time Gross Settlement (NRT ACH) | *Not applicable* |
| Deferred (or, Designated) | *Not applicable* | Deferred Net Settlement (DNS) |

- **Settlement types**: net vs. gross settlements
  - Gross settlement: each payment is processed one-by-one, in real-time or near real-time (seconds).
  - Net settlement: payments are deffered and processed in batch. Only the net difference is transferred between banks.
- **Settlement timing**: real-time, near real-time, or deferred
  - Real-Time: payment is processed instantly.
  - Near Real-Time: payments are processed almost instantly, with a slight delay of a few seconds.
  - Deferred (or, sometimes mentioned as "Designated"): payments are processed in batch at set time (for instance, at the end-of-day EOD).
- Practical settlement models in Vietnam
  - [Real-Time Gross Settlement (RTGS)](docs/Domestic%20High-Value%20Interbank%20Credit%20Transfers%20(RTGS).markdown) is used for high-value interbank transactions (>= 500 million VND), urgent transactions of any value, or transactions in foreign currencies.
  - [Low-value transfers (NRT ACH via NAPAS)](docs/Domestic%20Low-Value%20Interbank%20Credit%20Transfer%20(Near%20Real-Time%20ACH).markdown) is used for low-value interbank transactions (< 500 million VND).
  - [Deferred Net Settlement (DNS; or sometimes mentioned as Designated Time Net Settlement DTNS)](docs/Domestic%20Batch%20Interbank%20Credit%20Transfers%20(DNS).markdown) is used for recurring, one-to-many payments like salaries or supplier payments.
- The final settlement model is Real-Time Net Settlement (RTNS) or Real-Time Final Settlement (RTFS) is not applicable in Vietnam.

### Messaging Standards
- SWIFT proprietary standard (MT messages)
- (SWIFT) ISO 20022 standard (MX messages)
- EMVCo QR Code standard
- ISO 8583 standard

### Payment Risks and Mitigation Methods
- 6 types of risks are: settlement, liquidity, systemic, Herstatt, operational, and legal risks.
- 24 principles to mitigate risks.
- 9 mitigation actions
  - Proper selection of the direct participants
  - Net Debit Cap
  - Novation
  - Intraday Liquidity Arrangement
  - Collateral Requirement
  - Loss-sharing Arrangement
  - Third-party Guarantee
  - ISDA Master Agreement
  - Market Infrastructure Resiliency Service (MIRS)

## Companion Medium series
This repository pairs with a Medium article series, [Decoding Banking’s AI Shift: Non-Banker Learns Bank Out Loud](https://medium.com/@tungt3.pmp/decoding-bankings-ai-shift-non-banker-learns-bank-out-loud-2a88faadfb1d), where I explore how AI is transforming banking, from synthetic identity fraud to credit scoring and risk detection.

The series began first, but I realized the need to understand the foundations of Vietnam’s payment systems before diving into emerging challenges. This repository provides that foundation, offering detailed payment flows and concepts to help you grasp the topics discussed in the Medium series. Once completed, this repo will serve as a reference for readers of the Medium articles. Check out the series here for insights into AI’s impact on banking, grounded in the payment infrastructure you’ll learn about in this repo.

## Follow the author
Medium blog: https://medium.com/@tungt3.pmp/
LinkedIn profile: https://www.linkedin.com/in/tungtranthanh/
