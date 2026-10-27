# BFSI Domain Pack — shared domain knowledge for BMAD agents

Read the **Cross-cutting** section always, plus the sections that match `project.subdomains` in
`.github/bmad/bmad.config.yaml`. The rules agents enforce are in `bfsi-rules.md`; this file explains the domain so
that PRDs, stories, designs, code and tests use the right concepts and words.

> Orientation only. Regulatory facts and dates below are summaries for engineers and change over time. A qualified
> compliance, legal or tax professional confirms how any regulation applies. When the project rulebook and this pack
> disagree, the project rulebook wins.

Section index: 1 Cross-cutting · 2 Capital markets (orders, execution, allocation, confirmation, clearing and
settlement, positions, corporate actions, reference data, FIX) · 3 Retail banking and ledgers · 4 Payments ·
5 Lending · 6 Cards · 7 Wealth management · 8 Insurance · 9 Financial crime and compliance ·
10 Data protection, security and resilience · 11 Glossary

---

## 1. Cross-cutting — applies to every BFSI application

**Money is exact.** Decimal types, explicit currency, explicit rounding (`BFSI-MON-*`). Banker's rounding
(HALF_EVEN) is common for accruals; some fee and tax rules mandate HALF_UP. The rulebook states which.

**Two kinds of time.** Instants (when something happened, UTC) and business dates (which business day it belongs
to: trade date, value date, booking date). End-of-day and cut-offs are in the market's or entity's time zone.

**Calendars.** Each market, currency and payment scheme has its own holidays. T+1 means one *business* day after
trade date on that market's calendar. Roll conventions decide what happens when a date falls on a holiday.

**Lifecycles are forward-only.** Financial records are corrected by new records (reversal, cancel/correct, bust,
amendment with version), never by editing history. Auditors and regulators must be able to reconstruct what was
known when.

**Idempotency everywhere.** Networks retry and messages are delivered at least once. Every state-changing request
carries a key, and every consumer tolerates duplicates.

**Four-eyes (maker-checker).** Changes to limits, rates, fee schedules, reference data, user entitlements and manual
adjustments are made by one person and approved by another.

**Reconciliation.** Every book of record is reconciled against an independent source (exchange, broker, custodian,
nostro statement, scheme report). Breaks are investigated, not overwritten.

**Segregation of duties and entitlements.** Typical roles: front office (trader, relationship manager), middle office
(risk, trade support), back office (operations, settlements), compliance, finance, administrators. Entitlement
checks happen server-side per record.

**Typical non-functional needs.** High availability during market or banking hours, deterministic end-of-day batch,
audit retention measured in years, strict data classification, change control.

---

## 2. Capital markets — front to back

```
Order capture → Pre-trade risk & compliance → Routing / Execution → Fills
    → Allocation → Confirmation / Affirmation → Clearing → Settlement
    → Positions, P&L, Custody → Corporate actions → Regulatory & client reporting
```

### 2.1 Orders

- **Side:** Buy, Sell, Sell Short (needs a locate or borrow in many markets), Sell Short Exempt, Buy to Cover.
- **Order types:** Market, Limit, Stop, Stop Limit, plus algorithmic or venue-specific types (VWAP, TWAP, iceberg,
  pegged). Limit price is required for Limit and Stop Limit, and absent for Market.
- **Time in force:** DAY, GTC (good till cancelled, usually with a maximum life), IOC (immediate or cancel),
  FOK (fill or kill), GTD (good till date), OPG / CLS (at the open / close auction).
- **Quantities:** order quantity, cumulative filled quantity, leaves quantity (= order − filled for a live order),
  minimum quantity, display quantity. Lot size and tick size come from instrument reference data.
- **Typical order states** (aligned with FIX `OrdStatus`, tag 39): Pending New, New, Partially Filled, Filled,
  Pending Cancel, Cancelled, Pending Replace, Replaced, Rejected, Expired, Suspended, Done for Day.
  A project may use its own names; the rulebook's state machine is authoritative.
- **Care vs DMA:** a care order is worked by a trader for the client; a direct market access order goes straight to
  the venue after automated checks.
- **Parent / child:** a parent (client) order may be split into child orders sent to different venues; fills roll up
  to the parent.

### 2.2 Pre-trade risk and compliance

- **Limits:** buying power or credit limit, maximum order quantity and notional, daily traded value, position
  limits, concentration limits.
- **Fat-finger checks:** price deviation from a reference price, notional cap, quantity versus the client's typical
  size.
- **Restricted and watch lists:** instruments a client or the firm may not trade (for example during an M&A mandate).
- **Short-sale controls:** locate requirement and price tests where the market imposes them.
- **Market access controls:** in the US, SEC Rule 15c3-5 requires brokers with market access to have pre-trade
  financial and regulatory risk controls. Other markets have similar rules.
