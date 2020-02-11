---
title:  "Deploy GitHub provisioned application to Azure"
date:   2020-02-11 19:30:00 +0100
tag: 
    - GitHub
    - PowerShell
    - Automation
    - DevOps
    - Global DevOps Bootcamp
    - Secrets
    - Sodium
---

In my [last post](/provision-github-setup/){:target="_blank"} I described how we were able to provision an application into GitHub. The only problem then was there was no API for managing secrets in GitHub. Luckily two weeks ago [the API](https://developer.github.com/v3/actions/secrets/){:target="_blank"} to manage secrets was released. So to finish the setup, I added a script to create several Azure resources, get credentials from Azure and save them as secret in Azure. In the end a GitHub Actions task can use the credentials to deploy the application to Azure.

![picture](/assets/20200211/silas-kohler-C1P4wHhQbjM-unsplash.jpg)
_Photo by Silas Köhler on [Unsplash](https://unsplash.com/photos/C1P4wHhQbjM){:target="_blank"}_

### Creating the Azure Resources
I added the Create-AzureResources.ps1 which creates a ResourceGroup, AppPlan and a Web Application. This is all done with the most basic settings. Preferably I wanted to use the publish profile of the Web Application since it's the first mentioned way to use the [GitHub Action to deploy a Web App](https://github.com/Azure/webapps-deploy){:target="_blank"}. However the Azure Deploy task does only support XML while the [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/webapp/deployment?view=azure-cli-latest#az-webapp-deployment-list-publishing-profiles){:target="_blank"} only returns JSON. Another way to pass credentials to the deploy task is by using the [Azure Login task](https://github.com/Azure/login){:target="_blank"} in combination with creating an Azure Service Principal. The returned result after creating the service principal needs to be stored as a secret inside of GitHub.

### Storing the secret inside GitHub
So when the credentials are generated with the Azure CLI, the next step is to pass the secret to the GitHub API. According to the documentation it's advised to use [LibSodium](https://libsodium.gitbook.io/doc/){:target="_blank"} for encrypting the secrets. The idea is to get the public key for the new created GitHub repository and encrypt the secret with this key with the help of LibSodium. Unfortunately there is no up-to-date(!) PowerShell library of LibSodium, so I had to improvise. To reuse the existing Sodium library I imported the .Net dll and used the exposed methods to encrypt my secret in the same way as the C# example of the API documentation. 

{% highlight powershell %}
$PublicKeyBytes = [System.Convert]::FromBase64String($PublicKey)
$EncryptedMessageBytes = [Sodium.SealedPublicKeyBox]::Create($Secret, $PublicKeyBytes)
$EncryptedMessage = [System.Convert]::ToBase64String($EncryptedMessageBytes)
{% endhighlight %}

So the public key(Base64 string) gets converted to a byte array. Then the SealedPublicKeyBox.Create method from Sodium with the secret (Credentials in JSON string) and the public key (byte array) is used returning a byte array. The byte array then needs to be converted to a Base64 string which can be used in the body to post to the API. Don't forget to include the Id of the public key!

### Deploy
When the secret is stored in GitHub the Action can be triggered. The Action is now updated with the following steps:

{% highlight yaml %}
    - name: Azure Login
      uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}

    - name: 'Run Azure webapp deploy action using publish profile credentials.'
      uses: azure/webapps-deploy@v1
      with: 
        app-name: {{ApplicationNamePlaceHolder}} # Replace with your app name
        package: ${{env.DOTNET_ROOT}}/PartsUnlimited 

    - name: Azure Logout
      run: |
        az logout
{% endhighlight %}

After the Action is finished, the application should be deployed to the earlier created Azure resources. The complete code can be found [here on GitHub](https://github.com/thijslimmen/GitHub-InnovationDay). To summarize the scripts creates the following:
- A repository in GitHub
- Resources in Azure
- Azure credentials as secret in GitHub
- Check-in application to the GitHub repo
- Triggering GitHub action to build and deploy the application
- Add an issue
- Add a collaborator