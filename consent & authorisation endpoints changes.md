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

## 4 GET Authorisation by ID (read SCA status)

### Parameters

| Aspect | V1 Endpoints (AIS / PIS / PIIS) | V2 Unified Endpoint | Change |
|--------|----------------------------------|----------------------|--------|
| **Endpoint(s)** | - `GET /v1/consents/{consent-id}/authorisations/{authorisation-id}` (AIS)<br>- `GET /v1/{payment-service}/{payment-product}/{paymentId}/authorisations/{authorisationId}` (PIS)<br>- `GET /v1/{payment-service}/{payment-product}/{paymentId}/cancellation-authorisations/{authorisationId}` (PIS, cancellations)<br>- `GET /v1/consents/confirmation-of-funds/{CoF-consent-id}/authorisations/{authorisation-id}` (PIIS) | `GET /v2/{resource-path}/{resourceId}/{authorisation-category}/{authorisationId}` | Multiple V1 endpoints consolidated into single flexible V2 endpoint |
| **Path parameters** | - `consentId` (AIS)<br>- `payment-service`, `payment-product`, `paymentId` (PIS)<br>- `CoF-consent-id` (PIIS)<br>- `authorisationId` (all) | - `resource-path` (combines service + product-type)<br>- `resourceId` (consent/payment/subscription ID)<br>- `authorisation-category` (`authorisations` or `cancellation-authorisations`)<br>- `authorisationId` | Unified & generalized parameter model |
| **Header: X-Request-ID** | Mandatory UUID | Mandatory UUID | Same requirement |
| **Header: Digest** | Present only if `Signature` header is used. | Always required when `x-jws-signature` is used. Must follow [RFC3230], SHA-256 or SHA-512. | Stronger requirement and clarified algorithms |
| **Header: Signature** | `Signature` (RSA, headers list, base64) + `TPP-Signature-Certificate` | Replaced with `x-jws-signature` (JSON Web Signature). | Changed signature mechanism |
| **Header: TPP-Signature-Certificate** | Required if `Signature` is present. | Removed. | Certificate not sent directly, handled via JWS |
| **Header: PSU-* fields** | `PSU-IP-Address`, `PSU-IP-Port`, `PSU-Accept*`, `PSU-User-Agent`, `PSU-Http-Method`, `PSU-Device-ID`, `PSU-Geo-Location` | Not part of V2 spec. | Removed from V2 |
| **PSU-Http-Method** | Allowed: `GET`, `POST`, `PUT`, `PATCH`, `DELETE`. | Not used in V2. | Dropped |

### Response 200 — SCA Status (V1 vs V2)

| Aspect | V1 (all variants: PIS, AIS, PIIS) | V2 (unified) | Change |
|---|---|---|---|
| **Container** | `scaStatusResponse` | Inline object (no wrapper name) | Structure simplified |
| **`scaStatus`** | Required. Enum: `received`, `psuIdentified`, `psuAuthenticated`, `scaMethodSelected`, `started`, `unconfirmed`, `finalised`, `failed`, `exempted`. Example: `psuAuthenticated`. | Required. Same enum list, Example: `received`. | No change in values; example adjusted |
| **`psuMessage`** | Optional, maxLength 500. Text to PSU. | ❌ Not present | Removed |
| **`psuName`** | Optional, maxLength 140. Name of PSU. | Present, maxLength 140. Clarified: “Usage is following EBA Q&A 2020_5165.” | Kept but clarified |
| **`trustedBeneficiaryFlag`** | Optional boolean. Indicates if creditor is trusted beneficiary (only for final `scaStatus`). | ❌ Not present | Removed |
| **`_links`** | Very large set of links (`scaRedirect`, `scaOAuth`, `confirmation`, `startAuthorisation*`, `updatePsu*`, `updateProprietaryData*`, `selectAuthenticationMethod`, `creditorNameConfirmation`, `authoriseTransaction`, `self`, `status`, `scaStatus`, `account`, `balances`, `transactions`, `transactionDetails`, `cardAccount`, `cardTransactions`, `first`, `next`, `previous`, `last`, `download`, `<*>`, etc.) | Simplified: only `scaStatus` and `<*>` with generic `hrefType`. Description: “Should refer to next steps if resolvable via interface.” | Major reduction of link set |
| **TPP messages** | `tppMessage[]` of type `tppMessageGeneric` with `category`, `code`, `path`, `text`. Enums limited (ERROR/WARNING). | `tppMessages[]` of type `clientMessageInformation` with structured error codes: <br>• `messageCode_ServiceUnspecific` (400–409 sets) <br>• `messageCode_PisSpecific` <br>• `messageCode_AisSpecific` <br>• `messageCode_PiisSpecific` <br>• `messageCode_SigningBasketSpecific` <br>• `messageCode_PushAisSpecific` <br>Includes many new enums (e.g. `CONSENT_TYPE_NOT_SUPPORTED`, `KID_MISSING`, `PAYMENT_FAILED`, etc.). | Refactored & expanded error coding system |
| **Headers** | `X-Request-ID` | `X-Request-ID`, plus **`X-Reference-API-Name`**, **`X-Reference-API-Document`**, **`X-Reference-API-Version`** | New metadata headers added |



### Response 400 — Bad Request (V1 vs V2)

| Aspect | V1 (AIS / PIS / Cancellation-PIS) | V2 (unified AUTHORISATION) | Change |
|---|---|---|---|
| **Container** | `Error400_NG_AIS` / `Error400_NG_PIS` | `error_NG_400_AUTHORISATION` | Unified structure |
| **Error item array** | `tppMessages[]` of `tppMessage400_AIS` or `tppMessage400_PIS` | `apiClientMessages[]` of `clientMessageInformation_400_AUTHORISATION` | Renamed structure |
| **Category** | Enum: `ERROR`, `WARNING` | Same (string, only `ERROR` or `WARNING`) | No change |
| **Code enums** | **AIS:** `FORMAT_ERROR, PARAMETER_NOT_CONSISTENT, PARAMETER_NOT_SUPPORTED, SERVICE_INVALID, RESOURCE_UNKNOWN, RESOURCE_EXPIRED, RESOURCE_BLOCKED, TIMESTAMP_INVALID, PERIOD_INVALID, SCA_METHOD_UNKNOWN, SCA_INVALID, CONSENT_UNKNOWN, SESSIONS_NOT_SUPPORTED` <br> **PIS:** Same as AIS plus: `PAYMENT_FAILED, EXECUTION_DATE_INVALID` | `MessageCode_ServiceUnspecific_400` only: `FORMAT_ERROR, PARAMETER_NOT_CONSISTENT, PARAMETER_NOT_SUPPORTED, SERVICE_INVALID, CONSENT_UNKNOWN, RESOURCE_UNKNOWN, RESOURCE_EXPIRED, RESOURCE_BLOCKED, TIMESTAMP_INVALID, PERIOD_INVALID, SCA_METHOD_UNKNOWN, SCA_INVALID` | In V2: <br>• **Removed:** `PAYMENT_FAILED`, `EXECUTION_DATE_INVALID`, `SESSIONS_NOT_SUPPORTED` <br>• Codes consolidated under “ServiceUnspecific” |
| **Path / Text fields** | `path`, `text` (Max 500 chars) | Same (`path`, `text`, Max 500 chars) | No change |
| **Links (`_links`)** | Very large set (includes `scaRedirect`, `scaOAuth`, `confirmation`, all `startAuthorisation*`, `updatePsu*`, `updateProprietaryData*`, `updateEncrypted*`, `updateAdditional*`, `selectAuthenticationMethod`, `creditorNameConfirmation`, `authoriseTransaction`, `self`, `status`, `scaStatus`, `account`, `balances`, `transactions`, `transactionDetails`, `cardAccount`, `cardTransactions`, `first`, `next`, `previous`, `last`, `download`, `<*>`) | Expanded and renamed set: includes V1 base plus many new ones (**`updateResourceByDebtorAccountResource`**, **`transactionfees`**, **`savingsAccount`**, **`loanAccount`**, **`ibanCheck`**, **`paymentInitiation`**, **`securitiesAccount`**, **`positions`**, **`orders`**, **`orderDetails`**, **`relatedOrders[]`**, **`relatedTransactions[]`**, **`subscription`**, **`entryStatusRevoked[]`**, **`confirmInitiation`**, **`aspspParameters`**, **`aspspContacts`**, **`aspspDowntimes`**, **`onboardings`**, **`readConditions`**, **`confirmConditions`**) | In V2: <br>• **Removed:** `updateAdditionalPsuAuthentication`, `updateAdditionalEncryptedPsuAuthentication`, `<*>` <br>• **Added:** large new set (see bold) |
| **Headers** | `Location`, `X-Request-ID` | `Location`, `X-Request-ID`, plus **`X-Reference-API-Name`**, **`X-Reference-API-Document`**, **`X-Reference-API-Version`**, **`ASPSP-Notification-Support`** | Metadata headers added |


### 401 response

| Aspect | V1 (AIS / PIS / Cancellation-PIS) | V2 (unified `error_NG_401_AUTHORISATION`) | Change |
|---|---|---|---|
| **Container** | `Error401_NG_AIS` / `Error401_NG_PIS` | `error_NG_401_AUTHORISATION` | Unified container in V2 |
| **Error item array** | `tppMessages[]` of `tppMessage401_AIS` / `tppMessage401_PIS` | `apiClientMessages[]` of `clientMessageInformation_401_AUTHORISATION` | Structure renamed |
| **Category** | Enum: `ERROR`, `WARNING` | String, only `ERROR` or `WARNING` permitted | Same semantics |
| **Code enums (401)** | **AIS:** `CERTIFICATE_INVALID`, `ROLE_INVALID`, `CERTIFICATE_EXPIRED`, `CERTIFICATE_BLOCKED`, `CERTIFICATE_REVOKE`, `CERTIFICATE_MISSING`, `SIGNATURE_INVALID`, `SIGNATURE_MISSING`, `CORPORATE_ID_INVALID`, `PSU_CREDENTIALS_INVALID`, `CONSENT_INVALID`, `CONSENT_EXPIRED`, `TOKEN_UNKNOWN`, `TOKEN_INVALID`, `TOKEN_EXPIRED` <br> **PIS (incl. cancellation-authorisations):** *All AIS codes* **+** `KID_MISSING` | `MessageCode_ServiceUnspecific_401` (single list): `CERTIFICATE_INVALID`, `ROLE_INVALID`, `CERTIFICATE_EXPIRED`, `CERTIFICATE_BLOCKED`, **`CERTIFICATE_REVOKED`**, `CERTIFICATE_MISSING`, **`CLIENT_INVALID`**, **`CLIENT_INCONSISTENT`**, **`API_CONTRACT_ID_INVALID`**, `SIGNATURE_INVALID`, `SIGNATURE_MISSING`, `ROLE_INVALID` *(duplicate in spec; treat as same value)*, `PSU_CREDENTIALS_INVALID`, `CORPORATE_ID_INVALID`, `CONSENT_INVALID`, `CONSENT_EXPIRED`, `TOKEN_UNKNOWN`, `TOKEN_INVALID`, `TOKEN_EXPIRED` | **Renames/Additions/Removals:** <br>• **Renamed:** `CERTIFICATE_REVOKE` → **`CERTIFICATE_REVOKED`** (V2) <br>• **Added (V2 only):** `CLIENT_INVALID`, `CLIENT_INCONSISTENT`, `API_CONTRACT_ID_INVALID` <br>• **Missing in V2 list:** `KID_MISSING` (PIS-specific code in V1 is **not** present in V2 service-unspecific set) |
| **Path / Text** | `path`, `text` (max 500 chars) | `path`, `text` (max 500 chars) | No change |
| **_links** | Large set, e.g. `scaRedirect`, `scaOAuth`, `confirmation`, `startAuthorisation*`, `updatePsu*`, `updateProprietaryData*`, `startAuthorisationWithEncryptedPsuAuthentication`, `updateEncryptedPsuAuthentication`, **`updateAdditionalPsuAuthentication`**, **`updateAdditionalEncryptedPsuAuthentication`**, `selectAuthenticationMethod`, `creditorNameConfirmation`, `authoriseTransaction`, `self`, `status`, `scaStatus`, `account`, `balances`, `transactions`, `transactionDetails`, `cardAccount`, `cardTransactions`, paging (`first`,`next`,`previous`,`last`), `download`, **`<*>`** | Expanded & renamed set: retains base links and adds **`updateResourceByDebtorAccountResource`**, **`transactionfees`**, **`savingsAccount`**, **`loanAccount`**, **`ibanCheck`**, **`paymentInitiation`**, **`securitiesAccount`**, **`positions`**, **`orders`**, **`orderDetails`**, **`relatedOrders[]`**, **`relatedTransactions[]`**, **`subscription`**, **`entryStatusRevoked[]`**, **`confirmInitiation`**, **`aspspParameters`**, **`aspspContacts`**, **`aspspDowntimes`**, **`onboardings`**, **`readConditions`**, **`confirmConditions`** | **Removed in V2:** `updateAdditionalPsuAuthentication`, `updateAdditionalEncryptedPsuAuthentication`, wildcard **`<*>`**. **Added many new links** (see bold) |
| **Response headers** | `Location`, `X-Request-ID` | `Location`, `X-Request-ID`, **`X-Reference-API-Name`**, **`X-Reference-API-Document`**, **`X-Reference-API-Version`**, **`ASPSP-Notification-Support`** | New reference/feature headers in V2 |

---


