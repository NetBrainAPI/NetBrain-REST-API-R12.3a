
# Get Data Center Status API Design

## ***GET*** API/V1/data-center/status
This API is used to retrieve the current status of the data center, such as whether it is active, inactive, activating, or deactivating.

<b>Note</b>: Call this API against the web service URI of the data center whose status you want to query.

## Detail Information

> **Title** : Get Data Center Status<br>

> **Version** : 06/07/2026

> **API Server URL** : http(s):// IP address of your NetBrain Web API Server/ServicesAPI/API/V1/data-center/status

> **Authentication** :

|**Type**|**In**|**Name**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|Bearer Authentication| Headers | Authentication token |

## Request body(****required***)

>No request body.

## Parameters(****required***)

|**Name**|**Type**|**Description**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|||* - required<br />^ - optional|
|timeout^|int|Timeout setting for the operation, in seconds. Defaults to `30` if not provided.|


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
|success| bool | Indicates whether the operation was successful or not. |
|status| int | The current data center status. <br>`-2`: ERROR<br>`-1`: UNSUPPORTED<br>`0`: INACTIVE<br>`1`: ACTIVE<br>`2`: DEACTIVATING<br>`3`: ACTIVATING |
|statusCode| int | The returned status code of executing the API. |
|statusDescription| string | The explanation of the status code. |


# Full Example:
```python
import requests
import urllib3
import json
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

nb_url = "http(s)://<your-netbrain-web-api-server>"
token = "<your-auth-token>"

full_url = nb_url + "/ServicesAPI/API/V1/data-center/status"
headers = {'Content-Type': 'application/json', 'Accept': 'application/json'}
headers["Token"] = token

# `timeout` is optional; defaults to 30 seconds if omitted.
params = {"timeout": 30}

try:
    response = requests.get(full_url, params=params, headers=headers, verify=False)
    if response.status_code == 200:
        result = response.json()
        print(result)

        # Translate the numeric `status` into a human-readable label.
        status_labels = {
            -2: "ERROR",
            -1: "UNSUPPORTED",
             0: "INACTIVE",
             1: "ACTIVE",
             2: "DEACTIVATING",
             3: "ACTIVATING",
        }
        label = status_labels.get(result.get("status"), "UNKNOWN")
        print(f"Data center status: {label}")
    else:
        print("Failed to Get Data Center Status! - " + str(response.text))
except Exception as e:
    print(str(e))
```
```python
{'status': -1, 'statusCode': 790200, 'statusDescription': 'Success.'}
Data center status: ACTIVE

```

# cURL Code from Postman
```python
curl -X GET \
  'http://<your-netbrain-web-api-server>/ServicesAPI/API/V1/data-center/status?timeout=30' \
  -H 'Content-Type: application/json' \
  -H 'cache-control: no-cache' \
  -H 'token: <your-auth-token>'
```
