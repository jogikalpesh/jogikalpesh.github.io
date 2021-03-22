---
title: "Part 3: Calling Azure AD protected webapi from Azure AD protected Angular webapp"
last_modified_at: 2021-03-20T17:15:00+05:30
classes: wide
categories:
  - Blog
tags:
  - Azure Active Directory
  - Webapi
  - Azure App service
  - Angular 
---

This is a third part in the series of how to protect an application with Azure AD. Application contains angular based front end and .net core based webapi. 

In the first part we have protected an angular webapp using Azure AD and in the second part we have protected .net core based webapi. 

In this part, we will make a call to webapi from angular web application. We will embed necessary Authentication/Authorization details to the Api call.

At this point, we have two independent application components, one written in angular and another one in .net core. We have  **"application registration"** in Azure AD to represent each of these compoents.

We will go through following steps in this article
1. Calling a webapi from Angular web application without token.
2. Adding a scope.
3. Adding API permission.
4. Calling a webapi from Angular web application with token.

## 1. Calling a webapi from Angular web application.

We have created angular application [first part]({{ site.url }}{{ site.baseurl }}/blog/post-protecting-angular-front-end-using-azure-ad). I have added a simple page to show the temperature values. This page is simply making a call to the webapi we have create in [second part]({{ site.url }}{{ site.baseurl }}/blog/post-protecting-webapi-app-service-using-azure-ad) and it is showing the result on this page.

Both these applications are available in following github repository

```console
https://github.com/jogikalpesh/Quickstarts.git
```

**Angular website:**

```console
https://github.com/jogikalpesh/Quickstarts/tree/master/01-AzureAD-SPA/angular11-sample-app
```

**.netcore webapi:**
```console
https://github.com/jogikalpesh/Quickstarts/tree/master/02-AzureAD-Api/SampleApi
```
You can navigate to this new page by clicking **"weather"** menu in header. Since we are not supplying any authentication/authorization detail along with this call, the call should fail and you should get following error in "network" tab of "developer tool" in browser.

![image-center]({{ site.url }}{{ site.baseurl }}/assets/images/03-angular-call-api/01-first-Error-call.png){: .align-center}

![image-center]({{ site.url }}{{ site.baseurl }}/assets/images/03-angular-call-api/02-first-call-network-tab.png){: .align-center}

If you observe carefully then call to "WeatherForecast" api is returning **"302 Found"** status code. It is also returning **"location"** response header. This is happening because the call is not authorized and location header contains a link where user credentials can be provided. Since it is an **"Ajax"** call made to webapi, it would have been ideal if we had got **"401 Unauthorized"** instead. 

## 2. Adding a scope.
After you have logged into Angular application. ID token is issued to it. We can actually use this token, to acquire another token which is meant for webapi. 

In order to acquire token for the webapi, we have to first define a **"scope"** in webapi and give access to that scope to our web application. In the OAuth 2.0 parlance, webapi is a **resource** which is getting accessed on behalf of a user by web application. A Scope generally represents specific set of functionalities within the application. By defining different scopes, you can control access to specific functionalities within the application. Following are some of the microsoft hosted resources.

* Microsoft Graph: ```https://graph.microsoft.com```
* Azure Key Vault: ```https://vault.azure.net```

Microsoft Graph, for example, has defined following set of scopes

* Read a user's calendar
* Write to a user's calendar
* Send mail as a user

Similarly, we will also require to define a scope for our web api application. We have to define a scope in "Application registration" created for webapi in [second part]({{ site.url }}{{ site.baseurl }}/blog/post-protecting-webapi-app-service-using-azure-ad).

To add a scope,

1. Sign in to the Azure Portal
2. Switch to a tenant where you want to register an application. This is done by switching directory.
3. Search for and select **Azure Active Directory**
4. On the left panel, under **Manage**, click on **Application Registration**.
5. Open Application Registration created for **webapi**.
6. On the left panel, under **Manage**, click on **Expose an API**.
![image-center]({{ site.url }}{{ site.baseurl }}/assets/images/03-angular-call-api/03-Expose-an-api-menu.png){: .align-center}
7. click on **"+ Add a scope"**
8. provide  scope name, who can consent? as "Admins and Users", Admin consent display name, Admin consent description, User consent display name, User consent description. 
![image-center]({{ site.url }}{{ site.baseurl }}/assets/images/03-angular-call-api/04-Add-a-scope.png){: .align-center}
9. Click on save.
10. It should show as below
![image-center]({{ site.url }}{{ site.baseurl }}/assets/images/03-angular-call-api/05-added-scope.png){: .align-center}

