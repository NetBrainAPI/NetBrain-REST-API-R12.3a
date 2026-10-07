
# Topology API Design

## ***GET*** /V1/CMDB/Topology/Devices/Neighbors{?hostname}&{?topoType}
Use this API to get specific neighbors of a device according to the specified topology type.
<br>

<b>Important</b>: This page describes the `version=0` design of `GET /V1/CMDB/Topology/Devices/Neighbors` - this is not the recommended format.<br>
<br>
It is recommended to pass parameter <i>version=1</i> instead of <i>version=0</i>. <br>
For legacy `version=1` design, see [Get Device Neighbors by Topology Type API](https://github.com/NetBrainAPI/NetBrain-REST-API-R12.3a/blob/main/REST%20APIs%20Documentation/Topology%20Management/Get%20Device%20Neighbors%20by%20Topology%20Type%20API.md)

## Detail Information

> **Title** : Get Device Neighbors by Topology Type API_Version_0<br>

> **Version** : 10/07/2026.

> **API Server URL** : http(s)://IP address of NetBrain Web API Server/ServicesAPI/API/V1/CMDB/Topology/Devices/Neighbors


> **Authentication** : 

|**Type**|**In**|**Name**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|Bearer Authentication| Headers | Authentication token | 

## Request body(****required***)

>No request body.

## Query Parameters(****required***)

|**Name**|**Type**|**Description**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|||* - required <br>^ - optional|
|hostname* | list of string  | The devices name, such as ["US-BOS-R1"] or ["US-BOS-R2", "US-BOS-R3", "US-BOS-R4"]|
|ignoreCase^ | boolean | When set to `true`, each `hostname` is matched case-insensitively against the stored device names. <br>Default: `false`. |
|topoType* | list of strings  | Return the neighbors in specified topology types<br> 1: L3_Topo_Type, <br>2: L2_Topo_Type, <br>3: Ipv6_L3_Topo_Type, <br>4: VPN_Topo_Type, <br>such as [1] or [2,3,4].|
|||If both `hostname` and `topoType` are passed, only the <i>first</i> `hostname` and `topoType` will be used.|
|version^ | string | version=0 |

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
|neighbors | list of object | List of neribor devices and interface.  |
|neighbors.hostname | string | The peer device name.  |
|neighbors.interface | string | The peer interface name. |
|statusCode| integer | The returned status code of executing the API.  |
|statusDescription| string | The explanation of the status code.  |

# Full Example:
```python
full_url = nb_url + "/ServicesAPI/API/V1/CMDB/Topology/Devices/Neighbors"
headers = {'Content-Type': 'application/json', 'Accept': 'application/json'}
headers["Token"] = token

hostname = ["US-ORD-EDGE-SW1"] # Case-Sensitive!
topoType = ["L3_Topo_Type"] # takes strings, not integers

data = {
        "hostname" : hostname,
        "topoType" : topoType,
        "version": 0,
    }

try:
    response = requests.get(full_url, params = data, headers = headers, verify = False)
    if response.status_code == 200:
        result = response.json()
        print (result)
    else:
        print ("Failed to Get Neighbors by Topology! - " + str(response.text))
    
except Exception as e:
    print (str(e)) 
```
```python
{
  "neighbors": [
    {
      "hostname": "BUR-R206-Forti200F-1_(NB_Consult)",
      "interface": "Lab_DMZ_Vlan400 172.16.12.1/22"
    },
    {
      "hostname": "Core-ISP1-R1",
      "interface": "GigabitEthernet7 172.16.14.131/22"
    }
  ],
  "statusCode": 790200,
  "statusDescription": "Success."
}
```

# Full Example 2: Using `ignoreCase`

The `ignoreCase` parameter lets you supply `hostname` in any letter case; the server normalizes the match against the stored device name. In this `version=0` example, the device is stored as `ASA-Router` but the request sends `asa-router` (all lowercase) and still returns neighbor data.

```python
full_url = nb_url + "/ServicesAPI/API/V1/CMDB/Topology/Devices/Neighbors"
headers = {'Content-Type': 'application/json', 'Accept': 'application/json'}
headers["Token"] = token

hostname = ["asa-router"]
topoType = ['L3_Topo_Type']  # takes strings, not integers

data = {
        "hostname": hostname,
        "topoType": topoType,
        "version": 0,
        "ignoreCase": True
    }

try:
    response = requests.get(full_url, params=data, headers=headers, verify=False)
    if response.status_code == 200:
        result = response.json()
        print(result)
    else:
        print("Get neighbors by topology failed! - " + str(response.text))

except Exception as e:
    print(str(e))
```

Sample response:

```python
{
  "neighbors": [
    {
      "hostname": "!@#$%^&*()_-=+~`:;.'|\\/[]{}",
      "interface": "Ethernet1/3 172.25.52.9/24"
    },
    {
      "hostname": "ASA",
      "interface": "Ethernet1/3 172.25.52.1/24"
    },
    {
      "hostname": "ASA,Router",
      "interface": "Ethernet1/3 172.25.52.2/24"
    },
    {
      "hostname": "ASA.Switch",
      "interface": "Ethernet1/3 172.25.52.6/24"
    },
    {
      "hostname": "ASA@Switch",
      "interface": "Ethernet1/3 172.25.52.7/24"
    },
    {
      "hostname": "ASA\\Router",
      "interface": "Ethernet1/3 172.25.52.4/24"
    }
  ],
  "statusCode": 790200,
  "statusDescription": "Success."
}
```

# cURL Code from Postman:
```python
curl -X GET \
  'http://192.168.28.79/ServicesAPI/API/V1/CMDB/Topology/Devices/Neighbors?hostname=R1&topoType=L3_Topo_Type' \
  -H 'Postman-Token: d43de85c-8de9-4bcf-be28-9bc16ce7b329' \
  -H 'cache-control: no-cache' \
  -H 'token: 3d0f475d-dbae-4c44-9080-7b08ded7d35b'
```

# Error Examples:


```python
###################################################################################################################    

"""Error 1: empty inputs"""

Input:
        
        hostname = "" # Cannot be null.
        topoType = "" # Cannot be null.

Response:
    
  {'statusCode': 791001, 'statusDescription': 'The parameter "topoType" is required.'}

        
###################################################################################################################    

"""Error 2: wrong inputs"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Input:
        
        hostname = "hahahh" # No device with a hostname called "hahahh"
        topoType = "L3_Topo_Type"

Response:
    
  {'statusCode': 791006, 'statusDescription': 'hostname does not exist.'}


#--------------------------------------------------------------------------------------------------------------------        
    
Input:
        
        hostname = "R1" 
        topoType = "XXXX" # No topology type called "XXXX".

Response:

  {'statusCode': 791001, 'statusDescription': "Invalid parameter: the parameter 'topoType' is invalid."}

```
