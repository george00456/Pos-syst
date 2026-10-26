# OmniPOS — QuickBooks POS Edition: System Architecture & Implementation Blueprint

An offline-first, subscription-free Point of Sale (POS) application engineered with Intuit QuickBooks POS desktop & cloud aesthetic, complete manager security authorization matrix, Accounts Receivable Customer Center, End-of-Day Z-reports with bill & coin counter, QuickBooks General Ledger chart-of-accounts sync, and installable Progressive Web App (PWA) capabilities for mobile and desktop devices.

---

## User Review & Critical Decisions

> [!IMPORTANT]
> **Summary of Implemented Core Architecture**:
> - **$0 Monthly Fee Model**: Replaces traditional $79–$180/mo recurring SaaS charges with a transparent pay-per-sale micro-commission structure and built-in monthly savings calculator.
> - **Manager Authorization Matrix**: Sensitive operations (discounts exceeding thresholds, price overrides, voids/returns, inventory adjustments, and End-of-Day Z-Reports) strictly require Manager PIN (`1234`) authorization.
> - **QuickBooks Desktop & Cloud DNA**: Navigator Ribbon (`[F1]` Make a Sale, `[F6]` Customers, `[F2]` Item List, `[F3]` Receiving, `[F8]` End of Day, `[F10]` QB GL Sync, `[F12]` Manager Settings).
> - **Universal Mobile PWA**: Instant home-screen install on iOS Safari (via Share → Add to Home Screen) and Android Chrome (via 1-click `beforeinstallprompt`), complete with Service Worker offline caching.

---

## 1. Overview & Core Concept

### What It Does
OmniPOS is a high-speed retail checkout and store management system combining:
1. **Front-of-House Register**: Barcode scanning, item lookups, park/layaway orders (`[F4]`), Return & Exchange mode (`[F7]`), split tender, and printable 80mm thermal receipts.
2. **Customer Center**: Complete CRM with Accounts Receivable (A/R) balance tracking, credit limits, Net 15/30 invoice terms, and resale tax-exemption verification.
3. **Inventory & Purchasing**: QuickBooks Item List with reorder thresholds, vendor pricing, and Receiving Vouchers (`[F3]`) linked to Purchase Orders.
4. **Cash Drawer & Financial Audit**: End of Day Z-Out (`[F8]`) with denomination counters ($100 to 1¢), Over/Short variance audits, and paid-in/paid-out drawer logs.
5. **QuickBooks GL Integration**: Double-entry journal voucher generator posting to standard Chart of Accounts (1050 Till, 1060 Card Clearing, 1200 A/R, 2100 Sales Tax Payable, 4000 Sales Income, 5000 COGS, 6100 Commission Expense), with one-click `.IIF` / CSV export.

### Target Audience & Persona
- **Retail Store Owners & Managers**: Demanding total audit control, drawer oversight, and QuickBooks export without paying monthly software rent.
- **Cashiers & Baristas**: Needing rapid touchscreen checkout, numeric PIN terminal unlocking, and commission incentive tracking.
- **Mobile / Floor Sales Staff**: Using iPhones, iPads, and Android tablets to ring up sales and look up inventory anywhere in the store.

---

## 2. User Experience & Visual Design

### Aesthetic Direction & Intuit QuickBooks POS DNA
- **Navy Chrome (`#0c2340`)**: Classic QuickBooks POS header and application shell.
- **Intuit Blue (`#0077c5`)**: Interactive navigation buttons, links, and selected states.
- **QuickBooks Green (`#2ca01c`)**: Charge action buttons, positive cash totals, and successful sync indicators.
- **Typography**: `Plus Jakarta Sans` for clean, modern legibility paired with `JetBrains Mono` for currency figures, quantities, barcodes, and GL ledger numbers.
- **Viewport Layout**: 1-row fixed header, sticky horizontal Navigator Ribbon, flex-grow content area with no double scrollbars, and a bottom register status bar (`Workstation · Till Balance · Current Cashier · GL Status`).

