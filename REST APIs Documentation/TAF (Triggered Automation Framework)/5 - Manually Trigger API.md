
# Manually Trigger API Design

## ***POST*** /V1/TAF/Manually
Use this API to make a specific trigger task.

## Detail Information

> **Title** : Manually Trigger API

> **Version** : 06/04/2026

> **API Server URL** : http(s)://IP address of NetBrain Web API Server/ServicesAPI/API/V1/TAF/Manually

> **Authentication** : 

|**Type**|**In**|**Name**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|Bearer Authentication| Headers | Authentication token | 

## Request body(****required***)

|**Name**|**Type**|**Description**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|specificData* | Object  | JSON type. The event data from a third party system. |
|option.scope | String  | Mandatory parameter in multi-tenancy scenario, if the tenantId and domainId are not indicated in the request to specify a particular working domain.<br> If there is only 1 domain in the entire NetBrain system, this parameter is not required. |
|option.tenantId | String  | To specify a particular working tenant. |
|option.domainId | String  | To specify a particular working domain. |
|option.source* | String  | To specify a trigger source address of an Integrated IT System Data Field definition.<br>The information must be pre-defined in a record of Integrated IT System in System Management.  |
|option.category | String  | To specify a category of an Integrated IT System Data Field definition.<br>The information must be pre-defined in the same record of Integrated IT System in System Management as the indicated source of the same request. |
|option.nbIncidentId | String  | This parameter is used to indicate the incident ID returned from an existing triggered task. To do so, a previously matched Incident Type information can be directly picked instead of going through NetBrain incident type lookup process, to prevent a task from utilizing NetBrain system resource unnecessarily.<br>Use as needed to improve event processing performance. **Only use this parameter if you don't want NetBrain system to do the incident type lookup.**  |
|option.taskId | String  | The taskId returned from a particular trigger.<br>This parameter is used to directly use the taskId previously returned by Auto Trigger. A previously matched Incident Type information can be directly picked instead of going through NetBrain incident type lookup process, to prevent a task from utilizing NetBrain system resource unnecessarily.<br>Use as needed to improve event processing performance. **Only use this parameter if you don't want NetBrain system to do the incident type lookup.** |
|option.ticketId | string | Use this parameter to record the raw data ticketId. <br> This parameter is not required if ticketId doesn't exist. |
| option.accessCode | string | Use this parameter to generate a new incident with the specified value as accessCode. <br>If this value is not set, the accessCode will be randomly generated.|
|option.diagnosisId*| String | To specify a Triggered Diagnosis definition ID. |
|option.manuallyParams| Array | Triggered Diagnosis Filter for Member Network Intent setting. |
|option.manuallyParams[].key| String | Key name. Keys can be obtained by `API/V1/CMDB/TAF/ManuallyDiagnosis API` (`Get Triggered Diagnosis Definition API`) |
|option.manuallyParams[].value| String | Value of a key. |


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
|tenantId| String | The task execution context - Tenant ID. |
|domainId| String | The task execution context - Domain ID. |
|incidentId| String | Trigger task generated incident ID. |
|incidentUrl| String | Trigger task generated incident URL. |
|incidentPortalUrl| String | Trigger task generated incident portal URL. |
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

def ManualTrigger(endpoint, API_Body):
    API_URL = r"/V1/TAF/Manually"
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
        "category":"Event",
        "diagnosisId":"79756585-4d0f-ac3b-c26e-aab19bf0eaf1",
        "manuallyParams": [
            {
                "key":"NIDevice",
                "value":"US-BOS-R1",
            }
        ]
    }
}

headers = {
    "Accepted":"application/json",
    "Content-Type":"application/json"
}

token = getTokens(nb_endpoint, nb_username, nb_password)
headers["token"] = token

print(ManualTrigger(nb_endpoint, api_body))
```

    {'incidentId': '10015N', 'incidentUrl': 'i.html?id=10015N', 'incidentPortalUrl': 'i/10015N', 'taskId': '3065fa7a-d50b-4752-b9be-8f8e1a4b56e6', 'statusCode': 790200, 'statusDescription': 'Success.'}
    

## Formatted Sample JSON Response


```python
{
  "incidentId": "100035",
  "incidentUrl": "i.html?id=100035",
  "incidentPortalUrl": "i/100035",
  "taskId": "33702857-434c-4be0-8ae3-4e86c44e67a9",
  "statusCode": 790200,
  "statusDescription": "Success."
}
```
