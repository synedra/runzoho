# Zoho CRM Tasks Integration

This application uses Zoho CRM Tasks as the backend for a simplified todo list functionality, integrated through RunAlloy.  You can find a more detailed description in this [blog post](https://github.com/synedra/runzoho/blob/main/BLOG_POST.md).

## Prerequisites
- Httpie (instructions below)
- [Node.js](https://nodejs.org) (tested with 24.5)
- [Netlify](https://netlify.com) account
- [Zoho](https://zoho.com) account
- [RunAlloy](https://runalloy.com) API token



## Setup (click section name for details)

<details><summary>Environment Variables</summary>

**Get Your Credentials**

1. **RunAlloy API Key**: Get from RunAlloy dashboard → Settings → API Keys
2. **Zoho CRM Connector**: Configure the Zoho CRM connector in RunAlloy dashboard
3. **Zoho OAuth Credentials**: These are used by RunAlloy internally for the Zoho CRM connector

**Set Your Environment Variables**
Add these variables to your `.env` file:

```bash
# RunAlloy API Configuration
RUNALLOY_API_KEY=your_api_key_here
RUNALLOY_API_URL=https://production.runalloy.com

# Zoho OAuth Configuration 
ZOHO_CLIENT_ID=your_zoho_client_id
ZOHO_CLIENT_SECRET=your_zoho_client_secret
```
</details>

<details><summary>Httpie Setup</summary></summary>
### 2. Setup httpie

Curl is kind of a pain for making API calls, as you need to set the headers separately each time.  We'll use 'httpie', a command line option that allows you to set up your headers for every call.

1. **Install [Httpie](https://httpie.io/cli)**
2. **Setup httpie session**

Create your config file.  This instructs httpie to use the correct header file for each command.

```json
{
	"default_options": [
	    "--session-read-only=~/.config/httpie/default_headers_session.json"
	]
    }
```

Save this as `~/.config/httpie/config.json` (create the directory if it doesn't exist).

**Create session file**
Add the session header file to complete this setup:

```json
{
    "__meta__": {
	"about": "HTTPie session file",
	"help": "https://httpie.io/docs#sessions",
	"httpie": "3.2.4"
    },
    "auth": {
	"password": null,
	"type": null,
	"username": null
    },
    "cookies": [],
    "headers": [
	{
	    "name": "x-api-version",
	    "value": "2025-06"
	},
	{
	    "name": "Authorization",
	    "value": "Bearer j3oDqPBhf-ZuTGHeDf2Ru"
	},
	{
	    "name": "x-alloy-userid",
	    "value": "68f1e561ba205b5a3bf234c8"
	}
    ]
}
```

Save this as `~/.config/httpie/default_headers_session.json` 

**Test it out**

Run the following command to check your httpie setup:

``` bash
https https://production.runalloy.com/connectors
```

You can see all the connectors here.  We'll be using the 'zohoCRM' connector for the todo list.

For production, you will want to create this workflow dynamically, but for this example we will create the user and credential using the CLI, then plug them into the code.
</details>

<details><summary>Create user</summary>
**Create the user**
Run the following command to create a new user.
``` bash
https https://production.runalloy.com/users \
username="<your email address>" \
fullName="<Full name>"
```

This will return a string which you will use for your userId (like '68f1e561ba205b5a3bf234c8').  If you lose this string, you can find the user with the following command:

``` bash
https https://production.runalloy.com/users
```

Pick the user matching the email you used.  In the default_headers_session.json file (in ~/.config/httpie) change the x-alloy-userid to the user you just created.

Update the Authorization entry so that it is "Bearer <RUNALLOY_API_KEY>"
</details>

<details><summary>Setup netlify</summary>
	
**Setup your netlify environment**

To use this repository, you will need to get netlify set up.

**Setup the repository and application**

Click the following button to deploy the codebase to Netlify.  You need to have a Netlify account for this to work.

[![Deploy to Netlify](https://www.netlify.com/img/deploy/button.svg)](https://app.netlify.com/start/deploy?repository=https://github.com/synedra/runzoho)

Click "Connect with Github" to create a github repository and connect it to the netlify application.  Choose a unique name for your netlify application (like zoho-<github-username>)

Clone your newly created repository to your system and install the requirements.

```
git clone https://github.com/<github-username>/zoho-<github-username>
cd zoho-<github-username>
npm install
```

**Check the deploy**

Browse to `https://<yourappname>.netlify.app` and you should be given the chance to login with an email address.  Wait for a moment as we need to create the credential first.

</details>

<details><summary>Create credential</summary>

In your terminal window do the following:

``` bash
https https://production.runalloy.com/connectors/zohoCRM/credentials \ userId=68f1e561ba205b5a3bf234c8 \
authenticationType=oauth2 \
redirectUri=https://runzoho.netlify.app/.netlify/functions/zoho-auth \
data:='{"region":"com"}'
```

This will give you an oauth URL.  Copy and paste that URL into your browser window and it will let you login with Zoho - if you don't have a Zoho account already, create one now.

After you've done the OAuth login, your credential should be available.  Check the credentials to find the one you created.

``` bash
https://production.runalloy.com/connectors/zohoCRM/credentials
```

It will look like this:

``` json
        {
            "createdAt": "2025-10-20T17:48:10.015Z",
            "credentialId": "68f675dac4fc59f453aa25fb",
            "name": "Kirsten Hunter's Zoho CRM (10)",
            "type": "zohoCRM-oauth2",
            "updatedAt": "2025-10-21T15:46:57.982Z"
        }
```

Once you've found your credential you can update the code to use that credential.

In netlify/functions/zoho-tasks.cjs update the following lines:

```
    globalState.set("userId", "<userid from create command>");
    globalState.set("credentialId", "<credential from list of credentials>");
```
</details>

**Test locally**

In your working directory, start the test server.

```
netlify dev
```

**Deploy to the web**

```
netlify deploy --prod
```

