# Blue/Green deployment
## Exercice 2:

In the  previous Exercice we checked the application manually. We can use two kubernetes feature to check that our application is healthy.
The Livenness and readiness probe to  check that our application is healthy and ready to serve content.
The main difference between this feature is: 
* Liveness: is a check to know when a container need to restart
* readiness: indicate when a pod is ready to serve request 
further infromation are avaible in the [kubernetes documentation](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)

So, lets add this two check to our template pod.
The liveness probe is available via http on port 8081 and path /heathz. 
The readinees probe is available via http on port 8081 /ready

### Task One:
In this Task, we are going to discover the liveness impact on our deployment and pipeline.
You can add liveness probe to the deployment pipeline in the spec section. 
The application version with broken tag response 500 on path /heathz
Deploy this application version
What do you observe in the infrascture section ?

### Task Two:
The previous pipeline fails because our container is not ready to start 
So deploy a new application version with /heathz that response 200 
You can deploy application version V3
What do you observe in the infrascture section ?

### Task Three:
The liveness probe is a flag to indicate that our container is UP, but it can't verify that our application can serve traffic
The readiness probe can be used to check that the application critical component are ready to serve request
You can add readiness probe to the deployment pipeline in the spec section. 
Deploy The application version with brokenReadiness
What do you observe in the infrascture section? 
 
### Task Four:
The previous pipeline fails because our container is not ready to serve request 
So deploy a new application version with /ready that response 200 
You can deploy application version V4
What do you observe in the infrascture section ?
