# Using Dataverse Web API in Power Automate via REST API

This documentation will walk you through the steps to correctly fill out the environment variables upon import, ensuring a smooth setup process. Additionally, we'll explain how the key vault variable takes precedence if both are filled in. Finally, we will guide you on how to find your API URL in the schema section of your Dataverse table using the Power Apps site. Let's get started!

### _Important Warning_ ###

_This solution is provided for sample purposes only. It is intended to demonstrate how you can achieve similar outcomes in your own projects. There is no warranty from me or Microsoft, and it is not meant for production use out of the box. Please customize and thoroughly test the solution before deploying it in any production environment._

--------------------------------

## 1. Importing Your Solution

When you import your Dataverse Web API Demo Solution, you'll want to import this into your **Developer Environment** as this is an unmanaged solution.

## 2. Setting up Environment Variables
You'll need to set up the environment variables. Follow these steps:

1. **Navigate to Environment Variables**
   - Go to the solution that you've imported.
   - You will see a list of environment variables below.

2. **Fill Out the Environment Variables**
   - **DataverseWebAPI_ClientID**: Enter your Client ID here.
   - **DataverseWebAPI_ClientSecret_KeyVault**: Enter the key vault reference if you are using Azure Key Vault.
   - **DataverseWebAPI_ClientSecret_PlainText**: Enter the plain text client secret if you are not using Azure Key Vault.
   - **DataverseWebAPI_TenantID**: Enter your Tenant ID here.
   - **DataverseWebAPI_URL**: Enter your Dataverse API URL here.

### _Important Note_

_If both the `DataverseWebAPI_ClientSecret_KeyVault` and `DataverseWebAPI_ClientSecret_PlainText` variables are filled in, the key vault variable (`DataverseWebAPI_ClientSecret_KeyVault`) will take precedence. This ensures that your sensitive information is stored securely in Azure Key Vault. **It is highly recommended to use Azure Key Vault for storing of all secrets.**_

## 3. Finding Your API URL in the Dataverse Table Schema

To find your API URL in the schema section of your Dataverse table, follow these steps:

1. **Access Power Apps Site**
   - Navigate to the [Power Apps](https://make.powerapps.com) site.
   - Ensure you are logged in with your credentials.

2. **Navigate to Dataverse**
   - On the left-hand navigation pane, select **Tables**.

3. **Select Your Table**
   - Click on the table for which you need the API URL.

4. **Find the API URL**
   - Once you are in the table, click on the **Tools** option in the Table Properties pane.
   - Click **API link to table data** to get the URL to your org and table data.
   - The URL you receive will have a $top-10 filter on it. You can remove this or modify to include whatever filters you'd like later on inside of your HTTP calls.

## 4. Finalizing Your Setup

After filling out all necessary environment variables and ensuring the key vault precedence, you are ready to use your Dataverse Web API Demo Solution. Double-check all entries to ensure there are no mistakes. Your solution should now be configured correctly and ready for use.

If you encounter any issues or need further assistance, please refer to the [Power Platform documentation](https://docs.microsoft.com/powerapps/).

Thank you for exploring the Dataverse Web API Demo Solution! We hope this guide has been helpful. Happy building!

