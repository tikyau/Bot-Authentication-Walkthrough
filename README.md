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

1. &lt;appSettings&gt;
2.   &lt;!-- update these  **with**  your BotId, Microsoft App Id and your Microsoft App Password--&gt;
3.   &lt;add key=&quot;BotId&quot; value=&quot;YourBotId&quot; /&gt;
4.   &lt;add key=&quot;MicrosoftAppId&quot; value=&quot;&quot; /&gt;
5.   &lt;add key=&quot;MicrosoftAppPassword&quot; value=&quot;&quot; /&gt;
6.
7.   &lt;!-- AAD Auth v1 settings --&gt;
8.   &lt;add key=&quot;ActiveDirectory.Mode&quot; value=&quot;v1&quot; /&gt;
9.   &lt;add key=&quot;ActiveDirectory.ResourceId&quot; value=&quot;https://graph.windows.net/&quot; /&gt;
10.   &lt;add key=&quot;ActiveDirectory.EndpointUrl&quot; value=&quot;https://login.microsoftonline.com&quot; /&gt;
11.   &lt;add key=&quot;ActiveDirectory.Tenant&quot; value=&quot;dxdemos.net&quot; /&gt;
12.   &lt;add key=&quot;ActiveDirectory.ClientId&quot; value=&quot;2d3b5788-05a5-486d-b2a4-2772a4511396&quot; /&gt;
13.   &lt;add key=&quot;ActiveDirectory.ClientSecret&quot; value=&quot;wU3oFBJ1gjWcB8Lo/fMaaCwg7ygg8Y9zBJlUq+0yBN0=&quot; /&gt;
14.   &lt;add key=&quot;ActiveDirectory.RedirectUrl&quot; value=&quot;http://localhost:3979/api/OAuthCallback&quot; /&gt;
15.
16.
17. &lt;/appSettings&gt;

**Step 2

**Select Global.asax.cs file and call all the bot app setting property and assign to AuthBot model class, like below
1. using System.Configuration;
2. using System.Web.Http;
3.
4. namespace DevAuthBot
5. {
6.      **public**   **class**  WebApiApplication : System.Web.HttpApplication
7.     {
8.          **protected**   **void**  Application\_Start()
9.         {
10.             GlobalConfiguration.Configure(WebApiConfig.Register);
11.             AuthBot.Models.AuthSettings.Mode = ConfigurationManager.AppSettings[&quot;ActiveDirectory.Mode&quot;];
12.             AuthBot.Models.AuthSettings.EndpointUrl = ConfigurationManager.AppSettings[&quot;ActiveDirectory.EndpointUrl&quot;];
13.             AuthBot.Models.AuthSettings.Tenant = ConfigurationManager.AppSettings[&quot;ActiveDirectory.Tenant&quot;];
14.             AuthBot.Models.AuthSettings.RedirectUrl = ConfigurationManager.AppSettings[&quot;ActiveDirectory.RedirectUrl&quot;];
15.             AuthBot.Models.AuthSettings.ClientId = ConfigurationManager.AppSettings[&quot;ActiveDirectory.ClientId&quot;];
16.             AuthBot.Models.AuthSettings.ClientSecret = ConfigurationManager.AppSettings[&quot;ActiveDirectory.ClientSecret&quot;];
17.         }
18.     }
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

1. /// &lt;summary&gt;
2. /// Login and Logout
3. /// &lt;/summary&gt;
4. /// &lt;param name=&quot;context&quot;&gt;&lt;/param&gt;
5. /// &lt;param name=&quot;item&quot;&gt;&lt;/param&gt;
6. /// &lt;returns&gt;&lt;/returns&gt;
7. **public**  virtual async Task MessageReceivedAsync(IDialogContext context, IAwaitable&lt;IMessageActivity&gt; item)
8. {
9.      **var**  message = await item;
10.
11.     //endpoint v1
12.      **if**  (string.IsNullOrEmpty(await context.GetAccessToken(ConfigurationManager.AppSettings[&quot;ActiveDirectory.ResourceId&quot;])))
13.     {
14.         //Navigate to website for Login
15.         await context.Forward( **new**  AzureAuthDialog(ConfigurationManager.AppSettings[&quot;ActiveDirectory.ResourceId&quot;]),  **this**.ResumeAfterAuth, message, CancellationToken.None);
16.     }
17.      **else**
18.     {
19.         //Logout
20.         await context.Logout();
21.         context.Wait(MessageReceivedAsync);
22.     }
23. }
24.
25. /// &lt;summary&gt;
26. /// ResumeAfterAuth
27. /// &lt;/summary&gt;
28. /// &lt;param name=&quot;context&quot;&gt;&lt;/param&gt;
29. /// &lt;param name=&quot;result&quot;&gt;&lt;/param&gt;
30. /// &lt;returns&gt;&lt;/returns&gt;
31. **private**  async Task ResumeAfterAuth(IDialogContext context, IAwaitable&lt;string&gt; result)
32. {
33.     //AD resposnse message
34.      **var**  message = await result;
35.
36.     await context.PostAsync(message);
37.     context.Wait(MessageReceivedAsync);
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