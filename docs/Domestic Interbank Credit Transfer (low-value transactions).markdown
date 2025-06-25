# Domestic Near Real-Time Interbank Credit Transfers in Vietnam (applicable for low-value transactions)

This document describes the business and technical flow for domestic near-real-time interbank credit transfers in Vietnam, facilitated by the National Payment Corporation of Vietnam (NAPAS) using a Near Real-Time (NRT) Automated Clearing House (ACH) model. Final settlement occurs at the State Bank of Vietnam (SBV). The system supports transfers *up to less than **500 million VND per transaction**, operating 24/7.

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

|Step | Compliance & Risks. Failure case. |
|----|----|
| #1 **Initiation & Authentication** (Sender → Issuing Bank)<br>Hưng initiates a transfer to Phương via Bank A’s mobile banking app.<br>He provides the recipient’s bank, account number, amount, and remarks.<br>Hưng authenticates himself using the bank’s required MFA method (e.g., biometrics ,OTP). | \- |
| #2 **Validation & Fund Earmarking** (within Bank A)<br>Bank A’s core banking system verifies that Hưng has sufficient funds.<br>Upon successful validation, the system places a hold (earmark) on the transaction amount in Hưng's account. This prevents the funds from being used for other transactions.<br>This is an internal ledger operation within Bank A. The funds have not yet moved from the bank. | Compliance & Risks<br>- Customer Due Diligence (CDD) verification against its records.<br>- Screening against internal watchlists and sanctions lists.<br>- Device and behavioral analysis to generate a risk score.<br>- Reference: Law No. 14/2022/QH15 and Decree No. 19/2023/NĐ-CP.<br>- Perform funds validation and earmarking processes.<br><br>Failure case: If authentication fails or initial checks raise a high-risk flag (e.g., sanction hit, abnormal behavior), the transaction is blocked.<br>- Bank A may be required to file a Suspicious Transaction Report (STR) with the SBV. |
| #3 **Payment Instruction to Clearing House** (Bank A → NAPAS)<br>Bank A constructs and sends a payment instruction to NAPAS. | Technical Details<br>- Message: ISO 20022 `pacs.008` (FI to FI Payment Credit Transfer).<br>- Protocol: HTTPS POST request over a secure channel (VPN or dedicated line) using TLS 1.2 or higher.<br>- Authentication: Mutual TLS (mTLS) and OAuth 2.0 bearer token.<br><br>Integrity: The pacs.008 message is digitally signed by Bank A.<br>Non-Functional Requirements<br>- Timeliness: NAPAS must provide an asynchronous acknowledgment within seconds.<br>- Availability: NAPAS endpoints must maintain high availability (e.g., >99.95%). |
| #4 **Clearing & Routing** (within NAPAS)<br>NAPAS receives the pacs.008 message and performs clearing checks. | Checks<br>- Validates the message's digital signature and format.<br>- Checks for duplicate transactions (using a unique instruction ID).<br>- Performs real-time fraud analysis (e.g., velocity checks, watch-list checks, and pattern recognition).<br>- Validates the destination bank and account number format.<br>- Liquidity check: Verifies that the issuing bank (Bank A) has a sufficient balance in its pre-funded settlement account to cover the transaction amount. This is the most critical check to eliminate settlement risk.<br><br>Actions: If all checks pass,<br>- It decrements Bank A's available liquidity position by the transaction amount.<br>- It simultaneously logs a pending credit for the same amount, designated for Bank B. It also logs a pending debit for Bank A.<br>- NAPAS prepares and forwards a settlement instruction to the SBV.<br><br>Failure Case: If checks fail (e.g., invalid signature, suspected fraud, insufficient liquidity), NAPAS rejects the instruction and sends a `pacs.002` (Payment Status Report) back to Bank A with a rejection code. The transaction stops. |
| #5 **Transaction Confirmation** (NAPAS → Bank A & Bank B)<br>Upon successful clearing in step 4, NAPAS sends a final status update to both participating banks. | Technical Details<br>Message: ISO 20022 `pacs.002` is sent to both Bank A and Bank B.<br>- Status Code: The message includes a code, such as ACSP (Accepted Settlement Completed), to confirm successful completion.<br>- Actions: This message serves as the definitive trigger for banks to finalize the transaction on their internal ledgers.<br>- Bank A removes the hold on Hưng's account and formally debits the funds.<br>- It sends a confirmation notification to Hưng. |
| #6, 7 **Settlement** (NAPAS → SBV → NAPAS)<br>Instead of settling each transaction individually, NAPAS calculates the net positions of all banks over a short, defined clearing cycle (in seconds).<br>The SBV acts on the instruction from NAPAS to perform the final settlement. | Actions (NAPAS)<br>- At the end of a clearing cycle, NAPAS aggregates all cleared transactions.<br>- It calculates a single multilateral net position for each member bank.<br>- NAPAS submits a single, digitally signed net settlement file to the SBV.<br>- The SBV validates the file and executes the net transfers on the banks’ reserve accounts.<br>- Authentication: Verifies the digital signature on the settlement file to confirm it is an authentic instruction from NAPAS.<br>- Sufficient Funds: Checks that every bank with a net debit position has sufficient funds in its reserve account held at the SBV to cover its obligation. This is the most critical check.<br><br>Communication: SBV confirms the successful settlement back to NAPAS. Upon receiving confirmation of successful settlement from the SBV, NAPAS performs the final update on its internal position ledger.<br>- It marks the pending credits and debits for the settled cycle as done.<br>- The net amount transferred to Bank B's account at the SBV is now reflected in its overall available liquidity. Bank B can now allocate these settled funds to increase its sending liquidity position for future transactions.<br><br>Finality: Settlement is final and irrevocable upon completion of the net settlement cycle at the SBV.<br><br>Failure Case: This is a rare but serious event, primarily caused by a bank having insufficient funds in its reserve account to cover its net debit position.<br>- Rejection: The SBV will reject the entire settlement batch and notify NAPAS.<br>- Unwinding: NAPAS must then "unwind" the cycle. It sends notifications to all member banks that the settlement has failed.<br>- Impact: Bank B, which had already given a provisional credit to Phương based on the NAPAS guarantee, would be required to reverse the credit from her account. Bank A would remove the hold on Hưng's funds. The original transaction is effectively reversed, and the end-users are notified accordingly. This is a major disruptive event with systemic risk implications. |
| #8, 9, 10 **Credit Beneficiary & Notification** (Bank B → Receiver)<br>Bank B receives the `pacs.002` confirmation. | Actions<br>- Bank B's system performs final internal checks (e.g., account status) and runs last-mile AML.<br>- It credits Phương’s account with the transaction amount.<br>- It sends a notification (e.g., SMS, push notification) to Phương.<br>- Compliance: The receiving bank has a final opportunity to hold funds if its own checks detect high risk, placing them in a suspense account and filing an STR. |
| #11 Reconciliation | At the end of the day, all parties (Bank A, Bank B, NAPAS, and SBV) perform automated reconciliation to ensure their transaction logs match, maintaining the integrity of the financial system. |

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
