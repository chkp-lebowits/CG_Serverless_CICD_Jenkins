# CG_Serverless_CICD_Jenkins
working example of a Jenkins pipeline integrated with Cloudguard Dome9 Serverless CLI

## Overview

Cloudguad Dome9 supports CICD integration via the use of a small CLI client. The client allows users to 
 * scan function's code  in order to 
     + find known vulenerabilities in dependent libraries, 
     + find embedded credentials, or even 
     + find if the execution role for the function is too permissive, and calculate a role with all and only the permissions the code will actually need to execute its API calls to any AWS service. 
 * add to the function (really, to the CFT/SAM/Serverless framework template, instructions to add) the Cloudguard serverless runtime security layer

The cli and its use is documented with examples in the Cloudguard Dome9 Documentation, and in some more legth in [this repo](https://github.com/dome9/protego-examples).

In here, we're going to provide an annotated example of a Jenkinsfile that implements a pipeline that does both the scanning and the insertion of the runtime layer.

## setting up jenkins

To build this example, I 
1) cloned [the protego-example repo](https://github.com/dome9/protego-examples) that provides the information required about the cli itself. (I also made sure to change the remote on my local copies to be my own repo in github.
2) I added the annotated [Jenkinsfile](/Jenkinsfile) to the repo, commited and pushed it
3) added working github credentials to Jenkins
4) created a new pipepline of type "multibranch pipeline" where the scm source is my github clone repo, with the crendeials from step 1. (I used the "git" SCM plugin but you can just as well use "github")
![Alt setting up source](/Readmepics/scm.JPG?raw=true "scm setup")
5) I let the pipeline file name stay "Jenkinsfile", but you can change it
   
![Alt setting up source](/Readmepics/pipelinesettings.JPG?raw=true "pipeline setup")

and that's about it



## setting up the pipeline
See the accompanying [Jenkinsfile](/Jenkinsfile) for detailed explanation of the pipeline itself
