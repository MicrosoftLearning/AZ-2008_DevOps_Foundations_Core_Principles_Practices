---
lab:
    title: 'Implement CI/CD with GitHub Actions and IaC with Bicep'
    module: 'Deliver with DevOps'
---

# Lab 03 - Implement CI/CD with GitHub Actions and IaC with Bicep

## Objectives

In this lab, you will:

- Prepare the Azure subscription for the lab
- Implement continuous integration and continuous deployment (CI/CD) with GitHub Actions
- Implement Infrastructure as Code (IaC) and CI/CD with GitHub Actions and a Bicep template

> **Note:** In the previous lab, you configured GitHub Pages, which means that you actually already implemented continuous deployment (potentially without even realizing it). GitHub Pages use (in the background) GitHub Actions to perform automated deployment following any commits to the main branch. You can verify this by navigating to the **Actions** tab on the main page of the forked repo **Spoon-Knife**. Now you will enhance this functionality. First, you will create an Azure App Service web app and leverage its integration with GitHub Actions to implement a CI/CD workflow that builds and deploys a custom .NET web app into the existing Azure App Service. Next, you will use a custom-developed GitHub Actions workflow with a Bicep template to provision a couple of Azure App Service web apps in different Azure regions, and then build and deploy the same custom .NET web app into both of them.

> **Note:** For this and subsequent labs, use the same GitHub account you created for the purpose of the first lab.

## Prerequisites

