# Blue/Green deployment
## Exercice 1:

Spinnaker deployment Red/Black is implemented with Kubernetes ReplicaSet.

### Deploy Service

* Create a new pipeline called Xebicon-service.

* Add new stage to the pipeline.

* In the new stage, choose `Deploy manifest` and paste the yaml template bellow 

<details><summary>Service pipeline</summary>
<p>

```

apiVersion: v1
kind: Service
metadata:
  name: xebincon-app
  namespace: dev
spec:
  ports:
    - port: 80
      protocol: TCP
      targetPort: 8080
  selector:
    app: xebincon-app
  type: ClusterIP

```

</p>
</details>

* Save the pipeline

Your service pipeline should like the picture bellow.

![Deploy service](./images/deploy-service.png)

* Run the pipeline

When pipeline finish, you should see your service in the infrastructure section

![Deploy service](./images/deploy-service-pipeline.png)

### Deploy ReplicaSet

* Create a new pipeline called Xebicon-rs.

* Add new stage to the pipeline.

* In the new stage, choose `deploy manifest` and use the yaml template bellow 

<details><summary>Replicaset template</summary>
<p>

```

apiVersion: apps/v1beta2
kind: ReplicaSet
metadata:
  labels:
    applicationName: xebincon-app
  name: xebincon-app
  namespace: dev
spec:
  replicas: 3
  selector:
    matchLabels:
      applicationName: xebincon-app
  template:
    metadata:
      labels:
        applicationName: xebincon-app
    spec:
      containers:
        - image: 'chakch007/node-web-app:${parameters["version"]}'
          name: primary
          ports:
            - containerPort: 8080
```

</p>
</details>

* In the Configuration stage, Add a new parameter named `Version` 

![Deploy service](./images/add-version.png)

Your service pipeline should be like the picture bellow.

![Deploy service](./images/deploy-replicaset.png)

* Save the pipeline

* Run the pipeline with application v1

* You can check that you application is deployed in the infrastructure section

![Deploy service](./images/deploy-replicaset.png)

### Deploy new application version using Red/Black

We are going to update our application version to v2

* Go back to the pipeline Xebicon-rs

* Select the deploy `manifest stage`

* Go to Deploy `manifest stage configuration section`

* check the box, to let spinnaker manages traffic

* Select the namespace where did you deployed the service

* Select the service created in the beginning of the exercice

* Select Strategy: Red/Black

![Deploy service](./images/enable-red-black.png)

* Save the pipeline

* Run the pipeline with v2 version as parameter

* In the infrastructure, you should see a new replicaSet deployed and the previous one disabled


Great! now that we have implemented the Red/Black deployment, we need to implement the rollback pipeline

[previous](../README.md) | [next](../exercice2/README.md)


