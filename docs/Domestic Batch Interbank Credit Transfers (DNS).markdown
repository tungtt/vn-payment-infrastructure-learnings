# Part 2: Domestic Batch Interbank Credit Transfers (Payroll and Bulk Payments)

For recurring, one-to-many payments, such as salaries or supplier payments, batch processing through the Automated Clearing House (ACH) system, operated by the National Payment Corporation of Vietnam (NAPAS), is utilized. This section outlines the workflow, actors, and technical specifications for these transfers, ensuring compliance with regulatory requirements and alignment with industry standards.

## 2.1 Overview of the NAPAS ACH Batch Service

- **System Identification:** The ACH system, managed by NAPAS, facilitates batch processing of retail payments in Vietnam, connecting member banks for efficient transaction clearing.
- **Settlement Model:** Utilizes batch processing with Deferred Net Settlement (DNS). Payment instructions are collected into batches, processed at scheduled intervals, and netted for final settlement at the State Bank of Vietnam (SBV).
- **Common Use Cases:** Designed for high-volume, low-value payments, including corporate payroll, bulk supplier payments, and other mass disbursements.
- **Value Limit:** Individual transactions within a batch must be less than VND 500 million, though the cumulative value of a batch can be significantly higher.
- **Clearing Cycles:** NAPAS operates multiple clearing cycles daily, enabling several batch processing windows.

## 2.2 Actors

| Actor | Description |
| --- | --- |
| **Corporate Client** | Initiates the batch payment, typically a company’s finance or HR department preparing payroll or supplier payments. |
| **Originating Bank** | The corporate client’s bank, responsible for validating and processing the batch file. |
| **NAPAS** | The clearing house that processes, validates, and routes batch payment instructions. |
| **Receiving Banks** | Banks that receive and credit payments to beneficiaries’ accounts. |
| **SBV** | The State Bank of Vietnam, which performs final, irrevocable settlement by adjusting banks’ reserve accounts. |

## 2.3 Technical Flow of a Batch Transfer

The following table details the steps involved in processing a batch transfer, including compliance requirements, risks, and potential failure cases, ensuring a robust implementation framework for engineers.

| Step | Description | Compliance, Risks, and Failure Cases |
| --- | --- | --- |
| **1. File Preparation** | The corporate client creates a bulk payment file (e.g., Excel, CSV) containing transaction details such as recipient account numbers, bank details, names, amounts, and remarks. | **Compliance:** Ensure accuracy of recipient information to prevent misdirected payments. **Risks:** Errors in data entry may lead to payment delays or incorrect credits. |
| **2. File Upload and Authorization** | The corporate user uploads the file to the bank’s online platform (e.g., Vietcombank’s VCB-iB@nking) and authorizes it, often through a multi-level approval process (e.g., maker-checker workflow). | **Compliance:** Multi-level authorization mitigates unauthorized payment risks, per Decision No. 2345/QD-NHNN (effective July 1, 2024). **Failure Case:** Unauthorized or unapproved batches are rejected. |
| **3. File Validation** | The bank validates the file’s format, data integrity (e.g., checksums, valid record counts), and ensures sufficient funds in the corporate client’s account, placing a hold on the total batch amount. | **Compliance:** Performs Customer Due Diligence (CDD) and sanctions screening per Law No. 14/2022/QH15 and Decree No. 19/2023/NĐ-CP. **Failure Case:** Batch rejected if the file is malformed, contains errors, or lacks sufficient funds. The bank notifies the client via the online portal. |
| **4. Batch Submission to NAPAS** | The bank converts the validated file into an ISO 20022 `pain.001` (Customer Credit Transfer Initiation) message and transmits it to NAPAS via a secure channel (e.g., HTTPS with mutual TLS and digital signatures). | **Compliance:** Secure transmission and correct formatting are critical to meet NAPAS’s technical standards. **Failure Case:** NAPAS rejects the message if it fails format validation or security checks. |
| **5. Clearing** | NAPAS processes the batch during a designated clearing cycle, validating the message format, digital signature, and checking for duplicates using unique instruction IDs. It aggregates transactions, sorts them by receiving bank, and calculates multilateral net positions. | **Compliance:** NAPAS conducts fraud detection and compliance checks, including velocity checks and watchlist screening. **Failure Case:** Batch rejected if it fails validation or compliance checks, with NAPAS sending a `pacs.002` (Payment Status Report) with a rejection code to the originating bank. |
| **6. Settlement** | NAPAS compiles a net settlement file and sends it to SBV, which validates the file and executes net transfers by adjusting banks’ reserve accounts. Settlement is final and irrevocable. | **Compliance:** SBV ensures banks have sufficient reserves per regulatory requirements. **Failure Case:** If a bank lacks sufficient funds, SBV rejects the settlement, NAPAS unwinds the cycle, and banks are notified. The originating bank releases the hold on the client’s funds. |
| **7. Beneficiary Crediting** | NAPAS notifies receiving banks via ISO 20022 `pacs.002` messages. Receiving banks perform final checks (e.g., account status, AML) and credit beneficiaries’ accounts, notifying them via SMS or push notifications. | **Compliance:** Receiving banks may hold funds in a suspense account and file a Suspicious Transaction Report (STR) if checks detect high risk, per SBV’s AML regulations. **Failure Case:** Funds returned if accounts are invalid or closed, with NAPAS and the originating bank notified for reversal. |
| **8. Reconciliation** | Banks, NAPAS, and SBV perform end-of-day reconciliation to ensure transaction logs match, maintaining system integrity. | **Compliance:** Automated reconciliation ensures compliance with SBV’s oversight requirements. **Failure Case:** Discrepancies trigger manual investigation, but no direct transaction failure occurs at this step. |

## 2.4 File Format Specifications for Implementation

### Corporate-to-Bank Interface

Commercial banks in Vietnam typically provide corporate clients with proprietary file formats, such as Microsoft Excel (.xls, .xlsx) or CSV, for submitting batch payments. These formats are designed to be user-friendly, with fields for recipient account numbers, bank details, names, amounts, and remarks. Developers must consult their specific bank’s documentation for the exact file structure and requirements, as formats vary across institutions. For example, Vietcombank’s VCB-iB@nking platform specifies its own format for bulk payment files, which may include mandatory fields and validation rules.

### Bank-to-NAPAS Interface

The NAPAS ACH system, modernized in 2020, is built on the ISO 20022 standard. For batch payments, banks submit payment instructions to NAPAS using the `pain.001` (Customer Credit Transfer Initiation) message, an XML-based format that supports structured data for multiple transactions. This standard ensures interoperability and rich data exchange between banks and NAPAS. While NAPAS may support an ISO 8583-based interface for real-time transactions (e.g., card payments), batch payments rely on ISO 20022. As the industry progresses toward full ISO 20022 adoption, banks may offer corporates advanced file upload options aligned with this standard.

## Additional Notes

- **Clearing Cycles:** NAPAS operates multiple clearing cycles daily, typically at set intervals, allowing for efficient batch processing.
- **Transaction Types:** While the ACH system supports both credit and debit transactions, this section focuses on credit transfers for payroll and bulk payments.
- **Regulatory Compliance:** Banks are responsible for AML and KYC checks, particularly during file validation and beneficiary crediting, ensuring adherence to SBV regulations such as Law No. 14/2022/QH15 and Decree No. 19/2023/NĐ-CP.

## Citations

- NAPAS Official Website
- CMA.SE - NAPAS ACH System
- VnExpress - Vietnamese Banks ISO 20022 Migration
- State Bank of Vietnam Regulations
  