# Blue/Green deployment
## Exercice 2:

Now You become familiar with spinnker, you can create the rollback pipeline.
Let's create the pipeline bellow:

![Switch Back to the blue version](./images/rollback.png)

* Create new Stage: this stage is enable the previous application version
* Choose enable manifest and select the second newest version
* Create another stage: this stage will disable the newest version 
* Choose enable Manifest and select the second newest version

How can you improve this pipeline to prevent downtime?