### Step-by-Step User Flows
1. **Terminal Unlock**: Cashier selects their profile on the lock screen and enters their 4-digit PIN. Terminal unlocks instantly into the Register.
2. **Sale Ring-Up**: Scan or click items into the ticket ledger. Cashier can adjust quantity, apply discounts (with manager override if >15%), or assign a customer.
3. **Tender**: Click **[F1] Tender** to open payment modal. Tender options include Cash (with quick $20/$50/$100 bills and change calculation), Card, Check, Split Tender, or **On Account** (charges to customer A/R balance under credit terms).
4. **Receipt & Auto-Lock**: System prints 80mm thermal receipt with QuickBooks reference number and barcodes, updates real-time inventory, and auto-locks back to the employee PIN pad if configured.
5. **End of Day**: Manager opens **[F8] End of Day**, enters physical counts for bills and coins, reviews the automated Over/Short reconciliation, and submits the finalized Z-Report to the General Ledger.

---

## 3. Key Architectural Decisions & Trade-Offs

| Decision | Chosen Architecture | Rationale & Trade-Offs |
| :--- | :--- | :--- |
| **State Persistence** | Centralized `POSContext` with automatic `localStorage` synchronization | Instant reactivity, zero latency at checkout counter, and 100% functional without server network roundtrips. |
| **Offline Register** | `vite-plugin-pwa` with Workbox runtime caching and Service Worker | Critical for retail continuity; if store Wi-Fi drops, tickets, catalog items, and receipt generation continue uninterrupted. |
| **Manager Authorization** | Modal-driven PIN interceptor (`openPinModal`) | Instead of navigating away to a settings page, cashier actions that exceed privileges pop up a non-blocking 4-digit manager PIN pad right on the active screen. |
| **QuickBooks Compatibility** | Double-entry `.IIF` (Intuit Interchange Format) & CSV export | Native format supported by all versions of QuickBooks Desktop (Pro, Premier, Enterprise) and QuickBooks Online journal import. |
| **Subscription-Free Model** | Configurable micro-commission engine (default 1.25%) | Transparent, pay-per-sale model replaces fixed monthly fees; includes an interactive Savings Calculator showing ROI over standard POS systems. |

---

## 4. Technical Architecture & System Layout

### System Component Diagram

