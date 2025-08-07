## 📌 Endpoints

### AIS (Account Information Services)

| Method | Endpoint V1 | Endpoint V2 | Status | Path | Header | Request | Response | Query | Details |
|--------|-------------|-------------|-----------|------|--------|---------|----------|-------|----------------|
| POST   | /v1/consents | /v2/consents | ❌ removed |      |        |         |          |       | [View Details]modifications.md#accounts-get |
| GET    | /v1/consents/{consent-id} | /v2/consents/{consent-id} | ❌ removed |      |        |         |          |       | modifications.md#accounts-get |
| DELETE | /v1/consents/{consent-id} | /v2/consents/{consent-id} | ❌ removed |      |        |         |          |       | modifications.md#accounts-get |
| GET    | /v1/consents/{consent-id}/status | /v2/consents/{consent-id}/status | ❌ removed |      |        |         |          |       | modifications.md#accounts-get |
| POST   | /v1/consents/{consent-id}/authorisations | /v2/consents/{consent-id}/authorisations | ❌ removed |      |        |         |          |       | modifications.md#accounts-get |
| GET    | /v1/consents/{consent-id}/authorisations | /v2/consents/{consent-id}/authorisations | ❌ removed |      |        |         |          |       | modifications.md#accounts-get |
| PUT    | /v1/consents/{consent-id}/authorisations/{authorisation-id} | /v2/consents/{consent-id}/authorisations/{authorisation-id} | ❌ removed |      |        |         |          |      | Details |
| GET    | /v1/consents/{consent-id}/authorisations/{authorisation-id} | /v2/consents/{consent-id}/authorisations/{authorisation-id} | ❌ removed |      |        |         |          |      | Details |
| GET    | /v1/accounts | /v2/accounts | 🔶 modified |  | ✅ |  | ✅ |       | Details |
| GET    | /v1/accounts/{account-id} | /v2/accounts/{account-id} | 🔶 modified |  | ✅ |  | ✅ |       | Details |
| GET    | /v1/accounts/{account-id}/balances | /v2/accounts/{account-id}/balances | 🔶 modified |  | ✅ |  | ✅ |       | Details |
| GET    | /v1/accounts/{account-id}/transactions | /v2/accounts/{account-id}/transactions | 🔶 modified |  | ✅ |  | ✅ |  | Details |
| GET    | /v1/accounts/{account-id}/transactions/{transaction-id} | /v2/accounts/{account-id}/transactions/{transaction-id} | 🔶 modified |  | ✅ |  | ✅ |  | Details |
| GET    | /v1/accounts/{account-id}/cheques | /v2/accounts/{account-id}/cheques | ❌ removed |      |        |         |          |       | Details |
| GET    | /v1/card-accounts | /v2/card-accounts | 🔶 modified |  | ✅ |  | ✅ |       | Details |
| GET    | /v1/card-accounts/{account-id} | /v2/card-accounts/{account-id} | 🔶 modified |  | ✅ |  | ✅ |       | Details |
| GET    | /v1/card-accounts/{account-id}/balances | /v2/card-accounts/{account-id}/balances | 🔶 modified |  | ✅ |  | ✅ |     | Details |
| GET    | /v1/card-accounts/{account-id}/transactions | /v2/card-accounts/{account-id}/transactions | 🔶 modified |  | ✅ |  | ✅ | ✅ | Details |


### PIS (Payment Initiation Services)

