# **Steps for Installation on the Ubuntu 16.04 LTS:**

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
