```
┌────────────────────────────────────────────────────────────────────────┐
│                        OmniPOS Cloud Web Client                        │
│               (React 19 + TypeScript + Vite + Tailwind CSS)            │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │
    ┌───────────────────────────────┴───────────────────────────────┐
    ▼                                                               ▼
┌───────────────────────────┐                   ┌───────────────────────────┐
│     Top Chrome Header     │                   │   Terminal Lock Screen    │
│  - Branding & Till Total  │                   │  - Live Digital Clock     │
│  - Manager Lock/Unlock    │                   │  - Employee Avatar Picker │
│  - Active Cashier Switch  │                   │  - 4-Digit Numeric Keypad │
│  - Parked Orders Recall   │                   │  - Mobile QR / Install    │
│  - PWA Install Trigger    │                   └───────────────────────────┘
└─────────────┬─────────────┘
              │
    ┌─────────┴─────────────────────────────────────────────────────┐
    ▼                                                               ▼
┌────────────────────────────────────────────────────────┐  ┌───────────────────┐
│            QuickBooks POS Navigator Ribbon             │  │   Service Worker  │
│  [F1] Sale  [F6] CRM  [F2] Items  [F3] Receive         │  │   Offline Cache   │
│  [F8] Z-Out [F10] GL  [F12] Settings  [$0] Savings     │  │   (VitePWA / SW)  │
└───────────────────────────┬────────────────────────────┘  └───────────────────┘
                            │
  ┌─────────────────────────┼─────────────────────────┐
  ▼                         ▼                         ▼
┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐
│  Make a Sale     │  │ Customer Center  │  │  Item List &     │
│  - Barcode Scan  │  │  - A/R Balances  │  │  Receiving       │
│  - Return [F7]   │  │  - Net 30 Terms  │  │  - Reorder Points│
│  - Ledger Ticket │  │  - Credit Limits │  │  - Vendor Costs  │
│  - Tender Modal  │  │  - Tax Exemption │  │  - PO Vouchers   │
└─────────┬────────┘  └─────────┬────────┘  └─────────┬────────┘
          │                     │                     │
          └─────────────────────┼─────────────────────┘
                                ▼
┌────────────────────────────────────────────────────────────────────────┐
│                        POSContext State Engine                         │
│  - products, cart, customers, staffList, activeStaff, heldOrders       │
│  - transactions, drawerLogs, endOfDayReports, settings, isManager      │
│  - completeTransaction(), receiveStock(), submitEndOfDay(), syncQB()   │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │
  ┌─────────────────────────────────┼─────────────────────────────────┐
  ▼                                 ▼                                 ▼
┌──────────────────┐      ┌──────────────────┐      ┌──────────────────┐
│  Cash Drawer     │      │  Double-Entry    │      │  PWA & Mobile    │
│  Reconciliation  │      │  GL Journal Sync │      │  Integration     │
│  - Bill & Coin   │      │  - Accounts 1000 │      │  - 1-Click Install│
│    Counter ($100 │      │    through 6000  │      │  - iOS Safari Add│
│    to 1¢)        │      │  - .IIF / CSV    │      │    to Home Screen│
│  - Over / Short  │      │    Export        │      │  - Responsive UI │
└──────────────────┘      └──────────────────┘      └──────────────────┘
```

---

## 5. Core Data Models (`src/types/pos.ts`)

```typescript
export type StaffRole = 'manager' | 'cashier' | 'keyholder' | 'associate';

export interface StaffPermissions {
  canDiscountMaxPercent: number; // e.g. 15% without manager override
  canOverridePrice: boolean;
  canProcessReturns: boolean;
  canAdjustInventory: boolean;
  canPerformEndOfDay: boolean;
  canViewCostAndMargins: boolean;
}

export interface Customer {
  id: string;
  name: string;
  companyName?: string;
  email: string;
  phone: string;
  accountBalance: number;       // Outstanding A/R balance
  creditLimit: number;          // Maximum allowable on-account balance
  paymentTerms: 'Due on Receipt' | 'Net 15' | 'Net 30' | 'Net 60';
  isTaxExempt: boolean;
  taxExemptionNumber?: string;
}

export interface Transaction {
  id: string;
  timestamp: string;
  receiptNumber: string;
  items: CartItem[];
  subtotal: number;
  tax: number;
  total: number;
  paymentMethod: 'cash' | 'card' | 'split' | 'on_account';
  staffId: string;
  staffName: string;
  customerId?: string;
  customerName?: string;
  platformFee: number;
  qbJournalRef: string;        // Double-entry GL voucher reference
  isReturn?: boolean;
}

export interface EndOfDayReport {
  id: string;
  date: string;
  closingTime: string;
  managerId: string;
  openingFloat: number;
  cashSales: number;
  cardSales: number;
  onAccountSales: number;
  grossSales: number;
  returnsTotal: number;
  taxCollected: number;
  netSales: number;
  expectedCashInDrawer: number;
  actualCashCounted: number;
  overShort: number;
  billCounts: { [denom: string]: number };
  status: 'balanced' | 'over' | 'short';
  qbSyncStatus: 'synced' | 'pending';
}
```

---

## 6. Implementation Modules Breakdown

### Module 1: POS Central State Engine (`src/context/POSContext.tsx`)
- Coordinates store-wide state, synchronizing changes to `localStorage` with versioned keys (`omnipos_*_v2`).
- Implements `unlockManager(pin)`, `verifyManagerPin(pin)`, `lockTerminal()`, and `unlockTerminal(pin)`.
- Handles transaction completion with auto-lock check, customer A/R balance updates, stock decrements, and drawer cash accumulation.