## 3. Adding API permission.

Now that webapi has exposed a scope, we have to add permission in web application for this scope. We have to do this in "Application Registration" created for web application in [first part]({{ site.url }}{{ site.baseurl }}/blog/post-protecting-angular-front-end-using-azure-ad).

To add a permission,

1. Sign in to the Azure Portal
2. Switch to a tenant where you want to register an application. This is done by switching directory.
3. Search for and select **Azure Active Directory**
4. On the left panel, under **Manage**, click on **Application Registration**.
5. Open Application Registration created for **web application**.
![image-center]({{ site.url }}{{ site.baseurl }}/assets/images/03-angular-call-api/06-add-a-permission.png){: .align-center}
6. Click on **"Add a permission"**
7. Click on **"APIs my organization uses"** tab. It should display Application registration for **"webapi"**
![image-center]({{ site.url }}{{ site.baseurl }}/assets/images/03-angular-call-api/07-add-a-permission-1.png){: .align-center}
8. Click on webapi related registration and it should display scope we have defined in previous section. Select that scope and click on "Add Permission"
![image-center]({{ site.url }}{{ site.baseurl }}/assets/images/03-angular-call-api/08-add-a-permission-2.png){: .align-center}
9. After permission is added. It should show as below
![image-center]({{ site.url }}{{ site.baseurl }}/assets/images/03-angular-call-api/09-permission-added.png){: .align-center}

## 4. Calling a webapi from Angular web application with token.

As previously mentioned, After you have logged into Angular application. ID token is issued to it. We can actually use this token, to acquire another token which is meant for webapi. We are using **MSAL**(Microsoft Authentication Library) in our angular web application. By appropriately configuring MSAL, token for webapi can be  acquired and it can also automatically embed it to all the outogoing calls to webapi.

To achieve this,

1. open folder corresponding to angular webapplication which was created in [first part]({{ site.url }}{{ site.baseurl }}/blog/post-protecting-angular-front-end-using-azure-ad)
2. open ```src->app->app.module.ts``` file
3. Search for ```MSALInterceptorConfigFactory``` method.
4. Add ```Protected Resoruce Map``` for webapi. It is specified in following format. Specify webapi url for ```<url>``` and ```scope``` we have created in previous section.

```console
protectedResourceMap.set('<url>',[scope1,scope2,scope-n]);
```
Add it like below
```javascript
  export function MSALInterceptorConfigFactory(): MsalInterceptorConfiguration {
  const protectedResourceMap = new Map<string, Array<string>>();
  protectedResourceMap.set('https://graph.microsoft.com/v1.0/me', ['user.read']);
  protectedResourceMap.set('https://app-webapi.azurewebsites.net',['api://webapi.techlearning.com/generic']);

  return {
    interactionType: InteractionType.Redirect,
    protectedResourceMap
  };
}
```

Once this is done, MSAL will acquire appropriate token and it will embed it to all the webapi calls. Now start angular web application and login.  Upon login, you would now see consent page where it would ask for "permission" for the "scope" we have created for webapi. See below:

![image-center]({{ site.url }}{{ site.baseurl }}/assets/images/03-angular-call-api/10-consent-screen.png){: .align-center}

Keep browser's "developer tools" open. from the website, navigate to "weather" page. Observe In the network tab that call to webapi is returning **200 ok** response. MSAL is also embedding appropriate token to the call. Weather page should display a table with all the temperature values. See below:

![image-center]({{ site.url }}{{ site.baseurl }}/assets/images/03-angular-call-api/11-success-call.png){: .align-center}

![image-center]({{ site.url }}{{ site.baseurl }}/assets/images/03-angular-call-api/12-success-call-result.png){: .align-center}

## References
* [https://docs.microsoft.com/en-us/azure/app-service/configure-authentication-provider-aad](https://docs.microsoft.com/en-us/azure/app-service/configure-authentication-provider-aad)

## Related Posts
* [Part 1: Protecting Angular Frontend with Azure Active Directory]({{ site.url }}{{ site.baseurl }}/blog/post-protecting-angular-front-end-using-azure-ad)  
* [Part 2: Protecting WebApi deployed in Azure App service with Azure Active Directory]({{ site.url }}{{ site.baseurl }}/blog/post-protecting-webapi-app-service-using-azure-ad)