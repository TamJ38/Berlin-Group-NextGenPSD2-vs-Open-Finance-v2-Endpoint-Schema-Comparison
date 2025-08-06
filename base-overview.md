# Base Overview: Open Finance v2 vs NextGenPSD2 v1

> **Purpose**: This project provides a detailed comparative analysis between **Open Finance v2** (Berlin Group, 2024/2025) and the previous **NextGenPSD2 v1** specification.  
> It focuses on the evolution of API methods, endpoint structures, and protocol changes.  
> The goal is to support implementation teams and solution architects in understanding required migrations, upgrades, and deprecations.

---

## ✨ Repository Structure

| Sheet Name   | Description |
|--------------|-------------|
| `Endpoints`  | Lists **all endpoints** in v1 and v2. Shows the **HTTP method**, **path**, **status** (removed, added, modified), and what changed (Path, Header, Request, Response, Query). |
| `Modifications` | Describes changes in endpoints marked as **modified**. Specifies what changed (e.g. renamed path, added field, altered response). |
| `Common`     | Contains **shared/global changes** referenced in `Modifications` by coordinates (e.g. `E21`). Helps avoid duplication of recurring changes. |

---

## 📊 Change Types Tracked

- 🔶 **Modified** – Endpoint exists in both v1 and v2 but has at least one change:
  - Path parameters  
  - Request structure  
  - Response schema  
  - Headers or query parameters

- ➕ **Added** – New endpoint introduced in v2  
  _Example: `/v2/bulk-payments/{payment-product}/{paymentId}/extended-status`_

- ❌ **Removed** – Endpoint existed in v1 but is removed in v2  
  _Example: `/v1/payment-service/{payment-product}/{payment-id}/cancellation-authorisations`_

---

## 🔍 Endpoint Matrix Example

Example snippet from the full comparison matrix:

| Method | Endpoint V1                          | Endpoint V2                                 | Status   | Path | Header | Request | Response | Query |
|--------|--------------------------------------|----------------------------------------------|----------|------|--------|---------|----------|-------|
| POST   | `/v1/consents`                       | `/v2/consents`                               | ⚪removed  |      |        |         |          |       |
| GET    | `/accounts/{id}/balances`            | `/v2/accounts/{id}/balances`                 | 🔶modified | ✅   | ✅     | ✅       | ✅        | ✅     |
| GET    |                                      | `/v2/periodic-payments/{payment-product}`    | 🟢added    |      |        |         |          |       |

Legend:
- ✅ indicates the specific part (path/header/etc.) was **changed**
- Empty cell = no change

---

## 🔹 Usage Instructions

- Use the `Endpoints` tab to view the full change matrix
- Cross-reference `Modifications` for detailed diffs of changed endpoints
- Check the `Common` tab for global/query/header changes reused across endpoints
- Filter by:
  - API group: AIS / PIS / CoF
  - Method: GET / POST / PUT / DELETE
  - Status: Added / Modified / Removed

---

## 🔧 Future Extensions

This dataset can be easily extended into:

- Migration tools (e.g. endpoint mappers)
- OpenAPI diff viewers
- Frontend explorer (with filters for status/type/method)
- CI testing triggers for updated endpoint logic

---

## 🏁 Final Note

This comparison is curated based on official Berlin Group specifications.  
It is structured for high reusability in documentation, implementation planning, testing pipelines, and automated migration workflows.