| Method | Endpoint V1 | Endpoint V2 | Status | Path | Header | Request | Response | Query |
|--------|-------------|-------------|--------|------|--------|---------|----------|-------|
| POST   | /v1/payment-service/{payment-product} | /v2/payments/{payment-product} | 🔶 modified | ✅ | ✅ | ✅ | ✅ |       |
| POST   | /v1/payment-service/{payment-product}/authorisations | /v2/payment-service/{payment-product}/authorisations | ❌ removed |      |        |         |          |       |
| PUT    | /v1/payment-service/{payment-product}/{payment-id}/authorisations/{authorisation-id} | /v2/payment-service/{payment-product}/{payment-id}/authorisations/{authorisation-id} | ❌ removed |      |        |         |          |       |
| GET    |/v1/payment-service/{payment-product}/{payment-id}/authorisations/{authorisation-id} | /v2/payment-service/{payment-product}/{payment-id}/authorisations/{authorisation-id}  | ❌ removed |  |  |  |  |       |
| GET    | /v1/payment-service/{payment-product}/{payment-id} | /v2/payment-service/{payment-product}/{payment-id} | 🔶 modified | ✅ | ✅ |  | ✅ |       |
| PUT    | /v1/payment-service/{payment-product}/{payment-id} | /v2/payment-service/{payment-product}/{payment-id} | 🔶 modified | ✅ | ✅ | ✅ | ✅ |       |
| GET    | /v1/payment-service/{payment-product}/{payment-id}/status | /v2/payment-service/{payment-product}/{payment-id}/status | 🔶 modified | ✅ | ✅ |  | ✅ |       |
| GET    | /v1/{payment-service}/{payment-product}/{payment-id}/authorisations | /v2/{payment-service}/{payment-product}/{payment-id}/authorisations | ❌ removed |  |  |  |  |       |
| DELETE | /v1/payment-service/{payment-product}/{payment-id} | /v2/payment-service/{payment-product}/{payment-id} | 🔶 modified | ✅ | ✅ |  | ✅ |       |
| POST   | /v1/{payment-service}/{payment-product}/{payment-id}/cancellation-authorisations | /v2/{payment-service}/{payment-product}/{payment-id}/cancellation-authorisations | ❌ removed |      |        |         |          |       |
| PUT    | /v1/{payment-service}/{payment-product}/{paymentId}/cancellation-authorisations/{authorisationId}  | /v2/{payment-service}/{payment-product}/{paymentId}/cancellation-authorisations/{authorisationId}  | ❌ removed |      |        |         |          |       |
| GET    | /v1/{payment-service}/{payment-product}/{payment-id}/cancellation-authorisations | /v2/{payment-service}/{payment-product}/{payment-id}/cancellation-authorisations | ❌ removed |  |  |  |  |       |
| GET    | /v1/{payment-service}/{payment-product}/{payment-id}/cancellation-authorisations/{authorisation-id} | /v2/{payment-service}/{payment-product}/{payment-id}/cancellation-authorisations/{authorisation-id} | ❌ removed |  |  |  |  |       |
| POST   | - | /v2/bulk-payments/{payment-product} | ➕ added |     |     |     |     |       |
| GET    | /v1/bulk-payments/{payment-product}/{paymentId}/extended-status | /v2/bulk-payments/{payment-product}/{paymentId}/extended-status | 🔶 modified |  ✅   |  ✅   |     |  ✅   |       |
| POST   | - | /v2/periodic-payments/{payment-product} | ➕ added |     |     |     |     |       |

### PIIS (Payment Instrument Issuer Services)

| Method | Endpoint V1 | Endpoint V2 | Status | Path | Header | Request | Response | Query |
|--------|-------------|-------------|--------|------|--------|---------|----------|-------|
| POST   | /v1/consents/confirmation-of-funds | /v2/consents/confirmation-of-funds | ❌ removed |      |        |         |          |       |
| POST   | /v1/consents/confirmation-of-funds/{CoF-consent-id}/authorisations | /v2/consents/confirmation-of-funds/{CoF-consent-id}/authorisations | ❌ removed |      |        |         |          |       |
| PUT    | /v1/consents/confirmation-of-funds/{CoF-consent-id}/authorisations/{authorisation-id} | /v2/consents/confirmation-of-funds/{CoF-consent-id}/authorisations/{authorisation-id} | ❌ removed |      |        |         |          |       |
| POST   | /v1/funds-confirmations | /v2/funds-confirmations | 🔶 modified |      |    ✅    |   ✅   |   ✅   |       |
| GET    | /v1/consents/confirmation-of-funds//{CoF-consent-id} | /v2/consents/confirmation-of-funds//{CoF-consent-id} | ❌ removed |      |        |         |          |       |
| GET    | /v1/consents/confirmation-of-funds//{CoF-consent-id}/status | /v2/consents/confirmation-of-funds//{CoF-consent-id}/status | ❌ removed |      |        |         |          |       |
| GET    | /v1/consents/confirmation-of-funds/{CoF-consent-id}/authorisations/{authorisation-id} | /v2/consents/confirmation-of-funds/{CoF-consent-id}/authorisations/{authorisation-id} | ❌ removed |      |        |         |          |       |
| DELETE | /v1/consents/confirmation-of-funds//{CoF-consent-id} | /v2/consents/confirmation-of-funds//{CoF-consent-id} | ❌ removed |      |        |         |          |       |
