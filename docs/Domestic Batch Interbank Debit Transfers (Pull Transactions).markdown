# Domestic Batch Interbank Debit Transfers in Vietnam (DNS via NAPAS)

This document provides a detailed technical overview of the interbank payment flow for domestic batch debit transfers (pull transactions) in Vietnam, processed through the Automated Clearing House (ACH) system operated by the National Payment Corporation of Vietnam (NAPAS). It focuses on recurring payments such as subscriptions, utility bills, or loan repayments, typically for transactions valued at less than VND 500 million.

NAPAS ACH employs a Deferred Net Settlement (DNS) model with batch processing at scheduled intervals, ensuring efficient and cost-effective clearing of low to medium-value transactions.

*This guide is intended for software engineers and system architects integrating with Vietnam’s payment infrastructure.*

## Overview

NAPAS ACH is a critical component of Vietnam’s payment infrastructure, facilitating batch debit transfers for recurring payments. It operates on a DNS model, where transactions are accumulated and netted at scheduled intervals, reducing the number of interbank settlements.

**Key Characteristics**
- **Threshold**: Typically for transactions < VND 500 million.
- **Settlement Model**: Deferred Net Settlement (DNS) with batch processing.
- **Operating Hours**: Batches are processed at scheduled intervals throughout the business day.
- **Regulatory Framework**: Governed by relevant SBV regulations and circulars.
- **Messaging Standards**: ISO 20022 (e.g., `pain.008`, `pacs.003`, `pacs.002`).

## Actors

**Scenario**: A creditor (e.g., VieOn) initiates a batch debit transfer to collect recurring payments from a debtor (e.g., Dũng) via their respective banks.

| Actor               | Description                                                                 |
|---------------------|-----------------------------------------------------------------------------|
| Creditor            | Initiates the debit transfer (e.g., VieOn).                                 |
| Creditor's Bank     | The creditor’s bank (Bank A), responsible for initiating the debit request. |
| NAPAS               | The clearing house that processes and routes the debit requests.            |
| Debtor's Bank       | The debtor’s bank (Bank B), responsible for debiting the debtor’s account.  |
| Debtor              | Authorizes the debit (e.g., Dũng).                                          |
| SBV                 | The State Bank of Vietnam, responsible for final settlement of net positions. |

## Technical Flows

### Electronic Mandate Registration & Immediate First Debit

Electronic mandates are used to authorize recurring debit transfers, with the first debit occurring immediately after registration.

#### Steps for Mandate Registration

| Step | Description | Compliance and Regulatory Considerations |
|------|-------------|------------------------------------------|
| **1. Mandate Initiation** | The debtor accesses the creditor’s platform and selects the service. | - |
| **2. Mandate Details Entry** | The debtor enters their bank account details, bank identifier, and reviews the terms. | - |
| **3. Authentication and Authorization** | The debtor authenticates using OTP, biometrics, or digital signature. | Complies with the *Law on E-Transactions 2023* and *Decision No. 2345/QD-NHNN*. |
| **4. Mandate Submission** | The debtor submits the mandate, and the creditor stores the details securely. | Mandate details must be encrypted and stored securely. |
| **5. Mandate Registration with Banks** | The creditor sends the mandate to Bank A, which assigns a unique mandate ID and informs Bank B via NAPAS. | - |
| **6. Verification and Registration** | Bank B verifies the debtor’s identity and registers the mandate. | Bank B performs Customer Due Diligence (CDD) per *Law No. 14/2022/QH15*. |
| **7. Confirmation** | Bank B confirms the mandate registration to Bank A and the creditor, who then notifies the debtor. | - |

#### Steps for Immediate First Debit

| Step | Description | Compliance and Regulatory Considerations |
|------|-------------|------------------------------------------|
| **1. Debit Initiation** | The creditor initiates the first debit request immediately after mandate registration. | Explicit consent is required for the debit. |
| **2. Validation by Bank A** | Bank A validates the debit request against the mandate terms. | - |
| **3. Batch Submission to NAPAS** | Bank A includes the debit request in the next batch submitted to NAPAS. | - |
| **4. Clearing** | NAPAS processes the batch and routes the debit request to Bank B. | - |
| **5. Processing by Bank B** | Bank B validates the mandate, checks for sufficient funds, and provisionally debits the debtor’s account. | Bank B sends a `pacs.002` status report to Bank A via NAPAS, indicating acceptance (ACCP) or rejection (RJCT) with a reason code (e.g., insufficient funds, invalid account, mandate not found). |
| **6. Settlement** | NAPAS calculates the net positions, and SBV settles the accounts. | - |
| **7. Account Updates and Notifications** | Bank B finalizes the debit and notifies the debtor. Bank A credits the creditor and notifies them. | - |