- **Self-trade and wash-trade prevention:** an account must not trade with itself.
- A failed or unavailable check **rejects or holds** the order (fail closed, `BFSI-TRADE-01`).

### 2.3 Execution

- **Routing:** smart order routing chooses venues (exchanges, MTFs, ATSs, dark pools, systematic internalisers)
  identified by MIC codes.
- **Execution report** (FIX `35=8`) carries `ExecType` (150): New, Trade, Cancelled, Replaced, Rejected, Expired,
  Trade Correct, Trade Cancel, plus fill quantity (32 LastQty), fill price (31 LastPx), cumulative quantity (14),
  leaves (151) and average price (6).
- **Partial fills** accumulate; the order is Filled when cumulative quantity equals order quantity.
- **Busts and corrections:** a venue can cancel (bust) or correct a trade after the fact. Model them as new events
  linked to the original execution.
- **Best execution:** firms must take sufficient steps to get the best result for clients (price, cost, speed,
  likelihood of execution). Examples include MiFID II in the EU and FINRA Rule 5310 in the US. Execution data must
  be kept to evidence it.

### 2.4 Allocation

Institutional orders are often executed as one **block** and then **allocated** to underlying accounts or funds.

- **Allocation instruction:** the investment manager tells the broker how to split the block (FIX
  `AllocationInstruction`, `35=J`; or through a post-trade platform).
- **Methods:** fixed quantity per account, percentage, pro-rata by order size, or pro-rata by assets under management.
- **Average price:** each account usually receives the block's average price so all accounts are treated fairly.
  Average price precision is stated, for example 6 decimals.
- **Rounding and residuals:** quantities are rounded to lot size. The residual goes to an account chosen by a
  documented rule, for example the largest allocation or a designated house account. Allocations must sum exactly to
  the executed quantity (`BFSI-TRADE-03`).
- **Per-allocation economics:** gross amount = quantity × price; commission (rate × quantity or notional), fees,
  taxes and stamp duty per account; net amount; settlement date; standing settlement instructions (SSIs).
- **Partial executions:** if the block is only partly filled, allocations scale with a stated rule (typically pro-rata).
- **Step-outs and give-ups:** part of a trade is given to another broker for clearing or credit.
- **Fair allocation policy:** regulators expect allocations to be decided before or promptly after execution and not
  to favour particular accounts. Late or changed allocations need an audit trail.

### 2.5 Confirmation and affirmation

- **Confirmation:** the broker sends trade details per allocation (FIX `Confirmation`, `35=AK`; or a matching
  platform).
- **Affirmation / matching:** the client or its custodian agrees the details. Mismatches are exceptions to resolve
  before settlement.
- **US T+1:** under SEC Rule 15c6-2, broker-dealers must complete allocations, confirmations and affirmations as soon
  as technologically practicable and no later than the end of trade date, or have procedures to meet that standard.

### 2.6 Clearing and settlement

- **Clearing:** trades are often novated to a central counterparty (CCP), which nets obligations and manages margin.
- **Settlement:** delivery versus payment (DvP) at a central securities depository (CSD), for example DTC in the US.
- **Settlement cycles:** T+1 in the US, Canada and Mexico since May 2024, and in India since 2023. The EU, UK and
  Switzerland plan to move to T+1 on 11 October 2027. Other markets are T+2 or vary. Settlement date is always
  computed with the market calendar (`BFSI-TRADE-06`).
- **SSIs:** standing settlement instructions per account, market and currency (custodian, account, place of
  settlement, BIC).
- **Fails:** a trade that does not settle on the intended date. Some regimes, such as CSDR in the EU, apply cash
  penalties and buy-in rules. Track fail age and reason.

### 2.7 Positions, P&L and custody

- Trade-date position (economic exposure) versus settlement-date position (what is actually held).
- Realised P&L on closing trades; unrealised P&L marked to market.
- Cost basis methods such as FIFO, LIFO, average cost or specific lot. Which one is allowed depends on jurisdiction
  and account type.
- Cash projections per currency include pending settlements, fees and corporate-action payments.

### 2.8 Corporate actions

- **Mandatory:** cash dividend, stock split, reverse split, merger, spin-off, name or identifier change.
  **Voluntary:** tender offer, rights issue, dividend reinvestment election.
- **Key dates:** announcement, ex-date (trading without the entitlement), record date (who is entitled), payment date.
- **Effects:** adjust positions and cost basis; adjust or cancel open orders (for example GTC limit prices on a
  split); generate cash or stock entitlements; reconcile with the custodian.

