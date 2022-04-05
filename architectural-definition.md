# Deployment on APP Service
![spring-demo-app-service](https://user-images.githubusercontent.com/19845873/161813539-2ef2b15d-eb6d-4e2e-88e4-e4bc7a35df9f.jpg)

This architecture is more in line with the requirement, an app service (PaaS) is used to quickly deploy the application containers, monitor and scale the application, with lower implementation and administration costs.

The site is deployed via streaming when a push occurs in the ACR, then you can use the slots to manage blue green deployments

# Deployment on AKS (Optional)
![image](https://user-images.githubusercontent.com/19845873/161798820-d358ac45-97af-4ea7-8e80-66e6ec063a6c.png)

This implementation is recommended when you want to have more control over the platform and the Kubernetes API.

It also allows releases from Azure Devops with approval flows and stages

For this use case that consists of a simple application, it is not recommended to deploy the application on this infrastructure unless there is already a cluster enabled for this purpose, otherwise the costs of implementing the architecture, maintenance and consumption would be very high for this requirement

# **Azure container instance**s
It is not recommended for this requirement

According to the official microsoft documentation
"Azure Container Instances (ACI) provides a single pod of Hyper-V isolated containers on demand. It can be thought of as a lower-level "building block" option compared to Container Apps. **Concepts like scale, load balancing, and certificates are not provided with ACI containers**."
