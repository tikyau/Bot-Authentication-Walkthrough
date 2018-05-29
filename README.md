Introduction

The Azure AD is the identity provider, responsible for verifying the identity of users and applications and providing security tokens upon successful authentication of those users and applications. In this article, I have explained about creating Azure AD authentication and integrating into bot applications using AuthBot library.

Bot Application 

The Bot shows a very simple dialog with openUrl button and this button launches the web browser for validating user credentials and AD will respond to the message with an authentication code. You can copy the same code and reply back to the bot, the bot will validate and respond to the welcome message.

You can follow the below-given steps one by one and you will get to see an interesting demo at the end of this article.

Azure AD App registration

I will show the steps given below for Azure application creation, user creation, and permission configuration. While implementing the bot application, we need Client ID, tenant, return URL. So here, I will show how to get all the configuration information from the steps given below.

Step 1

Login to Microsoft Azure portal and choose Azure Active Directory from the sidebar.

Step 2

If you have not created Azure Active Directory, try to create a new AD creation for tenant URL or select or add tenant URL from the Domain names sections.

Bot Application 
Step 3

Select "Application Registration" and provide the details given below - Name for the application, application type as Web app/API, enter your application redirect URL, and click on "Create".

Bot Application 
Step 4

We need to give the permission to access the application from Bot, so grant the permission. Select the newly created application > select Required Permission > click Grant permission.

Step 5

Create a new user from "Users and Groups" section (optional).

Step 6

Create a client secret key from the application. Select Application > select keys > add new / copy client secret key.

Bot Application 
Step 4

You can copy the tenant, client ID, and Client Secret and you can follow the below steps for creating and implementing AD authentication in Bot.

Create new Bot application

Let's create a new bot application using Visual Studio 2017. Open Visual Studio > Select File > Create New Project (Ctrl + Shift +N) > select Bot application.

Bot Application 

The Bot application template gets created with all the components and all required NuGet references installed in the solutions.

Bot Application 

Install AuthBot Nuget Package

The AuthBot provide Azure Active Directory authentication library for implement Azure AD login in Bot .

Right click on Solution, select Manage NuGet Package for Solution > Search “ AuthBot” > select Project and install the package.

Bot Application 

You can follow given below steps for integrate AD authentication

Step 1

Select Web.config file and add Mode,resourceID,Endpointurl ,Tenant,clientID,clientSecret and redirect url appsettings property and replace Azure AD details as per below

<appSettings>  
  <!-- update these with your BotId, Microsoft App Id and your Microsoft App Password-->  
  <add key="BotId" value="YourBotId" />  
  <add key="MicrosoftAppId" value="" />  
  <add key="MicrosoftAppPassword" value="" />  
  
  <!-- AAD Auth v1 settings -->  
  <add key="ActiveDirectory.Mode" value="v1" />  
  <add key="ActiveDirectory.ResourceId" value="https://graph.windows.net/" />  
  <add key="ActiveDirectory.EndpointUrl" value="https://login.microsoftonline.com" />  
  <add key="ActiveDirectory.Tenant" value="dxdemos.net" />  
  <add key="ActiveDirectory.ClientId" value="2d3b5788-05a5-486d-b2a4-2772a4511396" />  
  <add key="ActiveDirectory.ClientSecret" value="wU3oFBJ1gjWcB8Lo/fMaaCwg7ygg8Y9zBJlUq+0yBN0=" />  
  <add key="ActiveDirectory.RedirectUrl" value="http://localhost:3979/api/OAuthCallback" />  
  
  
</appSettings>  
Step 2

Select Global.asax.cs file and call all the bot app setting property and assign to AuthBot model class, like below

using System.Configuration;  
using System.Web.Http;  
  
