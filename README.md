# Todo sample for Kubernetes


## Configurable settings

Store config in the environment (aka the [12-factor app](https://12factor.net/) principles).

Public accessible endpoint for todoapi backend, defined in [config.local.yml](k8s/local/config.local.yml):

 - `TODOAPI_HOST` : default = `localhost`
 - `TODOAPI_PORT` : default = `30080`
 - `TODOAPI_PATH` : default = `/api/todo`

A version for deployment on the cloud is also provided in [config.test.yml](k8s/test/config.test.yml):

 - `TODOAPI_HOST` : external IP (ephemeral or static) or domain name allocated by cloud providers.
 - `TODOAPI_PORT` : default = `80`
 - `TODOAPI_PATH` : default = `/api/todo`


## Architecture

A *dockerized* web app with separate frontend and backend services on *Kubernetes* (both locally and on the cloud).

**Frontend**

Static HTML5 files and jQuery scripts.

Local web endpoint:

- host = `localhost`
- port = `30000`

Cloud web endpoint:

- host = external IP (ephemeral or static) or domain name allocated by cloud providers
- port = `80`

**Backend**

Backend program written in ASP.NET Core.

Local API endpoint:

- host = `localhost`
- port = `TODOAPI_PORT` (default = `30080`)
- path = `TODOAPI_PATH` (default = `/api/todo`)

Cloud API endpoint:

- host = `TODOAPI_HOST` (to be revised in [config.test.yml](k8s/test/config.test.yml)), external IP (ephemeral or static) or domain name allocated by cloud providers
- port = `TODOAPI_PORT` (default = `80`)
- path = `TODOAPI_PATH` (default = `/api/todo`)



## Usage

### Preparation

1. If you're using GKE, do the [gke-steps](gke-steps.md) first.

2. Create a `todo` namespace for this app:

   ```
   % kubectl create ns todo
   ```

3. Load the ConfigMap content, if the workshop is to be run locally:

   ```
   % kubectl apply -f k8s/local/config.local.yml  -n todo
   % kubectl get configmaps  -n todo
   ```


4. Load the ConfigMap content, if the workshop is for test on the cloud (GCP/GKE for example):

   ```
   # reserve a new static external IP address for backend todoapi
   % gcloud compute addresses create todoapi --region=us-west1 --network-tier=PREMIUM

   # make sure the static external IP address has been allocated
   % gcloud compute addresses list

   # replace the placeholder string "111.222.333.444" with the allocated IP address
   % vi k8s/test/config.test.yml
   % vi k8s/test/todoapi-service.yml

   #...

   # now, load it!
   % kubectl apply -f k8s/test/config.test.yml  -n todo
   % kubectl get configmaps  -n todo
   ```

5. Fill in correct image names by modifying the `PROJECT_ID` string in the following files:

   - [docker-compose.yml](docker-compose.yml)
   - [k8s/local/todoapi-service.yml](k8s/local/todoapi-service.yml)
   - [k8s/local/todofrontend-service.yml](k8s/local/todofrontend-service.yml)
   - [k8s/test/todoapi-service.yml](k8s/test/todoapi-service.yml)
   - [k8s/test/todofrontend-service.yml](k8s/test/todofrontend-service.yml)


### Build

1. Build images:

   ```
   % docker-compose build
   ```

If you're running the workshop on the cloud, be sure to push the images to registry ([GCR](https://cloud.google.com/container-registry/) for example):

   ```
   % docker push gcr.io/PROJECT_ID/
   ```



### Run at local stage

1. Start the backend:

   ```
   % kubectl apply -f k8s/local/todoapi-service.yml -n todo
   % kubectl get svc  -n todo
   ```

2. Start the frontend:

   ```
   % kubectl apply -f k8s/local/todofrontend-service.yml -n todo
   % kubectl get svc  -n todo
   ```

3. Use your browser to visit the web app at http://localhost:30000


### Run at test stage

1. Start the backend:

   ```
   % kubectl apply -f k8s/test/todoapi-service.yml -n todo
   % kubectl get svc  -n todo
   ```

2. Start the frontend:

   ```
   % kubectl apply -f k8s/test/todofrontend-service.yml -n todo
   % kubectl get svc  -n todo
   ```

3. Use your browser to visit the web app at http://FRONTEND_EXTERNAL_IP:80


## Kubernetes dashboard

See [here](k8s-dashboard.md) if you'd like to use [Kubernetes dashboard](https://github.com/kubernetes/dashboard) locally.


## About the source code

The sample was extracted from the TodoApi demo in the Microsoft Docs site, retrieved on Feb 14, 2019:

 - Document - [Tutorial: Create a web API with ASP.NET Core MVC](https://docs.microsoft.com/zh-tw/aspnet/core/tutorials/first-web-api)

 - Source code - https://github.com/aspnet/Docs/tree/master/aspnetcore/tutorials/first-web-api/samples/2.2/TodoApi


The original source code to be used in this repo is packed in the `TodoApi-original.zip` file for your reference.


## LICENSE

Apache License 2.0.  See the [LICENSE](LICENSE) file.


## History

**6.0**: Support Kubernetes on the cloud (GKE for example).

**5.0**: Support ConfigMap and naming convention.

**4.1**: Use Kubernetes dashboard.

**4.0**: Support Kubernetes (locally).

**3.0**: Separate frontend and backend into 2 distinct containers.

**2.0**: Dockerize the app with simple `Dockerfile` and `docker-compose.yml`.

**1.0**: Extracted from Microsoft Docs.
