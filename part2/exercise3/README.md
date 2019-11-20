# Application configuration
In the [previous exercise](../exercise2/README.md) we learnt to parametrize our pipeline, 
and we added a new parameter "version".

This version v2 of xebicon-frontend use a Back-End application in order to fetch the Xebicon slots to display.  
In order to get it working, we'll need to deploy two new applications:
- xebicon-backend: the API consumed by xebicon-frontend
- xebicon-database: the database hosting the Xebicon slots data 

## Seting up the backend application
Create a new application "**xebicon-backend**", and set up a new pipeline in order to:
- deploy a new Service named "xebicon-backend"
- deploy a Kubernetes ReplicaSet (use the docker image ```jcalderan/xebicon-backend:v1```)

This application is a simple Node.js application which **listen on port 8080**, and exposes **two endpoints**:
- /slots: fetch all slots stored in database
- /health: return ```200 OK {"status": "up"}``` when the application is up and running 

## Setting up the database
Create a new application "**xebicon-database**", and set up a new pipeline in order to:
- deploy a new Service named "xebicon-database"
- deploy a Kubernetes replicaSet (use the docker image ```jcalderan/xebicon-database:v1```)

This application is a single node MongoDB database which **listen on port 27017**.

## Wiring up everything
We need to connect all three components.  

Both the Front-End and the Back-End applications have to be exposed to the Internet through a domain name, 
while the database must only be accessible from the inside of the Cluster.  
The following diagram illustrate this setup.
![Target deployment diagram](./xebia-stack.svg)

In order to access both applications from the Internet, we will use a domain name which resolve to our kubernetes cluster:
- Route53 is a DNS web service by AWS, which has already been provisioned in order to resolve the domain name ```*.my-cluster.com```
- Elastic Load Balancer, is an AWS Load Balancer which has been provisioned in order to forward traffic to the Kubernetes cluster nodes.  
- Ingress Controller is a Kubernetes components which implements Ingress rules in order to root incoming traffic to relevant Services.

Thus, route53 will resolve any sub-domain of ```*.my-cluster.com``` and forward the traffic to the ELB which in turn will forward the traffic to the kubernetes cluster node.
 
Services and Pods names can be resolved [using DNS names](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)
inside a Kubernetes cluster, using a name of the form:
- my-service.my-namespace.svc.cluster-domain.example
- my-pod.my-namespace.svc.cluster-domain.example

This means the application **xebicon-backend** can connect to the **xebicon-database** Service using the DNS name ```xebicon-database.your-namespace.svc.cluster.local```.

### Exercise
- Bake a set of Ingress Rules ???
- Update xebia-frontend service (TODO: give more insights)
- Update xebia-backend service(TODO: give more insights)
- Update xebia-database configmap(TODO: give more insights)

<details>
    <summary>xebicon-frontend solution</summary>
    <p>
    Click "**Pipeline Actions**" (upper right), then click "Edit as JSON", and copy paste the following JSON.

```json
{
  "appConfig": {},
  "keepWaitingPipelines": false,
  "lastModifiedBy": "anonymous",
  "limitConcurrent": true,
  "parameterConfig": [
    {
      "default": "v3",
      "description": "application version",
      "hasOptions": true,
      "label": "version",
      "name": "version",
      "options": [
        {
          "value": "v1"
        },
        {
          "value": "v2"
        },
        {
          "value": "v7"
        }
      ],
      "pinned": false,
      "required": true
    }
  ],
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
                "port": 80,
                "protocol": "TCP"
              }
            ],
            "selector": {
              "app": "xebicon-frontend",
              "environment": "dev",
              "version": "${parameters.version}"
            }
          }
        }
      ],
      "moniker": {
        "app": "xebicon-frontend"
      },
      "name": "Deploy Service",
      "namespaceOverride": "",
      "refId": "1",
      "requisiteStageRefIds": [
        "3"
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
                "environment": "dev",
                "version": "${parameters.version}"
              }
            },
            "template": {
              "metadata": {
                "labels": {
                  "app": "xebicon-frontend",
                  "environment": "dev",
                  "version": "${parameters.version}"
                }
              },
              "spec": {
                "containers": [
                  {
                    "image": "jcalderan/xebicon-frontend:${parameters.version}",
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
    },
    {
      "account": "kubernetes",
      "cloudProvider": "kubernetes",
      "manifests": [
        {
          "apiVersion": "networking.k8s.io/v1beta1",
          "kind": "Ingress",
          "metadata": {
            "annotations": {
              "nginx.ingress.kubernetes.io/rewrite-target": "/$2"
            },
            "name": "xebicon-frontend-ingress"
          },
          "spec": {
            "rules": [
              {
                "http": {
                  "paths": [
                    {
                      "backend": {
                        "serviceName": "xebicon-frontend-service",
                        "servicePort": 80
                      },
                      "path": "/xebicon-frontend(/|$)(.*)"
                    }
                  ]
                }
              }
            ]
          }
        }
      ],
      "moniker": {
        "app": "xebicon-test"
      },
      "name": "Deploy Ingress Rule",
      "refId": "3",
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
    }
  ],
  "triggers": [],
  "updateTs": "1574288839000"
}
```