- Completed [Lab 01 - Agile Planning and Management using GitHub](01-agile-planning-management-using-github.md)
- Completed [Lab 02 - Implement Flow of Work with GitHub](02-implement-manage-repositories-using-github.md)
- An Azure subscription to which you have at least the Contributor-level access. If you do not have an Azure subscription, you can sign up for a [free trial](https://azure.microsoft.com/free).

## Exercise 0: Prepare the Azure subscription for the lab

> **Note:** If you are using a new Azure subscription, some of the Azure resource providers might not be registered. Since the registration process might take a few minutes, to minimize the wait time, you will register them at the beginning of this lab.

> **Note:** In this lab, you will be using Azure Cloud Shell. If this is the first time you are using Azure Cloud Shell in your subscription, you will need to register the corresponding resource provider. 

1. Start a web browser and navigate to the Azure portal at `https://portal.azure.com`.
1. If prompted, sign in by using your Microsoft Entra ID account with the Owner access to the Azure subscription available to you.
1. In the web browser tab displaying the Azure portal, in the search text box at the top of the page, enter **Subscriptions** and, in the list of results, select **Subscriptions**.
1. On the **Subscriptions** page, select the subscription you want to use in this lab.
1. On the subscriptions page, in the vertical menu on the left side, select **Resource providers**.
1. In the list of resource providers, search for and select **Microsoft.CloudShell**.
1. With the **Microsoft.CloudShell** resource provider selected, in the toolbar, select **Register**.

   > **Note:** Don't wait until the registration completes but proceed directly to the next exercise. The registration might take a couple of minutes.

## Exercise 1: Implement continuous integration and continuous deployment (CI/CD) with GitHub Actions

In this exercise, you will implement continuous integration and continuous deployment to an Azure App Service web app with GitHub Actions.

> **Note:** The Azure App Service web app you will be deploying is a .NET-based implementation of the same HTML page you deployed in the previous lab. To simplify the initial setup, you will use a fork of another GitHub repo that contains the source code of the .NET app.

The exercise consists of the following tasks:

- Task 1: Fork a GitHub repo containing the source code of a web app
- Task 2: Create and configure an Azure App Service web app 
- Task 3: Validate the CI/CD functionality

### Task 1: Fork a GitHub repo containing the source code of a web app

1. Switch to the browser window displaying your GitHub account and ensure that you are still authenticated (if not, sign in by using your GitHub user account).
1. Open another tab in the same browser window and navigate to the [Spoon-Knife.NET](https://github.com/aaduser1/Spoon-Knife.NET) repo, which hosts the .NET version of the same web site you implemented in the previous lab.
1. On the **Spoon-Knife.NET** repo page, select **Fork**.
1. On the **Create a new fork** page, ensure that the **Owner** drop-down list entry displays your GitHub user name, accept the default entry **Spoon-Knife.NET** in the **Repository name** text box, leave the **Copy the main branch only** checkbox enabled, and then select **Create fork**.

   > **Note:** Your browser session will be automatically redirected to the newly forked repo.

### Task 2: Create and configure an Azure App Service web app

1. Switch to the web browser tab displaying the Azure portal at `https://portal.azure.com`.
1. In the Azure portal, in the search text box at the top of the page, enter **App Services** and select **App Services** in the list of results.
1. On the **App Services** page, select **+ Create** and, in the drop-down list, select **Web app**.
1. On the **Basics** tab of the **Create Web App** page, perform the following actions:

   - Verify that your Azure subscription appears in the **Subscription** drop-down list
   - Select the **Create new** link below the **Resource Group** drop-down list, in the **Name** text box, enter **devops-core-03a-RG**, and then select **OK**.
   - In the **Instance Details** section, in the **Name** text box, enter a globally unique name consisting of lower-case letters and digits, starting with a letter.
   - Ensure that the **Publish** **Code** option is selected.
   - In the **Runtime stack** drop-down list, select **.NET 6 (LTS)**.
   - Leave the **Operating System** option set to **Windows**.
   - In the **Region** drop-down list, select an Azure region close to your location.
   - In the **Pricing plans** section, accept the default pricing plan name, location (which is determined by your choice of the region), and the pricing plan.
   - In the **Zone redundancy** drop-down list, select **Disable**.

1. Back on the **Basics** tab of the **Create Web App** page, select **Next: Database >**.
1. On the **Database** tab of the **Create Web App** page, select **Next: Deployment**.
1. On the **Deployment** tab of the **Create Web App** page, in the **GitHub Actions settings** section, set **Continuous deployment** to **Enable**.
1. If needed, select **Change account** to point to your GitHub account and then perform the following actions:

   - In the **Organization** drop-down list, select the name of your GitHub organization.
   - In the **Repository** drop-down list, select **Spoon-Knife.NET**.
   - In the **Branch** drop-down list, select **main**.

   > **Note:** If prompted, first Authorize AzureAppService.

1. Select **Preview file**.
1. In the **Workflow configuration** pane, review the resulting workflow, which should have the following format (the details, such as the application name, will differ - so do not modify them):

   ```yaml
   # Docs for the Azure Web Apps Deploy action: https://github.com/Azure/webapps-deploy
   # More GitHub Actions for Azure: https://github.com/Azure/actions

   name: Build and deploy ASP.Net Core app to Azure Web App - docorewebapp03b

   on:
     push:
       branches:
         - main
     workflow_dispatch:

   jobs:
     build:
       runs-on: windows-latest

       steps:
         - uses: actions/checkout@v4

         - name: Set up .NET Core
           uses: actions/setup-dotnet@v1
           with:
             dotnet-version: '6.0.x'
             include-prerelease: true

         - name: Build with dotnet
           run: dotnet build --configuration Release

         - name: dotnet publish
           run: dotnet publish -c Release -o ${{env.DOTNET_ROOT}}/myapp

         - name: Upload artifact for deployment job
           uses: actions/upload-artifact@v3
           with:
             name: .net-app
          path: ${{env.DOTNET_ROOT}}/myapp

     deploy:
       runs-on: windows-latest
       needs: build
       environment:
         name: 'Production'
         url: ${{ steps.deploy-to-webapp.outputs.webapp-url }}
    
       steps:
         - name: Download artifact from build job
           uses: actions/download-artifact@v3
           with:
             name: .net-app
      
         - name: Deploy to Azure Web App
           id: deploy-to-webapp
           uses: azure/webapps-deploy@v2
           with:
             app-name: 'docorewebapp03b'
             slot-name: 'Production'
             package: .
             publish-profile: ${{ secrets.AZUREAPPSERVICE_PUBLISHPROFILE_B97B997AD06D4AAFAA3A1EBB388DA406 }}
   ```

1. In the **Workflow configuration** pane, select **Close**.
1. Back on the **Deployment** tab of the **Create Web App** page, select **Review + create** and then select **Create**.

   > **Note:** The deployment should complete within a couple of minutes. 

1. On the **Your deployment is complete** page, select **Go to resource**.
1. On the newly deployed web app page, in the **Essentials** section, copy the value of the **Default domain** link.
1. Open another tab in the web browser window and, from that tab, navigate to the URL you copied in the previous step.
1. Verify that the browser tab displays the web page that looks very similar to the one displayed at the end of the previous lab.

### Task 3: Validate the CI/CD functionality

1. Switch to the web browser tab displaying the GitHub home page at `https://github.com` with the **Spoon-Knife.NET** repo.
1. On the **Spoon-Knife.NET** repo page, in the listing of the content, note that the previous task resulted in automatic creation of the directory **.github/workflows**.
1. Select the **.github/workflows** directory to display its content.
1. In the directory listing, select the YAML file and display its content.
1. Review the content of the file and verify that it matches the one displayed earlier on in the **Workflow configuration** pane in the Azure portal. 

   > **Note:** In general, to follow the DevOps best practices, you should create a separate branch and a pull request whenever you want to introduce changes to the main branch. This is the approach we'll follow here as well.

1. In the upper left corner of the page, select the **main** entry to display the **Switch branches/tags** drop-down list.
1. In the **Find or create a branch...** text box, enter **test-app-service-web-app-cd** and then select **Create branch: test-app-service-web-app-cd from 'main'** entry to create a new branch.

   > **Note:** This will automatically make the newly created branch the current one, as indicated by the content of the drop-down list.

1. On the forked **Spoon-Knife.NET** repo page, select the **wwwroot** directory and, in the listing of files, select **index.html**.
1. On the **Spoon-Knife.NET/wwwroot/index.html** page, on the right side in the code editor toolbar, select the pencil icon to switch to the editor mode.
1. In the editor pane, replace the lines 16-18 with the following HTML code:

   ```html
   <p>
     Ready to team up and do some CD with Azure App Service web apps? Let's collaborate, @octocat!
   </p>
   ```

1. In the upper right corner of the editor page, select **Commit changes...**.
1. In the **Commit changes** window, perform the following actions:

   - In the **Commit message** text box, enter **Test Azure App Service web apps CD**.
   - In the **Extended description** text box, enter **Test Azure App Service web apps continuous deployment**.
   - Accept the default option **Commit directly to the test-app-service-web-app-cd branch**.
   - Select **Commit changes**.

1. Select the **Pull requests** tab.
1. On the **Pull requests** tab, select **Compare & pull request**.
1. On the **Open a pull request** page, in the **Choose a Base Repository** drop-down list, select the name of the forked repository you created at the beginning of this lab.

   > **Note:** The name will start with your GitHub name, followed by a forward slash, followed by **Spoon-Knife**. Once you select it, the entry should change to **base: main**.

   > **Note:** This is necessary because we want to update the main branch in the forked repo, rather than the main branch in the repo from which you created the fork.

1. On the **Open a pull request** page, select **Create pull request**.
1. On the **Test Azure App Service web apps CD** page, verify that the current branch has no conflicts with the base branch, select **Merge pull request**, and then select **Confirm merge**.
1. Verify that the pull request has been successfully merged and closed and select **Delete branch**.
1. Switch back to the web browser tab displaying the **Spoon-Knife.NET** page, refresh the page, and verify that its content includes your change.
1. Switch back to the GitHub page and, in the toolbar, select the **Actions** tab.
1. On the **Actions** page, in the **All workflows** section, review the list of workflows and select the most recent one.
1. On the workflow page, select the **build** and **deploy** job entries. For each job, review their individual steps.

## Exercise 2: Implement Infrastructure as Code (IaC) and CI/CD with GitHub Actions and a Bicep template

In this exercise, you will implement IaC with CI/CD to Azure App Service web apps with GitHub Actions and a Bicep template.

> **Note:** You will be using the same .NET-based web app you used in the first exercise of this lab. In this case, just as before, to simplify the initial setup, you will use a fork of an existing GitHub repo that contains the source code of the .NET app, the GitHub Actions workflow, and the Bicep template.

The exercise consists of the following tasks:

- Task 1: Fork and review the GitHub repo containing the source code of a web app, a GitHub Actions workflow, and a Bicep template
- Task 2: Configure the target environment
- Task 3: Validate the IaC and CI/CD functionality

### Task 1: Fork and review the GitHub repo containing the source code of a web app, a GitHub Actions workflow, and a Bicep template

1. Switch to the browser window displaying your GitHub account and ensure that you are still authenticated (if not, sign in by using your GitHub user account).
1. Open another tab in the same browser window and navigate to the [Spoon-Knife.Multi](https://github.com/polichtm/Spoon-Knife.Multi) repo, which hosts the .NET version of the same web site you implemented in the previous exercise of this lab, a GitHub Actions workflow, and a Bicep template (as well as a Bicep template parameters file).
1. On the **Spoon-Knife.Multi** repo page, select **Fork**.
1. On the **Create a new fork** page, ensure that the **Owner** drop-down list entry displays your GitHub user name, accept the default entry **Spoon-Knife.Multi** in the **Repository name** text box, leave the **Copy the main branch only** checkbox enabled, and then select **Create fork**.

   > **Note:** Your browser session will be automatically redirected to the newly forked repo.

1. On the **Spoon-Knife.Multi** page, review the repo structure and note that it consists of the following components:

   - the **.github/workflows** directory containing a YAML-based workflow named **main_docorewebapp03b.yml**
   - the **infra** directory containing the Bicep template and its parameters file
   - the **src** directory containing the .NET code of the source web app
   - the **wwwroot** directory containing the static elements of the web app (the same HTML and CSS files you worked with in the previous lab)
   - the **Spoon-Knife.NET.csproj** file used during the web app build process

1. Open the **.github/workflows** directory and then select the **main_docorewebapp03b.yml** file to view its content. Note that this GitHub Actions workflow contains a single job, which, in turn, consists of the following steps:

   - Checkout repository
   - Set up .NET Core
   - Build with dotnet
   - dotnet publish
   - Upload artifact for deployment job
   - Set up Azure CLI
   - Deploy Infrastructure with Bicep
   - Retrieve Web App Names
   - Retrieve Publish Profile for West US
   - Retrieve Publish Profile for East US
   - Download artifact from build job
   - Deploy to Azure Web App (West US)
   - Deploy to Azure Web App (East US)

   > **Note:** Do not focus on details of each job. It is much more important at this point to simply understand the purpose of the workflow.

1. Note that the process of provisioning Azure resources targets a single resource group which is defined at the beginning of the workflow as an environment variable `RESOURCE_GROUP: devops-core-03b-RG`.

   > **Note:** You will need to create this resource group before you run the workflow.

1. Note that the workflow relies on a secret to authenticate to the target Azure subscription during the Set up Azure CLI step, as evidenced by the statement `creds: ${{ secrets.AZURE_CREDENTIALS }}`.

   > **Note:** You will need to set up this secret before you run the workflow.

1. In addition, note that this workflow is configured to be triggered manually, as indicated by the `on: workflow_dispatch:` syntax.

   > **Note:** You could also use the same triggers as the workflow in the previous exercise of this lab, but in either case, you first have to create the target resource group and the secret that will authenticate and authorize access of the workflow to that resource group. You will set both up in the next task of this exercise.

### Task 2: Configure the target environment

> **Note:** You will start by creating the resource group.

1. Switch to the web browser tab displaying the Azure portal at `https://portal.azure.com`.
1. In the Azure portal, in the search text box at the top of the page, enter **Resource groups** and select **Resource groups** in the list of results.
1. On the **Resource groups** page, select **+ Create**.
1. In the **Resource groups** text box, enter **devops-core-03b-RG**.
1. In the **Region** drop-down list, select the same Azure region that was used in the previous exercise.
1. Select **Review + create** and then, on the **Review + create**, select **Create**.

    > **Note:** Next, you will create a service principal that will be used to authenticate from the GitHub Actions workflow to the target Azure subscription and assign to it the role of Contributor in the scope of the newly created resource group.

1. In the Azure portal, select the **Cloud Shell** icon to the right of the search text box.
1. If prompted to select either **Bash** or **PowerShell**, select **Bash**.

    > **Note**: If this is the first time you are starting **Cloud Shell** and you are presented with the **You have no storage mounted** message, select **Create storage** and wait for the storage mount to complete.

1. If necessary, select **Bash** in the drop-down menu in the upper-left corner of the Cloud Shell pane.
1. In the Bash session within the Cloud Shell pane, run the following command to store the value of your Azure subscription ID in a variable:

   ```cli
   SUBCRIPTION_ID=$(az account show --query id --output tsv) 
   ```

1. In the Bash session within the Cloud Shell pane, run the following command to create a Microsoft Entra ID service principal and assign to it the role of Contributor in the scope of the resource group **devops-core-03b-RG**:

   ```cli
   az ad sp create-for-rbac --name "docorewebapp03b" --role contributor --scopes /subscriptions/$SUBCRIPTION_ID/resourceGroups/devops-core-03b-RG --json-auth
   ```

1. Copy the entire JSON-formatted output of the command and record it. You will need it shortly. The output should have the format that resembles the following text:

   ```json
   {
     "clientId": "cccccccc-cccc-cccc-cccc-cccccccccccc",
     "clientSecret": "xxxxx~xxxxxxxxxxxx-xxxxxxxxxxxxx_xxxxxxx",
     "subscriptionId": "yyyyyyyy-yyyy-yyyy-yyyy-yyyyyyyyyyyy",
     "tenantId": "zzzzzzzz-zzzz-zzzz-zzzz-zzzzzzzzzzzz",
     "tenantId": "e7b5993f-5428-477c-8197-7c95209a48aa",
     "activeDirectoryEndpointUrl": "https://login.microsoftonline.com",
     "resourceManagerEndpointUrl": "https://management.azure.com/",
     "activeDirectoryGraphResourceId": "https://graph.windows.net/",
     "sqlManagementEndpointUrl": "https://management.core.windows.net:8443/",
     "galleryEndpointUrl": "https://gallery.azure.com/",
     "managementEndpointUrl": "https://management.core.windows.net/"
   }
   ```

1. Switch to the web browser window displaying the forked **Spoon-Knife.Multi** GitHub repo page and, in the toolbar, select **Settings**.
1. In the vertical menu on the left side, in the **Security** section, select **Secrets and variables** and, in the drop-down list, select **Actions**.
1. In the **Actions secrets and variables** pane, select **New repository secret**.
1. In the **Actions secrets / New secret** pane, in the **Name** text box, enter **AZURE_CREDENTIALS**.
1. In the **Secret** text box, paste the JSON-formatted text you recorded earlier in this task and select **Add secret**.

### Task 3: Validate the IaC and CI/CD functionality

1. In the web browser window displaying the forked **Spoon-Knife.Multi** GitHub repo page, select **Actions**.
1. If you are prompted to enable GitHub Actions, select **I understand my workflows, go ahead and enable them**.

    >**Note**: This is expected, since, by default GitHub will disable workflows in a forked repo for your own protection.

1. In the **All workflows** section on the left side, select **Build, provision, and deploy .NET app to Azure App Service web app**.
1. In the upper-right corner of the **Build, provision, and deploy .NET app to Azure App Service web app** pane, select **Run workflow**, verify that **Branch: main** appears in the **Use workflow from** drop-down list, and select **Run workflow** again.

    >**Note**: This should trigger a workflow run.

1. Select the **Build, provision, and deploy .NET app to Azure App Service web app** workflow run.
1. In the **Build, provision, and deploy .NET app to Azure App Service web app #1** pane, select **build and deploy**.
1. Monitor the progress of the workflow and verify that all steps of the **build and deploy** job complete successfully.

    >**Note**: In case any of the steps fail, from the same page that displays the workflow progress, in the upper right corner, select **Re-run all jobs** and then, in the **Re-run all jobs** pane, select **Re-run jobs**.

    >**Note**: Leave the deployed Azure resources running. You will need them in the next lab.
