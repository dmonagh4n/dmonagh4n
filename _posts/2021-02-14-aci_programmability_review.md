---
layout: post
title:  "ACI Programmability Review"
date:   2021-02-14 13:03:10 +0000
categories: jekyll update aci
---

# Abstract

After studying for new DevNet Core and Data Centre automation exams. I realised that there were areas of ACI which weren't clear to me from working through support tickets in day to day operations.

I think it's important to review the architecture of the Management Information Tree within ACI (MIT) before attempting to perform any operations against an ACI Policy Model.  All operations are performed against the Policy or, Logical Model, which in turn are then applied to the lower level, Concrete Model and subsequently, pushed to the Nexus 9Ks below.

### The Management Information Tree

The Management Information Tree (MIT) is composed of both logical and physical compoments. This is the heirarchical arrangement of these components, each node within this heirarchy represents a managed object (MO) or group objects, these can be concrete objects i.e. a switch or logical i.e. Application Profile (AP) or Endpoint Group (EPG), the combination of which results in a complete IP Fabric. 
An administrator will not interact directly with physical components but rather, include these in the ACI Policy Model, defined within the MIT.

```text 
Policy Universe
|
├── APIC Controllers
├── Fabric, Access, Inventory
├── VM Domains
├── Layer 4-7 Services
├── AAA, Security 
└── Tenants, Users, Common
```
Using the logical construct of the Tenant as an example (a commonly created, modified and deleted component), we can see the heirarchical components that build out the software defined constructs for an environment hosted in ACI.

```text 
Tenant
├── L3 Out
|
├── Application Profile
|   └── Endpoint Group
|
├── Bridge Domains
|   ├── VRF/CTX
|   └── Subnet
|
└── Contract
    ├── Filter
    └── Subject
```

While working with the programmability and automation tools within ACI, I constantly refer back to documentation to ensure I am working with the correct components as these map directly to different calls made to the APIC.  

### API Components


#### First Method : get_token()
{% highlight python %}
import requests
import json

def get_token():
''' 
Using basic auth, collect a token to be used for subsequent API calls. This method Uses POST method against the target APIC and returns the token resulting from the request.
'''
    try:
        url = "https://apic.example.com/api/aaaLogin.json"

        payload = {
        "aaaUser": {
            "attributes": {
                "name":"admin",
                "pwd":"correcthorsebatterystaple"
            }
        }
    }

        headers = {
            "Content-Type" : "application/json"
    }
    
        requests.packages.urllib3.disable_warnings()
        response = requests.post(url,data=json.dumps(payload), headers=headers, verify=False).json()
        token = response['imdata'][0]['aaaLogin']['attributes']['token']
        return token
    
    except:
     error = response.raise_for_status()
     printf("An error occured " + error)
{% endhighlight %}

### Cobra SDK    