# Blue/Green deployment
## Introduction
The Blue/Green deployment called Red/Black in Spinnaker is a technique of releasing application new version by switching traffic between to identical enviroment.
This technique prevent common risque associeted with deploying software suck as downtime and rollback capability.
The blue enviromment represent current application version serving production traffic. In Parrarel, the green enviroment is staged running a different version of the application. After the green enviroment is ready and tested, production traffic is redirected from blue to the green enviroment.
If any problems are identified, you can rollback by reverting traffic back to the blue enviromment.
For example, We have an application called Xebicon. This application is served by a service called xebicon service. The version V1 is our blue version of the application

![Blue version](images/B_G-Blue.svg)

We Need to update our application to a version V2. V2 become our green version. We should lunch this version a check if everything is working as exspected

![Lunch Green version](images/B_G-Green.svg)

After Checking that the green version is Ok, we switch the traffic from the blue version to the green version. 

![Lunch Green version](images/B_G-Switch.svg)

Now the user use the application version 2.

After a little time, we noticed that the application have some bugs that we couldn't identify during the validation process. Since, The V1 application is still running and not connected, we can switch back our service to serve the V1 instead of V2

![Switch Back to the blue version](images/B_G-KO.svg)

[previous](../part2/exercise2/README.md) | [next](./exercice1/README.md)

