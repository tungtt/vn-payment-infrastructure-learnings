# Domestic High-Value Interbank Credit Transfers in Vietnam (>= 500 Million VND or Urgent Transactions)

This document provides a detailed technical overview of the interbank payment flow for domestic high-value credit transfers in Vietnam, processed through the National Interbank Electronic Payment System (NIEPS) operated by the State Bank of Vietnam (SBV). It focuses on transactions valued at **500 million VND or more**, urgent payments of any value, and interbank transfers in foreign currencies (e.g., USD, EUR).

NIEPS employs a Real-Time Gross Settlement (RTGS) model, ensuring immediate and irrevocable settlement.

*This guide is intended for software engineers and system architects integrating with Vietnam’s payment infrastructure.*

## Overview

NIEPS is the cornerstone of Vietnam’s high-value payment infrastructure, managed by the SBV and succeeding the older Inter-Bank Electronic Payment System (IBPS). It operates on an RTGS model, where each payment is settled individually and in real-time. This process is final and irrevocable, which is critical for eliminating interbank settlement risk in large-value transactions.

**Key Characteristics**
- **Threshold**: Mandatory for VND transactions >= 500 million, urgent payments of any value, and all foreign currency transfers.
- **Settlement Model**: Real-Time Gross Settlement (RTGS).
- **Operating Hours**: Limited to official business days with strict cut-off times; instructions after cut-off are deferred to the next day.
- **Regulatory Framework**: Governed by Circular No. 08/2024/TT-NHNN (effective August 1, 2024).
- **Messaging Standards**: Transitioning from proprietary SBV formats to ISO 20022 (e.g., `pacs.009` for settlement, `camt.053` for statements).

## Actors

**Scenario**: A corporate or individual sender initiates a high-value transfer (>= 500M VND) to a receiver via their respective banks.

| Actor               | Description                                                                 |
|---------------------|-----------------------------------------------------------------------------|
| Sender              | Initiates the payment via the issuing bank’s online portal or branch.       |
| Receiver            | Receives the payment in their account at the receiving bank.                |
| Issuing Bank        | The sender’s bank, responsible for initiating and validating the transfer.  |
| Receiving Bank      | The receiver’s bank, responsible for crediting the beneficiary.             |
| Settlement Bank (SBV-NIEPS) | Executes final, irrevocable settlement through the NIEPS RTGS system. |

## Technical Flow

The table below outlines the step-by-step flow of a high-value RTGS transaction, including technical details, compliance requirements, and failure cases.

| Step                          | Description                                                                                                                                   | Compliance, Risks, and Failure Cases                                                                                   |
|-------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------|
| **1. Initiation & Authentication** | The sender initiates a transfer of >= 500M VND via the issuing bank’s online portal or branch. The bank authenticates using robust methods (e.g., biometrics, OTP). | **Compliance**: Per Decision No. 2345/QD-NHNN (effective July 1, 2024), biometric authentication is mandatory for online transactions > VND 10M or cumulative daily transactions > VND 20M.<br>**Failure Case**: Transaction blocked if authentication fails. |
| **2. Validation & Liquidity Check** | The issuing bank validates the payment instruction (compliance, risk parameters) and confirms sufficient funds in its SBV settlement account. For paper-based instructions, a "three-person rule" applies: creator, controller, and approver, each applying a digital signature. | **Compliance**: Adheres to Circular 08/2024/TT-NHNN for the three-person rule and digital signatures.<br>**Failure Case**: Rejected if compliance fails or funds are insufficient. |
| **3. Payment Instruction to NIEPS** | The issuing bank creates a payment order and transmits it to the NIEPS National Payment Service Center (NPSC). | **Technical Details**: Uses proprietary SBV formats, transitioning to ISO 20022 `pacs.009`. Sent over secure channels (VPN, TLS 1.2+).<br>**Non-Functional Requirements**: NPSC ensures >99.95% availability. |
| **4. Real-Time Gross Settlement** | The NPSC validates the order (signatures, format). If funds suffice, SBV debits the issuing bank’s account and credits the receiving bank’s account. Settlement is final and irrevocable. | **Failure Case**: If funds are insufficient, the payment is queued (FIFO) until liquidity improves or is canceled (by the issuing bank or SBV at EOD).<br>**Gridlock Resolution**: Includes bypass FIFO, reordering, and cancellation. |
| **5. Confirmation & Notification** | NIEPS sends a confirmation to both banks, confirming interbank transfer completion. | **Technical Details**: Uses ISO 20022 `pacs.002` (future state) for status updates.<br>**Failure Case**: None; failures handled in step 4. |
| **6. Credit Beneficiary** | The receiving bank performs final checks (account status, AML screening) and credits the receiver’s account. | **Compliance**: AML/CFT checks per Law No. 14/2022/QH15; STRs filed if risks detected.<br>**Failure Case**: If checks fail, NIEPS coordinates reversal with the issuing bank. |
| **7. Reconciliation** | At end-of-day (EOD), banks and SBV perform automated reconciliation to align records and balances. | **Failure Case**: Discrepancies trigger manual investigation. |

## Queue Management

- **Mechanism**: Centrally managed with a First-In-First-Out (FIFO) principle.
- **Gridlock Resolution**: Features bypass FIFO, reordering, and cancellation to unlock payments despite sufficient funds.
- **Monitoring**: Supports queue and settlement account monitoring.
- **Cancellation**: Allowed by the issuing bank during the day or by SBV at EOD if unsettled.

## Differences Between NRT ACH and RTGS

The table below compares the Near Real-Time Automated Clearing House (NRT ACH) system operated by NAPAS with the RTGS system of NIEPS.

| Feature                     | NRT ACH (NAPAS)                                 | RTGS (NIEPS)                                     |
|-----------------------------|-------------------------------------------------|--------------------------------------------------|
| **Transaction Value**       | < VND 500M per transaction                      | >= VND 500M, urgent payments, or FX transfers    |
| **Settlement Model**        | Near Real-Time with Deferred Net Settlement     | Real-Time Gross Settlement                       |
| **Processing**              | Batches transactions, nets positions            | Individual, real-time settlement                 |
| **Operating Hours**         | 24/7                                            | Strict business hours with cut-off times         |
| **Latency**                 | Near real-time (seconds)                        | Immediate (real-time)                            |
| **Queue Management**        | Rejects if liquidity fails                      | FIFO queue with gridlock resolution              |
| **Messaging**               | ISO 20022 (`pacs.008`, `pacs.002`)              | Transitioning to ISO 20022 (`pacs.009`, `camt`)  |
| **Use Case**                | Low-value retail payments (e.g., P2P transfers) | High-value, time-critical payments               |
| **Liquidity Requirement**   | Sufficient funds in SBV settlement account      | Sufficient funds in SBV settlement account       |

## Implementation Recommendations

- **Integration**: Partner with a bank connected to NIEPS for high-value transfers.
- **Messaging**: Support both legacy SBV formats and ISO 20022 to accommodate the transition.
- **Security**: Implement biometric authentication per Decision No. 2345/QD-NHNN for online transactions.
- **Source of Truth**: Consult the partner bank’s technical specifications for exact file formats, API endpoints, and security protocols.
