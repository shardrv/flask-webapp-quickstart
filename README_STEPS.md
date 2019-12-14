This tutorial shows you how to create and deploy a Flask web app in an Alpine Linux + NGINX Docker container to Azure Web Apps for Containers using the Azure CLI.

See [VSCODE.md](VSCODE.md) for using Visual Studio Code and the Azure Portal instead.

## Prerequisites
To complete this tutorial:

- [Install Python 3](https://www.python.org/downloads/)
- [Install Docker Community Edition](https://www.docker.com/community-edition)
- (Optional) [Install the Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli)

## Create app and run it locally

First create a root folder, name it e.g. ```flask-quickstart``` and create a single ```app``` folder inside the root folder. Now let's write our app, copy and paste the following code into ```app/main.py```:

```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello_world():
  return 'Hello, World!'

if __name__ == '__main__':
  app.run()
```

Now let's create a dockerfile for the app, we'll use an Alpine Linux container with an NGINX web server. Put the following in a file named `Dockerfile` inside of the ```flask-quickstart``` folder:

```Dockerfile
FROM tiangolo/uwsgi-nginx-flask:python3.6-alpine3.7

ENV LISTEN_PORT=8000
EXPOSE 8000

COPY /app /app
```

Now build and run the docker container:
```
docker build --rm -t flask-quickstart .
docker run --rm -it -p 8000:8000 flask-quickstart
```

Open a web browser, and navigate to the sample app at ```http://localhost:8000```.

You can see the Hello World message from the sample app displayed in the page.

![Flask app running locally](https://docs.microsoft.com/en-us/azure/app-service/media/app-service-web-get-started-python/localhost-hello-world-in-browser.png)

In your terminal window, press Ctrl+C to exit the web server and stop the container.

If you make code changes you can re-run the docker build and run commands above to update the container.

## Deploy the container to Azure

For this portion of the tutorial you will need the Azure CLI. Either install it locally, or you can run commands in the browser by navigating to the [Azure Cloud Shell](https://shell.azure.com/bash). As you are running the commands below, you can view the resources being created in the Azure portal by navigating to [portal.azure.com](https://portal.azure.com).

Create a resource group:
```
az group create --name FlaskApp --location "West US"
```

Create a container registry and retrieve the password, note that `<registry_name>` needs to be a unique name: 
```
az acr create --name <registry_name> --resource-group FlaskApp --location "West US" --sku Basic --admin-enabled true
az acr credential show -n <registry_name>
```

You see two passwords. Make note of the user name and the first password.
```JSON
{
  "passwords": [
    {
      "name": "password",
      "value": "<registry_password>"
    },
    {
      "name": "password2",
      "value": "<registry_password2>"
    }
  ],
  "username": "<registry_name>"
}
```

Log in to your registry. When prompted, supply the username and password shown above.
```bash
docker login <registry_name>.azurecr.io -u <registry_name>
```

Tag your container and push it to the registry:
```
docker tag flask-quickstart <registry_name>.azurecr.io/flask-quickstart
docker push <registry_name>.azurecr.io/flask-quickstart
```

Create the app service plan:
```
az appservice plan create --name FlaskAppPlan --resource-group FlaskApp --sku B1 --is-linux
```

Create the web app:
```
az webapp create --name <app_name> --resource-group FlaskApp --plan FlaskAppPlan --deployment-container-image-name "<registry_name>.azurecr.io/flask-quickstart"
```

Configure it to pull from the registry:
```
az webapp config container set --name <app_name> --resource-group FlaskApp --docker-custom-image-name <registry_name>.azurecr.io/flask-quickstart --docker-registry-server-url https://<registry_name>.azurecr.io --docker-registry-server-user <registry_name> --docker-registry-server-password <registry_password>
```

Run the following command to set the port number on the site and restart it:
```
az webapp config appsettings set --name <app name> --resource-group FlaskApp --settings  WEBSITES_PORT=8000
az webapp restart --name <app name> --resource-group FlaskApp
```

Browse to the web app at ```http://<app_name>.azurewebsites.net```

## Install additional libraries

To install additional libraries, first create a virtual environment locally, install packages and then generate a requirements.txt file.

On Windows:
```
py -3 -m venv env
env\scripts\activate
pip install flask <list of other libraries>
pip freeze > requirements.txt
deactivate
```

On Linux/Unix/macOS:
```
python3 -m venv env
env/bin/activate
pip install flask <list of other libraries>
pip freeze > requirements.txt
deactivate
```

Add the following code to the dockerfile so that the additional requirements are installed:
```
# Install additional requirements from a requirements.txt file
COPY requirements.txt /
RUN pip install --no-cache-dir -U pip
RUN pip install --no-cache-dir -r /requirements.txt
```

## Clean up

To clean up your Azure resources, delete the resource group
```az group delete --name FlaskApp```


# **Appended Steps for Installation on the Ubuntu 16.04 LTS:**

## Ubuntu system terminal 

- Make a new directory and clone the git repo
  ``` 
  mkdir -p Azure
  cd Azure
  ```
- Clone the repo
  ```
  git clone https://github.com/qubitron/flask-webapp-quickstart
  ```

- Navigate to the directory and install the requirements for the web app
  ```
  cd flask-webapp-quickstart
  pip install -r requirements.txt
  ```

- Within the directory, make the virtual environment and activate it
  ```
  virtualenv -p /usr/bin/python3 virtualenvironment/pyvirenv
  cd virtualenvironment/pyvirenv/bin
  source activate
  ```
    - Note: you can deactivate the virtualenv anytime using ```deactivate``` command. For more info please see the link: https://www.liquidweb.com/kb/creating-virtual-environment-ubuntu-16-04/

- Navigate to the main directory and open it in Visual Studio Code

  ```
  cd ~/Azure/flask-webapp-quickstart
  code .
  ```

- Select the correct python interpreter from the lower half corner of the VS Code screen and the select the virtual environment python interpreter

    ![Screenshot](https://github.com/shardrv/flask-webapp-quickstart/blob/master/Images/vscodeshot.png)

- A pop up showing the installation requirement of pylint comes up. Click Install and let it run in the VS Code terminal.
  - Fix for  
  ```Can not perform a '--user' install. User site-packages are not visible in this virtualenv``` 
  Remove the ```--user ``` from the VS Code terminal and run the command again

- Now right click on the main.py file from the navigation bar and select **Run Python File in Terminal**. This will start the web app with the server address. If you use ```Ctrl+Click``` on the web address, it opens it in the web page. 


## Dockerize and Deploy to Azure Web Apps

- Use ```Ctrl+Shift+P``` in VS Code to open the command interface on the app. Search for **Azure Sign In** and hit Enter. It will provide the login details for Microsoft Azure and then links the VS Code with Microsoft Azure. 
- Click on **Create Resource** in the dashboard.
  - Use the basic subscription, with unique username and login enabled to login with the credentials 

- Once the Container has been deployed, goto the Access Keys of the resource to see the login details

- Go back to VS Code command line 
  ```
  docker login [Container name].azurecr.io
  ```
  - Enter the username and password as in the resource group detials

- VS Code terminal hit ```Ctrl+Shift+P``` then ```Docker: Build Image``` select the ```Dockerfile``` . The prompt will ask for the image name so the prefix needs to be added with the name of the container as: ```[containername].azurecr.io/flask-webapp-quickstart:latest```

- Go to the **Docker** section of the VS Code and find the webapp currently built. Right click and click on **Push**. This will upload the container to Azure App service.  

- Check out the **Registries** tab in the **Docker** extension in VSCode. The version of the deployed app image will be listed here. Right click on the image file and click on **Deploy Image to Azure App Service**. Go through the prompts of creating the service name and deploy the app.

- Set the port number as the Dockerfile has the docker container listening to port 8000. Go to Azure App Service -> App name -> Application Settings -> Right click to add new setting -> Type ```WEBSITE_PORT``` set to ```8000```. Then right click and to restart the app and then click **Browse Website** to view.
















