---
title: "Post: Protecting WebApi deployed in Azure App service with Azure Active Directory"
last_modified_at: 2021-03-08T20:43:02+05:30
classes: wide
categories:
  - Blog
tags:
  - Azure Active Directory
  - Webapi
  - Azure App service
---

This is a second part in the series of how to protect an application with Azure AD. Application contains angular based front end and .net core based webapi. 

In this part we will take sample .net core based webapi and deploy it to Azure App service. Subsequently, we will protect it using Azure App service's **Authentication / Authorization** feature.

We are going to be following below steps in this article

1. Create .netcore webapi project
2. Deploy the project to azure app service 
3. Register an application in Azure AD
4. Configure Azure App service with authentication
5. Calling the protected Api using Postman
6. Accessing protected swagger page

## 1. Create .netcore webapi project

For the purpose of this article, i have created a simple webapi project using visual studio. I have integrated swagger to get nice api documentation page. This project has been uploaded to following github location.

```console
https://github.com/jogikalpesh/Quickstarts/tree/master/02-AzureAD-Api/SampleApi
```

Let's Run the project and navigate to swagger page by hitting following url. Replace port with appropriate value for your environment in following url.

```
http://localhost:54803/swagger/index.html
```
it should load following page

![image-center]({{ site.url }}{{ site.baseurl }}/assets/images/02-Protect-Api/01swagger.png){: .align-center}

## 2. Deploy the project to azure app service 

Azure App service running .net core 3.1 is required for completing this step. 

There are multiple ways in which webapi project can be deployed to Azure App service. We have used visual studio's **publish** feature to deploy this webapi. Simply right click on project and click on publish. Follow the wizard to deploy this project to Azure App service.

![image-center]({{ site.url }}{{ site.baseurl }}/assets/images/02-Protect-Api/publish01.png){: .align-center}

We can confirm the successful deployment by hitting following url. Just replace {Your_APP_Service_Name} with appropriate website's name. 

```
https://{Your_APP_Service_Name}.azurewebsites.net/swagger/index.html
```

After deploying, we can use postman to call the api. Currently, the API does not require any authentication.

![image-center]({{ site.url }}{{ site.baseurl }}/assets/images/02-Protect-Api/publish02.png){: .align-center}

## 3. Register an application in Azure AD

Currently, the deployed api is not secured. We were able to call the api without any authentication. In order to protect this api, we will begin by  registering application in Azure AD. This Application registration would represent our WebAPI in Azure AD.

Lets start by following below steps for registering our webapi in Azure AD

1. Sign in to the Azure Portal
2. Switch to a tenant where you want to register an application. This is done by switching directory.
3. Search for and select **Azure Active Directory**
4. On the left panel, under **Manage**, click on **Application Registration**.
![image-center]({{ site.url }}{{ site.baseurl }}/assets/images/02-Protect-Api/AppRegistration-01.png){: .align-center}
5. Click on "New registration".
5. Provide Name. 
6. In **Supported Account Types"**, since we are developing this website as single tenant , we will Select **"Accounts in this organizational Directory only"**. With this configuration, only users only from your directory can call this website. 
7. (Optional) In Redirect URI section, select **Web**. Specify redirect Url in `<app-url>/.auth/login/aad/callback` format. For example, `http://app-webapi.azurewebsites.net/.auth/login/aad/callback`. This step is not required if the webapi is not going to be accessed through browser. 
![image-center]({{ site.url }}{{ site.baseurl }}/assets/images/02-Protect-Api/AppRegistration-02.png){: .align-center}
8. From the overview section of the registered application, record **Application (client) ID** & **Directory (tenant) ID**. We will use these values in subsequent section.
9. from the left panel, under **Manage**, click on **Authentication**. Under Implicit grant and hybrid flows, set **ID Tokens(used for implicit and hybrid flows) to **True**.
10. From the left panel, under **Manage**, click on **Expose an API**. 
  * Set **Application ID URI**. You can use any valid url here. for example: `api://webapi.techlearning.com/`
  * Click on Save.
  

## 4. Configure Azure App service authentication

1. In Azure portal, navigate to your app service.
2. From the left panel, under settings, click on Authentication/Authorization.
3. Switch on **App Service Authentication**
4. By default, unauthenticated access is allowed. For authentication with Azure AD, set **Action to take when request is not authenticated** option to **Log in with Azure Active Directory**
  ![image-center]({{ site.url }}{{ site.baseurl }}/assets/images/02-Protect-Api/AuthenticationSettings1.png){: .align-center}
