# Jupyter GenAI TPCF Deployment Guide

This repository contains the necessary configuration and code to deploy a Jupyter Notebook service integrated with Tanzu GenAI services for TPCF. 

##### Gen AI tile Config for reference: 

<img width="850" height="406" alt="image" src="https://github.com/user-attachments/assets/15ac5c40-e944-43bb-8944-93d09cad4e0d" />


<img width="703" height="656" alt="image" src="https://github.com/user-attachments/assets/fad1e94a-b2ab-4798-a9f3-f91cf159b993" />


##### Clone and Navigate to the Directory

```
cd ./jupyter-genai-tpcf
```

##### Deploy the app

```
cf push
```

##### Create the GenAI Service Instance for Jupyter Notebook

```
cf create-service genai <Plan name> <Service Name>
```

Example:

```
cf create-service genai reddye-llama32-single-plan jupyter-llama-svc
```

##### Bind the created GenAI service with Jupyter Notebook Application

```
cf bind-service jupyter-notebook jupyter-llama-svc
```

##### Restart the Jupyter Notebook Application

```
cf restart jupyter-notebook
```

##### Confirm the Jupyter Notebook App Is Running

```
cf app jupyter-notebook
```

##### Access and Use Jupyter Notebook with GenAI

Open Your Jupyter Notebook

Access the route from the output above in your browser. In the Jupyter UI, go to File > New > Notebook.

##### Install Required Python Packages

Run these in a notebook cell:

```
pip install openai
pip install requests
```

##### Inspect Cloud Foundry Bound Services in Jupyter

To view your services and get credentials:

```
import os
import json
vcap_services = os.getenv("VCAP_SERVICES")
print(json.dumps(json.loads(vcap_services), indent=2))
```

##### Connect and Use the Tanzu GenAI Llama Model

Use this code snippet in your notebook:

```
import os, json, requests
services = json.loads(vcap_services)
llama_service = None
for svc_key, svc_list in services.items():
    for svc in svc_list:
        print("Found service instance:", svc['name'])  # List available service names
        if svc.get('name') == 'jupyter-llama-svc':
            llama_service = svc
            break
    if llama_service:
        break

assert llama_service, "Llama GenAI service binding not found"

credentials = llama_service["credentials"]
api_key = credentials["api_key"]
api_base = credentials["api_base"]
model = credentials.get("model_name", "llama3.2")

headers = {"Authorization": f"Bearer {api_key}", "Content-Type": "application/json"}
payload = {
    "model": model,
    "messages": [
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Hello from Tanzu GenAI Llama model!"}
    ],
    "max_tokens": 50
}

response = requests.post(
    f"{api_base}/chat/completions", headers=headers, json=payload
)
print(response.json())
print(response.json()['choices'][0]['message']['content'])
```

###### Notes
	•	Replace `jupyter-llama-svc` with your actual GenAI service instance name if different.
	•	Always check your credentials and bindings (see step 8) before running the code.
	•	Adjust the model name in `model = credentials.get("model_name", "llama3.2")` if your configuration is different.
