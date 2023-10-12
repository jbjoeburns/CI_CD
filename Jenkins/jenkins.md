# Jenkins CI-CD pipeline

Aim of CI-CD is to fully automate app deployment from scratch to help deliver software faster, speeding up updates and making more money.

Local code -> github -> automation server (via webhook trigger) -> jenkins -> master node (tests code on agent node) -> pushed to deployment if test successful

Error logs are returned on failed deploy.

![jenkins_pipeline.png](jenkins_pipeline.png)

If this was continuous deployment, then the app would be deployed from the master node.

![jenkins_pipeline_deliv.png](jenkins_pipeline_deliv.png)

This is essentially the same as when you tested deployment with render.

You'll want continuous delivery when you don't want the application to be available to customers right away.
- Continuous delivery lets you do new deploys instead when client specifications change
- Also lets you do additional, manual testing among other reasons

# Webhook

Webhook used for automation.

It always listens to changes in your github.

When changes occur, this repo will be cloned by jenkins.

# Building job in jenkins

- Click new item

![1.png](1.png)

- Name it appropriately 
- Click freestyle then OK

![2.png](2.png)

- Give appropriate description
- Tick 'discard old builds'

![3.png](3.png)

- Go to build and select 'execute shell'
- Include the following code
``` 
whoami
uname -a
```
- Click apply and save
- Can then run this on the list of jobs
- Go on job summary and click on the job under build history to see console log

# Automating start of next job
- Configure new/existing job
- go to bottom and under post build options click build other project
- Type the name of the next job you want

![8.png](8.png)

- **Practically** you'll use this to make jenkins clone the repo if the code deployed successfully

## Connect Jenkins to github
- Create new key pair
- Go to settings then deploy keys
- Put public key in field 
- Tick allow write access

This is the key we will be providing to Jenkins so it can clone our repo.

Then move to **Jenkins**

- When setting up github connection in jenkins
- Tick github project here

![Alt text](11-1.png)

- Then under source code management tick git and paste your repository HTTPS

![Alt text](12-1.png)

- Then add the private key for the key you linked to your GitHub earlier

![Alt text](13-1.png)

- Go to config on **Jenkins** and tick github trigger in build triggers

![14.png](14.png)

You also need to restrict where this can be run using the option under office 365 connector 
- We use the label expression `sparta-ubuntu-node`
- This ensures this only runs on a specific agent/slave node

Then go to settings in **github repo** and go to webhooks in the dropdown

- Then paste the payload URL, which is the URL of Jenkins 
    - Should be in this format `http://address:port/github-webhook/`
- Change content type to application/json

You should have set something like this up. With the exception of deployment.

![Alt text](diagCI-1.png)

## Letting Jenkins merge dev to main

Create dev branch.

Change branch in Jenkins config to **dev**.

![Alt text](17.png)

This will make it so this Jenkins job will try to build the contents of the dev branch to test if the app works.

Then, you create a new job with all the same setup as the first, minus the post build actions, and with a different **build trigger**.

We want this to be built once the test job succeeds.

![Alt text](19.png)

However! You also want to add **merge before build** as an additional behaviour, then specify this should merge with **main**.

![Alt text](18.png)

This creates the following pipeline, that has Jenkins test if the app works, then if it does it allows Jenkins to merge dev with main/master.

![Alt text](jenk_test_merge.png)

## Letting Jenkins create AWS instance

Make new job, and tick SSH agent and add AWS key (using doz' for this)

Make ASG that allows Jenkins IP with Jenkins port (8080)