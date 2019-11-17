# Blue/Green deployment
## Exercice 1:

You can start the application from this pipeline templates:

Create a new pipeline called Xebicon-service from this template:
You should replace the comment in the template by a values of your choise.

<details><summary>CLICK ME</summary>
<p>
```
{
  "isNew": true,
  "keepWaitingPipelines": false,
  "limitConcurrent": true,
  "stages": [
    {
      "account": "my-k8s-v2-account",
      "cloudProvider": "kubernetes",
      "isNew": true,
      "manifests": [
        {
          "apiVersion": "v1",
          "kind": "Service",
          "metadata": {
            "labels": null,
            "name": null
          },
          "spec": {
            "ports": [
              {
                "port": null,
                "protocol": "TCP",
                "targetPort": null
              }
            ],
            "selector": null,
            "type": "ClusterIP"
          }
        }
      ],
      "moniker": {
        "app": "test1"
      },
      "name": "Deploy (Manifest)",
      "refId": "1",
      "requisiteStageRefIds": [],
      "skipExpressionEvaluation": false,
      "source": "text",
      "trafficManagement": {
        "enabled": false,
        "options": {
          "enableTraffic": false,
          "namespace": null,
          "services": [],
          "strategy": null
        }
      },
      "type": "deployManifest"
    }
  ],
  "triggers": [],
}
```
</p>
</details>

Create a new pipeline called Xebicon-deployment from this template:

<details>
<p>
```
{
  "keepWaitingPipelines": false,
  "limitConcurrent": true,
  "parameterConfig": [
    {
      "default": "",
      "description": "",
      "hasOptions": false,
      "label": "Version",
      "name": "version",
      "options": [
        {
          "value": ""
        }
      ],
      "pinned": false,
      "required": true
    }
  ],
  "stages": [
    {
      "account": "my-k8s-v2-account",
      "cloudProvider": "kubernetes",
      "manifests": [
        {
          "apiVersion": "apps/v1",
          "kind": "Deployment",
          "metadata": {
            "labels": null,
            "name": "pod"
          },
          "spec": {
            "replicas": null,
            "selector": {
              "matchLabels": null
            },
            "template": {
              "metadata": {
                "labels": null
              },
              "spec": {
                "containers": [
                  {
                    "image": null,
                    "name": "pod"
                  }
                ]
              }
            }
          }
        }
      ],
      "moniker": {
        "app": "test1"
      },
      "name": "Deploy (Manifest)",
      "namespaceOverride": "dev",
      "refId": "1",
      "requisiteStageRefIds": [],
      "skipExpressionEvaluation": false,
      "source": "text",
      "trafficManagement": {
        "enabled": true,
        "options": {
          "enableTraffic": true,
          "namespace": "dev",
          "services": [
            "service pod"
          ],
          "strategy": "redblack"
        }
      },
      "type": "deployManifest"
    }
  ],
  "triggers": [],
  "updateTs": "1573933939000"
}
```
</p>
</details>