- **Common (V1 ↔ V2):**  
  `CERTIFICATE_INVALID`, `ROLE_INVALID`, `CERTIFICATE_EXPIRED`, `CERTIFICATE_BLOCKED`, `CERTIFICATE_MISSING`, `SIGNATURE_INVALID`, `SIGNATURE_MISSING`, `CORPORATE_ID_INVALID`, `PSU_CREDENTIALS_INVALID`, `CONSENT_INVALID`, `CONSENT_EXPIRED`, `TOKEN_UNKNOWN`, `TOKEN_INVALID`, `TOKEN_EXPIRED`

- **Only in V1 (PIS):**  
  `KID_MISSING`

- **Only in V1 (name form):**  
  `CERTIFICATE_REVOKE`  → **V2 uses `CERTIFICATE_REVOKED`**

- **Only in V2:**  
  `CLIENT_INVALID`, `CLIENT_INCONSISTENT`, `API_CONTRACT_ID_INVALID`, **`CERTIFICATE_REVOKED`** (renamed from V1)



### Response 403 — Differences Only (V1 vs V2)

| Aspect | V1 (AIS / PIS) | V2 (unified `error_NG_403_AUTHORISATION`) | Change |
|---|---|---|---|
| **Container** | `Error403_NG_AIS` / `Error403_NG_PIS` | `error_NG_403_AUTHORISATION` | Unified container in V2 |
| **Error item array** | `tppMessages[]` of `tppMessage403_AIS` / `tppMessage403_PIS` | `apiClientMessages[]` of `clientMessageInformation_403_AUTHORISATION` | Structure renamed |
| **Category** | Enum: `ERROR`, `WARNING` | String, only `ERROR` or `WARNING` permitted | Same semantics |
| **Code enums (403)** | **AIS:** `CONSENT_UNKNOWN`, `SERVICE_BLOCKED`, `RESOURCE_UNKNOWN`, `RESOURCE_EXPIRED` <br> **PIS (incl. cancellation-authorisations):** *AIS set* **+** `PRODUCT_INVALID` | `MessageCode_ServiceUnspecific_403`: `SERVICE_BLOCKED`, `CONSENT_UNKNOWN`, `RESOURCE_UNKNOWN`, `RESOURCE_EXPIRED` | **PIS-only `PRODUCT_INVALID` is not present** in the V2 service-unspecific set (may be handled under PIS-specific in other endpoints, but not here). |
| **Path / Text** | `path`, `text` (max 500 chars) | `path`, `text` (max 500 chars) | No change |
| **_links** | Large base set incl. `scaRedirect`, `scaOAuth`, `confirmation`, many `startAuthorisation*` / `update*` links, `selectAuthenticationMethod`, `creditorNameConfirmation`, `authoriseTransaction`, resources (`self`, `status`, `scaStatus`, `account`, `balances`, `transactions`, `transactionDetails`, `cardAccount`, `cardTransactions`), paging (`first`,`next`,`previous`,`last`), `download`, and **`updateAdditionalPsuAuthentication`**, **`updateAdditionalEncryptedPsuAuthentication`**, **`<*>`** | Retains core links and **adds**: `updateResourceByDebtorAccountResource`, `transactionfees`, `savingsAccount`, `loanAccount`, `ibanCheck`, `paymentInitiation`, `securitiesAccount`, `positions`, `orders`, `orderDetails`, `relatedOrders[]`, `relatedTransactions[]`, `subscription`, `entryStatusRevoked[]`, `confirmInitiation`, `aspspParameters`, `aspspContacts`, `aspspDowntimes`, `onboardings`, `readConditions`, `confirmConditions` | **Removed in V2:** `updateAdditionalPsuAuthentication`, `updateAdditionalEncryptedPsuAuthentication`, wildcard **`<*>`**. **Added many new links** (see bold). |
| **Response headers** | `Location`, `X-Request-ID` | `Location`, `X-Request-ID`, **`X-Reference-API-Name`**, **`X-Reference-API-Document`**, **`X-Reference-API-Version`**, **`ASPSP-Notification-Support`** | New reference/feature headers in V2 |

---


### Response 404 — Differences Only (V1 vs V2)

| Aspect | V1 (AIS / PIS) | V2 (unified `error_NG_404_AUTHORISATION`) | Change |
|---|---|---|---|
| **Container** | `Error404_NG_AIS` / `Error404_NG_PIS` | `error_NG_404_AUTHORISATION` | Unified container in V2 |
| **Error item array** | `tppMessages[]` of `tppMessage404_AIS` / `tppMessage404_PIS` | `apiClientMessages[]` of `clientMessageInformation_404_AUTHORISATION` | Structure renamed |
| **Category** | Enum: `ERROR`, `WARNING` | String, only `ERROR` or `WARNING` permitted | Same semantics |
| **Code enums (404)** | **AIS:** `RESOURCE_UNKNOWN` <br> **PIS:** `RESOURCE_UNKNOWN`, `PRODUCT_UNKNOWN` | `MessageCode_ServiceUnspecific_404`: `RESOURCE_UNKNOWN` | **`PRODUCT_UNKNOWN` (PIS-only) not present** in V2’s service-unspecific set for this method |
| **Path / Text** | `path`, `text` (max 500) | `path`, `text` (max 500) | No change |
| **_links** | Base set: `scaRedirect`, `scaOAuth`, `confirmation`, `startAuthorisation*`, `update*`, `selectAuthenticationMethod`, `creditorNameConfirmation`, `authoriseTransaction`, resources (`self`, `status`, `scaStatus`, `account`, `balances`, `transactions`, `transactionDetails`, `cardAccount`, `cardTransactions`), paging (`first`,`next`,`previous`,`last`), `download`, and **`updateAdditionalPsuAuthentication`**, **`updateAdditionalEncryptedPsuAuthentication`**, **`<*>`** | Retains core links and **adds** many: `updateResourceByDebtorAccountResource`, `transactionfees`, `savingsAccount`, `loanAccount`, `ibanCheck`, `paymentInitiation`, `securitiesAccount`, `positions`, `orders`, `orderDetails`, `relatedOrders[]`, `relatedTransactions[]`, `subscription`, `entryStatusRevoked[]`, `confirmInitiation`, `aspspParameters`, `aspspContacts`, `aspspDowntimes`, `onboardings`, `readConditions`, `confirmConditions` | **Removed in V2:** `updateAdditionalPsuAuthentication`, `updateAdditionalEncryptedPsuAuthentication`, wildcard **`<*>`**. **Added many new links** (see bold) |
| **Response headers** | `Location`, `X-Request-ID` | `Location`, `X-Request-ID`, **`X-Reference-API-Name`**, **`X-Reference-API-Document`**, **`X-Reference-API-Version`**, **`ASPSP-Notification-Support`** | New reference/feature headers in V2 |


---


### Response 405 — Differences Only (V1 vs V2)

| Aspect | V1 (AIS / PIS) | V2 (unified `error_NG_405_AUTHORISATION`) | Change |
|---|---|---|---|
| **Container** | `Error405_NG_AIS` / `Error405_NG_PIS` | `error_NG_405_AUTHORISATION` | Unified container in V2 |
| **Error item array** | `tppMessages[]` of `tppMessage405_AIS` / `tppMessage405_PIS` | `apiClientMessages[]` of `clientMessageInformation_405_AUTHORISATION` | Structure renamed |
| **Category** | Enum: `ERROR`, `WARNING` | String, only `ERROR` or `WARNING` permitted | Same semantics |
| **Code enums (405)** | **AIS & PIS:** `SERVICE_INVALID` | `MessageCode_ServiceUnspecific_405`: `SERVICE_INVALID` | Same code set; type refactored |
| **Path / Text** | `path`, `text` (max 500) | `path`, `text` (max 500) | No change |
| **_links** | Base set: `scaRedirect`, `scaOAuth`, `confirmation`, the `startAuthorisation*` & `update*` family, `selectAuthenticationMethod`, `creditorNameConfirmation`, `startAuthorisationWithTransactionAuthorisation`, `authoriseTransaction`, resources (`self`, `status`, `scaStatus`, `account`, `balances`, `transactions`, `transactionDetails`, `cardAccount`, `cardTransactions`), paging (`first`,`next`,`previous`,`last`), `download`, and **`updateAdditionalPsuAuthentication`**, **`updateAdditionalEncryptedPsuAuthentication`**, wildcard **`<*>`** | Retains core links and **adds** many: `updateResourceByDebtorAccountResource`, `transactionfees`, `savingsAccount`, `loanAccount`, `ibanCheck`, `paymentInitiation`, `securitiesAccount`, `positions`, `orders`, `orderDetails`, `relatedOrders[]`, `relatedTransactions[]`, `subscription`, `entryStatusRevoked[]`, `confirmInitiation`, `aspspParameters`, `aspspContacts`, `aspspDowntimes`, `onboardings`, `readConditions`, `confirmConditions` | **Removed in V2:** `updateAdditionalPsuAuthentication`, `updateAdditionalEncryptedPsuAuthentication`, wildcard **`<*>`**. **Added many new links** (see bold) |
| **Response headers** | `Location`, `X-Request-ID` | `Location`, `X-Request-ID`, **`X-Reference-API-Name`**, **`X-Reference-API-Document`**, **`X-Reference-API-Version`**, **`ASPSP-Notification-Support`** | New reference/feature headers in V2 |

---


### Response 406 — Not Acceptable (V1 vs V2)

| Aspect | V1 (per-endpoint specs) | V2 (unified) | Change |
|---|---|---|---|
| **Schema** | Not specified | Not specified | No body in both |
| **_links** | *No links* | *No links* | Unchanged |
| **Headers** | **Location**, **X-Request-ID** | **Location**, **X-Request-ID**, **X-Reference-API-Name**, **X-Reference-API-Document**, **X-Reference-API-Version**, **ASPSP-Notification-Support** | V2 adds reference/meta + notification capability headers |

**V2 header details**
- `X-Reference-API-Name` — `"Berlin Group openFinance API"`
- `X-Reference-API-Document` — Reference IG name (e.g., “Extended Payment Initiation Services”)
- `X-Reference-API-Version` — Version of the reference API
- `ASPSP-Notification-Support` — `boolean` indicating resource status notification support

**V1 header set**
- `Location` — Location of the created resource
- `X-Request-ID` — Request correlation ID

---

### Response 408 — Request Timeout (V1 vs V2)

| Aspect | V1 (per-endpoint specs) | V2 (unified) | Change |
|---|---|---|---|
| **Schema** | Not specified | Not specified | No body in both |
| **_links** | *No links* | *No links* | Unchanged |
| **Headers** | **Location**, **X-Request-ID** | **Location**, **X-Request-ID**, **X-Reference-API-Name**, **X-Reference-API-Document**, **X-Reference-API-Version**, **ASPSP-Notification-Support** | V2 adds reference/meta + notification capability headers |

**V2 header details**
- `X-Reference-API-Name` — `"Berlin Group openFinance API"`
- `X-Reference-API-Document` — Reference IG name (e.g., “Extended Payment Initiation Services”)
- `X-Reference-API-Version` — Version of the reference API
- `ASPSP-Notification-Support` — `boolean` indicating resource status notification support

**V1 header set**
- `Location` — Location of the created resource
- `X-Request-ID` — Request correlation ID

---

### Response 409 — Conflict (Unified V2 vs. V1 endpoints)

> Applies to:
> - **V2 (unified):** `GET /v2/{resource-path}/{resourceId}/{authorisation-category}/{authorisationId}`
> - **V1 (PIS):** `GET /v1/{payment-service}/{payment-product}/{paymentId}/authorisations/{authorisationId}`
> - **V1 (PIS – cancellation):** `GET /v1/{payment-service}/{payment-product}/{paymentId}/cancellation-authorisations/{authorisationId}`
> - **V1 (AIS):** `GET /v1/consents/{consentId}/authorisations/{authorisationId}`

#### Differences at a glance

| Aspect | V1 (per-endpoint) | V2 (unified) | Change |
|---|---|---|---|
| **Error container & item type** | `tppMessages[]` of `tppMessage409_AIS` or `tppMessage409_PIS` | `apiClientMessages[]` of `clientMessageInformation_409_AUTHORISATION` | Renamed and unified types |
| **Error code field** | `code` is `MessageCode409_AIS` or `MessageCode409_PIS` | `code` is `MessageCode_ServiceUnspecific_409` (refers to Data Dictionary) | Naming & scoping unified |
| **Known code values** | Only `STATUS_INVALID` (AIS & PIS) | Example shows `STATUS_INVALID` (Data Dictionary governs full set) | Equivalent known value; source of truth moved |
| **Message fields** | `category`, `code`, `path`, `text` | `category`, `code`, `path`, `text` | Same shape |
| **_links** | Large NextGen `_linksAll` set (includes `updateAdditionalPsuAuthentication`, `updateAdditionalEncryptedPsuAuthentication`, wildcard `<*>`) | Expanded `links` set (adds `transactionfees`, `savingsAccount`, `loanAccount`, `ibanCheck`, `paymentInitiation`, `securitiesAccount`, `positions`, `orders`, `orderDetails`, `relatedOrders[]`, `relatedTransactions[]`, `subscription`, `entryStatusRevoked[]`, `confirmInitiation`, `aspspParameters`, `aspspContacts`, `aspspDowntimes`, `onboardings`, `readConditions`, `confirmConditions`; **drops** the two `updateAdditional*` and wildcard) | Link model broadened & standardized |
| **Headers** | `Location`, `X-Request-ID` | `Location`, `X-Request-ID`, **`X-Reference-API-Name`**, **`X-Reference-API-Document`**, **`X-Reference-API-Version`**, **`ASPSP-Notification-Support`** | Reference & capability headers added |

---

#### Error code enumerations (by endpoint)