<details>
    <summary>xebicon-backend solution</summary>
    <p>
    Click "**Pipeline Actions**" (upper right), then click "Edit as JSON", and copy paste the following JSON.

```json
{
  "appConfig": {},
  "keepWaitingPipelines": false,
  "lastModifiedBy": "anonymous",
  "limitConcurrent": true,
  "parameterConfig": [
    {
      "default": "v2",
      "description": "application version",
      "hasOptions": true,
      "label": "version",
      "name": "version",
      "options": [
        {
          "value": "v1"
        },
        {
          "value": "v2"
        }
      ],
      "pinned": false,
      "required": true
    }
  ],
  "stages": [
    {
      "account": "kubernetes",
      "cloudProvider": "kubernetes",
      "manifests": [
        {
          "apiVersion": "v1",
          "kind": "Service",
          "metadata": {
            "name": "xebicon-backend-service"
          },
          "spec": {
            "ports": [
              {
                "port": 80,
                "protocol": "TCP"
              }
            ],
            "selector": {
              "app": "xebicon-backend",
              "environment": "dev",
              "version": "${parameters.version}"
            }
          }
        }
      ],
      "moniker": {
        "app": "xebicon-frontend"
      },
      "name": "Deploy Service",
      "namespaceOverride": "",
      "refId": "1",
      "requisiteStageRefIds": [
        "3"
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
              "app": "xebicon-backend"
            },
            "name": "xebicon-backend-deployment"
          },
          "spec": {
            "replicas": 1,
            "selector": {
              "matchLabels": {
                "app": "xebicon-backend",
                "environment": "dev",
                "version": "${parameters.version}"
              }
            },
            "template": {
              "metadata": {
                "labels": {
                  "app": "xebicon-backend",
                  "environment": "dev",
                  "version": "${parameters.version}"
                }
              },
              "spec": {
                "containers": [
                  {
                    "env": [
                      {
                        "name": "PORT",
                        "value": "80"
                      },
                      {
                        "name": "DB_URL",
                        "value": "mongodb://xebicon-database-service:27017"
                      }
                    ],
                    "image": "jcalderan/xebicon-backend:${parameters.version}",
                    "name": "xebicon-backend",
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
    },
    {
      "account": "kubernetes",
      "cloudProvider": "kubernetes",
      "manifests": [
        {
          "apiVersion": "networking.k8s.io/v1beta1",
          "kind": "Ingress",
          "metadata": {
            "annotations": {
              "nginx.ingress.kubernetes.io/rewrite-target": "/$2"
            },
            "name": "xebicon-backend-ingress"
          },
          "spec": {
            "rules": [
              {
                "http": {
                  "paths": [
                    {
                      "backend": {
                        "serviceName": "xebicon-backend-service",
                        "servicePort": 80
                      },
                      "path": "/xebicon-backend(/|$)(.*)"
                    }
                  ]
                }
              }
            ]
          }
        }
      ],
      "moniker": {
        "app": "xebicon-test"
      },
      "name": "Deploy Ingress",
      "refId": "3",
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
    }
  ],
  "triggers": [],
  "updateTs": "1574290517000"
}
```

<details>
    <summary>xebicon-database solution</summary>
    <p>
    Click "**Pipeline Actions**" (upper right), then click "Edit as JSON", and copy paste the following JSON.

```json
{
  "appConfig": {},
  "keepWaitingPipelines": false,
  "lastModifiedBy": "anonymous",
  "limitConcurrent": true,
  "parameterConfig": [
    {
      "default": "v1",
      "description": "application version",
      "hasOptions": true,
      "label": "version",
      "name": "version",
      "options": [
        {
          "value": "v1"
        }
      ],
      "pinned": false,
      "required": true
    }
  ],
  "stages": [
    {
      "account": "kubernetes",
      "cloudProvider": "kubernetes",
      "manifests": [
        {
          "apiVersion": "v1",
          "kind": "Service",
          "metadata": {
            "name": "xebicon-database-service"
          },
          "spec": {
            "ports": [
              {
                "port": 27017,
                "protocol": "TCP"
              }
            ],
            "selector": {
              "app": "xebicon-database",
              "environment": "dev",
              "version": "${parameters.version}"
            }
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
              "app": "xebicon-database"
            },
            "name": "xebicon-database-deployment"
          },
          "spec": {
            "replicas": 1,
            "selector": {
              "matchLabels": {
                "app": "xebicon-database",
                "environment": "dev",
                "version": "${parameters.version}"
              }
            },
            "template": {
              "metadata": {
                "labels": {
                  "app": "xebicon-database",
                  "environment": "dev",
                  "version": "${parameters.version}"
                }
              },
              "spec": {
                "containers": [
                  {
                    "image": "jcalderan/xebicon-database:${parameters.version}",
                    "name": "xebicon-database",
                    "ports": [
                      {
                        "containerPort": 27017
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
  "updateTs": "1574289097000"
}
```
