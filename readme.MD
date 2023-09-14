## Localy
```
export FLASK_ENV=development  
export FLASK_APP=app/main.py  
flask run 
```

## in Docker container
```
docker build -t zero2azure .  
docker run --rm -it -p 8000:8000 zero2azure
```  

## Deploy to Azure
login, create resource group, create container registry, check container registry credentials:
```
az login  
az group create --name FlaskApp --location "westus"  
az acr create --name dycontreg --resource-group FlaskApp --location "westus" --sku Basic --admin-enabled true  
az acr credential show -n dycontreg
```
login docker to Azure containers, on request use password from previous responce   
```
docker login dycontreg.azurecr.io -u dycontreg
```
Tag your container and push it to the registry:   
```
docker tag zero2azure dycontreg.azurecr.io/zero2azure
docker push dycontreg.azurecr.io/zero2azure
```
create Azure app service plan
```
az appservice plan create --name FlaskAppPlan --resource-group FlaskApp --sku B1
```
Create the web app:
```
az webapp create --name dyFlaskWebApp --resource-group FlaskApp --plan FlaskAppPlan --deployment-container-image-name dycontreg.azurecr.io/zero2azure
```
Configure it to pull from the registry:
```
az webapp config container set --name dyFlaskWebApp --resource-group FlaskApp  --docker-custom-image-name dycontreg.azurecr.io/zero2azure --docker-registry-server-url https://dycontreg.azurecr.io --docker-registry-server-user dycontreg --docker-registry-server-password odE9T6K/JYIWkZVjk1Pn+E+0xzayfrVNXJVUj9ovj6+ACRDjSYP8
```
set the port number on the site and restart it:
```
az webapp config appsettings set --name dyFlaskWebApp --resource-group FlaskApp --settings WEBSITES_PORT=8000
az webapp restart --name dyFlaskWebApp --resource-group FlaskApp
```
Browse to the web app at http://dyFlaskWebApp.azurewebsites.net