- **V2 unified (`error_NG_409_AUTHORISATION`)**
  - `category`: `"ERROR"` or `"WARNING"`
  - `code`: `MessageCode_ServiceUnspecific_409` (spec points to Data Dictionary; example `STATUS_INVALID`)
  - `path`: `string` (optional)
  - `text`: `string` (max 500)

- **V1 PIS (`Error409_NG_PIS`)**
  - `category`: `"ERROR"` or `"WARNING"`
  - `code`: `MessageCode409_PIS` → **`STATUS_INVALID`**
  - `path`: `string` (optional)
  - `text`: `string` (max 500)

- **V1 PIS — cancellation (`Error409_NG_PIS`)**
  - Same as PIS above:
    - `category`: `"ERROR"` or `"WARNING"`
    - `code`: **`STATUS_INVALID`**
    - `path`: `string` (optional)
    - `text`: `string` (max 500)

- **V1 AIS (`Error409_NG_AIS`)**
  - `category`: `"ERROR"` or `"WARNING"`
  - `code`: `MessageCode409_AIS` → **`STATUS_INVALID`**
  - `path`: `string` (optional)
  - `text`: `string` (max 500)

---

#### Links comparison (detail)

**V1 `_linksAll` (representative set)**
- `scaRedirect`, `scaOAuth`, `confirmation`
- `startAuthorisation`, `startAuthorisationWithPsuIdentification`, `updatePsuIdentification`
- `startAuthorisationWithProprietaryData`, `updateProprietaryData`
- `startAuthorisationWithPsuAuthentication`, `updatePsuAuthentication`
- `startAuthorisationWithEncryptedPsuAuthentication`, `updateEncryptedPsuAuthentication`
- **`updateAdditionalPsuAuthentication`**, **`updateAdditionalEncryptedPsuAuthentication`**
- `startAuthorisationWithAuthenticationMethodSelection`, `selectAuthenticationMethod`
- `creditorNameConfirmation`
- `startAuthorisationWithTransactionAuthorisation`, `authoriseTransaction`
- `self`, `status`, `scaStatus`
- `account`, `balances`, `transactions`, `transactionDetails`
- `cardAccount`, `cardTransactions`
- Paging: `first`, `next`, `previous`, `last`
- `download`
- `<?>` (wildcard)

**V2 `links` (delta vs V1)**
- **Added:** `updateResourceByDebtorAccountResource`, `transactionfees`, **`savingsAccount`**, **`loanAccount`**, **`ibanCheck`**, **`paymentInitiation`**, **`securitiesAccount`**, **`positions`**, **`orders`**, **`orderDetails`**, **`relatedOrders[]`**, **`relatedTransactions[]`**, **`subscription`**, **`entryStatusRevoked[]`**, **`confirmInitiation`**, **`aspspParameters`**, **`aspspContacts`**, **`aspspDowntimes`**, **`onboardings`**, **`readConditions`**, **`confirmConditions`**
- **Removed:** `updateAdditionalPsuAuthentication`, `updateAdditionalEncryptedPsuAuthentication`, wildcard `<*>`
- **Kept:** Core authorisation, OAuth/redirect, self/status/scaStatus, account & transaction navigation, pagination, download

---

#### Headers

- **V1 (all three endpoints):**  
  `Location`, `X-Request-ID`

- **V2 (unified):**  
  `Location`, `X-Request-ID`, **`X-Reference-API-Name`** (`"Berlin Group openFinance API"`), **`X-Reference-API-Document`**, **`X-Reference-API-Version`**, **`ASPSP-Notification-Support`** (`boolean`)


---

### Response 415 — Unsupported Media Type (Unified V2 vs. V1 endpoints)

> Applies to:
> - **V1 (PIS):** `GET /v1/{payment-service}/{payment-product}/{paymentId}/authorisations/{authorisationId}`
> - **V1 (PIS – cancellation):** `GET /v1/{payment-service}/{payment-product}/{paymentId}/cancellation-authorisations/{authorisationId}`
> - **V1 (AIS):** `GET /v1/consents/{consentId}/authorisations/{authorisationId}`
> - **V2 (unified):** `GET /v2/{resource-path}/{resourceId}/{authorisation-category}/{authorisationId}`

| Aspect | V1 (per-endpoint) | V2 (unified) | Change |
|---|---|---|---|
| **Presence** | Present on all three V1 endpoints | **Not provided in your V2 extract** | Unknown / not in provided spec |
| **Body** | None specified | — | — |
| **Headers** | `Location`, `X-Request-ID` | — | V2 not specified |
| **Links** | “No links” | — | — |


---

### Response 429 — Too Many Requests (Unified V2 vs. V1 endpoints)

> Applies to:
> - **V1 (PIS):** `GET /v1/{payment-service}/{payment-product}/{paymentId}/authorisations/{authorisationId}`
> - **V1 (PIS – cancellation):** `GET /v1/{payment-service}/{payment-product}/{paymentId}/cancellation-authorisations/{authorisationId}`
> - **V1 (AIS):** `GET /v1/consents/{consentId}/authorisations/{authorisationId}`
> - **V2 (unified):** `GET /v2/{resource-path}/{resourceId}/{authorisation-category}/{authorisationId}`

| Aspect | V1 (per-endpoint) | V2 (unified) | Change |
|---|---|---|---|
| **Presence** | Present (PIS + PIS cancellation: headers only; AIS: example body with `ACCESS_EXCEEDED`) | **Removed** (explicitly stated) | 429 no longer defined in V2 |
| **Body** | **AIS example (application/json):** array of `{ category, code: "ACCESS_EXCEEDED", text }` | — | Body removed with status removal |
| **Headers** | `Location`, `X-Request-ID` | — | — |
| **Links** | “No links” | — | — |


---

### Response 500 — Internal Server Error (Unified V2 vs. V1 endpoints)

> Applies to:
> - **V1 (PIS):** `GET /v1/{payment-service}/{payment-product}/{paymentId}/authorisations/{authorisationId}`
> - **V1 (PIS – cancellation):** `GET /v1/{payment-service}/{payment-product}/{paymentId}/cancellation-authorisations/{authorisationId}`
> - **V1 (AIS):** `GET /v1/consents/{consentId}/authorisations/{authorisationId}`
> - **V2 (unified):** `GET /v2/{resource-path}/{resourceId}/{authorisation-category}/{authorisationId}`

| Aspect | V1 (per-endpoint) | V2 (unified) | Change |
|---|---|---|---|
| **Presence** | Present on all three V1 endpoints | **Not provided in your V2 extract** | Unknown / not in provided spec |
| **Body** | None specified | — | — |
| **Headers** | `Location`, `X-Request-ID` | — | V2 not specified |
| **Links** | “No links” | — | — |


---

### Response 503 — Service Unavailable (Unified V2 vs. V1 endpoints)

> Applies to:
> - **V1 (PIS):** `GET /v1/{payment-service}/{payment-product}/{paymentId}/authorisations/{authorisationId}`
> - **V1 (PIS – cancellation):** `GET /v1/{payment-service}/{payment-product}/{paymentId}/cancellation-authorisations/{authorisationId}`
> - **V1 (AIS):** `GET /v1/consents/{consentId}/authorisations/{authorisationId}`
> - **V2 (unified):** `GET /v2/{resource-path}/{resourceId}/{authorisation-category}/{authorisationId}`

| Aspect | V1 (per-endpoint) | V2 (unified) | Change |
|---|---|---|---|
| **Presence** | Present on all three V1 endpoints | **Not provided in your V2 extract** | Unknown / not in provided spec |
| **Body** | None specified | — | — |
| **Headers** | `Location`, `X-Request-ID` | — | V2 not specified |
| **Links** | “No links” | — | — |


---

#### Notes / assumptions kept out of tables
- You stated **V2 removed 429** — this is reflected above.
- For **415/500/503 in V2**, you didn’t provide definitions. I’ve therefore marked those as *“Not provided in your V2 extract”* to stay precise and avoid guessing. If you share the V2 stubs for these statuses (if any), I’ll align the headers/links patterns (likely the `X-Reference-API-*` + `ASPSP-Notification-Support` headers you have on other V2 errors).
  




## 5.1 Delete Consent - AIS

**Endpoints**
- **V1:** `DELETE /v1/consents/{consent-id}`
- **V2:** `DELETE /v1/consents/confirmation-of-funds/{CoF-consent-id}`


```diff
- consentId * (path)

Path  
+ consent-category * 

Header
  Digest 
  - Only if Signature is present in request.  
  - Example: SHA-256=hl1/Eps8BEQW58FJhDApwJXjGY4nr1ArGDHIT25vq6A=
  + Must always follow RFC3230 + RFC5843 rules.  
  + Hash of body or empty byte list if no body.  
  + Algorithms: SHA-256, SHA-512.  
  + Example: SHA-256=hl1/Eps8BEQW58FJhDApwJXjGY4nr1ArGDHIT25vq6A=

- Signature 
- TPP-Signature-Certificate 

+ x-jws-signature 

- PSU-IP-Address 
- PSU-IP-Port 
- PSU-Accept 
- PSU-Accept-Charset 
- PSU-Accept-Encoding 
- PSU-Accept-Language 
- PSU-User-Agent 
- PSU-Http-Method 
- PSU-Device-ID 
- PSU-Geo-Location 
```


---


### Response 204

| Aspect      | V1 Response 204                          | V2 Response 204                          |
|-------------|------------------------------------------|------------------------------------------|
| **Headers** | **X-Request-ID** – ID of the request, unique to the call, as determined by the initiating party. | **X-Request-ID** – ID of the request, unique to the call, as determined by the initiating party.<br>**X-Reference-API-Name** – "Berlin Group openFinance API".<br>**X-Reference-API-Document** – The name of the Implementation Guideline document (e.g. "Extended Payment Initiation Services").<br>**X-Reference-API-Version** – Version of the reference API. |


---


### Response 400 — Differences Only (V1 vs V2)

| Aspect | V1 | V2 | Change |
|---|---|---|---|
| **Error container & item type** | `tppMessages[]` of `tppMessage400_AIS` | `apiClientMessages[]` of `clientMessageInformation_400_AIS` | Renamed structures |
| **Error code set** | `MessageCode400_AIS` (single enum list) | `MessageCode_400_AIS` **split into** `MessageCode_ServiceUnspecific_400` **+** `MessageCode_AisSpecific_400` | Refactored & partitioned |
| **New/removed codes** | Included: `SESSIONS_NOT_SUPPORTED` (together with others) | **Adds:** `CONSENT_TYPE_NOT_SUPPORTED` (AIS-specific). **Keeps:** `SESSIONS_NOT_SUPPORTED` but under AIS-specific. | Code set expanded/reorganized |
| **_links** | `scaRedirect`, `scaOAuth`, `confirmation`, `startAuthorisation`, `startAuthorisationWithPsuIdentification`, `updatePsuIdentification`, `startAuthorisationWithProprietaryData`, `updateProprietaryData`, `startAuthorisationWithPsuAuthentication`, `updatePsuAuthentication`, `startAuthorisationWithEncryptedPsuAuthentication`, `updateEncryptedPsuAuthentication`, `updateAdditionalPsuAuthentication`, `updateAdditionalEncryptedPsuAuthentication`, `startAuthorisationWithAuthenticationMethodSelection`, `selectAuthenticationMethod`, `creditorNameConfirmation`, `startAuthorisationWithTransactionAuthorisation`, `authoriseTransaction`, `self`, `status`, `scaStatus`, `account`, `balances`, `transactions`, `transactionDetails`, `cardAccount`, `cardTransactions`, `first`, `next`, `previous`, `last`, `download`, `<*>` | `scaRedirect`, `scaOAuth`, `confirmation`, `creditorNameConfirmation`, `startAuthorisation`, `startAuthorisationWithPsuIdentification`, `updatePsuIdentification`, `startAuthorisationWithProprietaryData`, `updateProprietaryData`, `startAuthorisationWithPsuAuthentication`, `updatePsuAuthentication`, `startAuthorisationWithEncryptedPsuAuthentication`, `updateEncryptedPsuAuthentication`, `startAuthorisationWithAuthenticationMethodSelection`, `selectAuthenticationMethod`, `startAuthorisationWithTransactionAuthorisation`, `authoriseTransaction`, `updateResourceByDebtorAccountResource`, `self`, `status`, **`transactionfees`**, `scaStatus`, `account`, **`savingsAccount`**, **`loanAccount`**, `balances`, `transactions`, `cardAccount`, `cardTransactions`, `transactionDetails`, **`ibanCheck`**, **`paymentInitiation`**, **`securitiesAccount`**, **`positions`**, **`orders`**, **`orderDetails`**, **`relatedOrders[]`**, **`relatedTransactions[]`**, **`subscription`**, **`entryStatusRevoked[]`**, `first`, `next`, `previous`, `last`, `download`, **`confirmInitiation`**, **`aspspParameters`**, **`aspspContacts`**, **`aspspDowntimes`**, **`onboardings`**, **`readConditions`**, **`confirmConditions`** | In V1: `updateAdditionalPsuAuthentication`, `updateAdditionalEncryptedPsuAuthentication`, `<*>` are removed. In V2: large set of new links are added (see bold). |
| **Response headers** | `Location`, `X-Request-ID` | `Location`, `X-Request-ID`, **`X-Reference-API-Name`**, **`X-Reference-API-Document`**, **`X-Reference-API-Version`**, **`ASPSP-Notification-Support`** | New headers added |


---


### Response 401 — Differences Only (V1 vs V2)

