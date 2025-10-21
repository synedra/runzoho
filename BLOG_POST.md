# Building a Modern Task Manager: Integrating Zoho CRM with React Using RunAlloy

*A step-by-step tutorial on creating a streamlined task management application that connects Zoho CRM Tasks to a React frontend through RunAlloy's unified API platform.*

<img width="1020" height="765" alt="image" src="https://github.com/user-attachments/assets/a2810272-f1d8-4ff1-b0b0-469bc4b35ca2" />

## The Challenge

Building integrations with third-party APIs like Zoho CRM can be complex. You need to handle OAuth flows, manage API credentials, deal with rate limiting, and navigate different API schemas. What t[...] 

In this tutorial, we'll build a React-based task manager that uses Zoho CRM's Tasks module as the backend, all routed through RunAlloy's connector platform. The result is a simplified, single-list tas[...] 

## Why This Architecture?

Instead of directly integrating with Zoho CRM's API, we're using RunAlloy as an intermediary layer. This approach provides several key benefits:

- **Centralized Authentication**: RunAlloy handles OAuth credentials and token refresh
- **Unified API**: Consistent interface across different connectors
- **Built-in Rate Limiting**: No need to implement your own throttling
- **Better Error Handling**: Standardized error responses
- **Monitoring**: Track API usage through RunAlloy's dashboard

## The Stack

Our application uses:
- **Frontend**: React with simplified single-task-list UI
- **Backend**: Netlify Functions for serverless API endpoints
- **Integration Layer**: RunAlloy for Zoho CRM connectivity
- **Task Storage**: Zoho CRM Tasks module
- **CLI Tool**: HTTPie for API testing and setup

## Architecture Overview

```
Frontend (React) - Single Task List
    ↓
Netlify Functions (zoho-tasks.cjs)
    ↓
RunAlloy Helper (runalloy-helper.cjs)
    ↓
RunAlloy API
    ↓
Zoho CRM Tasks Module
```

## Prerequisites

