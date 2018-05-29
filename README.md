**Introduction**

The Azure AD is the identity provider, responsible for verifying the identity of users and applications and providing security tokens upon successful authentication of those users and applications. In this article, I have explained about creating Azure AD authentication and integrating into bot applications using AuthBot library.

![Bot Application ](https://csharpcorner-mindcrackerinc.netdna-ssl.com/article/building-bot-application-with-azure-ad-login-authentication-using-authbot/Images/image001.jpg)

The Bot shows a very simple dialog with openUrl button and this button launches the web browser for validating user credentials and AD will respond to the message with an authentication code. You can copy the same code and reply back to the bot, the bot will validate and respond to the welcome message.

You can follow the below-given steps one by one and you will get to see an interesting demo at the end of this article.

**Azure AD App registration**

I will show the steps given below for Azure application creation, user creation, and permission configuration. While implementing the bot application, we need Client ID, tenant, return URL. So here, I will show how to get all the configuration information from the steps given below.

**Step 1

**Login to Microsoft [Azure portal](https://portal.azure.com/) and choose Azure Active Directory from the sidebar.

**Step 2

**If you have not created Azure Active Directory, try to create a new AD creation for tenant URL or select or add tenant URL from the Domain names sections.

![Bot Application ](https://csharpcorner-mindcrackerinc.netdna-ssl.com/article/building-bot-application-with-azure-ad-login-authentication-using-authbot/Images/image002.jpg)

**Step 3**

Select "Application Registration" and provide the details given below - Name for the application, application type as Web app/API, enter your application redirect URL, and click on "Create".

![Bot Application ](https://csharpcorner-mindcrackerinc.netdna-ssl.com/article/building-bot-application-with-azure-ad-login-authentication-using-authbot/Images/image003.jpg)

**Step 4**

We need to give the permission to access the application from Bot, so grant the permission. Select the newly created application > select Required Permission > click Grant permission.

**Step 5

**Create a new user from "Users and Groups" section (optional).

**Step 6

**Create a client secret key from the application. Select Application > select keys > add new / copy client secret key.

![Bot Application ](https://csharpcorner-mindcrackerinc.netdna-ssl.com/article/building-bot-application-with-azure-ad-login-authentication-using-authbot/Images/image004.jpg)

**Step 4

**You can copy the tenant, client ID, and Client Secret and you can follow the below steps for creating and implementing AD authentication in Bot.

**Create new Bot application**

Let's create a new bot application using Visual Studio 2017. Open Visual Studio > Select File > Create New Project (Ctrl + Shift +N) > select Bot application.

![Bot Application ](https://csharpcorner-mindcrackerinc.netdna-ssl.com/article/building-bot-application-with-azure-ad-login-authentication-using-authbot/Images/image005.jpg)

The Bot application template gets created with all the components and all required NuGet references installed in the solutions.

![Bot Application ](https://csharpcorner-mindcrackerinc.netdna-ssl.com/article/building-bot-application-with-azure-ad-login-authentication-using-authbot/Images/image006.jpg)

**Install AuthBot Nuget Package**

The [AuthBot](https://www.nuget.org/packages/AuthBot) provide Azure Active Directory authentication library for implement Azure AD login in Bot .

Right click on Solution, select Manage NuGet Package for Solution > Search " AuthBot" > select Project and install the package.

![Bot Application ](https://csharpcorner-mindcrackerinc.netdna-ssl.com/article/building-bot-application-with-azure-ad-login-authentication-using-authbot/Images/image007.jpg)

You can follow given below steps for integrate AD authentication

**Step 1

**Select Web.config file and add Mode,resourceID,Endpointurl ,Tenant,clientID,clientSecret and redirect url appsettings property and replace Azure AD details as per below

1.  <appSettings>  
2.  <!-- update these with your BotId, Microsoft App Id and your Microsoft App Password-->  
3.  <add key="BotId" value="YourBotId" />  
4.  <add key="MicrosoftAppId" value="" />  
5.  <add key="MicrosoftAppPassword" value="" />  

7.  <!-- AAD Auth v1 settings -->  
8.  <add key="ActiveDirectory.Mode" value="v1" />  
9.  <add key="ActiveDirectory.ResourceId" value="https://graph.windows.net/" />  
10. <add key="ActiveDirectory.EndpointUrl" value="https://login.microsoftonline.com" />  
11. <add key="ActiveDirectory.Tenant" value="dxdemos.net" />  
12. <add key="ActiveDirectory.ClientId" value="2d3b5788-05a5-486d-b2a4-2772a4511396" />  
13. <add key="ActiveDirectory.ClientSecret" value="wU3oFBJ1gjWcB8Lo/fMaaCwg7ygg8Y9zBJlUq+0yBN0=" />  
14. <add key="ActiveDirectory.RedirectUrl" value="http://localhost:3979/api/OAuthCallback" />  

17. </appSettings>  

**Step 2

**Select Global.asax.cs file and call all the bot app setting property and assign to AuthBot model class, like below

1.  using System.Configuration;  
2.  using System.Web.Http;  

4.  namespace DevAuthBot  
5.  {  
6.  public class WebApiApplication : System.Web.HttpApplication  
7.  {  
8.  protected void Application_Start()  
9.  {  
10. GlobalConfiguration.Configure(WebApiConfig.Register);  
11. AuthBot.Models.AuthSettings.Mode = ConfigurationManager.AppSettings["ActiveDirectory.Mode"];  
12. AuthBot.Models.AuthSettings.EndpointUrl = ConfigurationManager.AppSettings["ActiveDirectory.EndpointUrl"];  
13. AuthBot.Models.AuthSettings.Tenant = ConfigurationManager.AppSettings["ActiveDirectory.Tenant"];  
14. AuthBot.Models.AuthSettings.RedirectUrl = ConfigurationManager.AppSettings["ActiveDirectory.RedirectUrl"];  
15. AuthBot.Models.AuthSettings.ClientId = ConfigurationManager.AppSettings["ActiveDirectory.ClientId"];  
16. AuthBot.Models.AuthSettings.ClientSecret = ConfigurationManager.AppSettings["ActiveDirectory.ClientSecret"];  
17. }  
18. }  
19. }  

**Step 3

**You can create a new AzureADDialog class to show the default login and logout UI Design dialog. Rightclick on Project, select Add New Item, create a class that is marked with the [Serializable] attribute (so the dialog can be serialized to state), and implement the IDialog interface.

1.  using AuthBot;  
2.  using AuthBot.Dialogs;  
3.  using Microsoft.Bot.Builder.Dialogs;  
4.  using Microsoft.Bot.Connector;  
5.  using System;  
6.  using System.Configuration;  
7.  using System.Threading;  
8.  using System.Threading.Tasks;  

10. namespace DevAuthBot.Dialogs  
11. {  
12. [Serializable]  
13. public class AzureADDialog : IDialog<string>  
14. { 

**Step 4

**IDialog interface has only StartAsync() method. StartAsync() is called when the dialog becomes active. The method passes the IDialogContext object, used to manage the conversation.

1.  public async Task StartAsync(IDialogContext context)  
2.  {  
3.  context.Wait(MessageReceivedAsync);  
4.  } 

**Step 5

**Create aMessageReceivedAsyncmethod and write the following code for the login and logout default dialog and create a ResumeAfterAuth for after the user login, bot will reply the user name and email id details.

1.  /// <summary>  
2.  /// Login and Logout  
3.  /// </summary>  
4.  /// <param name="context"></param>  
5.  /// <param name="item"></param>  
6.  /// <returns></returns>  
7.  public virtual async Task MessageReceivedAsync(IDialogContext context, IAwaitable<IMessageActivity> item)  
8.  {  
9.  var message = await item;  

11. //endpoint v1  
12. if (string.IsNullOrEmpty(await context.GetAccessToken(ConfigurationManager.AppSettings["ActiveDirectory.ResourceId"])))  
13. {  
14. //Navigate to website for Login  
15. await context.Forward(new AzureAuthDialog(ConfigurationManager.AppSettings["ActiveDirectory.ResourceId"]), this.ResumeAfterAuth, message, CancellationToken.None);  
16. }  
17. else  
18. {  
19. //Logout  
20. await context.Logout();  
21. context.Wait(MessageReceivedAsync);  
22. }  
23. }  

25. /// <summary>  
26. /// ResumeAfterAuth  
27. /// </summary>  
28. /// <param name="context"></param>  
29. /// <param name="result"></param>  
30. /// <returns></returns>  
31. private async Task ResumeAfterAuth(IDialogContext context, IAwaitable<string> result)  
32. {  
33. //AD resposnse message   
34. var message = await result;  

36. await context.PostAsync(message);  
37. context.Wait(MessageReceivedAsync);  
38. } 

After the user enters the first message, our bot will reply and ask to login to the AD. Then, it waits for Authentication code and bot will reply the user details as a response like below.

![Bot Application ](https://csharpcorner-mindcrackerinc.netdna-ssl.com/article/building-bot-application-with-azure-ad-login-authentication-using-authbot/Images/image008.jpg)

**Run Bot Application**

The emulator is a desktop application that lets us test and debug our bot on localhost. Now, you can click on "Run the application" in Visual studio and execute in the browser

![Bot Application ](https://csharpcorner-mindcrackerinc.netdna-ssl.com/article/building-bot-application-with-azure-ad-login-authentication-using-authbot/Images/image009.jpg)\
**Test Application on Bot Emulator**

You can follow the below steps to test your bot application.

1.  Open Bot Emulator.
2.  Copy the above localhost url and paste it in emulator e.g. - http://localHost:3979
3.  You can append the /api/messages in the above url; e.g. - http://localHost:3979/api/messages.
4.  You won't need to specify Microsoft App ID and Microsoft App Password for localhost testing, so click on "Connect".

    ![Bot Application ](https://csharpcorner-mindcrackerinc.netdna-ssl.com/article/building-bot-application-with-azure-ad-login-authentication-using-authbot/Images/image010.jpg)