| Aspect | V1 | V2 | Change |
| --- | --- | --- | --- |
| **Error container & item type** | `tppMessages[]` of `tppMessage401_AIS` | `apiClientMessages[]` of `clientMessageInformation_401_AIS` | Renamed structures |
| **Error code set** | `MessageCode401_AIS` (single enum list: `CERTIFICATE_INVALID`, `ROLE_INVALID`, `CERTIFICATE_EXPIRED`, `CERTIFICATE_BLOCKED`, `CERTIFICATE_REVOKE`, `CERTIFICATE_MISSING`, `SIGNATURE_INVALID`, `SIGNATURE_MISSING`, `CORPORATE_ID_INVALID`, `PSU_CREDENTIALS_INVALID`, `CONSENT_INVALID`, `CONSENT_EXPIRED`, `TOKEN_UNKNOWN`, `TOKEN_INVALID`, `TOKEN_EXPIRED`) | `MessageCode_401_AIS` split into `MessageCode_ServiceUnspecific_401` (generic codes, ~19 total) + `MessageCode_AisSpecific_401` (AIS-specific codes, excludes `CONSENT_INVALID` by regex pattern) | Refactored & partitioned. Some codes reorganized into service-unspecific vs AIS-specific. |
| **_links** | `scaRedirect`, `scaOAuth`, `confirmation`, `startAuthorisation`, `startAuthorisationWithPsuIdentification`, `updatePsuIdentification`, `startAuthorisationWithProprietaryData`, `updateProprietaryData`, `startAuthorisationWithPsuAuthentication`, `updatePsuAuthentication`, `startAuthorisationWithEncryptedPsuAuthentication`, `updateEncryptedPsuAuthentication`, `updateAdditionalPsuAuthentication`, `updateAdditionalEncryptedPsuAuthentication`, `startAuthorisationWithAuthenticationMethodSelection`, `selectAuthenticationMethod`, `creditorNameConfirmation`, `startAuthorisationWithTransactionAuthorisation`, `authoriseTransaction`, `self`, `status`, `scaStatus`, `account`, `balances`, `transactions`, `transactionDetails`, `cardAccount`, `cardTransactions`, `first`, `next`, `previous`, `last`, `download`, `<*>` | `scaRedirect`, `scaOAuth`, `confirmation`, `creditorNameConfirmation`, `startAuthorisation`, `startAuthorisationWithPsuIdentification`, `updatePsuIdentification`, `startAuthorisationWithProprietaryData`, `updateProprietaryData`, `startAuthorisationWithPsuAuthentication`, `updatePsuAuthentication`, `startAuthorisationWithEncryptedPsuAuthentication`, `updateEncryptedPsuAuthentication`, `startAuthorisationWithAuthenticationMethodSelection`, `selectAuthenticationMethod`, `startAuthorisationWithTransactionAuthorisation`, `authoriseTransaction`, `updateResourceByDebtorAccountResource`, `self`, `status`, `transactionfees`, `scaStatus`, `account`, `savingsAccount`, `loanAccount`, `balances`, `transactions`, `cardAccount`, `cardTransactions`, `transactionDetails`, `ibanCheck`, `paymentInitiation`, `securitiesAccount`, `positions`, `orders`, `orderDetails`, `relatedOrders[]`, `relatedTransactions[]`, `subscription`, `entryStatusRevoked[]`, `first`, `next`, `previous`, `last`, `download`, `confirmInitiation`, `aspspParameters`, `aspspContacts`, `aspspDowntimes`, `onboardings`, `readConditions`, `confirmConditions` | In V1: `updateAdditionalPsuAuthentication`, `updateAdditionalEncryptedPsuAuthentication`, `<*>` removed. In V2: many new links added (e.g. `transactionfees`, `savingsAccount`, `loanAccount`, `ibanCheck`, `paymentInitiation`, `securitiesAccount`, `positions`, `orders`, `orderDetails`, `relatedOrders[]`, `relatedTransactions[]`, `subscription`, `entryStatusRevoked[]`, `confirmInitiation`, `aspspParameters`, `aspspContacts`, `aspspDowntimes`, `onboardings`, `readConditions`, `confirmConditions`). |
| **Response headers** | `Location`, `X-Request-ID` | `Location`, `X-Request-ID`, `X-Reference-API-Name`, `X-Reference-API-Document`, `X-Reference-API-Version`, `ASPSP-Notification-Support` | New headers added |


---


### Response 403 — Differences Only (V1 vs V2)

| Aspect | V1 | V2 | Change |
| --- | --- | --- | --- |
| **Error container & item type** | `tppMessages[]` of `tppMessage403_AIS` | `apiClientMessages[]` of `clientMessageInformation_403_AIS` | Renamed structures |
| **Error code set** | `MessageCode403_AIS` with enums: `CONSENT_UNKNOWN`, `SERVICE_BLOCKED`, `RESOURCE_UNKNOWN`, `RESOURCE_EXPIRED` | `MessageCode_403_AIS` under `anyOf` → `MessageCode_ServiceUnspecific_403` (same enums as V1) | Same set of codes, but reorganized under service-unspecific type |
| **_links** | `scaRedirect`, `scaOAuth`, `confirmation`, `startAuthorisation`, `startAuthorisationWithPsuIdentification`, `updatePsuIdentification`, `startAuthorisationWithProprietaryData`, `updateProprietaryData`, `startAuthorisationWithPsuAuthentication`, `updatePsuAuthentication`, `startAuthorisationWithEncryptedPsuAuthentication`, `updateEncryptedPsuAuthentication`, `updateAdditionalPsuAuthentication`, `updateAdditionalEncryptedPsuAuthentication`, `startAuthorisationWithAuthenticationMethodSelection`, `selectAuthenticationMethod`, `creditorNameConfirmation`, `startAuthorisationWithTransactionAuthorisation`, `authoriseTransaction`, `self`, `status`, `scaStatus`, `account`, `balances`, `transactions`, `transactionDetails`, `cardAccount`, `cardTransactions`, `first`, `next`, `previous`, `last`, `download`, `<*>` | `scaRedirect`, `scaOAuth`, `confirmation`, `creditorNameConfirmation`, `startAuthorisation`, `startAuthorisationWithPsuIdentification`, `updatePsuIdentification`, `startAuthorisationWithProprietaryData`, `updateProprietaryData`, `startAuthorisationWithPsuAuthentication`, `updatePsuAuthentication`, `startAuthorisationWithEncryptedPsuAuthentication`, `updateEncryptedPsuAuthentication`, `startAuthorisationWithAuthenticationMethodSelection`, `selectAuthenticationMethod`, `startAuthorisationWithTransactionAuthorisation`, `authoriseTransaction`, `updateResourceByDebtorAccountResource`, `self`, `status`, **`transactionfees`**, `scaStatus`, `account`, **`savingsAccount`**, **`loanAccount`**, `balances`, `transactions`, `cardAccount`, `cardTransactions`, `transactionDetails`, **`ibanCheck`**, **`paymentInitiation`**, **`securitiesAccount`**, **`positions`**, **`orders`**, **`orderDetails`**, **`relatedOrders[]`**, **`relatedTransactions[]`**, **`subscription`**, **`entryStatusRevoked[]`**, `first`, `next`, `previous`, `last`, `download`, **`confirmInitiation`**, **`aspspParameters`**, **`aspspContacts`**, **`aspspDowntimes`**, **`onboardings`**, **`readConditions`**, **`confirmConditions`** | In V1: `updateAdditionalPsuAuthentication`, `updateAdditionalEncryptedPsuAuthentication`, `<*>` are removed. In V2: many new links are added (see bold). |
| **Response headers** | `Location`, `X-Request-ID` | `Location`, `X-Request-ID`, **`X-Reference-API-Name`**, **`X-Reference-API-Document`**, **`X-Reference-API-Version`**, **`ASPSP-Notification-Support`** | New headers added |


---


### Response 404 — Differences Only (V1 vs V2)

| Aspect | V1 | V2 | Change |
| --- | --- | --- | --- |
| **Error container & item type** | `tppMessages[]` of `tppMessage404_AIS` | `apiClientMessages[]` of `clientMessageInformation_404_AIS` | Renamed structures |
| **Error code set** | `MessageCode404_AIS` with single enum: `RESOURCE_UNKNOWN` | `MessageCode_404_AIS` split into `MessageCode_ServiceUnspecific_404` (keeps `RESOURCE_UNKNOWN`) and `MessageCode_AisSpecific_404` (adds `CONTENT_TEMPORARILY_NOT_AVAILABLE`) | Code structure refactored and AIS-specific codes introduced |
| **_links** | `scaRedirect`, `scaOAuth`, `confirmation`, `startAuthorisation`, `startAuthorisationWithPsuIdentification`, `updatePsuIdentification`, `startAuthorisationWithProprietaryData`, `updateProprietaryData`, `startAuthorisationWithPsuAuthentication`, `updatePsuAuthentication`, `startAuthorisationWithEncryptedPsuAuthentication`, `updateEncryptedPsuAuthentication`, `updateAdditionalPsuAuthentication`, `updateAdditionalEncryptedPsuAuthentication`, `startAuthorisationWithAuthenticationMethodSelection`, `selectAuthenticationMethod`, `creditorNameConfirmation`, `startAuthorisationWithTransactionAuthorisation`, `authoriseTransaction`, `self`, `status`, `scaStatus`, `account`, `balances`, `transactions`, `transactionDetails`, `cardAccount`, `cardTransactions`, `first`, `next`, `previous`, `last`, `download`, `<*>` | `scaRedirect`, `scaOAuth`, `confirmation`, `creditorNameConfirmation`, `startAuthorisation`, `startAuthorisationWithPsuIdentification`, `updatePsuIdentification`, `startAuthorisationWithProprietaryData`, `updateProprietaryData`, `startAuthorisationWithPsuAuthentication`, `updatePsuAuthentication`, `startAuthorisationWithEncryptedPsuAuthentication`, `updateEncryptedPsuAuthentication`, `startAuthorisationWithAuthenticationMethodSelection`, `selectAuthenticationMethod`, `startAuthorisationWithTransactionAuthorisation`, `authoriseTransaction`, `updateResourceByDebtorAccountResource`, `self`, `status`, **`transactionfees`**, `scaStatus`, `account`, **`savingsAccount`**, **`loanAccount`**, `balances`, `transactions`, `cardAccount`, `cardTransactions`, `transactionDetails`, **`ibanCheck`**, **`paymentInitiation`**, **`securitiesAccount`**, **`positions`**, **`orders`**, **`orderDetails`**, **`relatedOrders[]`**, **`relatedTransactions[]`**, **`subscription`**, **`entryStatusRevoked[]`**, `first`, `next`, `previous`, `last`, `download`, **`confirmInitiation`**, **`aspspParameters`**, **`aspspContacts`**, **`aspspDowntimes`**, **`onboardings`**, **`readConditions`**, **`confirmConditions`** | V1: `updateAdditionalPsuAuthentication`, `updateAdditionalEncryptedPsuAuthentication`, `<*>` removed. V2: multiple new links added (see bold). |
| **Response headers** | `Location`, `X-Request-ID` | `Location`, `X-Request-ID`, **`X-Reference-API-Name`**, **`X-Reference-API-Document`**, **`X-Reference-API-Version`**, **`ASPSP-Notification-Support`** | New headers added |


---


### Response 405 — Differences Only (V1 vs V2)

| Aspect | V1 | V2 | Change |
| --- | --- | --- | --- |
| **Error container & item type** | `tppMessages[]` of `tppMessage405_AIS` | `apiClientMessages[]` of `clientMessageInformation_405_AIS` | Renamed structures |
| **Error code set** | `MessageCode405_AIS` with single enum: `SERVICE_INVALID` | `MessageCode_405_AIS` (split into `MessageCode_ServiceUnspecific_405`) — still only `SERVICE_INVALID` | Structure refactored, semantics unchanged |
| **_links** | `scaRedirect`, `scaOAuth`, `confirmation`, `startAuthorisation`, `startAuthorisationWithPsuIdentification`, `updatePsuIdentification`, `startAuthorisationWithProprietaryData`, `updateProprietaryData`, `startAuthorisationWithPsuAuthentication`, `updatePsuAuthentication`, `startAuthorisationWithEncryptedPsuAuthentication`, `updateEncryptedPsuAuthentication`, `updateAdditionalPsuAuthentication`, `updateAdditionalEncryptedPsuAuthentication`, `startAuthorisationWithAuthenticationMethodSelection`, `selectAuthenticationMethod`, `creditorNameConfirmation`, `startAuthorisationWithTransactionAuthorisation`, `authoriseTransaction`, `self`, `status`, `scaStatus`, `account`, `balances`, `transactions`, `transactionDetails`, `cardAccount`, `cardTransactions`, `first`, `next`, `previous`, `last`, `download`, `<*>` | `scaRedirect`, `scaOAuth`, `confirmation`, `creditorNameConfirmation`, `startAuthorisation`, `startAuthorisationWithPsuIdentification`, `updatePsuIdentification`, `startAuthorisationWithProprietaryData`, `updateProprietaryData`, `startAuthorisationWithPsuAuthentication`, `updatePsuAuthentication`, `startAuthorisationWithEncryptedPsuAuthentication`, `updateEncryptedPsuAuthentication`, `startAuthorisationWithAuthenticationMethodSelection`, `selectAuthenticationMethod`, `startAuthorisationWithTransactionAuthorisation`, `authoriseTransaction`, `updateResourceByDebtorAccountResource`, `self`, `status`, **`transactionfees`**, `scaStatus`, `account`, **`savingsAccount`**, **`loanAccount`**, `balances`, `transactions`, `cardAccount`, `cardTransactions`, `transactionDetails`, **`ibanCheck`**, **`paymentInitiation`**, **`securitiesAccount`**, **`positions`**, **`orders`**, **`orderDetails`**, **`relatedOrders[]`**, **`relatedTransactions[]`**, **`subscription`**, **`entryStatusRevoked[]`**, `first`, `next`, `previous`, `last`, `download`, **`confirmInitiation`**, **`aspspParameters`**, **`aspspContacts`**, **`aspspDowntimes`**, **`onboardings`**, **`readConditions`**, **`confirmConditions`** | V1: `updateAdditionalPsuAuthentication`, `updateAdditionalEncryptedPsuAuthentication`, `<*>` removed. V2: many new links added (see bold). |
| **Response headers** | `Location`, `X-Request-ID` | `Location`, `X-Request-ID`, **`X-Reference-API-Name`**, **`X-Reference-API-Document`**, **`X-Reference-API-Version`**, **`ASPSP-Notification-Support`** | New headers added |


