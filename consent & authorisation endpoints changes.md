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
