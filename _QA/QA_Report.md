# NetBrain REST API R12.3a — Documentation QA Report

**Repo:** `NetBrainAPI/NetBrain-REST-API-R12.3a` → `REST APIs Documentation/`
**Method:** Live execution of read/GET samples against `nblive2025.netbrain.com` (tenant *NBLIVE-Sessions*, domain *training* — a clean classic router/switch topology). Write endpoints reviewed statically (read-only policy). Live server build reports **R12.3** (repo is R12.3a).
**Date:** 2026-07-15

Severity key (per request):
- **ERROR** — following the documented endpoint/sample as written fails: sample errors out, mandatory parameter missing, or the documented path/method is wrong.
- **WARNING** — works if you follow the sample, but the doc has an inconsistency, typo, redundancy, or cosmetic defect.

---

## 1. Coverage

| Class | Count | How tested |
|---|---:|---|
| Files in `REST APIs Documentation/` | 247 | all parsed |
| Distinct API categories | 35 | all mapped |
| Read (GET) endpoints | 86 | **executed live** |
| Read-style POST (query) | 2 | executed live |
| Write endpoints (POST/PUT/DELETE/PATCH) | 155 | **static review** (path/method/JSON consistency, hand-verified); not executed (read-only policy) |
| Reference/guide docs (no testable endpoint) | 12 | n/a |

Auth prelude verified end-to-end: `POST /V1/Session` → `GET /V1/CMDB/Tenants` → `GET /V1/CMDB/Domains` → `PUT /V1/Session/CurrentDomain` → live GET (`/V1/System/ProductVersion` = R12.3, statusCode 790200). All OK.

---

## 2. ERRORS

### E1 — Get Device Neighbors by Topology Type (current) — sample returns HTTP 500
**File:** `Topology Management/Get Device Neighbors by Topology Type API.md`
The sample code and cURL use a **string** topology type:
```python
topoType = "L3_Topo_Type"
```
Live result: **HTTP 500 / `793001 Inner exception`** (both python sample value and cURL value fail).
The doc's *own parameter table* says `topoType` is `int` 1–4. Using `topoType=1` → HTTP 200 Success.
**Impact:** The documented sample does not work. **Fix:** change sample to `topoType = 1` (or `[1]`), and align the sample with the parameter table. (Server should also return a 4xx, not 500, for a bad value — worth a bug to engineering.)
*(The separate `...Version_1.md` doc correctly uses `topoType=[1]` and works — PASS.)*

### E2 — Update Role — documented endpoint path is a different API
**File:** `User Roles Management/Update Role API.md`
Header declares the wrong endpoint (copy-paste from the Path API):
```
## ***PUT*** /V1/CMDB/Path/Calculation
```
The API Server URL, `full_url`, and cURL all correctly use `/V1/CMDB/Roles`. A reader trusting the header would hit the wrong API. **Fix:** header → `## ***PUT*** /V1/CMDB/Roles`.

### E3 — Get Site Devices — header path 405s
**File:** `Site Management/Get Site Devices API.md`
Header: `## ***GET*** /V1/CMDB/Sites{?sitePath}|{?siteId}` → `GET /V1/CMDB/Sites` returns **405 Method not supported**.
API-URL/sample/cURL correctly use `/V1/CMDB/Sites/Devices` (works). **Fix:** header → `/V1/CMDB/Sites/Devices`.

### E4 — Get Search Results — header/title path 404s (V1 vs V3)
**File:** `Search Management/Get Search Results API.md`
Header + "API Server URL" say `V1/CMDB/Search` → **404 No resource**. Only `/V3/CMDB/Search` exists (sample + cURL use V3, works). **Fix:** header/URL → `/V3/CMDB/Search`.

### E5 — Get All Discovery Tasks — header path 404s
**File:** `Discovery Task Management/Get All Discovery Tasks API.md`
Header: `## ***GET*** /V1/CMDB/Devices/Discovery/Tasks` → **404** (extra `/Devices`). API-URL/sample/cURL use `/V1/CMDB/Discovery/Tasks` (works). **Fix:** remove `/Devices` from header.

### E6 — Change Analysis Report export endpoints — header path missing `/CMDB`
**Files:** `Change Analysis Report/Create Export Task For Change Analysis Report API.md` (+ `Check Export Task Status API.md`, `Download Change Analysis Report API.md`)
Headers use `/V1/ChangeAnalysis/Export/Tasks...` → **404**. The correct base is `/V1/CMDB/ChangeAnalysis/Export/Tasks...` (confirmed live: `/V1/CMDB/ChangeAnalysis/Export/Tasks/<id>/Status` → 200/`794004 Task does not exist`, i.e. valid endpoint). **Fix:** insert `/CMDB` in the three headers.

### E7 — Create/Set Interface Attribute — header path typo `Interface` (singular)
**Files:** `Device Interfaces Management/Create Interface Attribute API.md`, `Set Interface Attributes API.md`
Header: `## ***POST*** /V1/CMDB/Interface/Attributes`. Correct path is `/V1/CMDB/Interfaces/Attributes` (plural — used by API-URL/sample/cURL, and by the working Get Interface Attributes). **Fix:** header → `Interfaces`.

### E8 — CM Scheduling / Get Scheduled Task — three inconsistent paths; documented one 404s
**File:** `Change Management/CM Scheduling/Get Change Management Scheduled Task REST API.md`
Within one file: header + API-URL + cURL use `/V1/CMDB/CM/Tasks/ScheduledTask` (→ **404**); the Python sample uses `/V1/CM/Tasks/ScheduleTask` (→ **403**, i.e. the route that actually exists — my QA user lacked CM-scheduling permission, but 403≠404 proves the route resolves). Note also `ScheduleTask` vs `ScheduledTask` spelling drift. **Fix:** standardize on the real path `/V1/CM/Tasks/ScheduleTask` across header, URL, sample, and cURL. *(Applies across the CM Scheduling folder — Create/Update/Delete docs show the same `/V1/CM...` vs `/V1/CMDB/CM...` drift; verify each.)*