---


### 🔹 Response 406 

*(no schema shown in v2)*

| Aspect     | V1 | V2 |
|------------|----|----|
| **Schema** | *(schema shown)* | *(no schema shown)* |
| **Headers** | - **Location** – Location of the created resource *(string)*<br>- **X-Request-ID** – ID of the request *(string)* | - **X-Reference-API-Name** – `"Berlin Group openFinance API"` *(string)*<br>- **X-Reference-API-Document** – Implementation Guideline document name *(string)*<br>- **X-Reference-API-Version** – Version of the API *(string)*<br>- **Location** – Location of the created resource *(string)*<br>- **X-Request-ID** – ID of the request *(string)*<br>- **ASPSP-Notification-Support** – Indicates support for resource status notifications *(boolean)* |


---


### Response 408

| Aspect     | V1 | V2 |
|------------|----|----|
| **Schema** | *(no schema shown)* | *(no schema shown)* |
| **Headers** | - **Location** – Location of the created resource *(string)*<br>- **X-Request-ID** – ID of the request *(string)* | - **X-Reference-API-Name** – `"Berlin Group openFinance API"` *(string)*<br>- **X-Reference-API-Document** – Implementation Guideline document name *(string)*<br>- **X-Reference-API-Version** – Version of the API *(string)*<br>- **Location** – Location of the created resource *(string)*<br>- **X-Request-ID** – ID of the request *(string)*<br>- **ASPSP-Notification-Support** – Indicates support for resource status notifications *(boolean)* |


--- 


### Response 409 — Differences Only (V1 vs V2)

| Aspect | V1 | V2 | Change |
| --- | --- | --- | --- |
| **Error container & item type** | `tppMessages[]` of `tppMessage409_AIS` | `apiClientMessages[]` of `clientMessageInformation_409_AIS` | Renamed structures |
| **Error code set** | `MessageCode409_AIS` with enums: `RESOURCE_BLOCKED`, `RESOURCE_UNKNOWN`, `STATUS_INVALID` | `MessageCode_409_AIS` (split into `MessageCode_ServiceUnspecific_409`) — same set: `RESOURCE_BLOCKED`, `RESOURCE_UNKNOWN`, `STATUS_INVALID` | Structure refactored, semantics unchanged |
| **_links** | `scaRedirect`, `scaOAuth`, `confirmation`, `startAuthorisation`, `startAuthorisationWithPsuIdentification`, `updatePsuIdentification`, `startAuthorisationWithProprietaryData`, `updateProprietaryData`, `startAuthorisationWithPsuAuthentication`, `updatePsuAuthentication`, `startAuthorisationWithEncryptedPsuAuthentication`, `updateEncryptedPsuAuthentication`, `updateAdditionalPsuAuthentication`, `updateAdditionalEncryptedPsuAuthentication`, `startAuthorisationWithAuthenticationMethodSelection`, `selectAuthenticationMethod`, `creditorNameConfirmation`, `startAuthorisationWithTransactionAuthorisation`, `authoriseTransaction`, `self`, `status`, `scaStatus`, `account`, `balances`, `transactions`, `transactionDetails`, `cardAccount`, `cardTransactions`, `first`, `next`, `previous`, `last`, `download`, `<*>` | `scaRedirect`, `scaOAuth`, `confirmation`, `creditorNameConfirmation`, `startAuthorisation`, `startAuthorisationWithPsuIdentification`, `updatePsuIdentification`, `startAuthorisationWithProprietaryData`, `updateProprietaryData`, `startAuthorisationWithPsuAuthentication`, `updatePsuAuthentication`, `startAuthorisationWithEncryptedPsuAuthentication`, `updateEncryptedPsuAuthentication`, `startAuthorisationWithAuthenticationMethodSelection`, `selectAuthenticationMethod`, `startAuthorisationWithTransactionAuthorisation`, `authoriseTransaction`, `updateResourceByDebtorAccountResource`, `self`, `status`, **`transactionfees`**, `scaStatus`, `account`, **`savingsAccount`**, **`loanAccount`**, `balances`, `transactions`, `cardAccount`, `cardTransactions`, `transactionDetails`, **`ibanCheck`**, **`paymentInitiation`**, **`securitiesAccount`**, **`positions`**, **`orders`**, **`orderDetails`**, **`relatedOrders[]`**, **`relatedTransactions[]`**, **`subscription`**, **`entryStatusRevoked[]`**, `first`, `next`, `previous`, `last`, `download`, **`confirmInitiation`**, **`aspspParameters`**, **`aspspContacts`**, **`aspspDowntimes`**, **`onboardings`**, **`readConditions`**, **`confirmConditions`** | V1: `updateAdditionalPsuAuthentication`, `updateAdditionalEncryptedPsuAuthentication`, `<*>` removed. V2: many new links added (see bold). |
| **Response headers** | `X-Request-ID` | `X-Request-ID`, **`X-Reference-API-Name`**, **`X-Reference-API-Document`**, **`X-Reference-API-Version`**, **`ASPSP-Notification-Support`** | New headers added |


---


### 🔹 Response 415 – Request Timeout

| Aspect     | V1 | V2 |
|------------|----|----|
| **Schema** | *(no schema shown)* | *(no schema shown)* |
| **Headers** | - **Location** – Location of the created resource *(string)*<br>- **X-Request-ID** – ID of the request *(string)* | - **X-Reference-API-Name** – `"Berlin Group openFinance API"` *(string)*<br>- **X-Reference-API-Document** – Implementation Guideline document name *(string)*<br>- **X-Reference-API-Version** – Version of the API *(string)*<br>- **Location** – Location of the created resource *(string)*<br>- **X-Request-ID** – ID of the request *(string)*<br>- **ASPSP-Notification-Support** – Indicates support for resource status notifications *(boolean)* |


---


### 🔹 Response 429 – Request Timeout

Method is deleted in v2.


---


### 🔹 Response 500 – Request Timeout

| Aspect     | V1 | V2 |
|------------|----|----|
| **Schema** | *(no schema shown)* | *(no schema shown)* |
| **Headers** | - **Location** – Location of the created resource *(string)*<br>- **X-Request-ID** – ID of the request *(string)* | - **X-Reference-API-Name** – `"Berlin Group openFinance API"` *(string)*<br>- **X-Reference-API-Document** – Implementation Guideline document name *(string)*<br>- **X-Reference-API-Version** – Version of the API *(string)*<br>- **Location** – Location of the created resource *(string)*<br>- **X-Request-ID** – ID of the request *(string)*<br>- **ASPSP-Notification-Support** – Indicates support for resource status notifications *(boolean)* |


---


### 🔹 Response 503 – Request Timeout

| Aspect     | V1 | V2 |
|------------|----|----|
| **Schema** | *(no schema shown)* | *(no schema shown)* |
| **Headers** | - **Location** – Location of the created resource *(string)*<br>- **X-Request-ID** – ID of the request *(string)* | - **X-Reference-API-Name** – `"Berlin Group openFinance API"` *(string)*<br>- **X-Reference-API-Document** – Implementation Guideline document name *(string)*<br>- **X-Reference-API-Version** – Version of the API *(string)*<br>- **Location** – Location of the created resource *(string)*<br>- **X-Request-ID** – ID of the request *(string)*<br>- **ASPSP-Notification-Support** – Indicates support for resource status notifications *(boolean)* |


---


## 5.2 Delete Consent - PIIS

| Aspect        | V1 (NextGenPSD2) | V2 (OpenFinance) | Change |
|---------------|------------------|------------------|--------|
| **Method**    | `DELETE /v1/consents/confirmation-of-funds/{CoF-consent-id}` | `DELETE /v2/consents/{consent-category}/{consentId}` | **Renamed** endpoint (path) |
| **Availability** | Defined in PIIS (but full spec not included here) | Present with full schema | Kept, with new structure in v2 |


---


## 6.1 Get Consent Status - AIS

**Endpoints**
- **V1:** `GET /v1/consents/{consent-id}/status`
- **V2:** `GET /v2/consents/{consent-category}/{consentId}/status`

```diff
Path params:
- consentId (still present)
+ consent-category (NEW, required) → values: account-access, funds-confirmations, user-parameters-access, rtps


Headers:

- Signature
- TPP-Signature-Certificate
- PSU-IP-Address
- PSU-IP-Port
- PSU-Accept
- PSU-Accept-Charset
- PSU-Accept-Encoding
- PSU-Accept-Language
- PSU-User-Agent
- PSU-Http-Method
- PSU-Device-ID
- PSU-Geo-Location

+ x-jws-signature   (NEW, replaces Signature)
+ Digest            (kept, but stricter RFC3230/RFC5843 definition)
```

### Response 200 – V1 vs V2 Differences

| Aspect       | V1 (consentStatusResponse-200) | V2 (Response 200) |
|--------------|--------------------------------|-------------------|
| **Message object** | `consentStatusResponse-200` | *not named, directly returned object* |
| **consentStatus*** | Enum: <br>• received<br>• rejected<br>• valid<br>• revokedByPsu<br>• expired<br>• terminatedByTpp<br>• partiallyAuthorised <br><br>Additional codes may be added by ASPSP. | Enum: <br>• received<br>• rejected<br>• partiallyAuthorised<br>• valid<br>• revokedByPsu<br>• expired<br>• terminatedByTpp<br>• replacedByTpp |
| **psuMessage** | `psuMessageTextstring` <br>maxLength: 500 | `string` <br>maxLength: 500 |
| **Headers** | • `X-Request-ID` <br>ID of the request, unique to the call, as determined by the initiating party. | • `X-Request-ID` <br>ID of the request, unique to the call, as determined by the initiating party. <br><br>**New in V2:** <br>• `X-Reference-API-Name` – e.g. "Berlin Group openFinance API" <br>• `X-Reference-API-Document` – Implementation Guideline document name <br>• `X-Reference-API-Version` – API version |


---


### Response 400 – V1 vs V2 Differences

| Aspect | V1 (Error400_NG_AIS) | V2 (error_NG_400_AIS) |
|--------|----------------------|------------------------|
| **Main object** | `Error400_NG_AIS` | `error_NG_400_AIS` |
| **Error messages array** | `tppMessages[]` <br> Each item = `tppMessage400_AIS` | `apiClientMessages[]` <br> Each item = `clientMessageInformation_400_AIS` |
| **category*** | `tppMessageCategorystring` <br>Enum: [ ERROR, WARNING ] | `string` <br>Enum: [ ERROR, WARNING ] (same meaning) |
| **code*** | `MessageCode400_AISstring` <br>Enum (BAD_REQUEST codes): <br>• FORMAT_ERROR <br>• PARAMETER_NOT_CONSISTENT <br>• PARAMETER_NOT_SUPPORTED <br>• SERVICE_INVALID <br>• RESOURCE_UNKNOWN <br>• RESOURCE_EXPIRED <br>• RESOURCE_BLOCKED <br>• TIMESTAMP_INVALID <br>• PERIOD_INVALID <br>• SCA_METHOD_UNKNOWN <br>• SCA_INVALID <br>• CONSENT_UNKNOWN <br>• SESSIONS_NOT_SUPPORTED | `MessageCode_400_AIS` <br>Split into: <br>**Service Unspecific HTTP Error Codes** (same as V1 except cleaner definition) <br>**AIS Specific HTTP Error Codes:** <br>• CONSENT_TYPE_NOT_SUPPORTED <br>• SESSIONS_NOT_SUPPORTED |
| **path** | `string` | `string` |
| **text** | `tppMessageTextstring` <br>maxLength: 500 | `Max500Textstring` <br>maxLength: 500 |
| **_links** | `_linksAll` – very large set (authorisation flows, accounts, balances, transactions, etc.) | `links` – extended set with **more resources** than V1: <br> includes `updateResourceByDebtorAccountResource`, `savingsAccount`, `loanAccount`, `ibanCheck`, `paymentInitiation`, `securitiesAccount`, `positions`, `orders`, `orderDetails`, `relatedOrders`, `relatedTransactions`, `subscription`, `entryStatusRevoked`, `aspspParameters`, `aspspContacts`, `aspspDowntimes`, `onboardings`, `readConditions`, `confirmConditions` |
| **Headers** | • `Location` <br>• `X-Request-ID` | • `Location` <br>• `X-Request-ID` <br>**New in V2:** <br>• `X-Reference-API-Name` – e.g. "Berlin Group openFinance API" <br>• `X-Reference-API-Document` – Implementation Guideline name <br>• `X-Reference-API-Version` – API version <br>• `ASPSP-Notification-Support` – boolean for notification service support |


---


### Response 401 (UNAUTHORIZED) – AIS

