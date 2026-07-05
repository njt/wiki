---
url: https://docs.cashpoints.co.nz/partners-api-v1/
title: Cashpoints Partners API v1
author: Cashpoints (NZ) Limited
date_fetched: 2026-05-18
date_published: 2026-01-20
---

# Cashpoints Partners API v1

Documentation site for the Cashpoints Partners API, covering 9 endpoints across Card, Transactional, and Promotional APIs. Built with Material for MkDocs. Copyright 2026 Cashpoints (NZ) Limited.

## Authentication

- Production: `https://service.cashpoints.co.nz`
- UAT: `https://servicetest.cashpoints.co.nz`
- All requests require `CP-API-Key` header
- Content-Type: `application/json` or `application/x-www-form-urlencoded` (defaults to form-urlencoded if absent)
- Response formats: JSON (default) or XML via `Accept: application/xml`
- All responses return HTTP 200 regardless of business-level success/failure; the `code` field differentiates

## POS Interaction Flow

Best-case flow: Get card balance -> Simulate transaction -> Simulate and Lock -> Create transaction -> Unlock card balance (if payment fails).

## Definitions

- **Points**: Pseudo-currency offered by Cashpoints loyalty scheme
- **Partner**: A partner who offers Points
- **POS**: Partner's point-of-sale system
- **Cardholder**: Customer who has a Cashpoints card

## Error Codes

19 codes total. Key codes: 0=Success, 1=No result, 11=Invalid card number, 26=Invalid card number and/or pin, 28=Unauthorised access, 42=Inactive card, 59=Invalid Session/Session Expired, 71=Card not registered, 75=Card deactivated, 93=Invalid token, 96=Card balance locked, 97=Invalid card balance lock.

## Card API Endpoints

### POST /card/balance
Parameters: partner, terminalID, cardNumber
Response: code, message, registered (Y/N), active (Y/N), balance (int points), monetaryBalance (int cents)
Note: DO NOT use balance/monetaryBalance for sale amount deduction

### POST /card/balanceWithPin
Same as /card/balance but cardNumber includes PIN concatenated with `=` (e.g., `2777640012341234=1234`)
Security: 2+ incorrect PINs locks card for 15 minutes

### POST /card/exchangeTokenForDetails
Parameters: partner, terminalID, exchangeToken
Response: cardNumber, pin, registered, active, balance, monetaryBalance
Only available to some partners

### POST /card/unlockBalance
Parameters: partner, terminalID, cardBalanceLockID (from Simulate and Lock)
Used when payment fails after locking

## Transactional API Endpoints

### POST /transaction/simulate
Dry run - returns expected points without modifying card balance
Same request body as Create, but partnerTransactionRef can be reused across simulations and cardBalanceLockID can be empty

### POST /transaction/simulateAndLock
Locks balance AND simulates. Returns cardBalanceLockID (integer). Lock expires in 10 minutes.
Same request fields as Create transaction + returns cardBalanceLockID

### POST /transaction/create
Full parameters: partner, terminalID, partnerTransactionRef (alphanumeric+underscore+dash), transactionType (points_accumulation|points_redemption), siteDate (YYYYMMDD), siteTime (HHmmss), cardNumber (with =PIN for redemption), saleAmount (cents), pointsToRedeem (optional, 1-balance), cardBalanceLockID (optional), products array or flat item*XX fields

Products can be submitted as a `products` array (v1.3+) or flat `itemCodeXX`, `itemAmountXX`, `itemUnitXX`, `itemQuantityXX`, `itemUnitAmountXX`, `itemUnqualifiedXX` fields.

Response includes accumulationResult, redemptionResult, discountPromotionsResult with various promotion types (totalPercent, totalValue, itemPercent, itemValue).

monetaryTotalDiscount is the cents to deduct from the sale amount (includes redemptions + promotions). balance and monetaryBalance are explicitly NOT for sale amount deduction.

### POST /transaction/refund
Status: Draft
Parameters: partner, partnerTransactionRef, fullRefund (Y/N), refundProducts[] (if partial)
Cannot refund if: points already refunded, points expired, points redeemed, or discount promotions applied.
When system can't claw back points (insufficient balance), returns non-zero pointsToRefund/monetaryToRefund - partner deducts from cash refund.

## Promotional API

### POST /retailer/publicCardPromotions
Status: Draft
Parameters: partner, terminalID, cardNumber
Response: promotions[] with promotionID, partnerTemplateID, promotionText, itemCodes[], startDate, endDate (RFC 3339)

## Changelog

- 1.7 (2026-01-20): DO NOT use balance for sale amount deduction clarified; pointsToRedeem added; itemUnqualified added; lock expiry explanation updated
- 1.6 (2025-08-22): Card/PIN separator changed from # to =
- 1.5 (2025-07-31): cardBalanceLockID type changed from string to integer
- 1.4 (2025-07-30): Example JSON responses updated
- 1.3 (2025-06-24): products array added alongside flat item*XX fields
- 1.2 (2025-02-14): partnerTransactionRef restricted to alphanumeric+underscore+dash
- 1.1 (2024-12-10): PIN requirement introduced for redemption
- 1.0 (2024-07-23): V1 production ready
- 0.2 (2024-06-12): Draft ready
- 0.1 (2024-05-15): Initial draft
