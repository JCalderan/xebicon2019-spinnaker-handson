# Blue/Green deployment
## Exercice 2:

Now You become familiar with Spinnaker, you can create the rollback pipeline.
Let's create the pipeline bellow:

![Switch Back to the blue version](./images/rollback.png)

* Create new Stage: this stage enable the v1 application version
* This Stage should depends on the pipeline input
* Choose enable manifest and select the second newest version
* Create another stage: this stage will disable the v2 application version 
* This Stage should depends on the pipeline input
* Choose enable Manifest and select the newest version
* Run the pipeline
* You can check the infrastructure to check that the rollback is successful

How can you improve this pipeline to prevent downtime?


<details><summary>Solution</summary>
<p>


![Solution](./images/solution.png)


</p>
</details>


[previous](../exercice1/README.md) | [next](../exercice3/README.md)

