---
title:  "Provision GitHub setup"
date:   2019-12-16 21:00:00 +0200
tag: 
    - GitHub
    - PowerShell
    - Automation
    - DevOps
    - Global DevOps Bootcamp
---

This year I was part of the team that organized the [Global DevOps Bootcamp](https://globaldevopsbootcamp.com/){:target="_blank"}. An important part of the event was provisioning the setup for all teams. This consisted of creating an Azure DevOps project and Azure setup to have a complete pipeline for an application. During the last Xpirit's innovation day, I researched the possibility of replacing Azure DevOps with GitHub together with my colleague [Thijs](https://www.linkedin.com/in/thijs-limmen/){:target="_blank"}. In this post, I will share our findings.

![picture](/assets/20191216/junior-ferreira-7esRPTt38nI-unsplash.jpg)
_Photo by Júnior Ferreira on [Unsplash](https://unsplash.com/photos/7esRPTt38nI){:target="_blank"}_

### Goal
Our goal was to create a basic setup inside of GitHub with the use of the GitHub API and PowerShell. We decided the following items should be included in our setup:
- Create a repository
- Add collaborator(s) to the repository
- Add sample code to the repository
- Create and start a build based on GitHub Actions
- Create and start a release based on GitHub Actions
- Raise an issue

### Creating the resources
[Creating](https://developer.github.com/v3/repos/#create){:target="_blank"} a repository, [adding collaborators](https://developer.github.com/v3/repos/collaborators/#add-user-as-a-collaborator){:target="_blank"} and [raising an issue](https://developer.github.com/v3/issues/#create-an-issue){:target="_blank"} are really easy to do. Just call the corresponding API endpoints and you're good to go. The [GitHub API documentation](https://developer.github.com/v3/){:target="_blank"} is really clear and easy to follow.
From there the repository can be cloned locally and the sample code can be pushed to GitHub. For this part no API call is needed, just regular GIT commands can be used to achieve this. However you might want to [get the repository](https://developer.github.com/v3/repos/#get){:target="_blank"} and use the clone_url property to make sure you have the right URL.

### GitHub Actions
With the Azure DevOps setup the Azure DevOps API was used to create and trigger builds and releases. However for GitHub there isn't such an API as it seems. To achieve this it's possible to add a [predefined yml file](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/workflow-syntax-for-github-actions){:target="_blank"} inside the repository in a specific folder. So in our case we needed to add a yml file inside the /.github/workflows folder. In the yml file we included dotnet build and dotnet publish to create a package. In the end we included the [Azure Web app deploy task](https://github.com/Azure/webapps-deploy){:target="_blank"}. This task deploys the sample code to Azure. We could have split up the build part and the release to Azure part, but for the time being we merged them. Obviously the Azure web app could also be provisioned as part of this whole setup, but is currently not included. Both improvements could be the next step.

![Publish Code Coverage Result Task](/assets/20191216/yml.png)

### Secrets
To be able to release something to Azure a secret is needed. This secret should be included inside the repository to be reused and to be safe since it can be encrypted. Also this Azure deploy task uses a secret to deploy to Azure. Then we found out there is no API endpoint for storing secrets inside a repository which is a shame. According to [this request](https://github.community/t5/GitHub-Actions/Github-Apps-to-add-secrets/m-p/28259){:target="_blank"} more GitHub users need this functionality. While writing this post someone from the GitHub team announced they're currently working on this improvement, but no release date is given. Of course workarounds can be made, but that was not our goal for this tryout. I will follow this request and update this post as soon as it's available somehow.

### The result
In the end we have made a couple of PowerShell scripts that cover all planned resources. These scripts can be used to provision a basic setup for an application with a pipeline inside of GitHub. Unfortunately it was disappointing that it's not possible to store secrets and be able to also automate releases to Azure. Altogether it was a lot of fun to work this out and investigate the possibilities with GitHub.

![Publish Code Coverage Result Task](/assets/20191216/result.png)

Some things for us to keep in mind:
- No secret API available at the moment
- Private repositories only allow 4 collaborators to be added
- Builds and releases should be done by yml as it seems no API's are available

The code we made can be found here: [https://github.com/thijslimmen/GitHub-InnovationDay](https://github.com/thijslimmen/GitHub-InnovationDay){:target="_blank"}

UPDATE:
In the meantime GitHub launched the API for managing secrets, read about it in [my other post](/deploy-github-provisioned-application-to-azure/). Also private repositories are now able to have [unlimited collaborators](https://github.blog/2020-04-14-github-is-now-free-for-teams/).