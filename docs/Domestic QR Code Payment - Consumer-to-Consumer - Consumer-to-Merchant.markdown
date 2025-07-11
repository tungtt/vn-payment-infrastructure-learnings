# QR Code Payment Flows in Vietnam: Consumer-to-Consumer (C2C) and Consumer-to-Merchant (C2M)

This document provides a technical overview of QR Code Payment flows in Vietnam, focusing on Consumer-to-Consumer (C2C) and Consumer-to-Merchant (C2M) scenarios. It is tailored for software engineers integrating with Vietnam’s payment infrastructure, detailing the VietQR standard developed by the National Payment Corporation of Vietnam (NAPAS). VietQR leverages the NAPAS 24/7 Near Real-Time Automated Clearing House (ACH) network for standardized QR-based payments.

## Overview

VietQR, launched on June 15, 2021, by NAPAS, is Vietnam’s national QR code payment standard. It operates as an overlay on the existing NAPAS 24/7 ACH network, enhancing payment initiation rather than replacing the underlying infrastructure. This ensures interoperability across over 40 banks, covering more than 99% of the market, and supports transactions under 500 million VND, operating 24/7.

**Key Characteristics**  
- **Interoperability**: A single VietQR code works with any participating bank’s mobile app.  
- **Standardization**: Follows the EMVCo Merchant-Presented Mode (MPM) QR Code Specification.  
- **Underlying Service**: Processes transactions as Napas247 Quick Money Transfers.  
- **Availability**: Runs continuously, leveraging the ACH infrastructure.

## Actors

| Actor               | Description                                                                 |
|---------------------|-----------------------------------------------------------------------------|
| Sender (Consumer)   | Initiates payment by scanning the QR code.                                  |
| Receiver (Payee)    | Receives funds; an individual (C2C) or merchant (C2M).                      |
| Issuing Bank        | Sender’s bank, initiates the transfer.                                      |
| Receiving Bank      | Receiver’s bank, credits the beneficiary.                                   |
| Clearing House (NAPAS) | Validates, clears, and routes payment instructions.                      |
| Settlement Bank (SBV)  | Executes final settlement between banks’ reserve accounts.                |

For C2M flows, additional actors include:

| Actor               | Description                                                                 |
|---------------------|-----------------------------------------------------------------------------|
| Merchant            | Business receiving payment.                                                 |
| POS Provider        | Orchestrates payment process, generates QR codes, and handles reconciliation.|

## Technical Details: VietQR Code Generation

VietQR codes adhere to the EMVCo MPM standard, using a Tag-Length-Value (TLV) structure. Each data object includes:
- **Tag**: Two-digit identifier (e.g., `00` for Payload Format Indicator).  
- **Length**: Two-digit field specifying the value’s length.  
- **Value**: The data content.

### Key Tags in VietQR Payload

| Tag ID | Field Name                  | Format | Max Length | M/C/O | Description                                          |
|--------|-----------------------------|--------|------------|-------|------------------------------------------------------|
| 00     | Payload Format Indicator    | N      | 2          | M     | "01" indicates EMVCo compliance.                     |
| 01     | Point of Initiation Method  | N      | 2          | C     | "11" (Static QR), "12" (Dynamic QR).                 |
| 38     | Merchant Account Information| S      | 99         | M     | Nested TLV with BIN, account number, service code.   |
| 52     | Merchant Category Code (MCC)| N      | 4          | O     | Business type (e.g., "5411" for grocery stores).     |
| 53     | Transaction Currency        | N      | 3          | M     | "704" for VND.                                       |
| 54     | Transaction Amount          | N      | 13         | C     | Included in Dynamic QR, omitted in Static QR.        |
| 58     | Country Code                | S      | 2          | M     | "VN" for Vietnam.                                    |
| 62     | Additional Data Field       | S      | 99         | C     | Includes bill number, purpose, etc.                  |
| 63     | CRC                         | S      | 4          | M     | 16-bit CRC-16/CCITT-FALSE checksum.                  |

### QR Code Generation Steps

