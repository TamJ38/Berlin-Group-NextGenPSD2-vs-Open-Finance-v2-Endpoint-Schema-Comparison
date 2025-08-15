# Consent & Authorisation endpoints changes

In **V1 (NextGenPSD2)**, consents and their authorisation sub-resources lived inside each service domain (AIS/PIS/PIIS). <br> 
For example, AIS exposed `/v1/consents` and `/v1/consents/{consentId}/authorisations`.
In **V2 (openFinance)**, Berlin Group globalised consent management into a dedicated Consent API, with separate resources for account access, funds confirmation and user parameters—exposed as `/v2/consents/*`. This decouples consent lifecycle from AIS/PIS/PIIS and unifies request/response models, headers and error patterns.

### Endpoints replaced by the same globalised Consent or Authorisation method in V2

**1. POST Authorisations (start SCA)**  
_Replaces:_
- AIS:
  - `POST /v1/consents/{consent-id}/authorisations`  
- PIS:
  - `POST /v1/{payment-service}/{payment-product}/{paymentId}/authorisations`
  - `POST /v1/{payment-service}/{payment-product}/{payment-id}/cancellation-authorisations`
- PIIS:
  - `POST /v1/consents/confirmation-of-funds/{CoF-consent-id}/authorisations`  

_All replaced with:_  
`POST /v2/{resource-path}/{resourceId}/{authorisation-category}`  

---

**2. GET Authorisations (list SCA authorisations)**  
_Replaces:_
- AIS:
  - `GET /v1/consents/{consent-id}/authorisations`  
- PIS:
  - `GET /v1/{payment-service}/{payment-product}/{payment-id}/authorisations`  
  - `GET /v1/{payment-service}/{payment-product}/{payment-id}/cancellation-authorisations`
_All replaced with:_  
`GET /v2/{resource-path}/{resourceId}/{authorisation-category}`  

---

**3. PUT Authorisation by ID (update SCA status)**  
_Replaces:_
- AIS:
  - `PUT /v1/consents/{consent-id}/authorisations/{authorisation-id}`  
- PIS:
  - `PUT /v1/payment-service/{payment-product}/{payment-id}/authorisations/{authorisation-id}`
  - `PUT /v1/{payment-service}/{payment-product}/{paymentId}/cancellation-authorisations/{authorisationId}
- PIIS:
  - `PUT /v1/consents/confirmation-of-funds/{CoF-consent-id}/authorisations/{authorisation-id}`  

_All replaced with:_  
`PUT /v2/{resource-path}/{resourceId}/{authorisation-category}/{authorisationId}`  

---

**4. GET Authorisation by ID (read SCA status)**  
_Replaces:_
- AIS:
  - `GET /v1/consents/{consent-id}/authorisations/{authorisation-id}`  
- PIS: 
  - `GET /v1/payment-service/{payment-product}/{payment-id}/authorisations/{authorisation-id}`
  - `GET /v1/{payment-service}/{payment-product}/{paymentId}/cancellation-authorisations/{authorisationId}`
- PIIS: `GET /v1/consents/confirmation-of-funds/{CoF-consent-id}/authorisations/{authorisation-id}`  

_All replaced with:_  
`GET /v2/{resource-path}/{resourceId}/{authorisation-category}/{authorisationId}`  

---

**5. DELETE Consent**  
_Replaces:_
- AIS: `DELETE /v1/consents/{consent-id}`  
- PIIS: `DELETE /v1/consents/confirmation-of-funds/{CoF-consent-id}`  

_All replaced with:_  
`DELETE /v2/consents/{consent-category}/{consentId}`  

---

**6. GET Consent Status**  
_Replaces:_
- AIS: `GET /v1/consents/{consent-id}/status`  
- PIIS: `GET /v1/consents/confirmation-of-funds/{CoF-consent-id}/status`  

_All replaced with:_  
`GET /v2/consents/{consent-category}/{consentId}/status`  

---

**7. Create Consent**  
_Replaces:_
- AIS: `POST /v1/consents`  
- PIIS: `POST /v1/consents/confirmation-of-funds`  

_All replaced with:_  
- AIS → `POST /v2/consents/account-access`  
- PIIS → `POST /v2/consents/funds-confirmations`

---

# Detailed comparison

## Create AIS Consent — differences V1 → V2

**Endpoints**
- **V1:** `POST /v1/consents`
- **V2:** `POST /v2/consents/account-access`

### 📌 Headers - diff
```diff
- Signature
+ x-jws-signature
  X-Request-ID
