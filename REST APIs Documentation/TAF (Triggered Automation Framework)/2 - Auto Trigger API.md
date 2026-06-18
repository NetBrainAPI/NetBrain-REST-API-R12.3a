
# Auto Trigger API Design

## ***POST*** /V1/TAF/Auto
Use this API to send a raw event data from the 3rd party system to NetBrain.

## Detail Information

> **Title** : Auto Trigger API

> **Version** : 06/04/2026

> **API Server URL** : http(s)://IP address of NetBrain Web API Server/ServicesAPI/API/V1/TAF/Auto

> **Authentication** : 

|**Type**|**In**|**Name**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|Bearer Authentication| Headers | Authentication token | 

## Prerequisites

Before calling this API, the following must be configured in the NetBrain Workstation. If any of these prerequisites are missing, the API call will succeed at the HTTP level but no Triggered Diagnosis Task will be created, and the returned `incidentId` will not be associated with a meaningful workflow.

| # | Requirement | Where configured |
|---|---|---|
| 1 | **Domain Mapping** — define the relationship between each third-party `scope` value and a NetBrain Tenant/Domain pair | System Management |
| 2 | **IT System Mapping** — for each third-party system, define a `source` (address), a `category`, and the JSON `Field` definitions that describe the incoming raw ticket structure | System Management |
| 3 | **Incident Type** — define one or more incident types, each scoped to a `source` + `category` pair, optionally refined with raw-data conditions | Triggered Diagnosis Center |
| 4 | **Diagnosis (NIC, NI, or NIT)** — for each Incident Type, define the diagnosis logic that will run when a matching event arrives | Trigger Automation Manager (NIC) or Intent-Based Automation Center (NI / NIT) |

The relationship between third-party fields and NetBrain mapping is illustrated in the table below (example values):

| `scope` (third-party) | NetBrain Tenant / Domain |
|---|---|
| `USCompany` | `Initial_Tenant` / `Domain01` |
| `EUCompany` | `Initial_Tenant` / `Domain02` |

## Setup Steps

If the prerequisites above are not yet configured, follow these steps in order. Each step describes the UI screen and the inputs required.

**Step 1 — Configure Domain Mapping.** Open System Management and define the mapping between each third-party `scope` value and a NetBrain Tenant / Domain pair, using the table from the Prerequisites section above.

**Step 2 — Configure IT System Mapping.** In System Management, define the third-party system. Two sub-steps:

1. Define the **Source**, **Address**, and **Category** for the third-party system. The `source` and `category` values you record here must exactly match the values sent in `option.source` and `option.category` on each API call.
2. For each Category, define the **Field** entries that describe the JSON keys present in the third-party raw ticket. This tells NetBrain how to interpret the incoming `specificData` payload.

**Step 3 — Configure Incident Type.** Open the Triggered Diagnosis Center and create one or more Incident Types. Three sub-steps:

1. Select the **Source** and **Category** previously configured in Step 2.
2. Optionally, refine the Incident Type with a **Condition** that matches more specific raw-data values (e.g. only fire when `category = "inquiry"` and `severity = "3"`).
3. Optionally, configure **Define Incident** and **Incident Message** settings. Both are optional and can be left as defaults.

**Step 4 — Configure Diagnosis.** Define the diagnosis logic that will run when a matching event is received:

- For **NIC Diagnosis**, use the Trigger Automation Manager page. Select the Incident Type, define a condition, select the NIC, and set the Member NI Filter.
- For **NI Diagnosis** or **NIT Diagnosis**, use the Intent-Based Automation Center page. The configuration flow mirrors the NIC case.

**Step 5 — Run the script and review results.** Call this API with a representative request body (see the Example section below) and review the resulting incident under Incident Management.

## Request body(****required***)

|**Name**|**Type**|**Description**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|||* - mandatory <br> ^ - optional|
|specificData* | Object  | JSON type. The event data from a third party system. |
|option.scope^ | String  | Mandatory parameter in multi-tenancy scenario, if the tenantId and domainId are not indicated in the request to specify a particular working domain.<br> If there is only 1 domain in the entire NetBrain system, this parameter is not required. |
|option.tenantId^ | String  | To specify a particular working tenant. |
|option.domainId^ | String  | To specify a particular working domain. |
|*option.source^ | String  | To specify a trigger source address of an Integrated IT System Data Field definition.<br>The information must match and be pre-defined in a record of Integrated IT System in System Management.  |
|option.category^ | String  | To specify a category of an Integrated IT System Data Field definition.<br>The information must match and be pre-defined in the same record of Integrated IT System in System Management as the indicated source of the same request. |
|option.nbIncidentId^ | String  | Use this parameter to indicate an existing NetBrain incident. <br>This parameter is used to indicate an incident ID returned from an existing triggered task. To do so, a previously matched Incident Type information can be directly picked instead of going through NetBrain incident type lookup process, to prevent a task from utilizing NetBrain system resource unnecessarily.<br>Use as needed to improve event processing performance. **Only use this parameter if you don't want NetBrain system to do the incident type lookup.**  |
|option.taskId^ | String  | Must be provided with nbIncidentId together. The taskId returned from a particular trigger.<br>This parameter is used to indicate a taskId returned from an existing triggered task. To do so, a previously matched Incident Type information can be directly picked instead of going through NetBrain incident type lookup process, to prevent a task from utilizing NetBrain system resource unnecessarily.<br>Use as needed to improve event processing performance. **Only use this parameter if you don't want NetBrain system to do the incident type lookup.** |
|option.ticketId^ | string | Use this parameter to record the raw data. <br> This parameter is not required if ticketId doesn't exist.|
| option.accessCode^ | string | Use this parameter to generate a new incident with the specified value as accessCode. <br>If this value is not set, the accessCode will be randomly generated. |