Before we begin, make sure you have:
- [Node.js](https://nodejs.org) (v24.5 or later)
- [HTTPie CLI](https://httpie.io/cli) for API interactions
- A [Netlify](https://netlify.com) account
- A [Zoho](https://zoho.com) account
- A [RunAlloy](https://runalloy.com) API token

## Part 1: Setting Up HTTPie for API Management

One of the most tedious parts of working with APIs is managing headers for every request. We'll use HTTPie, a modern command-line HTTP client, to streamline our API interactions.

### Installing HTTPie

First, install HTTPie following the instructions at [httpie.io/cli](https://httpie.io/cli). On macOS, you can use Homebrew:

```bash
brew install httpie
```

### Configuring HTTPie Sessions

HTTPie sessions allow us to store headers and reuse them across requests. This is perfect for API keys and user IDs that don't change.

**Step 1: Create the HTTPie config file**

Create the directory and config file:

```bash
mkdir -p ~/.config/httpie
```

Create `~/.config/httpie/config.json` with:

```json
{
    "default_options": [
        "--session-read-only=~/.config/httpie/default_headers_session.json"
    ]
}
```

**Step 2: Create the session file**

Create `~/.config/httpie/default_headers_session.json` with your RunAlloy credentials:

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
            "value": "Bearer YOUR_RUNALLOY_API_KEY"
        },
        {
            "name": "x-alloy-userid",
            "value": "YOUR_USER_ID"
        }
    ]
}
```

**Step 3: Test the setup**

Test your HTTPie configuration:

```bash
https https://production.runalloy.com/connectors
```

You should see a list of available connectors, including `zohoCRM`.

## Part 2: Deploying the Application (do this before creating credentials)

Important: Deploy the Netlify app before creating your RunAlloy Zoho credential. The OAuth redirect URI that RunAlloy will ask for needs to point to your deployed site's Netlify Functions endpoint. Deploying first ensures you have a stable redirect URI to use when configuring the Zoho credential.

### Quick Deploy with Netlify

The fastest way to get started is using Netlify's one-click deploy:

[![Deploy to Netlify](https://www.netlify.com/img/deploy/button.svg)](https://app.netlify.com/start/deploy?repository=https://github.com/synedra/runzoho)

This will:
1. Fork the repository to your GitHub account
2. Create a Netlify application
3. Set up automatic deployments

Choose a unique name for your application (like `zoho-yourname`).

### Local Development Setup

Clone your newly created repository:

```bash
git clone https://github.com/YOUR_USERNAME/YOUR_REPO_NAME
cd YOUR_REPO_NAME
npm install
```

### Configuration

Update the configuration in `netlify/functions/zoho-tasks.cjs` with values you'll obtain later from RunAlloy:

```javascript
globalState.set("userId", "YOUR_USER_ID");
globalState.set("credentialId", "YOUR_CREDENTIAL_ID");
```

Note: You'll set the actual `userId` and `credentialId` after creating your RunAlloy user and Zoho credential in the next section. Deploying first ensures you know your site's URL so you can provide an accurate OAuth redirect URI when creating the credential.

### Environment Variables

Create a `.env` file with:

```bash
# RunAlloy API Configuration
RUNALLOY_API_KEY=your_api_key_here
RUNALLOY_API_URL=https://production.runalloy.com

# Zoho OAuth Configuration
ZOHO_CLIENT_ID=your_zoho_client_id
ZOHO_CLIENT_SECRET=your_zoho_client_secret
```

## Part 3: Setting Up RunAlloy Users and Credentials

With your Netlify app deployed and HTTPie configured, you can now set up the RunAlloy infrastructure needed for your application. Having the deployed Netlify URL at hand is important because you'll use it as the OAuth redirect URI when creating the Zoho credential.

### Creating a RunAlloy User

Every integration needs a user context. Create one with:

```bash
https https://production.runalloy.com/users \
    username="your-email@example.com" \
    fullName="Your Full Name"
```

This returns a user ID (like `68f1e561ba205b5a3bf234c8`). Update your HTTPie session file with this user ID in the `x-alloy-userid` header.

If you lose the user ID, you can retrieve it:

```bash
https https://production.runalloy.com/users
```

### Creating a Zoho CRM Credential

Now create the credential that will handle Zoho authentication. Use the redirect URI for your deployed Netlify app's function (for example: `https://YOUR_APP_NAME.netlify.app/.netlify/functions/zoho-auth`):

```bash
https https://production.runalloy.com/connectors/zohoCRM/credentials \
    userId=YOUR_USER_ID \
    authenticationType=oauth2 \
    redirectUri=https://YOUR_APP_NAME.netlify.app/.netlify/functions/zoho-auth \
    data:='{"region":"com"}'
```

This returns an OAuth URL. Open it in your browser to complete the Zoho authentication flow.

After authentication, list your credentials to get the credential ID:

```bash
https https://production.runalloy.com/connectors/zohoCRM/credentials
```

Note the `credentialId` from the response - you'll need this for your application and to update the `netlify/functions/zoho-tasks.cjs` configuration shown earlier.

## Part 4: Testing and Development

### Local Testing

Start the development server:

```bash
netlify dev
```

Navigate to `http://localhost:8888` to see your application. You should be able to:
- View your Zoho CRM tasks
- Create new tasks
- Update existing tasks (including status changes)
- Delete tasks
- See rich task information (priority, due dates, descriptions)

### Production Deployment

Deploy to production:

```bash
netlify deploy --prod
```

## Part 5: Understanding the Code Architecture

### Frontend Components

The React frontend is intentionally simplified:

- **App.js**: Main application container with authentication flow
- **BoardComponent.js**: Simplified task display (no board selection)
- **TodoList.js**: Enhanced task management with rich Zoho information
- **AppContext.js**: Global state management

### Backend Functions

The Netlify functions handle the API integration:

- **zoho-tasks.cjs**: Main CRUD operations for tasks
- **zoho-auth.cjs**: OAuth callback handler
- **runalloy-helper.cjs**: RunAlloy API integration layer

### Key Features Implemented

1. **Single Task List View**: Removed the complexity of multiple boards
2. **Enhanced Task Display**: Priority badges, due dates, owner information
3. **Task Completion Toggle**: Checkbox for easy status updates
4. **Rich Task Information**: Descriptions, creation dates, modification times
5. **Proper Error Handling**: User-friendly error messages
6. **Loading States**: Visual feedback during API operations

## Part 6: Zoho CRM Field Mappings

The application maps Zoho CRM task fields to our UI:

- **Subject**: Task name/title
- **Status**: Not Started, In Progress, Completed
- **Priority**: High, Normal, Low  
- **Description**: Task description
- **Due_Date**: Due date for task
- **Owner**: Task assignee
- **Created_Time**: Creation timestamp
- **Modified_Time**: Last modification timestamp

## Benefits of This Approach

### For Developers

- **Simplified Integration**: No need to handle Zoho OAuth flows directly
- **Consistent API**: Same patterns work across different connectors
- **Built-in Monitoring**: See API usage and errors in RunAlloy dashboard
- **Faster Development**: Focus on UI/UX instead of integration complexity

### For Users

- **Clean Interface**: Single task list without board complexity
- **Rich Information**: See priority, due dates, and descriptions at a glance
- **Real-time Sync**: Changes appear immediately in Zoho CRM
- **Mobile Friendly**: Responsive design works on all devices

## Testing Your Integration

Use HTTPie to test the RunAlloy integration directly:

```bash
# List tasks
https https://production.runalloy.com/actions/zohoCRM/listTasks \
    userId=YOUR_USER_ID

# Create a task
https https://production.runalloy.com/actions/zohoCRM/createTask \
    userId=YOUR_USER_ID \
    data:='{"Subject":"Test Task","Status":"Not Started","Priority":"Normal"}'
```

## Troubleshooting Common Issues

### "Authorization token required"
- Check your `RUNALLOY_API_KEY` in the environment variables
- Verify the API key is valid in RunAlloy dashboard

### "Credential ID required"  
- Ensure your Zoho connector is configured in RunAlloy dashboard
- Verify the credential exists and is active

### "Failed to fetch tasks"
- Check that Zoho CRM connector is properly configured
- Verify credential has necessary permissions for Tasks module
- Check RunAlloy dashboard for API errors

## What's Next?

This tutorial demonstrates a simplified but powerful approach to building modern integrations. You could extend this application by:

- Adding task filtering and sorting
- Implementing task categories or tags
- Adding due date notifications
- Integrating with other Zoho CRM modules
- Adding team collaboration features

## Conclusion

By combining React, Netlify Functions, and RunAlloy, we've created a modern task management application that's both powerful and maintainable. The key insight is using RunAlloy as an integration layer[...] 

The HTTPie setup provides a professional workflow for API testing and development, making it easy to iterate and debug during development.

This architecture pattern can be applied to integrate with any of RunAlloy's 200+ connectors, making it a scalable approach for building modern, integrated applications.

---

## Resources

- **Live Demo**: [Your deployed application URL]
- **Source Code**: [GitHub repository URL]
- **RunAlloy Documentation**: https://docs.runalloy.com
- **Zoho CRM API**: https://www.zoho.com/crm/developer/docs/api/
- **HTTPie Documentation**: https://httpie.io/docs
- **Netlify Functions**: https://docs.netlify.com/functions/

---

*This tutorial demonstrates real-world integration patterns using modern tools and best practices. The complete source code and deployment templates are available for immediate use.*