| Section   | V1: `Error401_NG_AIS` | V2: `error_NG_401_AIS` |
|-----------|------------------------|-------------------------|
| **Root Object** | `Error401_NG_AIS`<br/>contains: `tppMessages[]`, `_links` | `error_NG_401_AIS`<br/>contains: `apiClientMessages[]`, `_links` |
| **Message Array** | `tppMessages[]` | `apiClientMessages[]` |
| **Message Object** | **tppMessage401_AIS**<br/>- `category` (ERROR / WARNING)<br/>- `code` (AIS 401 codes)<br/>- `path`<br/>- `text` (max 500 chars) | **clientMessageInformation_401_AIS**<br/>- `category` (ERROR / WARNING)<br/>- `code` (combination of ServiceUnspecific + AISSpecific codes)<br/>- `path`<br/>- `text` (max 500 chars) |
| **Codes (Enum)** | AIS only:<br/>`CERTIFICATE_INVALID`, `ROLE_INVALID`, `CERTIFICATE_EXPIRED`, `CERTIFICATE_BLOCKED`, `CERTIFICATE_REVOKE`, `CERTIFICATE_MISSING`, `SIGNATURE_INVALID`, `SIGNATURE_MISSING`, `CORPORATE_ID_INVALID`, `PSU_CREDENTIALS_INVALID`, `CONSENT_INVALID`, `CONSENT_EXPIRED`, `TOKEN_UNKNOWN`, `TOKEN_INVALID`, `TOKEN_EXPIRED` | Extended:<br/>All AIS codes **plus**:<br/>`CERTIFICATE_REVOKED` (name changed from REVOKE), `CLIENT_INVALID`, `CLIENT_INCONSISTENT`, `API_CONTRACT_ID_INVALID`, more flexible structure (unspecific vs AIS-specific codes) |
| **Links** | `_linksAll` with many endpoints:<br/>`scaRedirect`, `scaOAuth`, `confirmation`, `startAuthorisation*`, `update*`, `creditorNameConfirmation`, `authoriseTransaction`, `self`, `status`, `scaStatus`, `account`, `balances`, `transactions`, `transactionDetails`, `cardAccount`, `cardTransactions`, navigation (`first`, `next`, `prev`, `last`), `download`, `<*>` etc. | `links` object (extended):<br/>Keeps all V1 links **plus adds new ones**:<br/>`updateResourceByDebtorAccountResource`, `transactionfees`, `savingsAccount`, `loanAccount`, `ibanCheck`, `paymentInitiation`, `securitiesAccount`, `positions`, `orders`, `orderDetails`, `relatedOrders`, `relatedTransactions`, `subscription`, `entryStatusRevoked`, `confirmInitiation`, `aspspParameters`, `aspspContacts`, `aspspDowntimes`, `onboardings`, `readConditions`, `confirmConditions` |
| **Headers** | - `Location`<br/>- `X-Request-ID` | - `X-Reference-API-Name`<br/>- `X-Reference-API-Document`<br/>- `X-Reference-API-Version`<br/>- `Location`<br/>- `X-Request-ID`<br/>- `ASPSP-Notification-Support` |


---


### 403 Error Response Comparison (AIS)

| Aspect | V1: `Error403_NG_AIS` | V2: `error_NG_403_AIS` |
|--------|-----------------------|-------------------------|
| **Message container** | `tppMessages` (array of `tppMessage403_AIS`) | `apiClientMessages` (array of `clientMessageInformation_403_AIS`) |
| **Category** | `category* : string`<br>Enum: `[ERROR, WARNING]` | `category* : string`<br>Only `[ERROR, WARNING]` permitted |
| **Code** | `code* : MessageCode403_AIS`<br>Enum: `[CONSENT_UNKNOWN, SERVICE_BLOCKED, RESOURCE_UNKNOWN, RESOURCE_EXPIRED]` | `code* : MessageCode_403_AIS`<br>Enum: `[SERVICE_BLOCKED, CONSENT_UNKNOWN, RESOURCE_UNKNOWN, RESOURCE_EXPIRED]`<br>(reference to *Service Unspecific HTTP Error Codes*) |
| **Path** | `path : string` | `path : string` |
| **Text** | `text : string (maxLength 500)` | `text : Max500Textstring (maxLength 500)` |
| **Links (`_links`)** | Wide set of links: <br>- `scaRedirect`, `scaOAuth`, `confirmation`<br>- `startAuthorisation`, `updatePsuIdentification`, `updateProprietaryData`<br>- `updatePsuAuthentication`, `updateEncryptedPsuAuthentication`, `updateAdditionalPsuAuthentication`, `updateAdditionalEncryptedPsuAuthentication`<br>- `selectAuthenticationMethod`, `authoriseTransaction`<br>- `creditorNameConfirmation`, `status`, `scaStatus`, `account`, `balances`, `transactions`, `transactionDetails`<br>- `cardAccount`, `cardTransactions`, `first`, `next`, `previous`, `last`, `download`, `self` | Much richer set of links: <br>- Includes all from V1<br>- **Plus new ones**: `updateResourceByDebtorAccountResource`, `transactionfees`, `savingsAccount`, `loanAccount`, `ibanCheck`, `paymentInitiation`, `securitiesAccount`, `positions`, `orders`, `orderDetails`, `relatedOrders`, `relatedTransactions`, `subscription`, `entryStatusRevoked`, `confirmInitiation`, `aspspParameters`, `aspspContacts`, `aspspDowntimes`, `onboardings`, `readConditions`, `confirmConditions` |
| **Headers** | - `Location: string`<br>- `X-Request-ID: string` | - `X-Reference-API-Name: string ("Berlin Group openFinance API")`<br>- `X-Reference-API-Document: string (e.g. "Extended Payment Initiation Services")`<br>- `X-Reference-API-Version: string`<br>- `Location: string`<br>- `X-Request-ID: string`<br>- `ASPSP-Notification-Support: boolean` |


---


### Error 404 (AIS) – V1 vs V2

| Category        | V1 (NextGenPSD2)                                                                                                                        | V2 (OpenFinance)                                                                                                                                                  |
|-----------------|------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Object Name** | `Error404_NG_AIS`                                                                                                                        | `error_NG_404_AIS`                                                                                                                                                |
| **Messages**    | `tppMessages` → `tppMessage404_AIS` <br> - **category***: `ERROR` / `WARNING` <br> - **code***: `RESOURCE_UNKNOWN` <br> - **path** <br> - **text** (max 500 chars) | `apiClientMessages` → `clientMessageInformation_404_AIS` <br> - **category***: `ERROR` / `WARNING` <br> - **code***: <br> • `RESOURCE_UNKNOWN` (Service Unspecific) <br> • `CONTENT_TEMPORARILY_NOT_AVAILABLE` (AIS specific) <br> - **path** <br> - **text** (max 500 chars) |
| **Links**       | `_linksAll`: <br> `scaRedirect`, `scaOAuth`, `confirmation`, `startAuthorisation`, `updatePsuIdentification`, <br> `updateProprietaryData`, `updatePsuAuthentication`, <br> `updateEncryptedPsuAuthentication`, `updateAdditionalPsuAuthentication`, `updateAdditionalEncryptedPsuAuthentication`, <br> `selectAuthenticationMethod`, `creditorNameConfirmation`, <br> `authoriseTransaction`, `self`, `status`, `scaStatus`, <br> `account`, `balances`, `transactions`, `transactionDetails`, <br> `cardAccount`, `cardTransactions`, `first`, `next`, `previous`, `last`, `download`, `<*>` | `links`: <br> `scaRedirect`, `scaOAuth`, `confirmation`, `creditorNameConfirmation`, `startAuthorisation`, <br> `updatePsuIdentification`, `updateProprietaryData`, `updatePsuAuthentication`, <br> `updateEncryptedPsuAuthentication`, `selectAuthenticationMethod`, `authoriseTransaction`, <br> `updateResourceByDebtorAccountResource`, `transactionfees`, <br> `savingsAccount`, `loanAccount`, `ibanCheck`, `paymentInitiation`, `securitiesAccount`, <br> `positions`, `orders`, `orderDetails`, `relatedOrders`, `relatedTransactions`, <br> `subscription`, `entryStatusRevoked`, `confirmInitiation`, <br> `aspspParameters`, `aspspContacts`, `aspspDowntimes`, `onboardings`, <br> `readConditions`, `confirmConditions`, `self`, `status`, `scaStatus`, <br> `account`, `balances`, `transactions`, `transactionDetails`, <br> `cardAccount`, `cardTransactions`, `first`, `next`, `previous`, `last`, `download` |
| **Headers**     | - `Location` <br> - `X-Request-ID`                                                                                                       | - `X-Reference-API-Name` <br> - `X-Reference-API-Document` <br> - `X-Reference-API-Version` <br> - `Location` <br> - `X-Request-ID` <br> - `ASPSP-Notification-Support` |


---


### Error 405 – AIS

| Aspect     | V1                                                                                                                                | V2                                                                                                                                                                                                                                                                                       |
|------------|-----------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Object** | `Error405_NG_AIS`                                                                                                                 | `error_NG_405_AIS`                                                                                                                                                                                                                                                                        |
| **Messages** | **`tppMessages`** → array of `tppMessage405_AIS`  <br>• `category` → `ERROR` \| `WARNING`  <br>• `code` → `SERVICE_INVALID`  <br>• `path` → string  <br>• `text` → max 500 chars | **`apiClientMessages`** → array of `clientMessageInformation_405_AIS`  <br>• `category` → `ERROR` \| `WARNING`  <br>• `code` → `MessageCode_405_AIS`  <br>&nbsp;&nbsp;– `SERVICE_INVALID` (service-unspecific)  <br>• `path` → string  <br>• `text` → max 500 chars |
| **Links**  | `_linksAll` → many link types:  <br>• `scaRedirect`, `scaOAuth`, `confirmation`, `creditorNameConfirmation`, `startAuthorisation`, … <br>• `self`, `status`, `scaStatus`, `account`, `balances`, `transactions`, `transactionDetails`, `cardAccount`, `cardTransactions`, … <br>• `first`, `next`, `previous`, `last`, `download` | `links` → extended set:  <br>• сите од V1 плус нови: `updateResourceByDebtorAccountResource`, `transactionfees`, `savingsAccount`, `loanAccount`, `ibanCheck`, `paymentInitiation`, `securitiesAccount`, `positions`, `orders`, `orderDetails`, `relatedOrders`, `relatedTransactions`, `subscription`, `entryStatusRevoked`, `confirmInitiation`, `aspspParameters`, `aspspContacts`, `aspspDowntimes`, `onboardings`, `readConditions`, `confirmConditions` |
| **Headers** | • `Location` → string <br>• `X-Request-ID` → string                                                                             | • `X-Reference-API-Name` → `"Berlin Group openFinance API"` <br>• `X-Reference-API-Document` → IG doc name <br>• `X-Reference-API-Version` → string <br>• `Location` → string <br>• `X-Request-ID` → string <br>• `ASPSP-Notification-Support` → boolean |


---


### 🔹 Response 406 – Not Acceptable

*(no schema shown in v2)*

| Aspect     | V1 | V2 |
|------------|----|----|
| **Schema** | *(schema shown)* | *(no schema shown)* |
| **Headers** | - **Location** – Location of the created resource *(string)*<br>- **X-Request-ID** – ID of the request *(string)* | - **X-Reference-API-Name** – `"Berlin Group openFinance API"` *(string)*<br>- **X-Reference-API-Document** – Implementation Guideline document name *(string)*<br>- **X-Reference-API-Version** – Version of the API *(string)*<br>- **Location** – Location of the created resource *(string)*<br>- **X-Request-ID** – ID of the request *(string)*<br>- **ASPSP-Notification-Support** – Indicates support for resource status notifications *(boolean)* |


---


### Response 408

| Aspect     | V1 | V2 |
|------------|----|----|
| **Schema** | *(no schema shown)* | *(no schema shown)* |
| **Headers** | - **Location** – Location of the created resource *(string)*<br>- **X-Request-ID** – ID of the request *(string)* | - **X-Reference-API-Name** – `"Berlin Group openFinance API"` *(string)*<br>- **X-Reference-API-Document** – Implementation Guideline document name *(string)*<br>- **X-Reference-API-Version** – Version of the API *(string)*<br>- **Location** – Location of the created resource *(string)*<br>- **X-Request-ID** – ID of the request *(string)*<br>- **ASPSP-Notification-Support** – Indicates support for resource status notifications *(boolean)* |


--- 


### Error 409 – AIS

| Aspect       | V1                                                                                                                                | V2                                                                                                                                                                                                                                                                                       |
|--------------|-----------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Object**   | `Error409_NG_AIS`                                                                                                                 | `error_NG_409_AIS`                                                                                                                                                                                                                                                                       |
| **Messages** | **`tppMessages`** → array of `tppMessage409_AIS`  <br>• `category` → `ERROR` \| `WARNING`  <br>• `code` → `STATUS_INVALID`  <br>• `path` → string  <br>• `text` → max 500 chars | **`apiClientMessages`** → array of `clientMessageInformation_409_AIS`  <br>• `category` → `ERROR` \| `WARNING`  <br>• `code` → `MessageCode_409_AIS` (→ service-unspecific, e.g. `STATUS_INVALID`)  <br>• `path` → string  <br>• `text` → max 500 chars |
| **Links**    | `_linksAll` → many link types:  <br>• `scaRedirect`, `scaOAuth`, `confirmation`, `startAuthorisation`, `creditorNameConfirmation`, … <br>• `self`, `status`, `scaStatus`, `account`, `balances`, `transactions`, `transactionDetails`, `cardAccount`, `cardTransactions`, … <br>• `first`, `next`, `previous`, `last`, `download` | `links` → extended set:  <br>• сите од V1 плус нови: `updateResourceByDebtorAccountResource`, `transactionfees`, `savingsAccount`, `loanAccount`, `ibanCheck`, `paymentInitiation`, `securitiesAccount`, `positions`, `orders`, `orderDetails`, `relatedOrders`, `relatedTransactions`, `subscription`, `entryStatusRevoked`, `confirmInitiation`, `aspspParameters`, `aspspContacts`, `aspspDowntimes`, `onboardings`, `readConditions`, `confirmConditions` |
| **Headers**  | • `Location` → string <br>• `X-Request-ID` → string                                                                             | • `X-Reference-API-Name` → `"Berlin Group openFinance API"` <br>• `X-Reference-API-Document` → IG doc name <br>• `X-Reference-API-Version` → string <br>• `Location` → string <br>• `X-Request-ID` → string <br>• `ASPSP-Notification-Support` → boolean |


