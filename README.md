# Vietnam Payment Infrastructure (July 2025)
Vietnam Payment Infrastructure Learnings &amp; Prototypes.

## Payment flows
These are the payment flows that will be discussed in this repository.

### 1. Bank-based Payments
- **Interbank Transfers**
  - *Domestic*
    - [High-value or urgent or foreign-currency transfers (RTGS via NIEPS)](docs/Domestic%20High-Value%20Interbank%20Credit%20Transfers%20(RTGS).markdown)
    - [Low-value transfers (NRT ACH via NAPAS)](docs/Domestic%20Low-Value%20Interbank%20Credit%20Transfer%20(Near%20Real-Time%20ACH).markdown)
    - Deferred Net Settlement (DNS via NAPAS)
  - *International*
    - SWIFT transfers
    - Other correspondent banking networks
- **Card Payments**
  - Domestic cards (NAPAS)
  - International cards (Visa, Mastercard, JCB, etc.)
  - Transactions: POS, ATM, online

### 2. Mobile-based Payments
- E-wallets (MoMo, ZaloPay, VNPay, etc.)
  - P2P transfers
  - Merchant payments
  - Bill payments
- QR code payments (VietQR)
- Mobile Money (telecom-based)

### 3. Online Payment Services (e.g., PayPal, Stripe, Ngân Lượng)

### 4. Money Transfer Operators (MTOs; e.g., using Western Union, MoneyGram)

### 5. Cryptocurrency Payments
- Via exchanges (e.g., Binance, Remitano)
- Via payment gateways (e.g., NOWPayments)

### 6. Other Payment Methods
- Buy Now, Pay Later (BNPL)
- Peer-to-Peer (P2P) lending
