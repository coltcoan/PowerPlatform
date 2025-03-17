# Supercharging REST APIs with Copilot Studio and AI Builder
<img width="750" alt="RW1lvmB" src="https://github.com/user-attachments/assets/6dad8ff7-32b0-474d-9b49-5f7428d8f0dd" />

## Overview

As organizations continue to embrace AI-driven automation, one of the most powerful transformations is enabling natural language interactions with enterprise systems. I recently built a Copilot Studio agent that dynamically queries Jira Cloud's REST API—but instead of static, predefined queries, it uses AI Builder to understand user intent, construct optimized JQL (Jira Query Language) queries, and retrieve only the most relevant results.

This solution removes complexity by allowing users to ask questions naturally and letting AI figure out the best API call. Instead of manually writing:

```
project = "Customer Support" AND status = "In Progress" AND assignee = “John Doe”
```

The user could simply ask:

```
"Show me all in-progress support tickets assigned to John Doe.”
```

## Technical Aspects

The Copilot agent, backed by AI Builder and HTTP actions, dynamically translates this into the correct API call, removing the burden of JQL knowledge from end users, as well as removing the need for many topics or branches for every possible scenario.

**Tech used:**
- Copilot Studio
- AI Builder
- Power Automate
- Dataverse
- Azure Key Vault
- Jira (can be any third-party REST API)

## Process

### Step 1. Secure API Access with Jira API Token

First, generate an API token from the Atlassian API token page. This will be necessary to authenticate in the case of Jira.

