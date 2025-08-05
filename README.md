Gen AI tile Config for reference: 

<img width="850" height="406" alt="image" src="https://github.com/user-attachments/assets/15ac5c40-e944-43bb-8944-93d09cad4e0d" />



##### Deploy

```
cd ./jupyter-genai-tpcf
cf push
```

##### Create Service Instance for jupyter-genai-tpcf

```
cf create-service genai <Plan name> <Service Name>

Example:

cf create-service genai reddye-llama32-single-plan jupyter-llama-svc
```

```
cf restart jupyter-notebook
```

cf app jupyter-notebook

Access the route in the browser > File > New > Notebook

```
pip install openai
```

```
pip install requests
```

```
import os
import json
vcap_services = os.getenv("VCAP_SERVICES")
print(json.dumps(json.loads(vcap_services), indent=2))
```

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
