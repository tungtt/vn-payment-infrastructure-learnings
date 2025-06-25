# Domestic Near Real-Time Interbank Credit Transfers in Vietnam (applicable for low-value transactions)

This document describes the business and technical flow for domestic near-real-time interbank credit transfers in Vietnam, facilitated by the National Payment Corporation of Vietnam (NAPAS) using a Near Real-Time (NRT) Automated Clearing House (ACH) model. Final settlement occurs at the State Bank of Vietnam (SBV). The system supports transfers **up to less than 500 million VND per transaction**, operating 24/7.

## Overview

The NRT ACH model enables rapid interbank transfers, distinct from deferred net-batch processing or real-time gross settlement (RTGS) systems. It uses prefunded intraday balances at SBV for instant settlement, ensuring low latency. ISO 20022 standardizes messaging, supporting features like real-time status checks and VietQR alias resolution (e.g., QR codes or phone numbers).

## Actors
**Scenario**: Hưng (sender) uses Bank A’s mobile banking app to transfer funds to Phương (recipient) at Bank B.

![High-level flow for domestic near-real-time interbank credit transfers in Vietnam](../media/Domestic%20Interbank%20Credit%20Transfer%20(low-value%20transactions).jpg)

| Actor | Description |
|----|----|
| Sender (Hưng) | Initiates the payment. |
| Receiver (Phương) | Receives the payment. |
| Issuing Bank (A) | The sender's bank. |
| Receiving Bank (B) | The receiver's bank. |
| Clearing House (NAPAS) | Processes, validates, and routes payment instructions between banks. |
| Settlement bank (SBV, or State Bank of Vietnam) | Performs final, irrevocable settlement by transferring funds between the commercial banks' reserve accounts. |

## Technical Flow

![Detailed sequence diagram for domestic near-real-time interbank credit transfers in Vietnam](../media/Domestic%20Interbank%20Credit%20Transfer%20(low-value%20transactions)%20Sequence%20Diagram.jpg)


| Step | Description | Compliance, Risks, and Failure Cases |
|------|-------------|-------------------------------------|
| **1. Initiation & Authentication** | Hưng initiates a transfer to Phương via Bank A’s mobile banking app, providing the recipient’s bank, account number, amount, and remarks. Hưng authenticates using Bank A’s required MFA (e.g., biometrics, OTP). | No specific compliance or risk checks at this step. |
| **2. Validation & Fund Earmarking** | Bank A’s core banking system verifies Hưng’s account has sufficient funds and places a hold (earmark) on the transaction amount, preventing its use elsewhere. This is an internal ledger operation; funds remain at Bank A. | **Compliance & Risks**: Performs Customer Due Diligence (CDD) against records, screens against internal watchlists and sanctions lists, and conducts device/behavioral analysis for a risk score (per Law No. 14/2022/QH15, Decree No. 19/2023/NĐ-CP). Validates funds and earmarks them.<br>**Failure Case**: Transaction blocked if authentication fails or checks flag high risk (e.g., sanction hit, abnormal behavior). Bank A may file a Suspicious Transaction Report (STR) with SBV. |
| **3. Payment Instruction to Clearing House** | Bank A sends a payment instruction to NAPAS. | **Technical Details**: Uses ISO 20022 `pacs.008` (FI to FI Payment Credit Transfer) via HTTPS POST over a secure channel (VPN or dedicated line, TLS 1.2+). Authenticates with mutual TLS (mTLS) and OAuth 2.0 bearer token. Message is digitally signed.<br>**Non-Functional Requirements**: NAPAS provides asynchronous acknowledgment within seconds; endpoints maintain >99.95% availability. |
| **4. Clearing & Routing** | NAPAS receives the `pacs.008` message and performs clearing checks: validates digital signature and format, checks for duplicates (using unique instruction ID), conducts fraud analysis (velocity checks, watchlists, patterns), verifies destination bank and account format, and confirms Bank A’s prefunded balance at SBV covers the amount. If checks pass, NAPAS decrements Bank A’s liquidity, logs a pending credit for Bank B and debit for Bank A, and forwards a settlement instruction to SBV. | **Failure Case**: If checks fail (e.g., invalid signature, suspected fraud, insufficient liquidity), NAPAS rejects the instruction, sends a `pacs.002` (Payment Status Report) with a rejection code to Bank A, and stops the transaction. |
| **5. Transaction Confirmation** | After successful clearing, NAPAS sends a final status update to Bank A and Bank B using ISO 20022 `pacs.002` with status code ACSP (Accepted Settlement Completed). Bank A removes the hold on Hưng’s account, debits funds, and notifies Hưng. | **Failure Case**: None at this step; failures are handled in steps 4 or 6. |
| **6 & 7. Settlement** | NAPAS aggregates cleared transactions over a short clearing cycle (seconds), calculates multilateral net positions for all banks, and submits a digitally signed net settlement file to SBV. SBV validates the file, checks banks’ reserve accounts for sufficient funds, and executes net transfers. SBV confirms settlement to NAPAS, which updates its ledger, marking credits and debits as complete. Bank B’s settled funds increase its liquidity for future transactions. Settlement is final and irrevocable. | **Failure Case**: If a bank lacks sufficient reserve funds, SBV rejects the settlement batch, notifies NAPAS, which unwinds the cycle and informs all banks. Bank B reverses any provisional credit to Phương; Bank A releases the hold on Hưng’s funds. Hưng is notified of the failure. This is a rare, high-impact event with systemic risk. |
| **8, 9, 10. Credit Beneficiary & Notification** | Bank B receives the `pacs.002` confirmation, performs final checks (e.g., account status, last-mile AML), credits Phương’s account, and sends a notification (e.g., SMS, push). | **Compliance**: Bank B may hold funds in a suspense account and file an STR if checks detect high risk.<br>**Failure Case**: If the account is invalid or checks fail, Bank B notifies NAPAS, which informs Bank A to reverse the transaction. Hưng is notified of the failure. |
| **11. Reconciliation** | Bank A, Bank B, NAPAS, and SBV perform end-of-day automated reconciliation to ensure transaction logs match, maintaining system integrity. | No specific failure case; discrepancies trigger manual investigation. |


