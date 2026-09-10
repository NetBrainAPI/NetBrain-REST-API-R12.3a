# Live-test findings (accumulator) — training domain, R12.3 (nblive2025)

Legend: ERROR = sample fails / missing mandatory param / wrong path-method. WARNING = works but doc defect.

## Confirmed via live execution

- **ERROR — Topology / Get Device Neighbors by Topology Type (current).**
  Sample code uses `topoType = "L3_Topo_Type"` (string) and curl `topoType=L3_Topo_Type`.
  Live result: **HTTP 500 `793001 Inner exception`**. The doc's own parameter table says `topoType` is `int` 1–4.
  Using `topoType=1` returns 200. => Sample is broken AND self-contradictory. (Server should also return 4xx not 500.)

- **WARNING — Site Management / Get Site Devices.**
  Header: `## ***GET*** /V1/CMDB/Sites{?sitePath}` (GET /V1/CMDB/Sites → **405 Method not supported**).
  Sample/API-URL/curl correctly use `/V1/CMDB/Sites/Devices` (works). Header path is wrong/misleading.

- **WARNING — Search Management / Get Search Results.**
  Header + "API Server URL" say `V1/CMDB/Search` (→ **404 No resource**). Sample full_url + curl use `/V3/CMDB/Search` (works).
  Header/title path is wrong; only V3 exists.

- **WARNING — Discovery / Get All Discovery Tasks.**
  Header: `## ***GET*** /V1/CMDB/Devices/Discovery/Tasks` (→ **404**). API-URL/sample/curl use `/V1/CMDB/Discovery/Tasks` (works).
  Header path has an extra `/Devices`.

- **WARNING/ENV — Data Center Operation / Get Data Center Status.**
  `/V1/data-center/status` → **404 No resource** on this instance (likely on-prem-only / not exposed on this SaaS build). Untestable here; verify against on-prem.

- **WARNING — Tenants&Domains / Get Domain License Template.**
  `/V3/CMDB/Domains/LicenseTemplate/{tenantId}` returns 200 but `statusCode: 0` (all other APIs return `790200`) and no `statusDescription`. Inconsistent status envelope.

## Confirmed PASS (sample works as documented)
Product Version; Get Tenants; Get Domains; Get Devices; Get Device Attributes (params); Get Device Configuration;
Get Interface(s) + Interface Attributes (params); Get Modules + Module Attributes (params); Get Device Group;
Get Devices of Group; Get One IP Table; Get Child Site; Get Site Info; Get Network Settings (telnetInfo/All);
Get Users; Get Roles; Get Shared Device Settings (+CLI/API/SNMP); Get Front Server Controller/Server List;
Get All API Servers; Get AWS/Azure Accounts; Get Domain License Usage Summary; Get Tenant License Template;
Get Saved Paths (params); Neighbors v1 (topoType=[1]); Neighbors current with int topoType.

## Note (not a bug — first-pass test artifact)
Device/Interface/Module Attributes & Saved Paths name their query dict `body`/`data` but pass it via `params=` in
`requests.get(...)`, so the samples DO work. (Sending it as a JSON body fails — but the docs don't do that.)