### Module 2: Front Register & Checkout (`src/components/Register/*`)
- **ProductCatalog**: Real-time category filtering, quick search, barcode scanner listener, Return/Exchange toggle (`[F7]`), and custom non-catalog item modal.
- **CartSidebar**: Tabular ledger with line-item discounts, price overrides (prompting for manager PIN if required), layaway parking (`[F4]`), and tender total.
- **PaymentModal**: Split tender, cash quick bills ($20, $50, $100), change calculator, and "On Account" payment charging customer A/R balances.
- **Thermal Receipt**: Printable 80mm receipt with business header, tax ID, cashier info, line items, and QuickBooks audit reference.

### Module 3: Customer Center (`src/components/Customers/CustomerCenterView.tsx`)
- Customer directory with live search, Accounts Receivable ledger balance, available credit bar, Net 15/30 credit terms, resale exemption certificate, and customer order history.

### Module 4: Inventory & Receiving (`src/components/Inventory/*`)
- **InventoryView**: QuickBooks Item List with SKU, department, on-hand count, reorder point, unit cost, selling price, and profit margin (protected by cashier visibility permissions).
- **ReceivingView**: Vendor receiving voucher intake with Purchase Order number, quantity received, unit cost updates, and automated stock replenishment.

### Module 5: End of Day Z-Report (`src/components/EndOfDay/EndOfDayView.tsx`)
- Step-by-step drawer reconciliation with bill counter ($100, $50, $20, $10, $5, $1) and coin counter ($1, 25¢, 10¢, 5¢, 1¢).
- Live Over/Short calculation against expected cash from cash sales, cash refunds, and opening float.
- Printable official Z-Report slip with drawer pay-in/pay-out audit log.

### Module 6: QuickBooks General Ledger Sync (`src/components/Accounting/QBSyncView.tsx`)
- Live Chart of Accounts balances (1050 Drawer Cash, 1060 Card Clearing, 1200 A/R, 2100 Sales Tax Payable, 4000 Sales Income, 5000 COGS, 6100 Commission Expense).
- Double-entry journal voucher logs with balanced debits and credits.
- One-click `.IIF` (Intuit Interchange Format) and CSV file export for direct import into QuickBooks Desktop and Online.

### Module 7: Manager Security Matrix & Settings (`src/components/Settings/SettingsView.tsx`)
- Protected by Master Manager PIN (`1234`).
- Cashier override policy configuration: maximum cashier discount %, price override approval, void/refund approval, inventory adjust approval, and margin hiding.
- Company details, tax rates, opening cash float, and staff roster management.

### Module 8: Multi-Platform PWA Installation (`src/components/Common/InstallAppModal.tsx`, `vite.config.ts`)
- Progressive Web App configured via `vite-plugin-pwa` with automatic service worker updates and asset precaching.
- High-resolution icons (`pwa-192x192.png`, `pwa-512x512.png`, `pwa-maskable-512x512.png`, and `apple-touch-icon.png`).
- Interactive install modal offering 1-click install for Android/Chrome/PC, step-by-step 4-stage guide for iPhone/iPad Safari (Share → Add to Home Screen), QR code scanner, and direct link copy.
- Non-intrusive `OfflineIndicator` toast alerting staff when operating on cached local data.

---

## 7. Execution & Build Verification

The application is fully compiled and running on the dev server:
- **Build Command**: `vite build` (`npm run build`)
- **Lint Validation**: `tsc --noEmit` (`npm run lint`) passed with 0 errors.
- **Preview / Shared URL**: `https://ais-pre-oysj5dbv5x5dxhgm2tctbl-335541482947.europe-west2.run.app`
- **Default Master PINs**:
  - Manager (David Miller): `1234`
  - Cashier (Sarah Connor): `1001`
  - Cashier (Marcus Vance): `1002`
  - Barista / Sales (Elena Rostova): `1003`