## Optional overlay service
- VietQR alias: when a QR code is used as an address for transfer, VietQR alias resolves to the standard account number and receiving bank. Will discuss this topic separately.

## What happened before NAPAS?
- Bank A communicates directly with SBV. It created a CITAD/IBPS payment message and sent it to the State Bank of Vietnam’s inter-bank network.
- Clearing and Settlement processes were combined. SBV gathered all the day’s messages and
  - Sorted them by receiving bank
  - Netted low-value payments in batch runs (cut-off 15:00 for low-value, 16:00 for high-value)
  - Immediately debited Bank A’s reserve account and credited Bank C’s reserve account in the same run.
- SBV released an “OK” message; Bank B booked the incoming funds and notified Phương.
- What it meant for customers
  - Not 24/7: payments only moved during SBV business windows (no nights, weekends, or holidays).
  - Waiting time: if Hùng missed the afternoon cut-off, Phương had to wait until the next working day.
  - Features, such as real-time status checks and alias (phone or QR) transfers, did not exist.

## Fraud detections, Network abuse detections, and AML checks
### Issuing and Receiving Banks
The legal obligation for AML transaction monitoring and reporting rests with the financial institutions that have the direct customer relationship (Bank A and Bank B).

Their internal AML systems will analyze the transaction after it has settled. If the transaction is flagged by their monitoring rules (e.g., it's part of a larger suspicious pattern, it's inconsistent with the customer's profile), the bank's compliance department is legally required to investigate and potentially file a Suspicious Transaction Report (STR) with the SBV's dedicated AML department.

### Clearing House (NAPAS)
Following settlement, NAPAS's role in this specific transaction is complete, except for providing data for reconciliation.

While NAPAS performs system-wide analysis to detect fraud patterns and network abuse, it lacks customer-level information to conduct AML checks and does not file STRs on behalf of its members for their customers' transactions.

### Settlement bank (SBV)
A completely separate division, the SBV's Anti-Money Laundering Department, is the government body that receives and analyzes the STRs filed by Bank A, Bank B, and other financial institutions. They perform macro-level analysis and can launch investigations, but they do not check individual transactions on the payment rails post-facto.

View more here: https://www.lexology.com/library/detail.aspx?g=a47a3db4-6c70-4f04-b311-ccab1d5dd3e7
