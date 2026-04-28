
This demo shows how to deploy a Docker image to an Azure Container Instance. The steps involved are:

1- Creating a custom container image for a Flask application

2- Creating an Azure Container Registry (ACR)

3- Pushing the custom container image to ACR

4 -Creating an Azure Container Instance (ACI) from the custom container image stored in the ACR.


## 1- Creating a custom container image 

```bash
cat > app.py <<EOF
from flask import Flask

app = Flask(__name__)

@app.route("/")
def hello_world():
    return "<p>Hello, World!</p>"

if __name__ == "__main__":
    app.run(debug=True, host='0.0.0.0', port=5000)
EOF
```


```bash
cat > Dockerfile <<EOF
FROM python:3.8
WORKDIR /app
COPY . /app
RUN pip install flask
EXPOSE 5000
ENTRYPOINT ["python"]
CMD ["app.py"]
EOF
```

```bash
podman build -t flask-app:v1 .
```


## 2- Creating an Azure Container Registry (ACR)

```bash
az acr create --name $ACRName --resource-group $RGName --sku standard --admin-enabled true
```

```bash
az acr credential show --name $ACRName --resource-group $RGName
```

```bash
podman login ${ACRName}.azurecr.io
```


## 3- Pushing the custom container image to ACR

```bash
podman tag localhost/flask-app:v1 ${ACRName}.azurecr.io/flask-app:v1
```

```bash
docker push ${ACRName}.azurecr.io/flask-app:v1
```

```bash
skopeo inspect docker://${ACRName}.azurecr.io/flask-app:v1
```

```bash
az acr repository list --name $ACRName --resource-group $RGName
```

```bash
az acr repository show --repository flask-app --name $ACRName --resource-group $RGName
```

## 4- Creating an Azure Container Instance (ACI)

```bash
ACInstanceName=instance-name
ContainerDNSName=container-dns-name
RegistryUsername=registry-username
RegistryPassword=registry-password

az container create --resource-group $RGName \
    --name $ACInstanceName --image ${ACRName}.azurecr.io/flask-app:v1 \
    --dns-name-label $ContainerDNSName
    --registry-username $RegistryUsername \
    --registry-password $RegistryPassword \
    --ports 5000
    --cpu 1 --memory 1.5
```


Get the IP address from the portal and access it.

```bash
curl 4.239.90.101:5000
<p>Hello, World!</p>%
```                   


