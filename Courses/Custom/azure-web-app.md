
This demo shows how to deploy to deploy and host an Azure Web App with Azure App Service. The steps involved are:

1- Creating a Flask application

2- Creating an Azure Web App

3- Deploying Code to the Web App


## 1- Creating a Flask application

Create a starter web application using the Flask web-application framework.

```bash
# Creating a working directory
mkdir ~/webapp-demo && cd ~/webapp-demo
```

```bash
# Creating a Python virtual environment
python3 -m venv venv
source venv/bin/activate
pip install flask
```

```bash
# Creating the Python application

cat > app.py <<EOF
from flask import Flask

app = Flask(__name__)

@app.route("/")
def hello_world():
    return "<html><body><h1>Hello, World!</h1></body></html>\n"

if __name__ == "__main__":
    app.run(debug=True, host='0.0.0.0', port=5000)
EOF
```

To deploy your application to Azure, you need to save the list of application requirements you made for it in a requirements.txt file. 

```bash
pip freeze > requirements.txt
```

To test your app locally, you need Python 3 and Flask installed on your system


## 2- Creating an Azure Web App

Create an App Service Plan that will be used to host the Web App

```bash
export APPPLAN=Demo-Plan
export APPRG=1-b63e2e29-playground-sandbox
export APPLOCATION="Canada East" 
export APPSKU=S1 # F1 for Free, B1 for Basic, S1 for Standard, or P1V3 for Premium
```
> [!NOTE]
> To see a full list of valid regions currently available for your subscription:
> ```bash
> az account list-locations --query "[].{DisplayName:displayName, Name:name}" -o table
> ```

```bash
# Create the App Service Plan
az appservice plan create \
  --name $APPPLAN \
  --is-linux \
  --resource-group $APPRG \
  --location "$APPLOCATION" \
  --sku $APPSKU 
```

Create the Web App

```bash
export APPNAME=Webapp-Demo
```

```bash
az webapp create \
  --resource-group $APPRG \
  --plan $APPPLAN \
  --name $APPNAME \
  --runtime "PYTHON:3.12"
```

```bash
az webapp list --output table
```

## 3- Deploying Code to the Web App

```bash
# Initialize the required resources names in bash variables
export APPNAME=$(az webapp list --query [0].name --output tsv)
export APPRG=$(az webapp list --query [0].resourceGroup --output tsv)
export APPPLAN=$(az appservice plan list --query [0].name --output tsv)
export APPSKU=$(az appservice plan list --query [0].sku.name --output tsv)
export APPLOCATION=$(az appservice plan list --query [0].location --output tsv)
```

```bash
az webapp up --name $APPNAME \
    --resource-group $APPRG \
    --plan $APPPLAN --sku $APPSKU \
    --location "$APPLOCATION"
```

An alternative method is to push your local code to the web app is using the Zip deploy method.


```bash
# cd to the working directory
cd ~/webapp-demo
```

First, zip your working directory content.

Using `Bash`
```
# Zip its content to a deploy.zip folder
zip -r webapp-deploy.zip .
```

Or using `PowerShell`

```powershell
Compress-Archive -Path * -DestinationPath deploy.zip
```

Then, push the zipped folder.

```bash
az webapp deploy source config-zip \
  --resource-group $APPRG \
  --name $APPNAME \
  --src webapp-deploy.zip
```





              