5. Under Authentication providers, select Azure Active Directory.
6. In the **Management mode**, select **Advanced**. 
  * for **Client ID**, Enter **Application (client) ID** recorded from **Application registration** step.
  * for **Issuer Url**, Use `https://login.microsoftonline.com/<tenant-id>/v2.0`, substitute TenantId recorded from **Application registration** step. For example: `https://login.microsoftonline.com/1372c663-c4a5-4eef-8e1e-8af9c94ff77a/v2.0`

![image-center]({{ site.url }}{{ site.baseurl }}/assets/images/02-Protect-Api/AuthenticationSettings2.png){: .align-center}
7. Click on OK, and then Save.

## 5. Calling the protected Api using Postman
At this point, our api is protected. On calling the Api endpoint using postman, it should return `401 Unauthorized` error.

![image-center]({{ site.url }}{{ site.baseurl }}/assets/images/02-Protect-Api/postmantest1.png){: .align-center}

We would need to send a **Bearer Token** to this Api. We would learn how to use logged in user's credentials to acquire token for calling webapi in future posts. Meanwhile, we would use something called as **Client Id** & **Client Secret** to acquire token 

### Generating **Client Secret**
  1. In the Azure Portal, search & navigate to Azure Active directory.
  2. Open the Application Registration we created for webapi in the previous steps.
  3. From left panel, select **Certificates & Secrets**
     ![image-center]({{ site.url }}{{ site.baseurl }}/assets/images/02-Protect-Api/AppRegistration-03.png){: .align-center}
  4. Under **Client Secrets**, Click on **New Client Secret**. Enter description, select expiry and click on Add.
  5. Copy the "Value" (not Id). You would not be able to copy this value later, so make sure you copy and record it somewhere.
    ![image-center]({{ site.url }}{{ site.baseurl }}/assets/images/02-Protect-Api/AppRegistration-04.png){: .align-center}

### Generating **Bearer Token**
  For generating bearer token, microsoft has provided the endpoint from where we can request the token. 
  1. Open Postman, create a new request
  2. Select `Get` http method, enter url as `https://login.microsoftonline.com/<tenant-id>/oauth2/v2.0/token`. Substitute right `<tenant-id>` recorded from application registration step. For example, `https://login.microsoftonline.com/1372c663-c4a5-4eef-8e1e-8af9c94ff77a/oauth2/v2.0/token`
  3. click on **Body**, enter following keys and values and click on send. Record the **Access Token** from the response.
  
  |Key|value|
  | --- | ----------- |
  |grant_type|client_credentials|
  |client_id | **Application (client) ID** from application registration step|
  |client_secret| **client secret** recorded from previous step|
  |scope |  Enter `<application-id-url>/.default` where Application-id-Url is url from application registration step. It is **not** App service url. For example, `api://webapi.techlearning.com/.default`|

  ![image-center]({{ site.url }}{{ site.baseurl }}/assets/images/02-Protect-Api/postmantest2.png){: .align-center}

### Call Api
After successfully generating the token, you should be able to call api. Use the **Access Token** generated in previous step and Add it in the authorization. Choose the type as **Bearer** and Put **Access Token** in Token field. See below for reference

![image-center]({{ site.url }}{{ site.baseurl }}/assets/images/02-Protect-Api/postmantest3.png){: .align-center}

## 6. Accessing swagger page

If you have followed the optional step of adding redirect url in Application registration section then you will be able to get to the swagger page from the browser. Navigate to your website, it will now redirect you to Azure AD authentication page. See below

![image-center]({{ site.url }}{{ site.baseurl }}/assets/images/02-Protect-Api/swagger1.png){: .align-center}

On providing the right credentials, it will redirect to the App service again

![image-center]({{ site.url }}{{ site.baseurl }}/assets/images/02-Protect-Api/swagger2.png){: .align-center}

## References
* [https://docs.microsoft.com/en-us/azure/app-service/configure-authentication-provider-aad](https://docs.microsoft.com/en-us/azure/app-service/configure-authentication-provider-aad)


## Next Steps
Calling Azure AD protected webapi from Azure AD protected Angular webapp

## Related Posts
* [Protecting Angular Website]({{ site.url }}{{ site.baseurl }}/blog/post-protecting-angular-front-end-azure-ad)  