**Note on Exception Handling**: If a debit is settled but later reversed (e.g., due to a customer dispute or error), the system uses a `pacs.004` message to return the funds. This ensures compliance with ISO 20022 standards for handling rejects, returns, and refunds.

### Recurring Debit Transactions

Subsequent debits are processed on the agreed dates following the initial setup.

#### Steps for Recurring Debits

| Step | Description | Compliance and Monitoring |
|------|-------------|---------------------------|
| **1. Scheduled Debit Initiation** | The creditor generates the debit request on the agreed date. | - |
| **2. Validation and Batch Submission** | Bank A validates the request and includes it in the next batch to NAPAS. | - |
| **3. Clearing and Settlement** | NAPAS processes the batch, routes the request to Bank B, and SBV settles the net positions. | - |
| **4. Processing by Bank B** | Bank B validates the mandate, checks funds, and debits the debtor’s account. | Bank B verifies mandate validity and performs ongoing AML/KYC checks. Suspicious transactions are reported to SBV. Sends `pacs.002` with ACCP or RJCT status. |
| **5. Account Updates and Notifications** | Bank B notifies the debtor of the debit. Bank A credits the creditor and notifies them. | - |

### Recurring Payment Cancellation

The debtor can revoke the mandate at any time to stop future debits.

#### Steps for Mandate Revocation

| Step | Description | Compliance Considerations |
|------|-------------|---------------------------|
| **1. Cancellation Request** | The debtor requests cancellation via the creditor’s platform. | The debtor has the right to revoke the mandate at any time. |
| **2. Mandate Revocation** | The creditor updates their records and notifies Bank A. | - |
| **3. Notification to Bank B** | Bank A deactivates the mandate and informs Bank B via NAPAS. | - |
| **4. Mandate Deactivation** | Bank B verifies the request and deactivates the mandate. | - |
| **5. Confirmation** | Bank B confirms the deactivation to Bank A, who then notifies the creditor. | - |
| **6. Future Transactions** | Bank B rejects any further debit requests for the revoked mandate. | - |

## Message Standards

The system uses ISO 20022 messages for communication between parties:

- **`pain.008`**: Used by the creditor to initiate the debit request to Bank A. Includes MandateRelatedInformation (e.g., Mandate ID, signature date).
- **`pacs.003`**: Sent by Bank A to NAPAS, then forwarded to Bank B to instruct the debit. Carries mandate details.
- **`pacs.002`**: Sent by Bank B to NAPAS (and then to Bank A) to report the debit request outcome—either acceptance (ACCP) or rejection (RJCT with a reason code, e.g., insufficient funds).

**Note**: Each `pacs.003` message should receive a corresponding `pacs.002` reply, especially if the debit is rejected.

## Compliance and Regulatory Considerations

- **Electronic Mandates**: Must comply with the *Law on E-Transactions 2023* and *Decision No. 2345/QD-NHNN*.
- **Explicit Consent**: No debits will be processed without a valid mandate on file, obtained with explicit consent from the debtor.
- **Mandate Data**: The Mandate ID and signature date must be included in ISO 20022 messages (e.g., `pain.008`, `pacs.003`) to link each debit to the debtor’s consent.
- **Data Protection**: All mandate and transaction data must be encrypted and securely stored.
- **Consumer Rights**: The debtor can revoke the mandate at any time, and banks must honor the revocation promptly.

## References

- [The National Payment Corporation of Vietnam (NAPAS) Launched a New ACH System](https://www.cma.se/news/the-national-payment-corporation-of-vietnam-napas-launched-a-new-ach-system)
- [PACS.003 ISO 20022 Message](https://www.cpg.de/en/glossary/pacs-003-iso-20022-message/)
- [SEPA Direct Debit (SDD) Process & Message Flow from Creditor to Debtor](https://www.linkedin.com/pulse/sepa-direct-debit-sdd-process-message-flow-from-creditor-samgra-malik-mb1hc)
- [PACS.002 ISO 20022 Message](https://www.cpg.de/en/glossary/pacs-002-iso-20022-message/)
- [Overview of Decision 2345/QD-NHNN: Important Points for Timo Users](https://timo.vn/en/blogs-en/overview-of-decision-2345-qd-nhnn-important-points-for-timo-users/)
- [Vietnam: Anti-Money Laundering Law Takes Effect](https://www.loc.gov/item/global-legal-monitor/2023-05-04/vietnam-anti-money-laundering-law-takes-effect/?loclr=ealln)
- [Navigating E-Transactions in Vietnam: Unveiling the 2023 Law on Electronic Transactions](https://vietnam.acclime.com/news-insights/navigating-e-transactions-in-vietnam-unveiling-the-2023-law-on-electronic-transactions/)