+ Client-Redirect-URI
+ Client-Nok-Redirect-URI
- TPP-Redirect-URI
- TPP-Nok-Redirect-URI

- TPP-Explicit-Authorisation-Preferred
+ Client-Explicit-Authorisation-Preferred

- TPP-Notification-URI
- TPP-Notification-Content-Preferred
- TPP-Brand-Logging-Information
+ Client-Notification-URI
+ Client-Notification-Content-Preferred
+ Client-Brand-Logging-Information

- TPP-Signature-Certificate
  Digest        (retained, clarified to RFC 3230/5843 usage)

+ Client-SCA-Approach-Preference   (replaces V1 redirect/decoupled prefs)
- TPP-Redirect-Preferred
- TPP-Decoupled-Preferred

+ Body-Sig-Profile
+ Body-Enc-Profile
+ Body-Enc-List
```

---

### 📌 Request example of v2 vs v1

<details>
<summary>Example Request – Create Consent (V2)</summary>

```json
POST /v2/consents/account-access
{
  "access": {
    "payments": [
      {
        "account": {
          "iban": "FR7612345987650123456789014",
          "bban": "BARC12345612345678",
          "pan": "5409050000000000",
          "maskedPan": "123456xxxxxx1234",
          "msisdn": "+49 170 1234567",
          "other": {
            "identification": "ACC-1234567890",
            "schemeNameCode": "AIIN",
            "schemeNameProprietary": "InternalScheme",
            "issuer": "ExampleIssuer"
          },
          "typeCode": "CACC",
          "typeProprietary": "InternalType",
          "currency": "EUR",
          "proxy": {
            "typeCode": "EMAL",
            "typeProprietary": "AltProxyType",
            "identification": "customer@example.com"
          },
          "name": "Primary Current Account",
          "owner": {
            "name": "John Q. Customer",
            "postaladdress": {
              "addressLines": [
                "Main Street 1"
              ],
              "department": "Finance",
              "subDepartment": "Retail",
              "streetName": "Main Street",
              "buildingNumber": "12A",
              "buildingName": "River House",
              "floor": "3",
              "postBox": "PO 456",
              "room": "305",
              "postCode": "10001",
              "townName": "Berlin",
              "townLocationName": "Mitte",
              "districtName": "Central",
              "countrySubDivision": "BE",
              "country": "SE"
            }
          },
          "servicer": {
            "bicfi": "ECBFDEFFFIM",
            "clearingSystemMemberId": {
              "memberId": "1234567",
              "clearingSystemIdentificationCode": "DEBLZ",
              "clearingSystemIdentificationProprietary": "LocalClearNet"
            },
            "name": "Example Bank AG",
            "postalAddress": {
              "addressLines": [
                "Bankplatz 2"
              ],
              "department": "Operations",
              "subDepartment": "Payments",
              "streetName": "Bankplatz",
              "buildingNumber": "2",
              "buildingName": "HQ Tower",
              "floor": "10",
              "postBox": "PO 999",
              "room": "1001",
              "postCode": "10115",
              "townName": "Berlin",
              "townLocationName": "Mitte",
              "districtName": "Central",
              "countrySubDivision": "BE",
              "country": "SE"
            },
            "other": {
              "identification": "BANK-98765",
              "schemeNameCode": "CUID",
              "schemeNameProprietary": "InternalBankId",
              "issuer": "National Registry"
            }
          }
        },
        "rights": [
          "ais",
          "accountDetails",
          "balances",
          "transactions",
          "ownerName",
          "trustedBeneficiaries",
          "ibanChecks"
        ]
      }
    ],
    "cards": [
      {
        "account": {
          "iban": "FR7612345987650123456789014",
          "bban": "BARC12345612345678",
          "pan": "5409050000000000",
          "maskedPan": "123456xxxxxx1234",
          "msisdn": "+49 170 1234567",
          "other": {
            "identification": "CARD-ACC-001",
            "schemeNameCode": "AIIN",
            "schemeNameProprietary": "InternalScheme",
            "issuer": "CardIssuer Ltd"
          },
          "typeCode": "CARD",
          "typeProprietary": "CardRecon",
          "currency": "EUR",
          "proxy": {
            "typeCode": "MBNO",
            "typeProprietary": "AltProxyType",
            "identification": "+491701234567"
          },
          "name": "Card Account",
          "owner": {
            "name": "John Q. Customer",
            "postaladdress": {
              "addressLines": ["Main Street 1"],
              "department": "Finance",
              "subDepartment": "Retail",
              "streetName": "Main Street",
              "buildingNumber": "12A",
              "buildingName": "River House",
              "floor": "3",
              "postBox": "PO 456",
              "room": "305",
              "postCode": "10001",
              "townName": "Berlin",
              "townLocationName": "Mitte",
              "districtName": "Central",
              "countrySubDivision": "BE",
              "country": "SE"
            }
          },
          "servicer": {
            "bicfi": "ECBFDEFFFIM",
            "clearingSystemMemberId": {
              "memberId": "1234567",
              "clearingSystemIdentificationCode": "DEBLZ",
              "clearingSystemIdentificationProprietary": "LocalClearNet"
            },
            "name": "Example Bank AG",
            "postalAddress": {
              "addressLines": ["Bankplatz 2"],
              "department": "Operations",
              "subDepartment": "Payments",
              "streetName": "Bankplatz",
              "buildingNumber": "2",
              "buildingName": "HQ Tower",
              "floor": "10",
              "postBox": "PO 999",
              "room": "1001",
              "postCode": "10115",
              "townName": "Berlin",
              "townLocationName": "Mitte",
              "districtName": "Central",
              "countrySubDivision": "BE",
              "country": "SE"
            },
            "other": {
              "identification": "BANK-98765",
              "schemeNameCode": "CUID",
              "schemeNameProprietary": "InternalBankId",
              "issuer": "National Registry"
            }
          }
        },
        "rights": ["accountDetails", "balances", "transactions"]
      }
    ],
    "cardAccounts": [
      {
        "account": {
          "iban": "FR7612345987650123456789014",
          "bban": "BARC12345612345678",
          "pan": "5409050000000000",
          "maskedPan": "123456xxxxxx1234",
          "msisdn": "+49 170 1234567",
          "other": {
            "identification": "CARD-RECON-ACC",
            "schemeNameCode": "AIIN",
            "schemeNameProprietary": "InternalScheme",
            "issuer": "CardIssuer Ltd"
          },
          "typeCode": "CASH",
          "typeProprietary": "Recon",
          "currency": "EUR",
          "proxy": {
            "typeCode": "EMAL",
            "typeProprietary": "AltProxyType",
            "identification": "recon@example.com"
          },
          "name": "Card Reconciliation Account",
          "owner": { "name": "Merchant Ltd", "postaladdress": { "addressLines": ["Market Street 9"], "country": "SE", "townName": "Stockholm", "postCode": "11122" } },
          "servicer": { "bicfi": "ECBFDEFFFIM", "name": "Example Bank AG" }
        },
        "rights": ["accountDetails", "balances"]
      }
    ],
    "savings": [
      {
        "account": {
          "iban": "FR7612345987650123456789014",
          "currency": "EUR",
          "name": "Savings Account",
          "owner": { "name": "John Q. Customer", "postaladdress": { "addressLines": ["Main Street 1"], "country": "SE", "townName": "Berlin", "postCode": "10001" } }
        },
        "rights": ["accountDetails", "balances", "ownerName"]
      }
    ],
    "loans": [
      {
        "account": {
          "iban": "FR7612345987650123456789014",
          "currency": "EUR",
          "name": "Loan Account",
          "owner": { "name": "John Q. Customer", "postaladdress": { "addressLines": ["Main Street 1"], "country": "SE", "townName": "Berlin", "postCode": "10001" } }
        },
        "rights": ["accountDetails"]
      }
    ],
    "securities": [
      {
        "account": {
          "iban": "FR7612345987650123456789014",
          "currency": "EUR",
          "name": "Securities Account",
          "owner": { "name": "John Q. Customer", "postaladdress": { "addressLines": ["Main Street 1"], "country": "SE", "townName": "Berlin", "postCode": "10001" } }
        },
        "rights": ["accountDetails", "orders"]
      }
    ]
  },
  "consentType": "detailed",
  "recurringIndicator": true,
  "validTo": "2025-12-31",
  "frequencyPerDay": 4
}
```
</details>

<details>
<summary>Example Request – Create Consent (V1)</summary>

```json
POST /v1/consents/account-access
{
  "access": {
    "accounts": [
      {
        "iban": "FR7612345987650123456789014",
        "bban": "BARC12345612345678",
        "pan": "5409050000000000",
        "maskedPan": "123456xxxxxx1234",
        "msisdn": "+49 170 1234567",
        "other": {
          "identification": "ACC-1234567890",
          "schemeNameCode": "AIIN",
          "schemeNameProprietary": "InternalScheme",
          "issuer": "ExampleIssuer"
        },
        "currency": "EUR",
        "cashAccountType": "CACC"
      }
    ],
    "balances": [
      {
        "iban": "FR7612345987650123456789014",
        "bban": "BARC12345612345678",
        "pan": "5409050000000000",
        "maskedPan": "123456xxxxxx1234",
        "msisdn": "+49 170 1234567",
        "other": {
          "identification": "ACC-1234567890",
          "schemeNameCode": "AIIN",
          "schemeNameProprietary": "InternalScheme",
          "issuer": "ExampleIssuer"
        },
        "currency": "EUR",
        "cashAccountType": "CACC"
      }
    ],
    "transactions": [
      {
        "iban": "FR7612345987650123456789014",
        "bban": "BARC12345612345678",
        "pan": "5409050000000000",
        "maskedPan": "123456xxxxxx1234",
        "msisdn": "+49 170 1234567",
        "other": {
          "identification": "ACC-1234567890",
          "schemeNameCode": "AIIN",
          "schemeNameProprietary": "InternalScheme",
          "issuer": "ExampleIssuer"
        },
        "currency": "EUR",
        "cashAccountType": "CACC"
      }
    ],
    "additionalInformation": {
      "ownerName": [
        {
          "iban": "FR7612345987650123456789014",
          "bban": "BARC12345612345678",
          "pan": "5409050000000000",
          "maskedPan": "123456xxxxxx1234",
          "msisdn": "+49 170 1234567",
          "other": {
            "identification": "ACC-1234567890",
            "schemeNameCode": "AIIN",
            "schemeNameProprietary": "InternalScheme",
            "issuer": "ExampleIssuer"
          },
          "currency": "EUR",
          "cashAccountType": "CACC"
        }
      ],
      "trustedBeneficiaries": [
        {
          "iban": "FR7612345987650123456789014",
          "bban": "BARC12345612345678",
          "pan": "5409050000000000",
          "maskedPan": "123456xxxxxx1234",
          "msisdn": "+49 170 1234567",
          "other": {
            "identification": "ACC-1234567890",
            "schemeNameCode": "AIIN",
            "schemeNameProprietary": "InternalScheme",
            "issuer": "ExampleIssuer"
          },
          "currency": "EUR",
          "cashAccountType": "CACC"
        }
      ]
    },
    "availableAccounts": "allAccountsWithOwnerName",
    "availableAccountsWithBalance": "allAccountsWithOwnerName",
    "allPsd2": "allAccountsWithOwnerName",
    "restrictedTo": [
      { "cashAccountType": "CACC" }
    ]
  },
  "recurringIndicator": true,
  "validUntil": "2025-12-31",
  "frequencyPerDay": 4,
  "combinedServiceIndicator": false
}
```
</details>

**1) High-level model shift in the request schema**

- **V1** centres on `consents.access` with three **lists of intentions per account**:
  - `accounts[]`, `balances[]`, `transactions[]` (+ optional `additionalInformation{ ownerName[], trustedBeneficiaries[] }`)
  - Global flags for “all accounts” (`availableAccounts`, `availableAccountsWithBalance`, `allPsd2`) and `restrictedTo[]`.
- **V2** centres on **`access`** with **domain buckets** and **rights**:
  - Buckets: `payments[]`, `cards[]`, `cardAccounts[]`, `savings[]`, `loans[]`, `securities[]`
  - Each entry is an `accountAccessRights` object: `{ account: {...}, rights: [AccessRightsCodes] }`
  - Adds `consentType` (enum), keeps `recurringIndicator`, renames `validUntil` → `validTo`, keeps `frequencyPerDay`.

---

**2) Top-level consent attributes in the request schema**

| V1 field                | V2 field        | Status     | Notes |
|-------------------------|-----------------|------------|-------|
| `recurringIndicator`    | `recurringIndicator` | **Unchanged** | Same meaning. |
| `validUntil`            | `validTo`       | **Renamed** | Same semantics; ASPSP may adjust date; `9999-12-31` convention remains. |
| `frequencyPerDay`       | `frequencyPerDay` | **Unchanged** | Same constraints (≥1, typically ≤4). |
| *(none)*                | `consentType`   | **Added**   | `global | detailed | aspspManaged | accountList` — influences whether `account` becomes mandatory. |
| `combinedServiceIndicator` | *(none)*     | **Removed** | Present in V1; not in V2 response schema excerpt. |

---

**3) Access model(in the request schema): account-lists (V1) → `rights` per `account` (V2)**

| Concept | V1 (inside `access`) | V2 (inside `access`) | Status / Mapping |
|---|---|---|---|
| Ask for **account details** | `accounts[]` | `rights` includes `accountDetails` | **Mapped** |
| Ask for **balances** | `balances[]` | `rights` includes `balances` | **Mapped** |
| Ask for **transactions** | `transactions[]` | `rights` includes `transactions` | **Mapped** |
| Ask for **owner name** | `additionalInformation.ownerName[]` | `rights` includes `ownerName` | **Mapped (moved to rights)** |
| Ask for **trusted beneficiaries** | `additionalInformation.trustedBeneficiaries[]` | `rights` includes `trustedBeneficiaries` | **Mapped (moved to rights)** |
| Ask for **all PSD2 accounts** | `allPsd2` (enum values) | `consentType = global` (or `accountList` + selection) | **Modelled via `consentType`** |
| Ask for **all accounts (with/without owner)** | `availableAccounts`, `availableAccountsWithBalance` | Use `consentType` + `rights` (e.g., include `ownerName`) | **Consolidated into type + rights** |
| Restrict by **account type** | `restrictedTo[]` of `cashAccountType` | Choose the **bucket** (`payments`, `cards`, `cardAccounts`, `savings`, `loans`, `securities`) | **Shifted to buckets** |

> **Reading tip:** In V1 they “tick boxes” by placing account refs into `accounts[]/balances[]/transactions[]`.  
> In V2 they **declare rights** (`rights[]`) for each **accountAccessRights** entry in the relevant **bucket**.

---

**4) Account reference in the request schema: structure evolution**

| Area | V1 `accountReference` | V2 `account` (inside `accountAccessRights`) | Status / Additions |
|---|---|---|---|
| Identifiers | `iban`, `bban`, `pan`, `maskedPan`, `msisdn`, `other{ identification, schemeNameCode, schemeNameProprietary, issuer }` | Same set available | **Retained** |
| Currency | `currency` | `currency` | **Retained** |
| Account type | `cashAccountType` | `typeCode` / `typeProprietary` *(remark: not used in consent model)* | **Superseded by buckets** |
| Proxy identifier | *(none)* | `proxy{ typeCode, typeProprietary, identification }` | **Added** |
| Party/owner data | `additionalInformation.ownerName[]` (as a consent intent) | `owner{ name, postalAddress{...} }` | **Owner structure added; right is still requested via `rights`** |
| Servicer (bank) | *(none)* | `servicer{ bicfi, clearingSystemMemberId{...}, name, postalAddress{...}, other{...} }` | **Added** |

---

**5) Rights catalogue (V2)**

V2 introduces a **unified rights list** (`rights: [AccessRightsCodes]`), e.g.:
`ais`, `accountDetails`, `balances`, `transactions`, `orders`, `ownerName`, `psuName`, `psuLeanIdentification`, `trustedBeneficiaries`, `initiatePayments`, `fundsConfirmations`, `userParameters`, `ibanChecks`, `corporateParameters`, `accountCheckParameters`.

This replaces V1’s multiple arrays/flags (`accounts[]`, `balances[]`, `transactions[]`, `additionalInformation.*`, `available*`, `allPsd2`, `restrictedTo`).

---

**6) Mini diff (conceptual)**

```diff
- access.accounts[]: accountReference per account to get details
- access.balances[]: accountReference per account to get balances
- access.transactions[]: accountReference per account to get transactions
- access.additionalInformation.ownerName[] / trustedBeneficiaries[]
- access.allPsd2 / availableAccounts / availableAccountsWithBalance
- access.restrictedTo[cashAccountType]
- combinedServiceIndicator
+ access.payments[] | cards[] | cardAccounts[] | savings[] | loans[] | securities[]:
+   - each item = accountAccessRights {
+       account: { iban | bban | pan | maskedPan | msisdn | other | currency | proxy? | owner? | servicer? }
+       rights: [AccessRightsCodes]  // e.g., accountDetails, balances, transactions, ownerName, trustedBeneficiaries
+     }
+ consentType: "global" | "detailed" | "aspspManaged" | "accountList"
+ validTo (was validUntil)
```

### Response 201 Differences – V1 vs V2 (POST /consents)

**Added in V2**
- `consentStatus` enum: Added value `replacedByTpp`.
- `consentId`:
  - Max length increased from unspecified in V1 to **70** in V2.
- `scaMethods.authenticationObject`:
  - `name` is now **mandatory** in V2 (was optional in V1).
- `chosenScaMethod`:
  - `name` is now **mandatory** in V2 (was optional in V1).
- `_links`:
  - Added `encryptionCertificates` array.

**Removed in V2**
- `consentStatus` enum: Removed value `received` description text from V1, replaced with shorter definition.
- `_links`:
  - Removed individual V1-specific description for each link (e.g., `scaRedirect`, `scaOAuth`, etc.), now only listed as possible links.
- Removed `otpFormat` enum values list ("characters", "integer") from V1, kept only text in V2.

**Changed in V2**
- `chosenScaMethod`:
  - Added explicit note that this appears only if Embedded SCA approach is chosen and method is implicitly selected.

---

 #### Response 201 Headers – V1 vs V2

| Header | V1 | V2 | Change |
|--------|----|----|--------|
| **X-Reference-API-Name** | *(not present)* | `"Berlin Group openFinance API"` *(string)* | **Added** in V2 |
| **X-Reference-API-Document** | *(not present)* | Name of Implementation Guideline document, e.g. `"Extended Payment Initiation Services"` *(string)* | **Added** in V2 |
| **X-Reference-API-Version** | *(not present)* | Version of the API *(string)* | **Added** in V2 |
| **ASPSP-SCA-Approach** | Values: `EMBEDDED`, `DECOUPLED`, `REDIRECT` *(OAuth SCA subsumed by REDIRECT)* | Values: `EMBEDDED`, `DECOUPLED`, `REDIRECT`, **`ASPSP-CHANNEL`** *(OAuth SCA subsumed by REDIRECT)* | **Value added:** `ASPSP-CHANNEL` |
| **ASPSP-Notification-Support** | Boolean, same description | Boolean, same description | **No change** |
| **ASPSP-Notification-Content** | String with constants: `SCA`, `PROCESS`, `LAST` – same description | Same constants and description | **No change** |
| **Location** | Present | Present | **No change** |
| **X-Request-ID** | Present | Present | **No change** |
| **ASPSP-Multiple-Consent-Support** | *(not present)* | Boolean – true if ASPSP supports Multiple Consent Service | **Added** in V2 |

---

### Response 400 – V1 vs V2 Differences

| Section | V1 | V2 | Change |
|---------|----|----|--------|
| **Error message container** | `tppMessages[]` (type: `tppMessage400_AIS`) | `apiClientMessages[]` (type: `clientMessageInformation_401_AIS`) | **Renamed** container and structure. |
| **Error code field** | `code` (enum: `FORMAT_ERROR`, `PARAMETER_NOT_CONSISTENT`, `PARAMETER_NOT_SUPPORTED`, `SERVICE_INVALID`, `RESOURCE_UNKNOWN`, `RESOURCE_EXPIRED`, `RESOURCE_BLOCKED`, `TIMESTAMP_INVALID`, `PERIOD_INVALID`, `SCA_METHOD_UNKNOWN`, `SCA_INVALID`, `CONSENT_UNKNOWN`, `SESSIONS_NOT_SUPPORTED`) | `code` split into two sets: **Service Unspecific** (`CERTIFICATE_INVALID`, `ROLE_INVALID`, `CERTIFICATE_EXPIRED`, `CERTIFICATE_BLOCKED`, `CERTIFICATE_REVOKED`, `CERTIFICATE_MISSING`, `CLIENT_INVALID`, `CLIENT_INCONSISTENT`, `API_CONTRACT_ID_INVALID`, `SIGNATURE_INVALID`, `SIGNATURE_MISSING`, `ROLE_INVALID`, `PSU_CREDENTIALS_INVALID`, `CORPORATE_ID_INVALID`, `CONSENT_INVALID`, `CONSENT_EXPIRED`, `TOKEN_UNKNOWN`, `TOKEN_INVALID`, `TOKEN_EXPIRED`) and **AIS Specific** codes (pattern-restricted). | **Expanded and restructured** error code sets; more certificate, client, token-related errors added. |
| **path** | `path` (string) | `path` (array / as defined in Data Dictionary) | **Type change**. |
| **text** | `text` (string, max 500 chars) | `text` (type: `Max500Text` from Data Dictionary) | **Type refactor**; semantic same. |
| **Links object** | `_links` with large set of AIS-specific link relations (e.g., `scaRedirect`, `scaOAuth`, `confirmation`, `startAuthorisation`, `updatePsuIdentification`, `balances`, `transactions`, `download`, etc.) | `_links` (type: `links`) from Data Dictionary – expanded with additional link relations such as `updateResourceByDebtorAccountResource`, `transactionfees`, `savingsAccount`, `loanAccount`, `ibanCheck`, `paymentInitiation`, `securitiesAccount`, `positions`, `orders`, `orderDetails`, `relatedOrders`, `relatedTransactions`, `subscription`, `entryStatusRevoked`, `confirmInitiation`, `aspspParameters`, `aspspContacts`, `aspspDowntimes`, `onboardings`, `readConditions`, `confirmConditions`, etc. | **Link list globalised and expanded**; many new link types introduced. |
| **Removed fields** | `creditorNameConfirmation` link in V1 | *(not present in V2)* | **Removed**. |