## Query Parameters(****required***)

> **No query parameter**

## Headers

> **Data Format Headers**

|**Name**|**Type**|**Description**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
| Content-Type | string  | support "application/json" |
| Accept | string  | support "application/json" |

> **Authorization Headers**

|**Name**|**Type**|**Description**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
| token | string  | Authentication token, get from login API. |

## Response

|**Name**|**Type**|**Description**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|incidentId| String | Trigger task generated incident ID. |
|incidentUrl| String | Trigger task generated incident URL. |
|incidentPortalUrl| String | Trigger task generated incident portal URL. |
|tenantId| String | Trigger created tenant ID.  |
|domainId| String | Trigger created domain ID.  |
|taskId| String | Trigger created task ID.  |
|statusCode| integer | The returned status code of executing the API.  |
|statusDescription| string | The explanation of the status code.  |


## Example
```python
import json
import requests
import requests.packages.urllib3 as urllib3
 
urllib3.disable_warnings()

def getTokens(endpoint, user,password):
    login_api_url = r"/V1/Session"
    Login_url = endpoint + login_api_url
    data = {
        "username": user,
        "password": password
    }
    token = requests.post(Login_url, data=json.dumps(
        data), headers=headers, verify=False)
    if token.status_code == 200:
#         print(token.json())
        return token.json()["token"]
    else:
        return "error"

def PublishEvent(endpoint, API_Body):
    API_URL = r"/V1/TAF/Auto"
    api_full_url = endpoint + API_URL
    try:
        api_result = requests.post(api_full_url, data=json.dumps(API_Body), headers=headers, verify=False)
        if api_result.status_code == 200:
            return api_result.json()
        else:
            return api_result.json()
    except Exception as e:
        return str(e)
    
nb_endpoint = "https://netbraintech.com/ServicesAPI/API"
nb_username = ""
nb_password = ""
tenantId = ""
domainId = ""
source = "Python Sample"

specificData = {
    "hostname":"US-BOS-R1",
    "status":"rebooted"
}

api_body = {
    "specificData":specificData,
    "option": {
        "scope":"Demo",
        "source":"https://notapplicable.com",
        "category":"Event"
    }
}

headers = {
    "Accepted":"application/json",
    "Content-Type":"application/json"
}

token = getTokens(nb_endpoint, nb_username, nb_password)
headers["token"] = token

print(PublishEvent(nb_endpoint, api_body))
```

    {'incidentId': '10017G', 'incidentUrl': 'i.html?id=10017G', 'incidentPortalUrl': 'i/10017G', 'taskId': 'f21e2e16-3bcb-4770-aa8f-9d9d0456882c', 'statusCode': 790200, 'statusDescription': 'Success.'}
    

## Formatted Sample JSON Response


```python
{
  "incidentId": "100033",
  "incidentUrl": "i.html?id=100033",
  "incidentPortalUrl": "i/100033",
  "taskId": "5c8930f8-c5c4-4645-94b9-d0f1a4c80a76",
  "statusCode": 790200,
  "statusDescription": "Success."
}
```

## Notes

- **Special characters in TAF Conditions.** When configuring conditions in the Triggered Diagnosis Center (Step 3 in Setup Steps), if the matching value contains a double-quote (`"`) or a backslash (`\`), it must be escaped — enter `\"` and `\\` respectively. Example: to match the literal value `hello!@#$%^&*()_-=+~`:;.\"'|\\/[]{}`, the Condition input must be entered as `hello!@#$%^&*()_-=+~`:;.\\"'|\\\\/[]{}`.
- **Verifying the prerequisites.** If the API returns `statusCode: 790200` but no Triggered Diagnosis Task fires, confirm that all four prerequisites (Domain Mapping, IT System Mapping, Incident Type, Diagnosis) are configured and that the `option.source` / `option.category` values in the request exactly match the IT System Mapping definition.
