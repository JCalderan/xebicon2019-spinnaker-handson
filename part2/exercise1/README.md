# Deployment pipeline
In this exercise, we'll deploy a Front-End application and set-up a pipeline for our development environment.

## Pipeline configuration
First create a new application named "**xebicon-frontend**".
> refer to [exercise 1](../../part1/exercise1/README.md) if you need a reminder about how to create an application

Then head to the "**Pipeline**" tab: 
- click "**configure a new pipeline**"
- enter the pipeline name "**Development**"
- click the "**Create**" button

You created an empty pipeline named "**Development**".
![Pipeline configuration page](./emptyPipelineConfig.png)

This page is the configuration step of the pipeline: every pipeline start with this step.  
We'll get back here later, for now add a new stage:
- select the "**Deploy (Manifest)**" type
- name the stage "**Deploy Service**"
- select the Kubernetes account Spinnaker will use in order to deploy the manifest (only one choice should be available)
- paste the following YAML file in the "**Manifest**" section
- save your changes (bottom right corner)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: xebicon-frontend-service
spec:
  # The service is exposed to the outside
  type: LoadBalancer
  # selectors are used to find and update pods
  selector:
    app: xebicon-frontend
    environment: dev
  ports:
    - protocol: TCP
      # port exposed by the service
      port: 9080
      # port listened by pods
      targetPort: 80
```

This manifest configure a [Kubernetes Service](https://kubernetes.io/docs/concepts/services-networking/service/) which allows other applications to access the xebicon-frontend application's pods.  
The following diagram, from the Kubernetes documentation on the [CoreOS website](https://coreos.com/kubernetes/docs/latest/services.html), 
shows how the service **select** the pods which will receive traffics according the their **labels**.
![Kubernetes Schema diagram](https://coreos.com/kubernetes/docs/latest/img/service.svg)

Now, we need to add a new step in order to deploy the application's pods:
- select the "**Deploy (Manifest)**" type
- name the stage "**Deploy Pods**"
- select the Kubernetes account spinnaker will use in order to deploy the manifest (only one choice should be available)
- paste the following YAML file in the "**Manifest**" section
- save your changes (bottom right corner)  

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: xebicon-frontend-deployment
  labels:
    app: xebicon-frontend
spec:
  replicas: 1
  selector:
    # labels used to find pods managed by this deployment
    matchLabels:
      app: xebicon-frontend
      environment: dev
  # pods template
  template:
    metadata:
      # labels set to pods created by this deployment
      labels:
        app: xebicon-frontend
        environment: dev
    spec:
      containers:
      - name: xebicon-frontend
        image: jcalderan/xebicon-frontend:v1
        ports:
        - containerPort: 80
```

We defined a [Kubernetes ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset) in order to manage our pods. 
All operations on pods (scaling up/down, updating, etc...) will be handled by this manifest.  

You should now have a pipeline resembling this.
![Your pipeline overview](./pipeineOverview.png)


Save your changes and go back to the Pipeline tab: you should see your pipeline named "Development".  
Click the button "**Start manual execution**". After a few seconds, your pipeline execution complete !
![Your pipeline execution complete](./pipelineCompleted1.png)

Take a moment to click on each steps and play with the pipeline results output.  

Go to the "**Infrastructure**" tab : a new Cluster Group has been created, as well as a new Load Balancer.  
Our pipeline ensure new pods won't be deployed if an error occurs when deploying the corresponding Service.

> Using Spinnaker Pipeline, we are able to coordinate deployment stages which depends on each others completion status.

<details>
    <summary>Solution</summary>
    <p>
    Click "**Pipeline Actions**" (upper right), then click "Edit as JSON", and copy paste the following JSON.

```json
{
  "keepWaitingPipelines": false,
  "lastModifiedBy": "anonymous",
  "limitConcurrent": true,
  "stages": [
    {
      "account": "kubernetes",
      "cloudProvider": "kubernetes",
      "manifests": [
        {
          "apiVersion": "v1",
          "kind": "Service",
          "metadata": {
            "name": "xebicon-frontend-service"
          },
          "spec": {
            "ports": [
              {
                "port": 9080,
                "protocol": "TCP",
                "targetPort": 80
              }
            ],
            "selector": {
              "app": "xebicon-frontend",
              "environment": "dev"
            },
            "type": "LoadBalancer"
          }
        }
      ],
      "moniker": {
        "app": "xebicon-frontend"
      },
      "name": "Deploy Service",
      "namespaceOverride": "",
      "refId": "1",
      "requisiteStageRefIds": [],
      "skipExpressionEvaluation": false,
      "source": "text",
      "trafficManagement": {
        "enabled": false,
        "options": {
          "enableTraffic": false,
          "services": []
        }
      },
      "type": "deployManifest"
    },
    {
      "account": "kubernetes",
      "cloudProvider": "kubernetes",
      "manifests": [
        {
          "apiVersion": "apps/v1",
          "kind": "ReplicaSet",
          "metadata": {
            "labels": {
              "app": "xebicon-frontend"
            },
            "name": "xebicon-frontend-deployment"
          },
          "spec": {
            "replicas": 1,
            "selector": {
              "matchLabels": {
                "app": "xebicon-frontend",
                "environment": "dev"
              }
            },
            "template": {
              "metadata": {
                "labels": {
                  "app": "xebicon-frontend",
                  "environment": "dev"
                }
              },
              "spec": {
                "containers": [
                  {
                    "image": "jcalderan/xebicon-frontend:v1",
                    "name": "xebicon-frontend",
                    "ports": [
                      {
                        "containerPort": 80
                      }
                    ]
                  }
                ]
              }
            }
          }
        }
      ],
      "moniker": {
        "app": "xebicon-frontend"
      },
      "name": "Deploy (Manifest)",
      "refId": "2",
      "requisiteStageRefIds": [
        "1"
      ],
      "skipExpressionEvaluation": false,
      "source": "text",
      "trafficManagement": {
        "enabled": false,
        "options": {
          "enableTraffic": false,
          "services": []
        }
      },
      "type": "deployManifest"
    }
  ],
  "triggers": [],
  "updateTs": "1574199631000"
}
```
    </p>
</details>

[previous](../../part1/exercise1/README.md) | [next](../exercise2/README.md)
