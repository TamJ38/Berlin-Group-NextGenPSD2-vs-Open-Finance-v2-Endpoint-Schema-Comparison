# Common Modifications

This file documents shared changes that apply across multiple endpoints in the Open Finance v2 specification. 
Grouped by type of change and API group.

---

## 📁 Headers

### AIS / PIS / PIIS
- #### 🔸 **Removed everywhere**:
  - `TPP-Signature-Certificate` 
- #### 🔸 **Changed everywhere**:
  - `Signature changed to x-jws-signature`
  - `Digest - the description is more detailed and written according to RFC3230 and RFC5843`

### AIS
- #### 🔸 **Removed AIS**:
  - `PSU-Geo-Location`, `PSU-IP-Port`, ` PSU-Accept-Charset`, `PSU-Device-ID`, `PSU-Http-Method`,`PSU-Accept-Encoding`, ` PSU-Accept-Language`, `PSU-User-Agent`
- #### 🔸 **Added AIS**:
  -  in 201, 200, 400, 401, 403, 404, 405, 406, 408, 409, 415, 429, 500, 503 response: `X-Reference-API-Name`, `X-Reference-API-Version`, `X-Reference-API-Document`
  -  in 400, 401, 403,404,405,406, 408, 409,415,429, 500, 503 response: `ASPSP-Notification-Support`

### PIS
- #### 🔸 **Removed PIS**:
  - `TPP-Redirect-Preferred`, `TPP-Decoupled-Preferred` 
- #### 🔸 **Added PIS**:
  - `Client-SCA-Approach-Preference(decoupled, redirect, embedded)`,   
  - in 400, 401, 403,404,405,406, 408, 409,415, 500, 503 response: `ASPSP-Notification-Support`,
  - in 201,200, 400,401, 403, 404, 405,406,408, 409, 415, 500, 503 response: `X-Reference-API-Name`, `X-Reference-API-Version`, `X-Reference-API-Document`

### PIIS
- #### 🔸 **Removed PIIS**:
  - `TPP-Redirect-Preferred`, `TPP-Decoupled-Preferred` 
- #### 🔸 **Added PIIS**:
  - `Client-SCA-Approach-Preference(decoupled, redirect, embedded)`,   
  - in 400, 401, 403,404,405,406, 408, 409,415, 500, 503 response: `ASPSP-Notification-Support`,
  - in 201,200, 400,401, 403, 404, 405,406,408, 409, 415, 500, 503 response: `X-Reference-API-Name`, `X-Reference-API-Version`, `X-Reference-API-Document`
---


## 📁 Response Body

### AIS / PIS / PIIS
#### 🔸 Name changed
- `tppMessage` renamed to `apiClientMessages`
#### 🔸 Error response extended
- Standard error code `401` enums extended:
  - `CLIENT_INVALID`, `ROLE_INVALID`,`CLIENT_INCONSISTENT`, `API_CONTRACT_ID_INVALID`
#### 🔸 Parameter _links
- added more options for the parameter `_links` in schema of the response:
  - `transactionfees`, `updateResourceByDebtorAccountResource`, `savingsAccount`, `loanAccount`, `ibanCheck`, `securitiesAccount`, `paymentInitiation`, `positions`, `orders`, `relatedOrders`, `orderDetails`, `relatedTransactions`, `subscription`, `entryStatusRevoked`, `confirmInitiation`, `aspspParameters`, `aspspContacts`, `aspspDowntimes`, `onboardings`, `readConditions`, `confirmConditions`
- the following values are removed from the parameter `_links` in the schema of the response if they exist:
  - `updateAdditionalPsuAuthentication`, `updateAdditionalEncryptedPsuAuthentication`, `< * >`

### PIS 
#### 🔸 Error `400`, `401`, `403`, `404` response changed - PIS
- In the response with error code `400`, `401`, `403`, `404`:
  - `apiClientMessage.code(enum)` is divided into two parts: `MessageCode_ServiceUnspecific_40X(enum)` and `MessageCode_PisSpecific_40X(enum)`
#### 🔸 Error `405`, `409` response changed - PIS
- In the response with error code `405`, `409`:
  - `apiClientMessage.code(enum)` has one part MessageCode_ServiceUnspecific_40X(enum); 
 