### 2.9 Reference data

- **Instruments:** ISIN, CUSIP, SEDOL, FIGI, exchange ticker plus MIC; asset class; currency; lot size; tick size;
  trading calendar; status (active, halted, delisted).
- **Parties:** LEI for legal entities; BIC for financial institutions; internal client, account and sub-account ids.
- **Market data:** last price, bid/ask, close, reference price for fat-finger checks, halts and auction states.
- Reference data changes follow maker-checker and are versioned with effective dates.

### 2.10 FIX protocol — common messages

| MsgType (35) | Message |
|---|---|
| D | New Order Single |
| F | Order Cancel Request |
| G | Order Cancel/Replace Request |
| 8 | Execution Report |
| 9 | Order Cancel Reject |
| J | Allocation Instruction |
| P | Allocation Instruction Ack |
| AS | Allocation Report |
| AK | Confirmation |
| 3 / j | Session Reject / Business Message Reject |

Common tags: 11 ClOrdID, 37 OrderID, 17 ExecID, 54 Side, 38 OrderQty, 40 OrdType, 44 Price, 59 TimeInForce,
55 Symbol, 48/22 SecurityID/IDSource, 1 Account, 60 TransactTime.

### 2.11 Trade and transaction reporting (examples)

US: CAT (Consolidated Audit Trail) for equities and options orders, and TRACE for many fixed-income trades.
EU and UK: MiFIR transaction reporting, and EMIR for derivatives. Requirements are jurisdiction-specific and change.
The rulebook lists which ones apply.

---

## 3. Retail banking and ledgers

- **Accounts:** current, savings, term deposit, loan, internal (suspense, GL). Account status: active, dormant,
  frozen, closed. Joint holders and mandates (who may operate).
- **Double-entry ledger:** each transaction posts balanced debits and credits. The general ledger aggregates
  sub-ledgers.
- **Balances:** ledger (book) balance, available balance (= ledger − holds + overdraft limit, as defined), cleared
  and uncleared funds.
- **Dates:** booking date (when posted) versus value date (when interest starts or stops counting).
- **Interest accrual:** daily accrual using the product's day count; capitalisation monthly or quarterly; tiered
  rates; withholding tax where applicable.
- **End of day:** cut-over of the business date, accruals, fee charging, statement generation. The batch must be
  restartable without double-posting.
- **Statements and notices:** generated from postings; immutable once issued.
- **Customer due diligence** gates account opening (section 9).

---

## 4. Payments

- **Lifecycle:** initiation → validation (format, account, limits) → fraud and sanctions screening → funds
  reservation → routing to scheme → clearing → settlement → status report → reconciliation. Exceptions: rejects
  before settlement, returns after settlement, recalls and investigations.
- **ISO 20022 messages:** `pain.001` (customer credit transfer initiation), `pacs.008` (FI-to-FI customer credit
  transfer), `pacs.009` (FI credit transfer), `pacs.002` (payment status report), `pacs.004` (return),
  `camt.053` (statement), `camt.054` (debit/credit notification), `camt.056` (cancellation request).
- **SWIFT cross-border:** legacy MT103 and MT202 are replaced by ISO 20022 (CBPR+). The coexistence period for
  cross-border payment instructions ended in November 2025.
- **Domestic schemes (examples):** US ACH (NACHA), Fedwire, FedNow, RTP; EU SEPA Credit Transfer and SCT Inst;
  UK Faster Payments and CHAPS; India NEFT, RTGS, IMPS and UPI.
- **Cut-offs and value dating** are per scheme, in the scheme's time zone and calendar.
- **Duplicate prevention:** an end-to-end id and an idempotency key per instruction (`BFSI-ID-01`).
- **Reason codes:** reject and return codes map to internal states; unknown codes go to manual review.
- **Nostro / vostro:** correspondent accounts reconciled daily against statements.

---

## 5. Lending

- **Lifecycle:** application → KYC and credit assessment → decision → offer and acceptance → disbursement →
  servicing (instalments, prepayment, restructuring) → collections → closure or write-off.
- **Instalment (EMI) for an amortising loan:** `P × r × (1+r)^n / ((1+r)^n − 1)` with periodic rate `r` and `n`
  periods. Rounding goes to the last instalment so principal reconciles (`BFSI-LEND-02`).
- **Day count conventions:** ACT/360, ACT/365 Fixed, 30/360, ACT/ACT. The product defines one.
- **Delinquency:** days past due (DPD) buckets such as 1–30, 31–60, 61–90, 90+, computed from due dates and
  payments.