1. **Gather Data**: Collect beneficiary details (mandatory) and transaction specifics (conditional).  
2. **Build TLV Elements**: Create TLV strings (e.g., `540575000` for 75,000 VND).  
3. **Concatenate**: Combine TLV elements in order.  
4. **Calculate CRC**: Compute CRC-16 over the string, append as a 4-character hex (e.g., `F6B3`).  
5. **Generate QR**: Use a library (e.g., ISO/IEC 18004-compliant) to create the QR image.

**Static vs. Dynamic QR Codes**  
- **Static QR**: Tag 01 = "11", reusable, no amount (C2C).  
- **Dynamic QR**: Tag 01 = "12", one-time use, includes amount (C2M).

## Flow 1: Consumer-to-Consumer (C2C) QR Code Payment

**Scenario**: Hưng (Bank A) transfers 75,000 VND to Phương (Bank B) using a static QR code.

### C2C Flow Steps

| Step | Description | Technical Details |
|------|-------------|-------------------|
| **1. QR Generation** | Phương generates a static QR code via Bank B’s app. | Payload: Tags 00 ("01"), 01 ("11"), 38 (BIN, account, "QRIBFTTA"), 53 ("704"), 58 ("VN"), 63 (CRC). |
| **2. QR Scanning** | Hưng scans the QR with Bank A’s app. | App parses TLV, populates beneficiary fields. |
| **3. Data Entry** | Hưng enters 75,000 VND and authenticates. | SBV Decision No. 2345/QD-NHNN applies (e.g., OTP). |

The subsequent steps mirror [the NRT ACH process post-initiation](Domestic%20Low-Value%20Interbank%20Credit%20Transfers%20(Near%20Real-Time%20ACH).markdown), with QR scanning replacing manual data entry.

## Flow 2: Consumer-to-Merchant (C2M) QR Code Payment

**Scenario**: Dũng pays 95,000 VND at a coffee shop using a dynamic QR code via KiotViet POS.

### Role of the POS Provider (KiotViet)

KiotViet, a non-financial orchestrator, bridges commerce and finance:  
- **QR Generation**: Requests dynamic QR codes via a Payment Gateway API.  
- **Reconciliation**: Updates merchant systems via webhooks post-payment.

### C2M Flow Steps

| Step | Description | Technical Details |
|------|-------------|-------------------|
| **1. Bill Finalization** | Cashier totals 95,000 VND on KiotViet POS. | Generates bill ID (e.g., "KV-1138"). |
| **2. QR Request** | KiotViet requests a QR from the Payment Gateway. | API POST: `{"amount": 95000, "billNumber": "KV-1138"}`. |
| **3. QR Generation** | Gateway creates a dynamic QR code. | Tags 01 ("12"), 54 ("95000"), 62 (bill number), 38 ("QRPUSH"). |
| **4. QR Scan** | Dũng scans the QR with his app. | Pre-filled details displayed. |
| **5. Authentication** | Dũng authenticates the payment. | Similar to C2C, SBV compliance. |
| **6. Clearing & Settlement** | Processed via NAPAS and SBV. | See [Domestic Low-Value Interbank Credit Transfers](Domestic%20Low-Value%20Interbank%20Credit%20Transfers%20(Near%20Real-Time%20ACH).markdown). |
| **7. Confirmation** | Receiving bank credits merchant, notifies Gateway. | API or message queue integration. |
| **8. Webhook** | Gateway sends webhook to KiotViet. | JSON: `{"status": "SUCCESS", "billNumber": "KV-1138"}`, HMAC-SHA256 signed. |
| **9. Reconciliation** | KiotViet marks bill as paid. | Automated via webhook validation. |

## Implementation Recommendations

- **QR Generation**:  
  - Static: Use libraries like `vietqr-generator` for C2C.  
  - Dynamic: Integrate with Payment Gateway APIs for C2M.  
- **Security**: Use TLS 1.2+ for APIs, HMAC-SHA256 for webhook verification.  
- **Compliance**: Adhere to SBV Decision No. 2345/QD-NHNN for authentication.

## References

- [Napas launches VietQR trademark, QR code quick money transfer service](https://en.vietnamplus.vn/napas-launches-vietqr-trademark-qr-code-quick-money-transfer-service-post203137.vnp)
- [ISO 20022](www.iso20022.org)
- [EMVCo QR Code standard](https://www.emvco.com/emv-technologies/qr-codes/)
- [VietQR Generation Library](https://pypi.org/project/vietqr-generator/)
