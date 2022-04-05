# react-and-spring-data-rest

The application has a react frontend and a Spring Boot Rest API, packaged as a single module Maven application.

You can build the application running (`./mvnw clean verify`), that will generate a Spring Boot flat JAR in the target folder.

To start the application you can just run (`java -jar target/react-and-spring-data-rest-*.jar`), then you can call the API by using the following curl (shown with its output):

---

\$ curl -v -u greg:turnquist localhost:8080/api/employees/1
{
"firstName" : "Frodo",
"lastName" : "Baggins",
"description" : "ring bearer",
"manager" : {
"name" : "greg",
"roles" : [ "ROLE_MANAGER" ]
},
"\_links" : {
"self" : {
"href" : "http://localhost:8080/api/employees/1"
}
}
}

---

To see the frontend, navigate to http://localhost:8080. You are immediately redirected to a login form. Log in as `greg/turnquist`


# **Deployment On Azure Devops + Azure Cloud**

## Requirements:
- An Azure Cloud and Azure Devops subscription
- Permission to create resources and service principal
- Admin role on azure devOps project
- Clone or Import the repository from the github to Azure devOps repository


# **Necessary Project files and folders**
![image](https://user-images.githubusercontent.com/19845873/161797657-63055952-2c39-41cb-9cbf-3fb527a3bdb7.png)

- **IaC**: Contains all the ARM templates to deploy Azure Container Registry, App sevice plan, App Service Plan, Autoscale  and role assignment.

![image](https://user-images.githubusercontent.com/19845873/161797830-abbeccd6-161a-43b8-b99c-22684bce28de.png)

- **kubernetes (optional)**:  Contains the yaml manifests  files to deploy on K8 (deployment, service and HPA). Note the cluster, namespaces and the acr-secret must be previously created, it is not cover for this assignment.

![image](https://user-images.githubusercontent.com/19845873/161797864-333beb0c-d816-4434-ae69-cc1c36eb81ff.png)

- **Azure pipelines and docker files**:
1.  azure_pipeline_iac.yaml: This pipeline is necesary to deploy the Azure Container Registry, Web App + App Service Plan and role assignment. It also allows cleanup/detroy function of the all components created for this assignment.
2. azure_pipeline.yaml: This is the CI pipeline to Build the docker image and push to the ACR. In addition, this pipeline publishes an artifact that contains the necessary k8 and IaC files if you want to do continuous delivery using the options of the azure devOps release.
3. Dockerfile: Contains the instrucctions to build the app in a docker image

![image](https://user-images.githubusercontent.com/19845873/161797933-c9579493-8620-4bab-a6a5-856b5492dc95.png)

## Steps:
****
## Infraestructure as Code Pipeline (IaC)

1. Create new pipeline (Pipelines -> New Pipelines)
2. Connect to: "**Azure Repos Git**"
3. Select your project repository
4. Scroll down and select: "**Existing Azure Pipelines YAML file**"
5. Select branch master and path "**/azure_pipeline_iac.yaml**"
6. Press next and in the "**RUN**" button press the arrow and press "**SAVE**"

Important!!: 
Before running the pipeline it is necessary to create the connection service in azure devops
Project settings -> Service connections -> New Service connections -> Azure Resource Manager -> scope: "subscription" -> Name: "sc-ARM"



(Optional) Rename pipeline
1. Go to Pipelines -> Pipelines and press in more options "**Rename/move**"
2. In the input "Name" type: "iac" and press "**Save**"


1. Go to more options and press run pipeline
2. There are two parameters to control the manual execution of the pipeline:
-    **isDeployment** (boolean): To deploy de infraestructure.
-    **isCleaning** (boolean): To use the cleanup/destroy functionality 

## RUN pipeline iac pipeline to deploy the infraestructure
-    **isDeployment**: checked
-    **isCleanup**: unchecked

## RUN pipeline iac pipeline to clean and destroy the infraestructure created infraestructure 
-    **isDeployment**: unchecked
-    **isCleanup**: checked

**Optional if you prefer to use cloud shell (Bash) to deploy the ARM templates**
1. Create the resources groups

`$az group create --location centralus --name rg-demo-spring-acr`
`$az group create --location centralus --name rg-demo-spring-app`

2. Upload the templates
3. Deploy the ARM template using Azure CLI

- Azure Container Registry
`$IaC/azure_container_registry`
`$az deployment group create \`
  `--name deployment-demo-spring-acr \`
  `--resource-group rg-demo-spring-acr \`
  `--template-file  template.json \`
  `--parameters '@parameters.json'`

- App sevice plan, App Service Plan, Autoscale
`$cd IaC/app_service`
`$az deployment group create \`
`--name deployment-demo-spring-app \`
`--resource-group rg-demo-spring-app \`
`--template-file  template.json \`
`--parameters '@parameters.json'`

- Role Assignment system managed identity to pull images from ACR
`$cd IaC/role_assignment`
`$az deployment group create --resource-group rg-demo-spring-acr --template-file template.json`

- Cleanup / detroy
`az group delete --name rg-demo-spring-acr`
`az group delete --name rg-demo-spring-app`

## CI Pipeline

1. Create new pipeline (Pipelines -> New Pipelines)
2. Connect to: "**Azure Repos Git**"
3. Select your project repository
4. Scroll down and select: "**Existing Azure Pipelines YAML file**"
5. Select branch master and path "**/azure_pipeline.yaml**"
6. Press next and in the "**RUN**" button press the arrow and press "**CI-APP**"

Important!!: 
Before running the pipeline it is necessary to create the connection service in azure devops
Project settings -> Service connections -> New Service connections -> Docker Registry -> Registry type: "Azure Container Registry", select your previous created ACR: "demospringcr" -> Service connection name: "sc-acr"

## RUN CI/CD pipeline
Make a push on trigger branch (dev/master) o run manually, the pipeline will build and push the docker image to the acr and deploy on the web app

Then could you use blue green deployment with the slots

If you have and error on the assignment task on iac pipeline run this 
Follow the steps on the Role Assignment system managed identity to pull images from ACR

Refresh the web browser and check the app running: http://demo-spring-app.azurewebsites.net/login
