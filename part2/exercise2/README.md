# Pipeline parametrization
Each pipeline begin with a step called "Configuration", which contains six sections.
We will go overs all of them, and see how they allow us to make our pipeline reusable. 

## Execution Options
The options here are self explanatory.  
They allow us to adapt ours CD workload to the need of our development platform and the resources available.

## Expected Artifacts
Because pipelines can be triggered by stages in other pipelines, or be triggered by remote services (Circle CI for instance), 
they may depends on external resources produced by theses upstream stages or services in order to work correctly.  
Theses external resources are called [Artifacts](https://www.spinnaker.io/reference/artifacts/in-pipelines/).

Spinnaker defines a specification in order to describe artifacts, and all artifact must match this specification.  
```json
{
  "type":       // How this artifact is classified. Allows for easy distinction
                // between docker images and debian packages, for example.

  "reference":  // The URI.

  "artifactAccount": // The account configured within Spinnaker that has
                // permission to download this resource.

  "name":       // (Optional) A human-readable name that makes matching
                // artifacts simpler.

  "version":    // (Optional) The version of this artifact. By convention, the
                // "version" should be compared against other artifacts with
                // the same "type" and "name".

  "provenance": // (Optional) A link to whatever produced this artifact. This
                // is used for deep-linking into other systems from Spinnaker.

  "metadata":   // (Optional) Arbitrary k/v metadata useful for scripting
                // within stages.

  "location":   // (Optional) The region/zone/namespace this artifact can be
                // found in. This doesn't add information to the URI, but makes
                // multi-regional deployments easier to configure.

  "uuid":       // (Assigned by Spinnaker) Used for tracing artifacts within
                // Spinnaker.
}
```
Thus when an artifact is injected into a pipeline execution context, the pipeline actually receive a JSON payload compliant with the artifact specification.  

When we defined an "**Expected Artifact**", we actually defined a template against which each artifact received by the pipeline will be matched, until one and only one artifact match.  
If no artifacts match, we can choose to use backup strategies, other wise the pipeline will fail.

An artifact matched successfully against an expected artifact is said to be "**bound**" to this expected artifact.  
This allows to reference expected artifacts in downstream stages, with the actual artifact being bound at runtime.

## Automated Triggers
This section allows you to configure [automatic triggers](https://www.spinnaker.io/guides/user/pipeline/triggers/).  

Their is many different triggers with different configuration options each.  
Keep in mind that all triggers can supply artifacts for your pipelines, or supply parameters.

Here is an example payload for a [webHook automated trigger](https://www.spinnaker.io/guides/user/pipeline/triggers/webhooks/):
```json
{
  "parameters": {
    "environment": "dev"
  },
  "artifacts": [
      {
        "type": "docker/image",
        "name": "myImage",
        "reference": "repo/myImage"
      }
    ]
}
```

Note that a pipeline can always be trigger manually, even if it specify an automated trigger.

## Parameters
This section allows us to define parameters for our pipeline.  
Theses parameters can be accessed by downstream stages, and they take the form of key/value pairs with additional meta-data attached, 
as a default value and/or a set of options from which users can pick the value from.  

Parameters, as well as other variables injected in the pipeline execution context, can be referenced using the Spinnaker's [Pipeline Expression Language](https://www.spinnaker.io/reference/pipeline/expressions/).  
In fact, the Pipeline Expression Language is implemented using [SpEL](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#expressions).

> In order to access parameters, you can use SpEL: ${parameters.myParam}

## Notification
This section allows to setup notification channels hooked on the pipeline's lifecycle steps (started, completed, failed). 

# Exercise
Go to the configuration page of the pipeline created in the [previous exercise](./../exercise1/README.md).   

We hardcoded the version of Xebifront to the value 'v1' (see the Service and Deployment manifest).  
Modify the pipeline in order be able to specify the version at runtime:
- add a parameter 'version'
- modify the Service manifest to use this parameter
- modify the Deployment manifest to use this parameter