namespace DevAuthBot  
{  
    public class WebApiApplication : System.Web.HttpApplication  
    {  
        protected void Application_Start()  
        {  
            GlobalConfiguration.Configure(WebApiConfig.Register);  
            AuthBot.Models.AuthSettings.Mode = ConfigurationManager.AppSettings["ActiveDirectory.Mode"];  
            AuthBot.Models.AuthSettings.EndpointUrl = ConfigurationManager.AppSettings["ActiveDirectory.EndpointUrl"];  
            AuthBot.Models.AuthSettings.Tenant = ConfigurationManager.AppSettings["ActiveDirectory.Tenant"];  
            AuthBot.Models.AuthSettings.RedirectUrl = ConfigurationManager.AppSettings["ActiveDirectory.RedirectUrl"];  
            AuthBot.Models.AuthSettings.ClientId = ConfigurationManager.AppSettings["ActiveDirectory.ClientId"];  
            AuthBot.Models.AuthSettings.ClientSecret = ConfigurationManager.AppSettings["ActiveDirectory.ClientSecret"];  
        }  
    }  
}  
Step 3

You can create a new AzureADDialog class to show the default login and logout UI Design dialog. Rightclick on Project, select Add New Item, create a class that is marked with the [Serializable] attribute (so the dialog can be serialized to state), and implement the IDialog interface.

using AuthBot;  
using AuthBot.Dialogs;  
using Microsoft.Bot.Builder.Dialogs;  
using Microsoft.Bot.Connector;  
using System;  
using System.Configuration;  
using System.Threading;  
using System.Threading.Tasks;  
  
namespace DevAuthBot.Dialogs  
{  
    [Serializable]  
    public class AzureADDialog : IDialog<string>  
    { 
Step 4 

IDialog interface has only StartAsync() method. StartAsync() is called when the dialog becomes active. The method passes the IDialogContext object, used to manage the conversation.

public async Task StartAsync(IDialogContext context)  
{  
    context.Wait(MessageReceivedAsync);  
} 
Step 5

Create a MessageReceivedAsync method and write the following code for the login and logout default dialog and create a ResumeAfterAuth for after the user login, bot will reply the user name and email id details.

/// <summary>  
/// Login and Logout  
/// </summary>  
/// <param name="context"></param>  
/// <param name="item"></param>  
/// <returns></returns>  
public virtual async Task MessageReceivedAsync(IDialogContext context, IAwaitable<IMessageActivity> item)  
{  
    var message = await item;  
  
    //endpoint v1  
    if (string.IsNullOrEmpty(await context.GetAccessToken(ConfigurationManager.AppSettings["ActiveDirectory.ResourceId"])))  
    {  
        //Navigate to website for Login  
        await context.Forward(new AzureAuthDialog(ConfigurationManager.AppSettings["ActiveDirectory.ResourceId"]), this.ResumeAfterAuth, message, CancellationToken.None);  
    }  
    else  
    {  
        //Logout  
        await context.Logout();  
        context.Wait(MessageReceivedAsync);  
    }  
}  
  
/// <summary>  
/// ResumeAfterAuth  
/// </summary>  
/// <param name="context"></param>  
/// <param name="result"></param>  
/// <returns></returns>  
private async Task ResumeAfterAuth(IDialogContext context, IAwaitable<string> result)  
{  
    //AD resposnse message   
    var message = await result;  
  
    await context.PostAsync(message);  
    context.Wait(MessageReceivedAsync);  
} 
After the user enters the first message, our bot will reply and ask to login to the AD. Then, it waits for Authentication code and bot will reply the user details as a response like below.

Bot Application 
Run Bot Application

The emulator is a desktop application that lets us test and debug our bot on localhost. Now, you can click on "Run the application" in Visual studio and execute in the browser

Bot Application 
Test Application on Bot Emulator

You can follow the below steps to test your bot application.

Open Bot Emulator.
Copy the above localhost url and paste it in emulator e.g. - http://localHost:3979
You can append the /api/messages in the above url; e.g. - http://localHost:3979/api/messages.
You won't need to specify Microsoft App ID and Microsoft App Password for localhost testing, so click on "Connect".

Bot Application 