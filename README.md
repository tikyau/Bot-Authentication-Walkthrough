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