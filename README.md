# Adding optional Azure content

### Prerequisites

**Setup that the learner needs to do OUTSIDE of the course repository**

- Every user/learner will need an Azure account, which they can create [here](https://azure.microsoft.com/en-us/free/search/?&ef_id=EAIaIQobChMIsYvv96eg5wIVyh6tBh0tYQqKEAAYASAAEgJ6WPD_BwE:G:s&OCID=AID2000128_SEM_bp6n8E3v&MarinID=bp6n8E3v_287547098344_azure%20free%20trial_e_c__44568975817_kwd-298648055668&lnkd=Google_Azure_Brand&gclid=EAIaIQobChMIsYvv96eg5wIVyh6tBh0tYQqKEAAYASAAEgJ6WPD_BwE)
- Once the account is created the user/learner will need to create a new [subscription](https://docs.microsoft.com/en-us/azure/cost-management-billing/manage/create-subscription)
  - This process **requires** billing information, however all of the resources we use will fall under the FREE tier for the first 12 months after initial account creation
- Install the [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest) on a local machine

**Additional setup required for the existing course repository**

- Place the three workflow files located in *this* repository into the `.github/` directory on the `master` branch of the *course template* repository.
  - We can't place these in the `.github/workflows` directory becuase they wont copy if we do.  This is desired for their use-case however, we will have the user/learner add them to the `.github/workflows` directory when they need to be used.
- Two branches need to be created.  These will be used for automatic environment provisioning and tear-down.
  - `create-env`
  - `destroy-env`
  
---

### First step: configuring secrets

The workflows rely on actions that consume [secrets](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/creating-and-using-encrypted-secrets#creating-encrypted-secrets) located within the repository.  

The first secret that needs to be configured is the user/learners `AZURE_SUBSCRIPTION_ID` 

**Getting the user/learners `AZURE_SUBSCRIPTION_ID`:**

1. Open a terminal and run `az login`.  This command will open a web-browser where the user/learner will login with the account they created in the **prerequisites** steps.
2. Once the user/learner logs in via web-browser an object will be printed to the terminal that looks similar to this:
  ```
  [
  {
    "cloudName": "AzureCloud",
    "id": "f****a09-****-4d1c-98**-f**********c",
    "isDefault": true,
    "name": "some-subscription-name",
    "state": "Enabled",
    "tenantId": "********-a**c-44**-**25-62*******61",
    "user": {
      "name": "mdavis******@*********.com",
      "type": "user"
      }
    }
  ]
  ```
3. Copy the value of the `id:` field and add it as a secret named `AZURE_SUBSCRIPTION_ID` in the course repository.
4. Keep this value handy, you will need it to create the `AZURE_CREDENTIALS` secret next.

**Creating the user/learners `AZURE_CREDENTIALS`:**

1. From the terminal run the following command:
```
   az ad sp create-for-rbac --name "GitHub-Actions" --role contributor \
                            --scopes /subscriptions/{subscription-id} \
                            --sdk-auth
                            
  # Replace {subscription-id} with the same id stored in AZURE_SUBSCRIPTION_ID
```
                            
2. Make sure the user/learner adds their real `subscription id` in place of the one shown above.
3. This command returns an object to the terminal similar to what is shown below.  Place that **entire** object as the value for the `AZURE_CREDENTIALS` secret in the course repository.
```
  {
    "clientId": "<GUID>",
    "clientSecret": "<GUID>",
    "subscriptionId": "<GUID>",
    "tenantId": "<GUID>",
    (...)
  }
```

---

### Second step: provision environment

Have the learner edit and make the following changes to the `provision-azure.yml` workflow.

1. Move `provision-workflow.yml` into the `.github/workflows` folder
2. Edit `AZURE_WEBAPP_NAME` of the file by replacing `USERNAME` with the user/learners **GitHub handle**
```
  #################################################
  ### USER PROVIDED VALUES ARE REQUIRED BELOW   ###
  #################################################
  #################################################
  ### REPLACE USERNAME WITH GH USERNAME         ###
  AZURE_WEBAPP_NAME: USERNAME-ttt-app
  #################################################
```
3. Commit those changes to a branch named `create-env` and create a new pull request
4. Once the workflow completes this PR can be merged and the `create-env` branch should be deleted so this workflow never runs again during the course.  It's a nice piece of notes for the learner to leave the course with though.
  - As always, if the initial creation of the pull request doesn't trigger the workflow, make an arbitrary push to the `create-env` branch to trigger it.
  - This workflow takes about 2.5 minutes to run fully

---

### Third step: teach the normal content

The normal lessons around CD with actions work just fine.  This time though the responses need to guide the user/learner to use the `deploy-azure.yml` workflow.  

  - This workflow takes about 2.5 minutes to run fully

This workflow contains the steps necessary to deploy the containerized tic-tac-toe app to Azure.  Just like provisioning the environment, the `deploy-azure.yml` requires the `AZURE_WEBAPP_NAME` to be changed.  **It needs to be consistent with what the user named it when the environment was provisioned**.

:warning: There is a huge "gotcha" that exists with this GitHub Packages.  No one can reuse a package name across repositories, for that reason the user MUST name the `DOCKER_IMAGE_NAME` in this workflow something they have not used in a different repository.

You can however create a newer version of a package as long as the repository/packagename stays the same.  For our purposes the user/learner gets a repository to take the course in which shouldn't change so this woudln't be a problem for users/learners taking this course a second or third time.

**For example, I cannot create another docker image named `woo-hoo` in ANY repository other than the `azure-test` reposoitory I created while figuring this out.**

:computer: The `Deploy web app container` step in this workflow (second to the last step in the entire file) produces output in the **Actions tab** that contains the URL to visit so the user/learner can see their app deployed.  This can also be found from within the Azure UI.

---

### Final step: destroy the Azure resources

In a final step help the user/learner tear down the Azure resources we create throughout this course.  This prevents any accidental charges in the event the user/learner is outside of their FREE tier usage.

Have the learner edit and make the following changes to the `destroy-azure.yml` workflow.

1. Move `destory-workflow.yml` into the `.github/workflows` folder
2. Commit those changes to a branch named `destroy-env` and create a new pull request
3. Once the workflow completes this PR can be merged and the `destroy-env` branch should be deleted so this workflow never runs again.  It's a nice piece of notes for the learner to leave the course with though.
  - As always, if the initial creation of the pull request doesn't trigger the workflow, make an arbitrary push to the `destory-env` branch to trigger it.
  - This workflow takes about 2.5 minutes to run fully
  
___

**For fun I included a CI/CD Souffle workflow that does everything but destroy the deployment (in traditional DevOps order)**

Not sure it's needed for the course, but I figured it may be fun to demo this every now and then :smile:

*For it to work the secrets in the `first step` still need to be created*