---


### 🔹 Response 415 – Request Timeout

| Aspect     | V1 | V2 |
|------------|----|----|
| **Schema** | *(no schema shown)* | *(no schema shown)* |
| **Headers** | - **Location** – Location of the created resource *(string)*<br>- **X-Request-ID** – ID of the request *(string)* | - **X-Reference-API-Name** – `"Berlin Group openFinance API"` *(string)*<br>- **X-Reference-API-Document** – Implementation Guideline document name *(string)*<br>- **X-Reference-API-Version** – Version of the API *(string)*<br>- **Location** – Location of the created resource *(string)*<br>- **X-Request-ID** – ID of the request *(string)*<br>- **ASPSP-Notification-Support** – Indicates support for resource status notifications *(boolean)* |


---


### 🔹 Response 429 – Request Timeout

Method is deleted in v2.


---

## 6.2 Get Consent Status - PIIS

| Aspect        | V1 (NextGenPSD2) | V2 (OpenFinance) | Change |
|---------------|------------------|------------------|--------|
| **Method**    | `GET /v1/consents/confirmation-of-funds/{CoF-consent-id}/status` | `GET /v2/consents/{consent-category}/{consentId}/status` | **Renamed** endpoint (path) |
| **Availability** | Defined in PIIS (but full spec not included here) | Present with full schema | Kept, with new structure in v2 |


---


## 7. Create AIS Consent — differences V1 → V2

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

### 🔹 Response 201 Differences – V1 vs V2 (POST /consents)

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

### 🔹 Response 400 – V1 vs V2 Differences

| Section | V1 | V2 | Change |
|---------|----|----|--------|
| **Error message container** | `tppMessages[]` (type: `tppMessage400_AIS`) | `apiClientMessages[]` (type: `clientMessageInformation_401_AIS`) | **Renamed** container and structure. |
| **code** | `MessageCode400_AIS` enum:<br>`FORMAT_ERROR`, `PARAMETER_NOT_CONSISTENT`, `PARAMETER_NOT_SUPPORTED`, `SERVICE_INVALID`, `RESOURCE_UNKNOWN`, `RESOURCE_EXPIRED`, `RESOURCE_BLOCKED`, `TIMESTAMP_INVALID`, `PERIOD_INVALID`, `SCA_METHOD_UNKNOWN`, `SCA_INVALID`, `CONSENT_UNKNOWN`, `SESSIONS_NOT_SUPPORTED` | Split into:<br>**Service Unspecific 400** → `FORMAT_ERROR`, `PARAMETER_NOT_CONSISTENT`, `PARAMETER_NOT_SUPPORTED`, `SERVICE_INVALID`, `CONSENT_UNKNOWN`, `RESOURCE_UNKNOWN`, `RESOURCE_EXPIRED`, `RESOURCE_BLOCKED`, `TIMESTAMP_INVALID`, `PERIOD_INVALID`, `SCA_METHOD_UNKNOWN`, `SCA_INVALID`<br>**AIS Specific 400** → `CONSENT_TYPE_NOT_SUPPORTED`, `SESSIONS_NOT_SUPPORTED` | Repartitioned & extended |
| **path** | `string` | `string` | Unchanged |
| **text** | `text` (string, max 500 chars) | `text` (type: `Max500Text` from Data Dictionary) | **Type refactor**; semantic same. |
|**_links**| `scaRedirect`, `scaOAuth`, `confirmation`, `startAuthorisation`, `startAuthorisationWithPsuIdentification`, `updatePsuIdentification`, `startAuthorisationWithProprietaryData`, `updateProprietaryData`, `startAuthorisationWithPsuAuthentication`, `updatePsuAuthentication`, `startAuthorisationWithEncryptedPsuAuthentication`, `updateEncryptedPsuAuthentication`, `updateAdditionalPsuAuthentication`, `updateAdditionalEncryptedPsuAuthentication`, `startAuthorisationWithAuthenticationMethodSelection`, `selectAuthenticationMethod`, `creditorNameConfirmation`, `startAuthorisationWithTransactionAuthorisation`, `authoriseTransaction`, `self`, `status`, `scaStatus`, `account`, `balances`, `transactions`, `transactionDetails`, `cardAccount`, `cardTransactions`, `first`, `next`, `previous`, `last`, `download`, `<*>` | `scaRedirect`, `scaOAuth`, `confirmation`, `startAuthorisation`, `startAuthorisationWithPsuIdentification`, `updatePsuIdentification`, `startAuthorisationWithProprietaryData`, `updateProprietaryData`, `startAuthorisationWithPsuAuthentication`, `updatePsuAuthentication`, `startAuthorisationWithEncryptedPsuAuthentication`, `updateEncryptedPsuAuthentication`, `startAuthorisationWithAuthenticationMethodSelection`, `selectAuthenticationMethod`, `startAuthorisationWithTransactionAuthorisation`, `authoriseTransaction`, `updateResourceByDebtorAccountResource`, `self`, `status`, `transactionfees`, `scaStatus`, `account`, `savingsAccount`, `loanAccount`, `balances`, `transactions`, `cardAccount`, `cardTransactions`, `transactionDetails`, `ibanCheck`, `paymentInitiation`, `securitiesAccount`, `positions`, `orders`, `orderDetails`, `relatedOrders[]`, `relatedTransactions[]`, `subscription`, `entryStatusRevoked[]`, `first`, `next`, `previous`, `last`, `download`, `confirmInitiation`, `aspspParameters`, `aspspContacts`, `aspspDowntimes`, `onboardings`, `readConditions`, `confirmConditions` | **Expanded set** (many new links added; `creditorNameConfirmation` removed) |
| **Removed fields** | `creditorNameConfirmation` link in V1 | *(not present in V2)* | **Removed**. |
| Headers | `Location`, `X-Request-ID` | Same + `X-Reference-API-Name`, `X-Reference-API-Document`, `X-Reference-API-Version`, `ASPSP-Notification-Support` | New headers |


---


### 🔹 Response 401 – V1 vs V2 Differences

| Section | v1 | v2 | diff |
|---------|----|----|----------|
| **Array name** | `tppMessages` | `apiClientMessages` | Name changed |
| **Item type** | `tppMessage401_AIS` | `clientMessageInformation_401_AIS` | Changed structure |
| **category** | Enum: `ERROR`, `WARNING` | Enum: `ERROR`, `WARNING` | Same |
| **code** | `MessageCode401_AIS` <br> Enum: `CERTIFICATE_INVALID`, `ROLE_INVALID`, `CERTIFICATE_EXPIRED`, `CERTIFICATE_BLOCKED`, `CERTIFICATE_REVOKE`, `CERTIFICATE_MISSING`, `SIGNATURE_INVALID`, `SIGNATURE_MISSING`, `CORPORATE_ID_INVALID`, `PSU_CREDENTIALS_INVALID`, `CONSENT_INVALID`, `CONSENT_EXPIRED`, `TOKEN_UNKNOWN`, `TOKEN_INVALID`, `TOKEN_EXPIRED` | `MessageCode_401_AIS` <br> (Service Unspecific + AIS Specific) <br> Enum: `CERTIFICATE_INVALID`, `ROLE_INVALID`, `CERTIFICATE_EXPIRED`, `CERTIFICATE_BLOCKED`, `CERTIFICATE_REVOKED`, `CERTIFICATE_MISSING`, `SIGNATURE_INVALID`, `SIGNATURE_MISSING`, `CORPORATE_ID_INVALID`, `PSU_CREDENTIALS_INVALID`, `CONSENT_INVALID`, `CONSENT_EXPIRED`, `TOKEN_UNKNOWN`, `TOKEN_INVALID`, `TOKEN_EXPIRED`, **`CLIENT_INVALID`**, **`CLIENT_INCONSISTENT`**, **`API_CONTRACT_ID_INVALID`** | New values added in v2 and change `CERTIFICATE_REVOKED` |
| **path** | string | string | Same |
| **text** | `tppMessageTextstring` (max 500) | `Max500Textstring` (max 500) | Same (only name changed) |
|**_links**|`scaRedirect`, `scaOAuth`, `confirmation`, `startAuthorisation`, `startAuthorisationWithPsuIdentification`, `updatePsuIdentification`, `startAuthorisationWithProprietaryData`, `updateProprietaryData`, `startAuthorisationWithPsuAuthentication`, `updatePsuAuthentication`, `startAuthorisationWithEncryptedPsuAuthentication`, `updateEncryptedPsuAuthentication`, `updateAdditionalPsuAuthentication`, `updateAdditionalEncryptedPsuAuthentication`, `startAuthorisationWithAuthenticationMethodSelection`, `selectAuthenticationMethod`, `creditorNameConfirmation`, `startAuthorisationWithTransactionAuthorisation`, `authoriseTransaction`, `self`, `status`, `scaStatus`, `account`, `balances`, `transactions`, `transactionDetails`, `cardAccount`, `cardTransactions`, `first`, `next`, `previous`, `last`, `download`, `< * >` | `scaRedirect`, `scaOAuth`, `confirmation`, `startAuthorisation`, `startAuthorisationWithPsuIdentification`, `updatePsuIdentification`, `startAuthorisationWithProprietaryData`, `updateProprietaryData`, `startAuthorisationWithPsuAuthentication`, `updatePsuAuthentication`, `startAuthorisationWithEncryptedPsuAuthentication`, `updateEncryptedPsuAuthentication`, `startAuthorisationWithAuthenticationMethodSelection`, `selectAuthenticationMethod`, `startAuthorisationWithTransactionAuthorisation`, `authoriseTransaction`, **`updateResourceByDebtorAccountResource`**, `self`, `status`, **`transactionfees`**, `scaStatus`, `account`, **`savingsAccount`**, **`loanAccount`**, `balances`, `transactions`, `cardAccount`, `cardTransactions`, `transactionDetails`, **`ibanCheck`**, **`paymentInitiation`**, **`securitiesAccount`**, **`positions`**, **`orders`**, **`orderDetails`**, **`relatedOrders`**, **`relatedTransactions`**, **`subscription`**, **`entryStatusRevoked`**, `first`, `next`, `previous`, `last`, `download`, **`confirmInitiation`**, **`aspspParameters`**, **`aspspContacts`**, **`aspspDowntimes`**, **`onboardings`**, **`readConditions`**, **`confirmConditions`** |
| **Removed fields** | `creditorNameConfirmation` link in V1 | *(not present in V2)* | **Removed**. |
| **Headers**, | `Location`, `X-Request-ID` | Same + `X-Reference-API-Name`, `X-Reference-API-Document`, `X-Reference-API-Version`, `ASPSP-Notification-Support` | New headers |


---


### 🔹 Response 403 – V1 vs V2 Differences

| Aspect                | **V1** (`Error403_NG_AIS`) | **V2** (`error_NG_403_AIS`) |
|------------------------|-----------------------------|------------------------------|
| **Message container**  | `tppMessages[]`            | `apiClientMessages[]`        |
| **Message object**     | `tppMessage403_AIS`        | `clientMessageInformation_403_AIS` |
| **Category**           | `tppMessageCategorystring`<br>Enum: `[ERROR, WARNING]` | `category* string`<br>Only `"ERROR"` or `"WARNING"` permitted |
| **Code**               | `MessageCode403_AISstring`<br>Enum: `[CONSENT_UNKNOWN, SERVICE_BLOCKED, RESOURCE_UNKNOWN, RESOURCE_EXPIRED]` | `MessageCode_403_AIS->MessageCode_ServiceUnspecific_403`<br>Enum: `[SERVICE_BLOCKED, CONSENT_UNKNOWN, RESOURCE_UNKNOWN, RESOURCE_EXPIRED]`  |
| **Text**               | `tppMessageTextstring`<br>maxLength: 500 | `Max500Text[...]` |
| **Path**               | `string`                   | `string` |
| **Links** | `scaRedirect`, `scaOAuth`, `confirmation`, `startAuthorisation`, `updatePsuIdentification`, `updateProprietaryData`, `updatePsuAuthentication`, `updateEncryptedPsuAuthentication`, `updateAdditionalPsuAuthentication`, `updateAdditionalEncryptedPsuAuthentication`, `selectAuthenticationMethod`, `authoriseTransaction`, etc. | `updateResourceByDebtorAccountResource`, `transactionfees`, `savingsAccount`, `loanAccount`, `ibanCheck`, `paymentInitiation`, `securitiesAccount`, `positions`, `orders`, `orderDetails`, `relatedOrders[]`, `relatedTransactions[]`, `subscription`, `entryStatusRevoked[]`, `confirmInitiation`, `aspspParameters`, `aspspContacts`, `aspspDowntimes`, `onboardings`, `readConditions`, `confirmConditions`  **extended** list|
| **Headers**, | `Location`, `X-Request-ID` | Same + `X-Reference-API-Name`, `X-Reference-API-Document`, `X-Reference-API-Version`, `ASPSP-Notification-Support` | New headers |