### E9 — AAM Get Application (by name) — header/API-URL path 404s (plural vs singular)
**File:** `AAM (Application Assurance Module)/Get Application Information.md`
Header + API-URL: `V3/AAM/Applications?Application={name}` (plural) → **404**. Sample uses `/V3/AAM/Application` (singular) → works (400 "application does not exist" = valid endpoint). **Fix:** header/URL → `/V3/AAM/Application`.

---

## 3. WARNINGS

### W1 — Get Front Server of A Device — header typo `decive`
**File:** `Devices Management/Get Front Server of A Device API.md`
Header: `/V1/CMDB/Devices/decive/FrontServer` (typo). Sample/cURL use `device`. Live: **both spellings resolve** to the same business response, so it still works — cosmetic. **Fix:** header `decive` → `device`.

### W2 — Get Domain License Template — inconsistent status envelope
**File:** `Tenants and Domains Management/Get Domain License Template.md`
Returns HTTP 200 but `statusCode: 0` and no `statusDescription`, whereas every other API returns `790200` / `"Success."`. **Fix:** confirm intended envelope; doc/response should show `790200`.

### W3 — Benchmark "task does not exist" — inconsistent status codes
**Files:** `Benchmark Task Management/Get Benchmark Task Status API.md`, `Get Benchmark Task Runs API.md`
Same condition (missing task) returns `794004 "Task 'X' does not exist."` from *Status* but `791006 "task X does not exist."` from *Runs* (different code + casing). Cosmetic inconsistency across sibling APIs.

### W4 — Endpoint header paths missing the leading slash (V3-era docs) — 34 files
Headers written as `## ***POST*** V3/AAM/Application` instead of `.../V3/...` (no leading `/`). Clusters: AAM (11), ADT (10), TAF Lite (6), Network Definition (4), Change Analysis (1), Incident Report (1), Search (1). Cosmetic/consistency — samples still work. **Fix:** prefix header paths with `/`.

### W5 — Get Devices — self-contradictory `limit` description
**File:** `Devices Management/Get Devices API.md`
The `limit` parameter says both *"No upper bound for this parameter"* and *"The `limit` value's valid range is 10 - 100 … must be greater than or equal to 10 and less than or equal to 100."* Contradictory. **Fix:** state the real constraint once.

### W6 — External Authentication endpoint name misspelled (`ExternalAuthtication`) — informational
**Files:** `External Authentication/*`
The endpoint is `/V1/CMDB/ExternalAuthtication` (missing an "n"). Live confirms the **server itself** uses the misspelling (correct spelling → 404), so the doc is *accurate to the product*. Flagging as a known product quirk, not a doc error — the docs correctly match the server.

---

## 4. Not doc bugs (recorded to prevent false positives)
- **Device/Interface/Module Attributes & Get Saved Paths** name their query dict `body`/`data` but pass it via `params=` in `requests.get(...)`, so samples work. (Sending it as a JSON body fails, but the docs don't do that.)
- **Get AAM Report** header is correctly `POST` (initial GET probe was a test error).
- **Resolve Device Gateway** correctly documents the `ipOrHost` parameter.
- **~115 "invalid JSON" candidates** were NetBrain's Python-dict response style (single quotes, `True/False/None`). Strict ```json-fenced parsing found **0** malformed blocks.

---

## 5. Untestable in this environment (not scored — verify separately)
- **Data Center Operation** (`/V1/data-center/status`, `/opr`, fsc migrate/transfer) → **404** on this SaaS instance; likely on-prem-only. Verify on an on-prem build.
- **CM Scheduling** execution blocked by **403** (QA user lacks permission) — path validity confirmed, behavior not exercised.
- **AAM / ADT** end-to-end flows need the AAM module + pre-existing applications/tables (absent in *training*). Endpoints reachable; data flows not exercised.
- **Path Calc / Benchmark / Discovery / Topology-build / CM task** *result/status* GETs validated as **reachable** (proper business errors for fake IDs) but not exercised end-to-end, since that requires creating tasks (writes, excluded by policy).

---

## 6. PASS (documented read sample works as written)
Product Version · Get Tenants · Get Domains · Get Devices · Get Device Attributes · Get Device Configuration · Get Device Data · Get Device Baseline Table · Get Interfaces · Get Interface Attributes · Get Modules · Get Module Attributes · Get Device Group · Get Devices of Group · Get One IP Table · Get Device Neighbors (Version_1) · Neighbors current (with int topoType) · Get Child Site · Get Site Info · Get Site Devices (`/Sites/Devices`) · Network Settings (telnetInfo/All) · Get Users · Get Roles · Shared Device Settings (+CLI/API/SNMP) · Front Server Controller/Server List · All API Servers · AWS/Azure Accounts · Domain License Usage Summary · Tenant License Template · Get Saved Paths · Resolve Device Gateway · Path Gateways · Search (V3) · Network Definition · License Node Info · Usage Report of Users · Tune Devices · Event Console · Event Driven status · Audit Logs (From+To) · Discovery Tasks (`/Discovery/Tasks`).

---

## 7. Remaining depth available on request
This pass fully covered the **live-testable read surface** and the **mechanical** correctness of write docs (path/method/JSON). A deeper per-field review of each of the 155 write request-body param tables (every mandatory `*` present in the sample, field-description accuracy, response-schema completeness) was **not** exhaustively done. I can go category-by-category on the write bodies next if you want that depth.
