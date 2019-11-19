# Blue/Green deployment
## Introduction


The Blue/Green deployment called Red/Black in Spinnaker is a technique of releasing application new version by switching traffic between two identical environments.

This technique prevents common risque associated with deploying software suck as downtime and rollback capability.

The blue environment represents the current application version serving production traffic. In parallel, the green environment is staged running a different version of the application. After the green environment is ready and tested, production traffic is redirected from blue to the green environment.

If any problems are identified, you can rollback by reverting traffic back to the blue environment.

For example, We have an application called Xebicon-app. This application is served by a service called Xebicon-service. The version V1 is our blue version of the application

<p align="center">
  <img src="images/B_G-Blue.svg"/>
</p>

We Need to update our application to a version V2. V2 become our green version. We should lunch this version a check if everything is working as expected

<p align="center">
  <img src="images/B_G-Green.svg"/>
</p>

After Checking that the green version is Ok, we switch the traffic from the blue version to the green version. 

<p align="center">
  <img src="images/B_G-Switch.svg"/>
</p>

Now the user use the application version 2.

After a little time, we noticed that the application has some bugs that we couldn't identify during the validation process. Since, The V1 application is still running and not connected, we can switch back our service to serve the V1 instead of V2

<p align="center">
  <img src="images/B_G-KO.svg"/>
</p>

[previous](../part2/exercise2/README.md) | [next](./exercice1/README.md)