---


### 🔹 Response 404 – V1 vs V2 Differences

#### Root / Messages

| Aspect | **v1** | **v2** | Difference |
|---|---|---|---|
| **Messages array** | `tppMessages[]` | `apiClientMessages[]` | Renamed |
| **Message item type** | `tppMessage404_AIS` | `clientMessageInformation_404_AIS` | Renamed |
| **category** | `tppMessageCategorystring` (Enum: `ERROR`, `WARNING`) | `string` (Only `ERROR` or `WARNING`) | Same values |
| **code** | `MessageCode404_AISstring` (Enum: `RESOURCE_UNKNOWN`) | `MessageCode_404_AIS` = anyOf: <br>• **Service Unspecific**: `RESOURCE_UNKNOWN` <br>• **AIS Specific**: `CONTENT_TEMPORARILY_NOT_AVAILABLE` | New AIS-specific code in v2 |
| **path** | `string` | `string` | — |
| **text** | `tppMessageTextstring` (max 500) | `Max500Textstring` (max 500) | Type alias renamed |
| **links** | `scaRedirect`, `scaOAuth`, `confirmation`, `startAuthorisation`, `startAuthorisationWithPsuIdentification`, `updatePsuIdentification`, `startAuthorisationWithProprietaryData`, `updateProprietaryData`, `startAuthorisationWithPsuAuthentication`, `updatePsuAuthentication`, `startAuthorisationWithEncryptedPsuAuthentication`, `updateEncryptedPsuAuthentication`, `startAuthorisationWithAuthenticationMethodSelection`, `selectAuthenticationMethod`, `startAuthorisationWithTransactionAuthorisation`, `authoriseTransaction`, `account`, `balances`, `transactions`, `transactionDetails`, `cardAccount`, `cardTransactions`, `self`, `status`, `scaStatus, `first`, `next`, `previous`, `last`, `download`,  `creditorNameConfirmation`, `< * >` | Same base + **`savingsAccount`**, **`loanAccount`**, **`paymentInitiation`**, **`ibanCheck`**, **`transactionfees`**, **`securitiesAccount`**, **`positions`**, **`orders`**, **`orderDetails`**, **`relatedOrders[]`**, **`relatedTransactions[]`**, **`subscription`**, **`entryStatusRevoked[]`**, **`aspspParameters`**, **`aspspContacts`**, **`aspspDowntimes`**, **`onboardings`**, **`readConditions`**, **`confirmConditions`**, **`updateResourceByDebtorAccountResource`** and `creditorNameConfirmation` is removed|
| **Headers**, | `Location`, `X-Request-ID` | Same + `X-Reference-API-Name`, `X-Reference-API-Document`, `X-Reference-API-Version`, `ASPSP-Notification-Support` | New headers |


---


### 🔹 Response 405 – V1 vs V2 Differences

| Aspect / Section | **v1** | **v2** | Difference |
|---|---|---|---|
| **Messages array** | `tppMessages[]` | `apiClientMessages[]` | Renamed |
| **Message item type** | `tppMessage405_AIS` | `clientMessageInformation_405_AIS` | Renamed |
| **category** | `tppMessageCategorystring` (Enum: `ERROR`, `WARNING`) | `string` (Only `ERROR` or `WARNING`) | Same values, wording tightened |
| **code** | `MessageCode405_AIS` (Enum: `SERVICE_INVALID`) | `MessageCode_405_AIS` **anyOf** → Service-Unspecific: `SERVICE_INVALID` | Same code set; structure switched to anyOf |
| **path** | `string` | `string` | — |
| **text** | `tppMessageTextstring` (max 500) | `Max500Textstring` (max 500) | Type alias renamed |
| **_links** — Core Authorisation / SCA | `scaRedirect`, `scaOAuth`, `confirmation`, `startAuthorisation`, `startAuthorisationWithPsuIdentification`, `updatePsuIdentification`, `startAuthorisationWithProprietaryData`, `updateProprietaryData`, `startAuthorisationWithPsuAuthentication`, `updatePsuAuthentication`, `startAuthorisationWithEncryptedPsuAuthentication`, `updateEncryptedPsuAuthentication`, `startAuthorisationWithAuthenticationMethodSelection`, `selectAuthenticationMethod`, `startAuthorisationWithTransactionAuthorisation`, `authoriseTransaction` | Same set | — |
| **_links** — Accounts (AIS) | `account`, `balances`, `transactions`, `transactionDetails`, `cardAccount`, `cardTransactions` | Same + **`savingsAccount`**, **`loanAccount`** | New in v2 |
| **_links** — Payments / Misc | — | **`paymentInitiation`**, **`ibanCheck`**, **`transactionfees`** | New in v2 |
| **_links** — Securities | — | **`securitiesAccount`**, **`positions`**, **`orders`**, **`orderDetails`**, **`relatedOrders[]`**, **`relatedTransactions[]`** | New in v2 |
| **_links** — Subscriptions / Status | `self`, `status`, `scaStatus` | Same + **`subscription`**, **`entryStatusRevoked[]`** | New in v2 |
| **_links** — Paging / Download | `first`, `next`, `previous`, `last`, `download` | Same | — |
| **_links** — ASPSP Ops | — | **`aspspParameters`**, **`aspspContacts`**, **`aspspDowntimes`**, **`onboardings`**, **`readConditions`**, **`confirmConditions`**, **`confirmInitiation`**, **`updateResourceByDebtorAccountResource`** | New in v2 |
| **_links** — Other | `creditorNameConfirmation`, `< * >` | — | `creditorNameConfirmation` not listed in v2 |
| **Headers** | `Location`, `X-Request-ID` | Same + **`X-Reference-API-Name`**, **`X-Reference-API-Document`**, **`X-Reference-API-Version`**, **`ASPSP-Notification-Support`** | New headers introduced in v2 |

---

### 🔹 Response 406 – Not Acceptable

*(no schema shown in v2)*

| Aspect     | V1 | V2 |
|------------|----|----|
| **Schema** | *(schema shown)* | *(no schema shown)* |
| **Headers** | - **Location** – Location of the created resource *(string)*<br>- **X-Request-ID** – ID of the request *(string)* | - **X-Reference-API-Name** – `"Berlin Group openFinance API"` *(string)*<br>- **X-Reference-API-Document** – Implementation Guideline document name *(string)*<br>- **X-Reference-API-Version** – Version of the API *(string)*<br>- **Location** – Location of the created resource *(string)*<br>- **X-Request-ID** – ID of the request *(string)*<br>- **ASPSP-Notification-Support** – Indicates support for resource status notifications *(boolean)* |


---


### 🔹 Response 408 – Request Timeout

| Aspect     | V1 | V2 |
|------------|----|----|
| **Schema** | *(no schema shown)* | *(no schema shown)* |
| **Headers** | - **Location** – Location of the created resource *(string)*<br>- **X-Request-ID** – ID of the request *(string)* | - **X-Reference-API-Name** – `"Berlin Group openFinance API"` *(string)*<br>- **X-Reference-API-Document** – Implementation Guideline document name *(string)*<br>- **X-Reference-API-Version** – Version of the API *(string)*<br>- **Location** – Location of the created resource *(string)*<br>- **X-Request-ID** – ID of the request *(string)*<br>- **ASPSP-Notification-Support** – Indicates support for resource status notifications *(boolean)* |


--- 


### 🔹 Response 409 – Request Timeout

| Section       | v1                                                                                                                        | v2                                                                                                                        |
|---------------|----------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------|
| **Message Array** | `tppMessages` → array of `tppMessage409_AIS`                                                                            | `apiClientMessages` → array of `clientMessageInformation_409_AIS`                                                           |
| **Message Category** | `category` (Enum: `ERROR`, `WARNING`)                                                                                 | `category` (Enum: `ERROR`, `WARNING`)                                                                                      |
| **Message Code** | `code` = `MessageCode409_AISstring` (Enum: `[STATUS_INVALID]`)                                                           | `code` = `MessageCode_409_AIS` (anyOf → `MessageCode_ServiceUnspecific_409`, e.g. `STATUS_INVALID`)                         |
| **Path**        | `path` (string)                                                                                                          | `path` (string)                                                                                                            |
| **Text**        | `text` (string, maxLength 500, example text)                                                                             | `text` (Max500Textstring, maxLength 500, example text)                                                                     |
| **Links**       | `_links` (`_linksAll`) with many link types:<br> `scaRedirect`, `scaOAuth`, `confirmation`, `startAuthorisation`, `updatePsuIdentification`, `updateProprietaryData`, `updatePsuAuthentication`, `updateEncryptedPsuAuthentication`, `updateAdditionalPsuAuthentication`, `updateAdditionalEncryptedPsuAuthentication`, `selectAuthenticationMethod`, `creditorNameConfirmation`, `authoriseTransaction`, `self`, `status`, `scaStatus`, `account`, `balances`, `transactions`, `transactionDetails`, `cardAccount`, `cardTransactions`, `first`, `next`, `previous`, `last`, `download`, `<*>` | `_links` (`links`) with expanded list:<br> `scaRedirect`, `scaOAuth`, `confirmation`, `startAuthorisation`, `updatePsuIdentification`, `updateProprietaryData`, `updatePsuAuthentication`, `updateEncryptedPsuAuthentication`, `selectAuthenticationMethod`, `authoriseTransaction`, `updateResourceByDebtorAccountResource`, `self`, `status`, `transactionfees`, `scaStatus`, `account`, `savingsAccount`, `loanAccount`, `balances`, `transactions`, `cardAccount`, `cardTransactions`, `transactionDetails`, `ibanCheck`, `paymentInitiation`, `securitiesAccount`, `positions`, `orders`, `orderDetails`, `relatedOrders`, `relatedTransactions`, `subscription`, `entryStatusRevoked`, `first`, `next`, `previous`, `last`, `download`, `confirmInitiation`, `aspspParameters`, `aspspContacts`, `aspspDowntimes`, `onboardings`, `readConditions`, `confirmConditions` |
| **Headers** | - **Location** – Location of the created resource *(string)*<br>- **X-Request-ID** – ID of the request *(string)* | - **X-Reference-API-Name** – `"Berlin Group openFinance API"` *(string)*<br>- **X-Reference-API-Document** – Implementation Guideline document name *(string)*<br>- **X-Reference-API-Version** – Version of the API *(string)*<br>- **Location** – Location of the created resource *(string)*<br>- **X-Request-ID** – ID of the request *(string)*<br>- **ASPSP-Notification-Support** – Indicates support for resource status notifications *(boolean)* |                                          |
| **Example**     | `[{ "category": "ERROR", "code": "STATUS_INVALID", "text": "additional text information..." }]`                          | `{ "category": "ERROR", "code": "STATUS_INVALID", "text": "Text, maximum of 500 characters" }`                              |


### 🔹 Response 415 – Request Timeout

| Aspect     | V1 | V2 |
|------------|----|----|
| **Schema** | *(no schema shown)* | *(no schema shown)* |
| **Headers** | - **Location** – Location of the created resource *(string)*<br>- **X-Request-ID** – ID of the request *(string)* | - **X-Reference-API-Name** – `"Berlin Group openFinance API"` *(string)*<br>- **X-Reference-API-Document** – Implementation Guideline document name *(string)*<br>- **X-Reference-API-Version** – Version of the API *(string)*<br>- **Location** – Location of the created resource *(string)*<br>- **X-Request-ID** – ID of the request *(string)*<br>- **ASPSP-Notification-Support** – Indicates support for resource status notifications *(boolean)* |


---


### 🔹 Response 429 – Request Timeout

Method is deleted in v2.


---


### 🔹 Response 500 – Request Timeout

| Aspect     | V1 | V2 |
|------------|----|----|
| **Schema** | *(no schema shown)* | *(no schema shown)* |
| **Headers** | - **Location** – Location of the created resource *(string)*<br>- **X-Request-ID** – ID of the request *(string)* | - **X-Reference-API-Name** – `"Berlin Group openFinance API"` *(string)*<br>- **X-Reference-API-Document** – Implementation Guideline document name *(string)*<br>- **X-Reference-API-Version** – Version of the API *(string)*<br>- **Location** – Location of the created resource *(string)*<br>- **X-Request-ID** – ID of the request *(string)*<br>- **ASPSP-Notification-Support** – Indicates support for resource status notifications *(boolean)* |


---


### 🔹 Response 503 – Request Timeout

| Aspect     | V1 | V2 |
|------------|----|----|
| **Schema** | *(no schema shown)* | *(no schema shown)* |
| **Headers** | - **Location** – Location of the created resource *(string)*<br>- **X-Request-ID** – ID of the request *(string)* | - **X-Reference-API-Name** – `"Berlin Group openFinance API"` *(string)*<br>- **X-Reference-API-Document** – Implementation Guideline document name *(string)*<br>- **X-Reference-API-Version** – Version of the API *(string)*<br>- **Location** – Location of the created resource *(string)*<br>- **X-Request-ID** – ID of the request *(string)*<br>- **ASPSP-Notification-Support** – Indicates support for resource status notifications *(boolean)* |


---


## 2. PIIS – Consent for Funds Confirmation
| Aspect        | V1 (NextGenPSD2) | V2 (OpenFinance) | Change |
|---------------|------------------|------------------|--------|
| **Method**    | `POST /v1/consents/confirmation-of-funds` | `POST /v2/consents/funds-confirmations` | **Renamed** endpoint (path + resource name) |
| **Availability** | Defined in PIIS (but full spec not included here) | Present with full schema | Kept, with new structure in v2 |
