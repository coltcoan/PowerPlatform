# Copilot Navigator

Copilot Navigator is a Power Platform solution designed to help users discover and access the Microsoft Copilot tools they are licensed for. It provides a seamless interface to check licensing status and launch available Copilot products, or initiate access requests for those not yet provisioned.

---

## 🚀 Purpose

Copilot Navigator enables users to:

- Identify which Copilot tools they have access to:
  - **M365 Copilot**
  - **M365 Copilot Chat**
  - **Copilot Studio**
- Automatically check licensing by querying Microsoft Graph and evaluating the user's SKUs through an AI-powered prompt.
- Launch the Copilot tools they are licensed for.
- Navigate to request forms for tools they are not yet licensed to use.

---

## 🛠️ Pre-requisites

Before deploying Copilot Navigator, ensure the following requirements are met:

### Licensing & Environment

- Power Apps **Premium** license is required.
- The solution file is **unmanaged** and **customizable**.
- Environment variables must be configured during import.

### Security Configuration

- Assign the **Copilot Product Viewers** security role to the security group managing access to the Copilot Navigator app.

### Environment Variables

  - Security group object ID for **Copilot Studio Authors** (configured in Power Platform Admin Center under Tenant Settings).
  - Security group object ID for **M365 Copilot license assignments**.
  - URL for requesting **M365 Copilot** licensing.
  - URL for requesting **M365 Copilot Chat** licensing.
  - URL for requesting **Copilot Studio** licensing.

### Data

    - Import the CopilotProductList.csv file to the 'Copilot Products' Dataverse table to start with an initial list of products.

---

## 📦 Deployment Notes

- This solution is intended for internal enterprise use and may require customization to align with your organization's licensing structure and access control policies.
- Ensure all environment variables and security roles are correctly configured before publishing the app to users.

---

## 📄 License

This project is provided as-is under your organization’s licensing terms. Please consult your internal compliance team before external distribution.

