# Blue/Green deployment
## Exercice 3:

To improve our deployment process, we can use two kubernetes feature to check that our application is healthy and ready to serve traffic.

The Livenness and Readiness probe check that our application is healthy and ready to serve content.

The main difference between this feature is: 

* Liveness: indicates when a pod is "alive"

* readiness: indicate when a pod is ready to serve request 

Further information are available in the [kubernetes documentation](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)

In this exerice we are going to use this two features to improve the Blue/Green deployement

For this exercice:

* The liveness probe is available via http GET on port 8080 and path /heathz. 

* The readinees probe is available via http on port 8080 and path / .

### Task 1: Add LivenessProbe

In this task, we are going to discover the impact of Liveness of on our deployment and pipeline:

* You can add Liveness probe to the deployment pipeline in the spec section. 

* Deploy the version v3.1 of the application

* What do you observe in the infrastructure section ?

<details><summary>Solution</summary>
<p>
The previous pipeline fails because our container is not ready to start 
Screen shot here 
</p>
</details>

* You can deploy application version v3.2

* What do you observe in the infrastructure section ?

### Task 2: Add ReadinessProbe

The liveness probe is a flag to indicate that our container is UP, but it can't verify that our application can serve traffic

The Readiness probe can be used to check that the application critical component are ready to serve request

* Add Readiness probe to the deployment pipeline in the spec section. 

* Deploy The application version v3.3

* What do you observe in the infrastructure section? 

<details><summary>Solution</summary>
<p>
The previous pipeline fails because / response 503
Screen shot here 
</p>
</details>

A new application version is available with / that response 200 

* You can deploy application version v3.4

[previous](../exercice2/README.md)