### PIIS 
#### 🔸 Error response changed - PIS
- The response with error code `400`:
  - `apiClientMessage.code(enum)` is divided into two parts: `MessageCode_ServiceUnspecific_40X(enum)` and `MessageCode_PisSpecific_40X(enum)`
- In the response with error code `401`, `403`, `404`, `405`, `409`:
  - `apiClientMessage.code` have one part `MessageCode_ServiceUnspecific_40X(enum)`;
 
### AIS 
- #### 🔸 Account object changes
  - `psuName` **deleted**
  - `resourceId` is now **mandatory**
  - **New fields added**: `other`, `tariffs`, `applicableFees`, `interest`, `relatedDates`, `collateralsInvolved`, `guaranteeInvolved`
  - **New enum values for `cashAccountType`**:
    `CACC`, `CARD`, `CASH`, `CHAR`, `CISH`, `COMM`, `CPAC`, `LLSV`, `LOAN`, `MGLD`, `MOMA`, `NREX`, `ODFT`, `ONDP`, `OTHR`, `SACC`, `SLRY`, `SVGS`, `TAXE`, `TRAN`, `TRAS`, `VACC`, `NFCA`
  - `_links` extended: 
    - `cheques` in `_links` was **deleted**
    - `_links` is extended with many values:
    `scaRedirect`, `scaOAuth`, `confirmation`
    `startAuthorisation`, `updatePsuIdentification`, `updatePsuAuthentication`
    `startAuthorisationWithEncryptedPsuAuthentication`
    `selectAuthenticationMethod`, ...
- #### 🔸 In the response with error code `400`, `401`, `404`:
  - `apiClientMessage.code` is divided into two parts: `MessageCode_ServiceUnspecific_40X(enum)` and `MessageCode_PisSpecific_40X(enum)`
- #### 🔸 In the response with error code `403`, `405`, `409`:
  - `apiClientMessage.code` have one part `MessageCode_ServiceUnspecific_40X(enum)`;
- #### 🔸 In the response with error code `404`:
  - new enum value for `code` is **added**: `CONTENT_TEMPORARILY_NOT_AVAILABLE`;
- #### 🔸 In the response with error code `400`:
  - new enum value for `code` is **added**: `CONSENT_TYPE_NOT_SUPPORTED`;
- #### 🔸 Transaction reference structure changes
  - In every transaction (`booked`, `pending`, `information`) → added references with additional fields:
  - `localInstrumentCode`, `localInstrumentProprietary`, `amountDetails`, `interbankSettlementDate`, `cardTransaction`
  - `creditor` and `debtor` – with additional fields
- #### 🔸 Transaction fields moved/deleted (for every type of transaction in transactions)
  - **Moved**: `endToEndId`, `mandateId`, `checkId`, `creditorId` in references
  - **Deleted**: 
    `creditorName`, `debtorName`,
    `remittanceInformationUnstructuredArray`, `remittanceInformationStructuredArray`
- #### 🔸 Account structure enriched
  - `account.cashAccountType` → **deleted**
  - **New fields added**:
    `typeCode`, `typeProprietary`, `proxy`, `name`, `owner`, `servicer(with many additional fields) `
  - `account.other.schemeNameCode` → extended with enum values:  
    `AIIN`, `BBAN`, `CUID`, `UPIC`
- #### 🔸 For every type of transaction in transactions:
  **Added**: 
  - **New mandatory field** for:
    `creditorAgent`, `debtorAgent` → `financialInstitutionId`
  - Fields for `ultimateCreditor` and `ultimateDebtor`:
    `name`, `identification`
  - Fields for `remittanceInformationStructured`:
    `referredDocumentInformation`, `creditorReferenceInformation`, `additionalRemittanceInformation`
- #### 🔸 for every type of transaction in transactions:
  - `additionalInformation` → renamed to `additionalTransactionInformation`
  - New fields added:
    `additionalTransactionInformationStructured`
    `rtpDetails`  in additionalInformationStructured, 
    `standingOrderDetails.monthsOfExecution` in additionalInformationStructured
- #### 🔸 **New descripton to additionalInformationStructured.standingOrderDetails. monthsOfExecution**:
  - `"31"` is ultimo. First day = `"1"`, time zone = ASPSP.
---