![image-1](https://github.com/user-attachments/assets/a8616d9c-e381-424e-ae2a-517497a5b516)

Stored the API token in Azure Key Vault to avoid exposing credentials directly in Copilot Studio or cloud flows.

I also created a Secret environment variable to be able to securely reference the secret later on. Additionally, you may want to create a second environment variable to store the username (email address) that is tied to this API token, as you’ll need both items to successfully authenticate.

![image-2](https://github.com/user-attachments/assets/ce9a687a-51f2-44fa-9898-646b26e4e322)

### Step 2. Setting Up Copilot Studio Agent

Create a new solution in Power Platform to encapsulate the entire project, ensuring portability and manageability.

> PS: If you’re new to solutions and Application Lifecycle Management, check out this section on Pipelines in Power Platform from our white paper on establishing a Power Platform tenant strategy: https://learn.microsoft.com/en-us/power-platform/guidance/white-papers/environment-strategy#pipelines-in-power-platform

In Power Apps, navigate to your solution you’ve just created and select “Agent”, under the “Create new” menu dropdown.

Assign a clear agent name and description, ensuring it aligns with Jira issue tracking for this example.

Configure the agent's personality and behavior using the instructions field.

Configure the agent's purpose and prompting by setting explicit instructions for how it should interpret and respond to user queries.

![image-3](https://github.com/user-attachments/assets/b201142c-1130-4012-89f6-74234a996d2e)

### Step 3. Optimizing Queries with AI Builder (GPT Model)

Create an AI Builder model leveraging GPT to understand and process natural language queries.

In Power Apps, navigate to the new AI Prompts section and select "Build your own prompt."

Define training examples mapping user input to API query structures. I’ve included an example of my prompt for reference.

> A key part of the prompting and instructions is configuring your AI Builder model to output only the necessary API endpoint, ensuring that the generated queries were optimized for retrieving the most relevant data while not providing additional commentary.

![image-4](https://github.com/user-attachments/assets/7da98372-aa2f-4900-a7a4-ad207c7bf5d2)

### Step 4. Preparing Authentication to Jira

Create another topic in your agent, responsible for querying Jira.

To be able to authenticate your REST API requests to Jira, we’ll need to create a Base64 string for the authentication. 

To do this, we’re going to create a Power Automate flow to securely get our API token and convert what we need to a Base64 string and pass it back to the agent.

- Create a flow action
- Use the “perform an unbound action” action in your flow
- Look for the RetrieveEnvironmentVariableSecretValue action
- Set the EnvironmentVariableName parameter to the name of your environment variable you created (Note: This is NOT the display name, but rather the logical name of your environment. This will have your publisher prefix in front of it. Ex: contoso_MCS_APIToken, not “MCS API Token”.
- Under the settings section of the Unbound action step, turn “secure outputs” to on. This will ensure the outputs of this action are secured and your plain text token is not visible in your flow runs.

![image-5](https://github.com/user-attachments/assets/0d4438ba-05bc-4730-8e1d-02dea1c1d10d)

In the “Respond to Copilot” action, we’re going to create an expression to convert our API token and username into a Base64 string. Your expression will look similar to this:
```
base64(concat(parameters('MCS_Username (contoso_MCS_Username)'), outputs('Perform_an_unbound_action')?['body/EnvironmentVariableSecretValue']))
```
Save and publish your flow.

![image-5a](https://github.com/user-attachments/assets/ebec2d2e-62dd-477b-8e95-4b0e0b0a9cdf)

Go back to your agent and now add the flow you’ve created as an action.

### Step 5. Setting up HTTP Call

Now that we’ve established our AI prompt, our authentication, and converting the authentication to the proper format, we can set up our HTTP call to Jira.

Add an HTTP action in your topic, after the Base64 conversion flow action.

In the URL parameter, set this to the Global Variable we created from the output of our Query Optimization flow action.

Click Edit in the Headers and body section

In the header section, Add a new key and call it Authorization

In the value, create a formula and it should look something like this:
```
 "Basic "&Topic.Base64Output
 ```
> For the Response data type, you’ll want to select “From sample data”. This is where you’re going to want to paste a sample response payload from Jira into here, so you can easily parse and use information from the API’s response downstream in your agent. You can find this on Jira’s API reference documentation.

![image-6](https://github.com/user-attachments/assets/37ccd09a-f470-417f-a658-97575f481e93)
![image-6a](https://github.com/user-attachments/assets/61d07f34-9635-4b90-a0d9-916ad90855fb)

### Step 6. Processing and Presenting Results

Once we query Jira for this information, we’re going to want to be able to generate an answer to the end user. This is where Generative Answers comes in. Typically, this action is used with knowledge sources such as SharePoint, Dataverse, or public websites. 

However, you can use custom data sources such as JSON responses in a Generative Answers node. Let me show you how:

- First, you’ll want to add a “Create generative answers” node, after the HTTP action node
- Click “edit” on the Data sources section.
- You’ll want to make sure “Allow the AI to use its own general knowledge” is Off in this instance. We only want this action to use the data provided by us.
- Click the dropdown next to Classic data
- We’re going to create a Formula here. This formula is running a For All loop on all of the issues that were returned based on our HTTP query, and formatting them into the expected format, calling out the Content, as well as ContentLocation. For ContentLocation, I’ve left this as blank for this example.
- Keep in mind, this formula will need to be tweaked to account for any specific fields you want the LLM to reason over. I’ve accounted for several in my example formula below:
   
![image-7](https://github.com/user-attachments/assets/f17ecddb-d7c6-40fe-9047-e4d67a314bcd)
```
ForAll(

    Topic.JiraParsedRecord.issues,

    {

        Content:

            Text(id) & ", " &

            key & ", " &

            fields.summary & ", " &

            fields.issuetype.name & ", " &

            fields.project.name & ", " &

            fields.status.name & ", " &

            fields.priority.name & ", " &

            fields.reporter.displayName & ", " &

            fields.assignee.displayName & ", " &

            Text(fields.created) & ", " &

            Text(fields.updated) & ", " &

            If(IsBlank(fields.description), "No description provided",fields.description) & ", " &

            Text(fields.resolutiondate) & ", " &

            If(IsBlank(fields.resolution.name), "Unresolved", fields.resolution.name) &", " &

            If(IsBlank(fields.duedate), "No due date", Text(fields.duedate)),

 

        ContentLocation: Blank()

    }

)
```
Click insert. Next, you’ll want to make sure your instructions are clear in this node. Here’s an example of some instructions I used:

> Do not remove any records when summarizing the response. If the user asks for 5 records as an example, they should receive 5 items in response, if the items exist.
>If you do not see the value they're looking for in the results, use this type of talk track: "The closest results I found were:" and then provide the results using the sample formatting provided, in markdown >format.

Here's a sample response:

> The issues currently assigned to John Doe are:
> ISSUE-5
> Status: Done
> Priority: Medium
> 
> ISSUE-1
> Status: In Progress
> Priority: High
> 
> ISSUE-7
> Status: To Do
> Priority: Low

Once you feel the instructions are to your liking, Save your changes. You should be ready to start using your new Copilot Studio agent and dynamically querying Jira about issues in your instance!

## Conclusions

Let’s recap what we’ve done:

>- Generated an API token in Jira and tied both the token and our username securely to Azure Key Vault and created environment variables so we can easily but securely reference this data.
>- Created an AI Builder prompt model that can reason over API reference documentation and dynamically provide the proper query information for Jira
>- Built a Copilot Studio agent that can utilize the query optimization of our AI Builder prompt, and then call Jira via HTTP
>- Finally, we converted the output from Jira into a custom data source we can have Generative Answers reason over and provide a friendly response back to the end user.

This approach can be expanded beyond Jira—the same principles can be applied to integrations with other enterprise platforms where maybe their Power Platform connector doesn’t have very flexible or usable filtering options or simply doesn’t exist (yet!). I can see this type of solution really allowing organizations to make AI-driven automation a core part of business workflows.

Feel free to message me with any questions or feedback. I hope this tutorial was helpful!

## References

### Environment strategy white paper
https://aka.ms/powerplatformenvironmentstrategy
 
### Using environment variables for Azure Key Vault secrets
https://learn.microsoft.com/en-us/power-apps/maker/data-platform/environmentvariables-azure-key-vault-secrets
 
### Using AI Builder Prompt actions in Copilot Studio
https://learn.microsoft.com/en-us/ai-builder/use-a-custom-prompt-in-mcs
 
### Use a custom data source for Generative Answers nodes
https://learn.microsoft.com/en-us/microsoft-copilot-studio/nlu-generative-answers-custom-data
