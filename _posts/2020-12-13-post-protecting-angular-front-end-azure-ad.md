---
title: "Post: Protecting Angular Frontend with Azure Active Directory"
last_modified_at: 2020-12-13T16:20:02-05:00
classes: wide
categories:
  - Blog
tags:
  - Azure Active Directory
  - Angular
---

This is a first part in the series of how to protect an application with Azure AD. Application contains angular based front end and .net core based webapi. 

We are going to be following below steps in this article

1. Register an application in Azure AD
2. Download quick start angular application template
3. Configure Angular website with Azure AD
4. Run the application

## 1. Register an application in Azure AD

We would need to first create application registeration in azure active directory. This application registration represents our angular front end web application. If we have webapi backend then we will require to create separate application registration for that component. 

Lets start by following below steps for registering our website in Azure AD

1. Sign in to the Azure Portal
2. Switch to a tenant where you want to register an application. This is done by switching directory.
3. Search for and select **Azure Active Directory**
4. On the left panel, under **Manage**, click on **Application Registration**.
![image-center]({{ site.url }}{{ site.baseurl }}/assets/images/01-Protect-Angular/AppRegistration-01.png){: .align-center}
5. Click on "New registration".
5. Provide Name. 
6. In **Supported Account Types"**, since we are developing this website as single tenant , we will Select **"Accounts in this organizational Directory only"**. With this configuration, only users only from your directory can login to this website. 
7. In Redirect URI section, select **Single-page-application(SPA)**. Specify appropriate redirect uri where Azure AD should redirect user after successful login. In our case, we would just specify **"http://localhost:4200"**.
![image-center]({{ site.url }}{{ site.baseurl }}/assets/images/01-Protect-Angular/AppRegistration-02.png){: .align-center}
8. From the overview section of the registered application, record **Application (client) ID
** & **Directory (tenant) ID**. In my case, ApplicationId= 35cf9790-0ba2-4916-b348-032d308a3e99 & TenantId=1372c663-c4a5-4eef-8e1e-8af9c94ff77a.

## 2. Download quick start angular application template

You find many samples in below github repository for javascript based applications

```
https://github.com/AzureAD/microsoft-authentication-library-for-js.git 
```

For our website, we would use following quick start template from this repository

```
https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/samples/msal-angular-v2-samples/angular11-sample-app
```

## 3. Configure Angular website with Azure AD

We would use **Application (client) ID** & **Directory (tenant) ID** recorded in the first step to configure angular application.

1. Open the application in your favorite editor. 
2. Open **app.module.ts** located in **app** folder
3. Search for **MSALInstanceFactory** method in this file, and update it with correct **clientId** & **Authority** values. Update the Authority value with appropriate TenantId. If Authority key is not present then add it as below.


```typescript
  export function MSALInstanceFactory(): IPublicClientApplication {
  return new PublicClientApplication({
    auth: {
      clientId: '35cf9790-0ba2-4916-b348-032d308a3e99', // <--- Update ClientId here
      redirectUri: '/',
      postLogoutRedirectUri: '/',
      authority: "https://login.microsoftonline.com/1372c663-c4a5-4eef-8e1e-8af9c94ff77a" // <-- Update TenantId here
    },
    cache: {
      cacheLocation: BrowserCacheLocation.LocalStorage,
      storeAuthStateInCookie: isIE, // set to true for IE 11. Remove this line to use Angular Universal
    },
    system: {
      loggerOptions: {
        loggerCallback,
        logLevel: LogLevel.Info,
        piiLoggingEnabled: false
      }
    }
  });
}
```

## 4. Run the application

Now our website is configured with appropriate details. Now it's the time to see everything in action. start you website by executing following command

```
ng serve
```

1. Open the website in the browser, it should be available at http://localhost:4200. 
2. You will see three buttons at the top. Click on Login. 
![image-center]({{ site.url }}{{ site.baseurl }}/assets/images/01-Protect-Angular/AppRun-01.png){: .align-center}
3. This will redirect you to following Azure AD page where you can specify username & password. 
![image-center]({{ site.url }}{{ site.baseurl }}/assets/images/01-Protect-Angular/AppRun-02.png){: .align-center}
4. Upon providing correct credentials, it will show you a consent page like below. Click on Accept.
![image-center]({{ site.url }}{{ site.baseurl }}/assets/images/01-Protect-Angular/AppRun-03.png){: .align-center}
5. Azure AD redirects you to a "Redirect URL" specified in appliction registration along with appropriate token. 
![image-center]({{ site.url }}{{ site.baseurl }}/assets/images/01-Protect-Angular/AppRun-04.png){: .align-center}
5. You can click on "Profile" button at the top to see some information from your Azure AD profile. Application basically does a "Graph Api" call using logged in user's credentials.


Thats it!! You have an Angular website protected with Azure AD. 

Join me in the next session for **protecting webapi hosted in Azure App Service using Azure AD**.

 

