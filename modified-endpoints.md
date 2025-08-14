# Modifications

This file contains detailed changes for each modified endpoint. Where applicable, shared changes are referenced from [`common.md`](common.md).

## 🗂️ Modified Endpoints Overview

| API | Method | Endpoint V2 | Changes | Details |
|-----|--------|-------------|---------|---------|
| PIS | POST | `/v2/payments/{payment-product}` | 📬 Path, 🧩 Header, 🧾 Request, 🟦 Response | [See details](#v2-payments-payment-product) |
| PIS | GET | `v2/{payment-service}/{payment-product}/{payment-id}/status` | 🧩 Header, 🟦 Response | [See details](#v2-payment-service-payment-product-payment-id-status) |
| PIS | GET | `v2/{payment-service}/{payment-product}/{payment-id}` | 📬 Path, 🧩 Header, 🟦 Response | [See details](#v2-payment-service-payment-product-payment-id) |
| PIS | DELETE | `v1/{payment-service}/{payment-product}/{payment-id}` | 🧩 Header, 🟦 Response | [See details](#v1-payment-service-payment-product-payment-id) |
| PIS | GET | `/v2/bulk-payments/{payment-product}/{paymentId}/extended-status` | 🧩 Header, 🟦 Response | [See details](#v2-bulk-payments-payment-product-paymentid-extended-status) |
| PIIS | POST | `v1/funds-confirmations` | 🧩 Header, 🧾 Request, 🟦 Response | [See details](#v1-funds-confirmations) |
| AIS | GET | `v2/accounts` | 🧩 Header, 🟦 Response | [See details](#v2-accounts) |
| AIS | GET | `v2/accounts/{account-id}` | 🧩 Header, 🟦 Response | [See details](#v2-accounts-account-id) |
| AIS | GET | `v2/accounts/{account-id}/balances` | 🧩 Header, 🟦 Response | [See details](#v2-accounts-account-id-balances) |
| AIS | GET | `v2/accounts/{account-id}/transactions` | 🧩 Header, 🟦 Response | [See details](#v2-accounts-account-id-transactions) |
| AIS | GET | `v2/accounts/{account-id}/transactions/{transaction-id}` | 🧩 Header, 🟦 Response | [See details](#v2-accounts-account-id-transactions-transaction-id) |
| AIS | GET | `/v2/card-accounts` | 🧩 Header, 🟦 Response | [See details](#v2-card-accounts) |
| AIS | GET | `/v2/card-accounts/{account-id}` | 🧩 Header, 🟦 Response | [See details](#v2-card-accounts-account-id) |
| AIS | GET | `/v2/card-accounts/{account-id}/balances` | 🧩 Header, 🟦 Response | [See details](#v2-card-accounts-account-id-balances) |
| AIS | GET | `/v2/card-accounts/{account-id}/transactions` | 🧩 Header, 🟦 Response, 🔎 Query | [See details](#v2-card-accounts-account-id-transactions) |

---

### v2-payments-payment-product
**Group**: PIS  
**Method**: `POST`  
**Endpoint v2**: `/v2/payments/{payment-product}`
**Endpoint v1**: `/v1/{payment-service}/{payment-product}`

#### 📬 Path
Path changed from /v1/{payment-service}/{payment-product} to /v2/payments/{payment-product}

#### 🧩 Header
[See - AIS/PIS/PIIS and PIS Part](https://github.com/TamJ38/Berlin-Group-NextGenPSD2-vs-Open-Finance-v2-Endpoint-Schema-Comparison/blob/dev/common.md#-headers)  from the file: common.md;
- **Additional changes are:**
- `Body-Sig-Profile`, `Body-Enc-Profile`, `Body-Enc-List`, `Content-Type` - added;
- `TPP-Redirect-URI` ->`Client-Redirect-URI`,
- `TPP-Nok-Redirect-URI`->`Client-Nok-Redirect-URI`,
- `TPP-Explicit-Authorisation-Preferred`-> `Client-Explicit-Authorisation-Preferred( async authorization)`,
- `TPP-Notification-UR`I->`Client-Notification-URI`,
- `TPP-Notification-Content-Preferred`->`Client-Notification-Content-Preferred`
- `TPP-Brand-Logging-Information`->`Client-Brand-Logging-Information`

#### 🧾 Request
- **v1 request-body**: 	
- `paymentInitiation_json{...}`
- `periodicPaymentInitiation_json{...}`
- `bulkPaymentInitiation_json{...}`
- changed to
- **v2 request body** :	
- `SinglePayment_SCT_Core{...}`
- `SinglePayment_SCT_inst_Core{...}`
- `SinglePayment_Target2_Core{...}`
- `SinglePayment_Cross_Border_CT_Core{...}`

#### 🟦 Response
- **201** - response
  - [See here - Name changed part](https://github.com/TamJ38/Berlin-Group-NextGenPSD2-vs-Open-Finance-v2-Endpoint-Schema-Comparison/blob/dev/common.md#-name-changed); 
  - scaMethods.name and chosenScaMethod.name are mandatory
  - apiClientMessages->code(enum) is divided into more parts and there are additional enum values for each of them:
  ```
  oneOf ->	
  messageCode_ServiceUnspecific{...}
  messageCode_PisSpecific{...}
  messageCode_AisSpecific{...}
  messageCode_PiisSpecific{...}
  messageCode_SigningBasketSpecific{...}
  messageCode_PushAisSpecific{...}
  };
  ```
  - added enum value for transactionStatus:PRES
  - added _links value:encryptionCertificates;
- **400,401, 403, 404,405, 409 response**
  - [See here - Name changed part](https://github.com/TamJ38/Berlin-Group-NextGenPSD2-vs-Open-Finance-v2-Endpoint-Schema-Comparison/blob/dev/common.md#-name-changed); 
  - [See here - Error response changed - PIS](https://github.com/TamJ38/Berlin-Group-NextGenPSD2-vs-Open-Finance-v2-Endpoint-Schema-Comparison/edit/dev/common.md#pis-1);
  - [See here - Parameter _links](https://github.com/TamJ38/Berlin-Group-NextGenPSD2-vs-Open-Finance-v2-Endpoint-Schema-Comparison/edit/dev/common.md#-parameter-_links);
- **401 response** -[AIS / PIS / PIIS part - Error response extended](https://github.com/TamJ38/Berlin-Group-NextGenPSD2-vs-Open-Finance-v2-Endpoint-Schema-Comparison/edit/dev/common.md#-error-response-extended);
- **429 response** - removed;


---

### v2-payment-service-payment-product-payment-id-status
**Group**: PIS  
**Method**: `GET`  
**Endpoint v2**: `v2/{payment-service}/{payment-product}/{payment-id}/status`
**Endpoint v1**: `v1/{payment-service}/{payment-product}/{payment-id}/status`

#### 🧩 Header
- [See here AIS / PIS / PIIS part](https://github.com/TamJ38/Berlin-Group-NextGenPSD2-vs-Open-Finance-v2-Endpoint-Schema-Comparison/edit/dev/common.md#ais--pis--piis);
- [See here Added PIS part](https://github.com/TamJ38/Berlin-Group-NextGenPSD2-vs-Open-Finance-v2-Endpoint-Schema-Comparison/edit/dev/common.md#-added-pis);
- `PSU-IP-Address`, `PSU-IP-Port, PSU-Accept`, `PSU-Accept-Charset`, `PSU-Accept-Encoding`, `PSU-Accept-Language`, `PSU-User-Agent`, `PSU-Http-Method`, `PSU-Device-ID`, `PSU-Geo-Location` - deleted;

#### 🟦 Response
- **200 response**
  - [See here - Name changed part](https://github.com/TamJ38/Berlin-Group-NextGenPSD2-vs-Open-Finance-v2-Endpoint-Schema-Comparison/blob/dev/common.md#-name-changed);
  - added  enum value: PRES in transactionStatus, reasonCode, reasonProprietary, transactionFees, currencyConversionFees, estimatedTotalAmount, estimatedInterbankSettlementAmount;
  - deleted psuName;
  - _links is extended with:
``` scaRedirect,
scaOAuth,
confirmation,
startAuthorisation,
startAuthorisationWithPsuIdentification,
startAuthorisationWithPsuAuthentication,
startAuthorisationWithEncryptedPsuAuthentication,	
startAuthorisationWithAuthenticationMethodSelection,	
startAuthorisationWithTransactionAuthorisation, self	
status, scaStatus, encryptionCertificates, < * >; 
(_links.scaRedirect and self still exist, but are now in a richer _links object in V2, with additional links);
```
  - apiClientMessages.code is defined as:
``` {
oneOf ->	
messageCode_ServiceUnspecific{...}
messageCode_PisSpecific{...}
messageCode_AisSpecific{...}
messageCode_PiisSpecific{...}
messageCode_SigningBasketSpecific{...}
messageCode_PushAisSpecific{...}
};
```
- **401 response** - [AIS / PIS / PIIS part - Error response extended](https://github.com/TamJ38/Berlin-Group-NextGenPSD2-vs-Open-Finance-v2-Endpoint-Schema-Comparison/blob/dev/common.md#-error-response-extended)
- **400,401, 403, 404,405, 409 response**
  - [See here - Name changed part](https://github.com/TamJ38/Berlin-Group-NextGenPSD2-vs-Open-Finance-v2-Endpoint-Schema-Comparison/blob/dev/common.md#-name-changed); 
  - [See here - Error response changes - PIS part](https://github.com/TamJ38/Berlin-Group-NextGenPSD2-vs-Open-Finance-v2-Endpoint-Schema-Comparison/blob/dev/common.md#-error-400-401-403-404-response-changed---pis);
  - [See here - Parameter _links part](https://github.com/TamJ38/Berlin-Group-NextGenPSD2-vs-Open-Finance-v2-Endpoint-Schema-Comparison/blob/dev/common.md#ais--pis--piis-1);
- **429 response** - deleted;

---

### v2-payment-service-payment-product-payment-id
**Group**: PIS  
**Method**: `GET`  
**Endpoint v2**: `v2/{payment-service}/{payment-product}/{payment-id}`
**Endpoint v1**: `v1/{payment-service}/{payment-product}/{payment-id}`

#### 📬 Path
- added new enum value for the path parameter **payment-product**: pain.001-proprietary-credit-transfers 
#### 🧩 Header
- [See here AIS/PIS/PIIS part](https://github.com/TamJ38/Berlin-Group-NextGenPSD2-vs-Open-Finance-v2-Endpoint-Schema-Comparison/blob/dev/common.md#ais--pis--piis)
- [See here Added PIS part](https://github.com/TamJ38/Berlin-Group-NextGenPSD2-vs-Open-Finance-v2-Endpoint-Schema-Comparison/edit/dev/common.md#-added-pis) **200 response is not included**
PSU-IP-Address, PSU-IP-Port, PSU-Accept, PSU-Accept-Charset, PSU-Accept-Encoding, PSU-Accept-Language, PSU-User-Agent, PSU-Http-Method, PSU-Device-ID, PSU-Geo-Location - deleted;


#### 🟦 Response
- **200 response** 
``` oneOf ->	
paymentInitiationWithStatusResponse{...}
periodicPaymentInitiationWithStatusResponse{..}
bulkPaymentInitiationWithStatusResponse{...}
```
changed to:
``` anyOf ->	
SinglePayment_generic{...}
BulkPayment_generic{...}
PeriodicPayment_generic{...}; (whole response is changed)
```
- **401 response** - [AIS / PIS / PIIS part - Error response extended](https://github.com/TamJ38/Berlin-Group-NextGenPSD2-vs-Open-Finance-v2-Endpoint-Schema-Comparison/blob/dev/common.md#-error-response-extended)
- **400,401, 403, 404,405, 409 response**
  - [See here - Name changed part](https://github.com/TamJ38/Berlin-Group-NextGenPSD2-vs-Open-Finance-v2-Endpoint-Schema-Comparison/blob/dev/common.md#-name-changed); 
  - [See here - Error response changes - PIS part](https://github.com/TamJ38/Berlin-Group-NextGenPSD2-vs-Open-Finance-v2-Endpoint-Schema-Comparison/blob/dev/common.md#-error-400-401-403-404-response-changed---pis);
  - [See here - Parameter _links part](https://github.com/TamJ38/Berlin-Group-NextGenPSD2-vs-Open-Finance-v2-Endpoint-Schema-Comparison/blob/dev/common.md#ais--pis--piis-1);
- **429 response** - deleted;

---

### v1-payment-service-payment-product-payment-id
**Group**: PIS  
**Method**: `DELETE`  
**Endpoint v2**: `v2/{payment-service}/{payment-product}/{payment-id}`
**Endpoint v1**: `v1/{payment-service}/{payment-product}/{payment-id}`

#### 🧩 Header
- [See here AIS/PIS/PIIS part](https://github.com/TamJ38/Berlin-Group-NextGenPSD2-vs-Open-Finance-v2-Endpoint-Schema-Comparison/blob/dev/common.md#ais--pis--piis)
- [See here PIS part](https://github.com/TamJ38/Berlin-Group-NextGenPSD2-vs-Open-Finance-v2-Endpoint-Schema-Comparison/blob/dev/common.md#pis)
``` TPP-Redirect-URI ->Client-Redirect-URI,
TPP-Nok-Redirect-URI->Client-Nok-Redirect-URI, 
TPP-Explicit-Authorisation-Preferred-> Client-Explicit-Authorisation-Preferred(async authorization);
PSU-ID, PSU-ID-Type, PSU-Corporate-ID, PSU-Corporate-ID-Type, Client-Notification-URI, Client-Notification-Content-Preferred, Client-Brand-Logging-Information added;
```


#### 🟦 Response
- **202 Received** changed to **202 Accepted**;
- **202 response** 
- added ENUM value: PRES in transactionStatus;  
- scaMethods[].name and chosenScaMethod.name are required in v2;
- added in links: `scaRedirect`, `scaOAuth`, `confirmation`, 
`startAuthorisationWithTransactionAuthorisation`, `self`, `status`, `scaStatus`, `encryptionCertificates`, < * >;
- **401 response** - [AIS / PIS / PIIS part - Error response extended](https://github.com/TamJ38/Berlin-Group-NextGenPSD2-vs-Open-Finance-v2-Endpoint-Schema-Comparison/blob/dev/common.md#-error-response-extended)
- **400,401, 403, 404,405, 409 response**
  - [See here - Name changed part](https://github.com/TamJ38/Berlin-Group-NextGenPSD2-vs-Open-Finance-v2-Endpoint-Schema-Comparison/blob/dev/common.md#-name-changed); 
  - [See here - Error response changes - PIS part](https://github.com/TamJ38/Berlin-Group-NextGenPSD2-vs-Open-Finance-v2-Endpoint-Schema-Comparison/blob/dev/common.md#-error-400-401-403-404-response-changed---pis); **405 response also included in Error 400, 401, 403, 404 response changed - PIS part**
  - [See here - Parameter _links part](https://github.com/TamJ38/Berlin-Group-NextGenPSD2-vs-Open-Finance-v2-Endpoint-Schema-Comparison/blob/dev/common.md#ais--pis--piis-1);
- **429 response** - deleted;

---

### v2-bulk-payments-payment-product-paymentid-extended-status
**Group**: PIS  
**Method**: `GET`  
**Endpoint**: `/v2/bulk-payments/{payment-product}/{paymentId}/extended-status`

#### 🧩 Header
- [See here AIS/PIS/PIIS part](https://github.com/TamJ38/Berlin-Group-NextGenPSD2-vs-Open-Finance-v2-Endpoint-Schema-Comparison/blob/dev/common.md#ais--pis--piis)
- [See here Added PIS part](https://github.com/TamJ38/Berlin-Group-NextGenPSD2-vs-Open-Finance-v2-Endpoint-Schema-Comparison/edit/dev/common.md#-added-pis)
- PSU-IP-Address, PSU-IP-Port, PSU-Accept, PSU-Accept-Charset, PSU-Accept-Encoding, PSU-Accept-Language, PSU-User-Agent, PSU-Http-Method, PSU-Device-ID, PSU-Geo-Location deleted;

#### 🟦 Response
- **200 response** - added new values for reasonCode(enum) and statusReasonInformationCode(enum):`AM21`, `BEXX`;
- reasonProperty description changed: If an ISO Code is available it should be used instead of Proprietary Reasons;
- originalTransactionInformationAndStatus.transactionStatus added value: `PRES`;
- added in _links `scaOAuth`, `confirmation`, `startAuthorisation`, `startAuthorisationWithPsuIdentification`, `startAuthorisationWithPsuAuthentication`, `startAuthorisationWithEncryptedPsuAuthentication`, `startAuthorisationWithAuthenticationMethodSelection`, `startAuthorisationWithTransactionAuthorisation`, `status`, `scaStatus`, `encryptionCertificates`; 
- apiClientMessages.code extended from string to: 
``` code*	{
oneOf ->	
messageCode_ServiceUnspecific{...}
messageCode_PisSpecific{...}
messageCode_AisSpecific{...}
messageCode_PiisSpecific{...}
messageCode_SigningBasketSpecific{...}
messageCode_PushAisSpecific{...}
};
```
- **401 response** - [AIS / PIS / PIIS part - Error response extended](https://github.com/TamJ38/Berlin-Group-NextGenPSD2-vs-Open-Finance-v2-Endpoint-Schema-Comparison/blob/dev/common.md#-error-response-extended)
- **400,401, 403, 404,405, 409 response**
  - [See here - Name changed part](https://github.com/TamJ38/Berlin-Group-NextGenPSD2-vs-Open-Finance-v2-Endpoint-Schema-Comparison/blob/dev/common.md#-name-changed); 
  - [See here - Error response changes - PIS part](https://github.com/TamJ38/Berlin-Group-NextGenPSD2-vs-Open-Finance-v2-Endpoint-Schema-Comparison/blob/dev/common.md#-error-400-401-403-404-response-changed---pis);
  - [See here - Parameter _links part](https://github.com/TamJ38/Berlin-Group-NextGenPSD2-vs-Open-Finance-v2-Endpoint-Schema-Comparison/blob/dev/common.md#ais--pis--piis-1);
- **429 response** - deleted;
**See common:** [changed:](common.md#headers), [Code](common.md#headers), [confirmation](common.md#headers), [code*](common.md#headers)

---

### v1-funds-confirmations
**Group**: PIIS  
**Method**: `POST`  
**Endpoint v2**: `v2/funds-confirmations`
**Endpoint v1**: `v1/funds-confirmations`

#### 🧩 Header
- [See here AIS/PIS/PIIS part](https://github.com/TamJ38/Berlin-Group-NextGenPSD2-vs-Open-Finance-v2-Endpoint-Schema-Comparison/blob/dev/common.md#ais--pis--piis)
- [See here PIIS part](https://github.com/TamJ38/Berlin-Group-NextGenPSD2-vs-Open-Finance-v2-Endpoint-Schema-Comparison/blob/master/common.md#piis)
- Body-Sig-Profile, Body-Enc-Profile, Body-Enc-List, Consent-ID added;

#### 🧾 Request
- `account.cashAccountType` deleted;
- `typeCode`, `typeProprietary`, `proxy`, `name`, `owner` and `servicer` (with many additional fields)  added;
- `account.other.schemeNameCode` is extended with enum values: `AIIN`, `BBAN`, `CUID`, `UPIC`;

#### 🟦 Response
- [See here the Error response extended and Name changed parts in AIS/PIS/PIIS part](https://github.com/TamJ38/Berlin-Group-NextGenPSD2-vs-Open-Finance-v2-Endpoint-Schema-Comparison/blob/dev/common.md#ais--pis--piis)
- [See here PIIS part](https://github.com/TamJ38/Berlin-Group-NextGenPSD2-vs-Open-Finance-v2-Endpoint-Schema-Comparison/edit/master/common.md#piis-1)
- **400,401,403,404,405 response** - [See here the Parameter _links part](https://github.com/TamJ38/Berlin-Group-NextGenPSD2-vs-Open-Finance-v2-Endpoint-Schema-Comparison/edit/master/common.md#-parameter-_links)
- deleted value for code.enum: `SESSIONS_NOT_SUPPORTED`;
- added values for code.enum: `CARD_INVALID`, `NO_PIIS_ACTIVATION`
- **429 response** deleted

---

### v2-accounts
**Group**: AIS  
**Method**: `GET`  
**Endpoint**: `v2/accounts`

#### 🧩 Header
C15, C16, C17,C18,C19, C20 from the sheet:common;

**See common:** [C15](common.md#headers), [C16](common.md#headers), [C17C18C19](common.md#headers), [C20](common.md#headers)

#### 🟦 Response
200 response - E15, E16 from the sheet:common;
400 response - E23 from the sheet:common;
400, 401 ,403, 404, 405, 409 response  -  E17, E18 from the sheet:common;
401 response - E21 from the sheet:common;
400, 401, 404 response -  E19 from the sheet:common;
 403, 405, 409 response  E20 from the sheet:common;
404 response - E22 from the sheet:common;

---

### v2-accounts-account-id
**Group**: AIS  
**Method**: `GET`  
**Endpoint**: `v2/accounts/{account-id}`

#### 🧩 Header
C15, C16, C17, C18, C19, C20 from the sheet:common;

**See common:** [C15](common.md#headers), [C16](common.md#headers), [C17](common.md#headers), [C18](common.md#headers), [C19](common.md#headers), [C20](common.md#headers)

#### 🟦 Response
200 response - E15, E16 from the sheet:common;
400 response - E23 from the sheet:common;
400, 401, 404 response -  E19 from the sheet:common;
  403, 405, 409 response  - E20 from the sheet:common;
401 response -  E21 from the sheet:common;
404 response - E22 from the sheet:common;
400, 401 ,403, 404, 405, 409 response  -  E17, E18 from the sheet:common;

---

### v2-accounts-account-id-balances
**Group**: AIS  
**Method**: `GET`  
**Endpoint**: `v2/accounts/{account-id}/balances`

#### 🧩 Header
C15, C16, C17, C18, C19, C20 from the sheet:common;

**See common:** [C15](common.md#headers), [C16](common.md#headers), [C17](common.md#headers), [C18](common.md#headers), [C19](common.md#headers), [C20](common.md#headers)

#### 🟦 Response
200 response - account.cashAccountType deleted;
typeCode, typeProprietary, proxy, name, owner and servicer(with many additional fields)  added in account;
account.other.schemeNameCode is extended with enum values(AIIN, BBAN, CUID, UPIC);
400 response -  E23 from the sheet:common;
400, 401, 404 response -  E19 from the sheet:common;
401 response - E21 from the sheet:common;
404 response - E22 from the sheet:common;
403, 405, 409 response  - E20 from the sheet:common;
400, 401 ,403, 404, 405, 409 response  -  E17, E18 from the sheet:common;

**See common:** [CUID](common.md#headers)

---

### v2-accounts-account-id-transactions
**Group**: AIS  
**Method**: `GET`  
**Endpoint**: `v2/accounts/{account-id}/transactions`

#### 🧩 Header
C15, C16, C17, C18, C19, C20 from the sheet:common;

**See common:** [C15](common.md#headers), [C16](common.md#headers), [C17](common.md#headers), [C18](common.md#headers), [C19](common.md#headers), [C20](common.md#headers)

#### 🟦 Response
200 response -E26 from the sheet:common;
account is mandatory; 
field download in _links is not required
creditorAccount and debtorAccount - E26 from the sheet:common;
for every type of transaction in transactions:
E24,E25,E27, E28 from the sheet:common;


for every entry in entry details: E25,E27,E18 from the sheet:common; 
 added enum values for purpose code;
transactions._links - E18 from the sheet:common; 
400 response - E23 from the sheet:common;
400, 401, 404 response -  E19 from the sheet:common;
  403, 405, 409 response  - E20 from the sheet:common;
401 response -  E21 from the sheet:common;
404 response - E22 from the sheet:common;
400, 401 ,403, 404, 405, 409 response  -  E17, E18 from the sheet:common;

**See common:** [creditorAccount](common.md#headers), [code](common.md#headers)

---

### v2-accounts-account-id-transactions-transaction-id
**Group**: AIS  
**Method**: `GET`  
**Endpoint**: `v2/accounts/{account-id}/transactions/{transaction-id}`

#### 🧩 Header
C15, C16, C17, C18, C19, C20 from the sheet:common;

**See common:** [C15](common.md#headers), [C16](common.md#headers), [C17](common.md#headers), [C18](common.md#headers), [C19](common.md#headers), [C20](common.md#headers)

#### 🟦 Response
200 response - E24,E25,E27,E28 from the sheet:common;
transactions._links - E18 from the sheet:common;
for every entry in entry details: E25,E27,E18 from the sheet:common, added enum values for purpose code;
400 response - E23 from the sheet:common;
400, 401, 404 response -  E19 from the sheet:common;
  403, 405, 409 response  - E20 from the sheet:common;
401 response -  E21 from the sheet:common;
404 response - E22 from the sheet:common;
400, 401 ,403, 404, 405, 409 response  -  E17, E18 from the sheet:common;

**See common:** [code](common.md#headers)

---

### v2-card-accounts
**Group**: AIS  
**Method**: `GET`  
**Endpoint**: `/v2/card-accounts`

#### 🧩 Header
C15, C16, C17, C18, C19, C20 from the sheet:common;

**See common:** [C15](common.md#headers), [C16](common.md#headers), [C17](common.md#headers), [C18](common.md#headers), [C19](common.md#headers), [C20](common.md#headers)

#### 🟦 Response
200 response - resourceId is mandatory; _links In account - E16  from the sheet:common;
400 response - E23 from the sheet:common;
400, 401, 404 response -  E19 from the sheet:common;
  403, 405, 409 response  - E20 from the sheet:common;
401 response -  E21 from the sheet:common;
404 response - E22 from the sheet:common;
400, 401 ,403, 404, 405, 409 response  -  E17, E18 from the sheet:common;

---

### v2-card-accounts-account-id
**Group**: AIS  
**Method**: `GET`  
**Endpoint**: `/v2/card-accounts/{account-id}`

#### 🧩 Header
C15, C16, C17, C18, C19, C20 from the sheet:common;

**See common:** [C15](common.md#headers), [C16](common.md#headers), [C17](common.md#headers), [C18](common.md#headers), [C19](common.md#headers), [C20](common.md#headers)

#### 🟦 Response
200 response - resourceId is mandatory; _links In account - E16  from the sheet:common;
400 response - E23 from the sheet:common;
400, 401, 404 response -  E19 from the sheet:common;
  403, 405, 409 response  - E20 from the sheet:common;
401 response -  E21 from the sheet:common;
404 response - E22 from the sheet:common;
400, 401 ,403, 404, 405, 409 response  -  E17, E18 from the sheet:common;

---

### v2-card-accounts-account-id-balances
**Group**: AIS  
**Method**: `GET`  
**Endpoint**: `/v2/card-accounts/{account-id}/balances`

#### 🧩 Header
C15, C16, C17, C18, C19, C20 from the sheet:common;

**See common:** [C15](common.md#headers), [C16](common.md#headers), [C17](common.md#headers), [C18](common.md#headers), [C19](common.md#headers), [C20](common.md#headers)

#### 🟦 Response
200 response
cardAccount - E26 from the sheet:common; cardAccount is required;
400 response - E23 from the sheet:common;
400, 401, 404 response -  E19 from the sheet:common;
  403, 405, 409 response  - E20 from the sheet:common;
401 response -  E21 from the sheet:common;
404 response - E22 from the sheet:common;
400, 401 ,403, 404, 405, 409 response  -  E17, E18 from the sheet:common;

**See common:** [cardAccount](common.md#headers), [cardAccount](common.md#headers)

---

### v2-card-accounts-account-id-transactions
**Group**: AIS  
**Method**: `GET`  
**Endpoint**: `/v2/card-accounts/{account-id}/transactions`

#### 🧩 Header
C15, C16, C17, C18, C19, C20 from the sheet:common;

**See common:** [C15](common.md#headers), [C16](common.md#headers), [C17](common.md#headers), [C18](common.md#headers), [C19](common.md#headers), [C20](common.md#headers)

#### 🟦 Response
200 response
cardAccount - E26 from the sheet:common; cardAccount is required;
field download in _links is not required;
cardTransactions - cardAcceptorAddress field for every transactions(booked,..): department, subDepartment,buildingName,floor, postBox, room, postCode, townName, townLocationName, districtName, countrySubDivision are deleted;
_links in cardTransactios are extended - E18 from the sheet:common;
400 response - E23 from the sheet:common;
400, 401, 404 response -  E19 from the sheet:common;
  403, 405, 409 response  - E20 from the sheet:common;
401 response -  E21 from the sheet:common;
404 response - E22 from the sheet:common;
400, 401 ,403, 404, 405, 409 response  -  E17, E18 from the sheet:common;

**See common:** [cardAccount](common.md#headers), [cardAccount](common.md#headers), [cardTransactions](common.md#headers), [cardAcceptorAddress](common.md#headers), [countrySubDivision](common.md#headers), [cardTransactios](common.md#headers)

#### 🔎 Query
deleted entryReferenceFrom;