- **Provisioning:** expected credit loss under IFRS 9 (stages 1, 2 and 3) or CECL in the US, as the rulebook
  states. Engineering stores the inputs and the stage history; finance owns the models.
- **Collateral and limits:** collateral value, haircut, loan-to-value, facility limits and utilisation.

---

## 6. Cards

- **Parties:** cardholder, issuer, acquirer, merchant, network (scheme).
- **Flow:** authorisation (hold) → clearing → settlement; chargebacks and representments; refunds.
- **PCI DSS** (version 4.0 is current): never store sensitive authentication data (CVV/CVC, PIN, full track) after
  authorisation. Mask PAN to at most the first 6 and last 4 digits. Prefer tokenisation, and keep systems that touch
  card data in a minimal, segregated scope (`BFSI-SEC-03`).

---

## 7. Wealth management

- **Suitability and appropriateness:** recommendations match the client's risk profile, objectives and knowledge.
  The profile has a date and expires.
- **Portfolios:** model portfolios, target weights, drift thresholds, rebalancing orders (which then follow the
  capital-markets lifecycle), cash sweeps, fees (AUM-based, performance).
- **Performance:** time-weighted versus money-weighted returns; benchmark comparison.

---

## 8. Insurance

- **Policy lifecycle:** quote → underwriting → bind → issue → endorsements → renewal → cancellation or lapse.
- **Premium:** calculation, instalments, earned versus unearned premium.
- **Claims:** first notice of loss → assessment → reserve → settlement → recovery and subrogation.
- **Reinsurance:** ceded share, treaties, bordereaux.

---

## 9. Financial crime and compliance

- **KYC / CDD:** identity verification, beneficial ownership, risk rating; enhanced due diligence (EDD) for high-risk
  clients and politically exposed persons (PEPs); periodic review; KYC expiry blocks activity.
- **Sanctions screening:** names, parties and payments are screened against lists such as OFAC SDN, UN, EU and UK
  HM Treasury. Potential matches are held for review (`BFSI-PAY-01`).
- **AML transaction monitoring:** rules and models flag unusual patterns (structuring, rapid movement, high-risk
  jurisdictions). Alerts go to investigation and possibly a suspicious activity or transaction report (SAR/STR).
  Engineering never tips off the customer.
- **Market abuse surveillance:** spoofing, layering, wash trades, insider dealing and front-running. Order and trade
  data must be complete and time-stamped accurately enough to support surveillance.
- **Record keeping:** examples include SEC Rule 17a-4 and FINRA Rule 4511 in the US, and MiFID II in the EU. They
  require records to be retained for set periods, often several years, in a tamper-evident form.

---

## 10. Data protection, security and resilience

- **Data classification:** public, internal, confidential, restricted. Restricted typically covers government ids,
  card data, credentials, health data and full account numbers.
- **Privacy laws (examples):** GLBA in the US, GDPR in the EU and UK, and the DPDP Act 2023 in India. These cover
  purpose limitation, minimisation, retention, data subject rights and breach notification.
- **Security baseline:** least privilege, MFA for privileged access, secrets in a vault, encryption in transit and at
  rest, key rotation, dependency and container scanning, secure SDLC evidence.
- **Operational resilience:** examples include DORA in the EU (applies from January 2025) and national regulators'
  outsourcing and cloud guidance. They expect ICT risk management, incident classification and reporting, resilience
  testing and third-party risk management. Engineering contributes RTO/RPO targets, runbooks, kill switches and
  observability.

---

## 11. Glossary

| Term | Meaning |
|---|---|
| Block trade | One execution later split across several accounts |
| Allocation | Assignment of part of a block to an account, with its own economics and settlement |
| Average price | Quantity-weighted mean of fill prices |
| Leaves quantity | Quantity still open on a live order |
| Notional | Quantity × price (× multiplier for derivatives) |
| Trade date / T | Business day the trade was executed |
| Settlement date | Business day securities and cash change hands (T+n on the market calendar) |
| Value date | Date from which a cash movement counts for interest or availability |
| DvP | Delivery versus payment: securities move only if cash moves |
| CCP / CSD | Central counterparty / central securities depository |
| SSI | Standing settlement instructions |
| Nostro / vostro | Our account at another bank / another bank's account with us |
| Break | Difference found by a reconciliation |
| Maker-checker | Four-eyes control: one person makes a change, another approves it |
| Fail closed | On a control failure, block the action rather than allow it |
| Idempotency key | Client-supplied key that makes a retried request safe |
| DPD | Days past due |
| ECL | Expected credit loss |
| PEP | Politically exposed person |
| SAR / STR | Suspicious activity / transaction report |
| MIC / LEI / BIC | Market id / legal entity id / bank identifier code |
