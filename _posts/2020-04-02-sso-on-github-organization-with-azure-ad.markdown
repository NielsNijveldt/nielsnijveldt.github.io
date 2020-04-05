---
title:  "SSO on GitHub organization with Azure AD"
date:   2020-04-02 19:30:00 +0100
tag: 
    - GitHub
    - Azure
    - SSO
    - Azure AD
---

GitHub offers [SAML single sign-on(SSO)](https://help.github.com/en/github/authenticating-to-github/about-authentication-with-saml-single-sign-on) in the enterprise plan. I created an enterprise account as a trial to see how this works together with Azure AD. GitHub offers documentation to help to set this up, but it's quite a lot and also some things are not up to date anymore. In this post I will try to explain how to do it and what you could expect of it.

![picture](/assets/20200402/shane-avery-OHnvp41aDzE-unsplash.jpg)
_Photo by Shane Avery on [Unsplash](https://unsplash.com/photos/OHnvp41aDzE){:target="_blank"}_

First you will need both a GitHub enterprise organization and an Azure AD. If you don't have GitHub enterprise, create a free trial for 14 days on [this page](https://github.com/enterprise). Now open both Azure AD and your GitHub organization settings. As the first step we need to create an enterprise application in Azure AD. Click "Add application", search for GitHub and then select "github.com". Fill in a custom name or use the default value and create the application. After creation you will see a couple of options. Step 1 is to "Assign users and groups". Create a couple of users or groups in AD and assign them to this application. By doing this those users will be able to use this connection to sign in to GitHub. Step 2 is to "Set up single sign on". After opening this you will see a 5 step configuration for single sign-on.

![picture](/assets/20200402/AzureAD-Overview.png)

1. Press edit on "Basic SAML Configuration" and fill in these fields as:
- Identifier (Entity ID): https://github.com/orgs/<organizationname>
- Reply URL (Assertion Consumer Service URL): https://github.com/orgs/<organizationname>/saml/consume
- Sign on URL: https://github.com/orgs/<organizationname>/sso

2. Press edit on "User Attributes & Claims" and select the required claim. Make sure the "nameidentifier" claim is set as:
- "Email address"
- Source: "Attribute"
- Source attribute: "user.mail"

![picture](/assets/20200402/AzureAD-Settings.png)

Step 3 and 4 contain information we need to configure in the GitHub settings. Also make sure to download the base64 certificate and open the file in a text editor. Now open the "Settings" option in your GitHub organization. Enable the feature "Enable SAML authentication", after doing this you can fill in the forms based on the information and certificate from Azure. Beneath the certificate field there is a text line, make sure it is set to "Your SAML provider is using the RSA-SHA256 Signature Method and the SHA256 Digest Method.". You can do this by press the edit button in the end of the line and select both "SHA256" options.
Now click the "Test SAML configuration" button and login with any account with access to the GitHub application in Azure AD. If everything is setup fine you will see the message "Passed: Successfully authenticated your SAML SSO identity". Now users should be able to go to https://github.com/orgs/<organizationname>/sso and login to access the GitHub resources. Even if you didn't add them to the users of the organization. Note that you cannot just go https://github.com/login and login with the Azure AD.

![picture](/assets/20200402/GitHub-Security.png)

After a user logs in, I would have expected it could access the GitHub resources straightaway. Unfortunately this is not the case. When the user logs in for the first time GitHub asks to create a GitHub account or to login with an existing one. After this a linked identity is added to the account. So will this make sure you only have to be logged in to Azure AD? Unfortunately this is also not true. If you open a new browser and login in Azure AD it will redirect you to GitHub and ask for the password of the linked identity. This feels more like a double sign-on, then a single sign-on.

## Group synchronization
Also a feature with the Azure AD connection is group synchronization. I would have expected it to synchronize my Azure AD groups to my GitHub organization automatically. However this is not how it works. For example if you want to grant a specific Azure AD group to a specific repository you need to create a group in the organization and use the "Identity Provider Groups" feature. If you select the dropdown it will show all groups from Azure AD. After doing this you can add this group to the repository. Now users from the AD group will be able to access the repository. In my opinion it would have been better if I was able to select Azure AD groups straightaway. Another downside from this is that you are not able to use the "Identity Provider Groups" option when creating a GitHub team from the API.

## Summary
Overall the single sign-on feature works fine on GitHub and could be used as additional security option. It's also easy to configure. However some things could be a little bit better. For example the double login and group synchronization feature could be done differently. 