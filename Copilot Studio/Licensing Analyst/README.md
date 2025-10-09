# Licensing Analyst Solution Documentation

---

## Overview
The **Licensing Analyst** solution is an intelligent, autonomous agent system designed to streamline Office 365 license management and upgrade requests across the organization.  
It leverages **Power Automate**, **Dataverse**, and multiple **AI agents** to handle the full lifecycle of a license request — from intake to approval and provisioning.

---

## Key Capabilities
- Accepts **license requests** via Microsoft Forms or email  
- Handles **user inquiries** such as:
  - “What licenses do I currently have?”
  - “How does the license request process work?”
  - “How long does an upgrade usually take?”
- Sends **AI-generated updates** to requesters when Dataverse records are created or modified  
- Performs **automated reasoning** using grounded knowledge from the *Microsoft Enterprise Licensing Comparison Guide*  
- Manages **multi-step approvals** and auto-provisions licenses via Entra ID

---

## Pre-requisites
### Environment Variables
- There are environment variables you will need to fill in to have the solution function properly for your organization
  - **MSFormsID** - This is the ID guid for a Microsoft Form if you wanted to allow users to submit requests via a Form.
  - **Licensing EntraID Group Object ID** - This should be the object ID of your security group that is controlling your licensing. This could be expanded to referencing a table of many different licensing groups where you could have multiple licensing request options.
  - **Procurement Email** - This is an email address where you'd like approval requests to be sent in the second/final stage of your licensing request approvals.
  - **Manager Demo Skip** - Put an email address here if you'd like to demo the solution and have your manager stage approvals come to a test account instead.
### Table Data
- There is a .csv file in the repository called **Product names and service plans**. You can import this into the Product Service Plans dataverse table within the solution. You can always get an up-to-date version of this file from http://aka.ms/serviceplanids 

---

## Agent Instructions
### Licensing Analyst
```
# Licensing Approvals Agent

## Purpose
Assists license approvers—primarily non-technical financial stakeholders—with:
- New licensing and upgrade requests
- Reviewing a user’s current licenses
- Sending status emails based on approvals and updates

---

## New Request Workflow (run in order)

1. **Query Active Directory & Knowledge**
   - Retrieve the requester’s basic details using **Active Directory Analyst**.
   - Search grounded knowledge sources to assess whether the request is reasonable.

2. **Log Decision**
   - Use **Dataverse → Add row to `LicensingRequest`**.
   - Record the recommendation and metadata.
   - If a field is unknown, set it to `null` (do not prompt the user).

3. **Send Receipt**
   - Use **Send an email (V2)** to confirm receipt to the requester.
   - Do **not** estimate timelines or status—just confirm it was submitted for approval.

> _End of new-request workflow._

---

## Triggers

### 1) Dataverse Record Updated
When a Dataverse record update triggers the agent:
- Review **Manager Approval** and **Procurement Approval** statuses.
- **Final outcome logic**
  - If both Manager **and** Procurement approve → **Approved**
  - These approvals always supersede the AI recommendation.
- **Notify the requester** using **Send an email (V2)**  
  Format the email using **HTML** with:
  - Bold headings and clear sections
  - Bullet list showing **who** responded and the **status** of each approval
  - A clearly stated **Final Status**

> **Note:** Do not include the AI recommendation in the final outcome email.

> _End of update-trigger workflow._

---

### 2) Email Received
- Use the above instructions + grounded knowledge + available tools to answer the question.
- After drafting the summary, reply with **Reply to email**.
```
### Active Directory Analyst
```
# User Validation & Licensing Verification Workflow

This workflow retrieves user details, validates their current licensing, and passes the data back to the Licensing Agent.

---

## Step 1 – Get User Details

**Goal:** Retrieve the user’s identity and metadata.

**Actions:**
1. Use **Search for users** to get their ID and User Principal Name (UPN).  
   - Query with:  
     - Requesting user’s **email address**, **ID**, or **UPN**

2. Pull detailed user metadata with **Send an HTTP request**:

Retrieves:
- `id`
- `userPrincipalName`
- `jobTitle`
- `department`
- `companyName`
- `employeeid`

3. Retrieve current assigned licenses using another **HTTP request**:

Returns all assigned license SKU IDs for that user.

---

## Step 2 – Get Licensing Entitlements

**Goal:** Understand what the user is entitled to based on existing licenses.

**Action:**  
Pass the output from the `/licenseDetails` call into the  
**Check Licensing Entitlements of User** tool.

---

## Step 3 – Return Results to Licensing Agent

**Goal:** Feed the collected data back into the main workflow.

**Action:**  
Send all gathered user details and license entitlement data back to the **Licensing Agent**.  
Once complete, continue the original workflow steps as designed.

---

### Summary Table

| Step | Tool / Action | Purpose | Key Output |
|------|----------------|----------|-------------|
| 1 | Search for users | Retrieve user ID & UPN | `id`, `userPrincipalName` |
| 1 | HTTP GET /users/{id} | Get user metadata | `jobTitle`, `department`, `companyName`, `employeeid` |
| 1 | HTTP GET /users/{id}/licenseDetails | Get assigned licenses | `skuId` list |
| 2 | Check Licensing Entitlements of User | Evaluate current entitlements | Entitlement summary |
| 3 | Send data to Licensing Agent | Continue workflow | Updated request context |

---

**Tip:**  
Use bold headers, code blocks, and tables like above—GitHub renders these cleanly, and they’re far more readable than nested blockquotes.
```
---


## Process Flow

### 1. Intake & Triggering
The agent can be triggered through multiple channels:
- **Microsoft Form submissions** for new license requests  
- **Direct email inquiries** for process or status questions  
- **Dataverse record updates** that initiate automated notifications  

**Screenshot:**
<img width="867" height="245" alt="Screenshot 2025-10-09 at 3 42 55 AM" src="https://github.com/user-attachments/assets/74aaf318-f11a-4e57-9075-8956e513d664" />

---

### 2. Request Evaluation
When a new license request is received, the Licensing Analyst triggers its child agent — **Active Directory Analyst** — to gather contextual details about the user.  
This includes:
- Fetching user information and current licenses from **Azure AD**
- Running an **AI-powered evaluation** to determine if an E3 upgrade is justified
- Consulting its **grounded knowledge base**, sourced from the *Microsoft Enterprise Licensing Comparison Guide*, to rationalize the request

The agent:
- Logs all details in **Dataverse** (custom table: `LicenseRequests`)
- Sends a **confirmation email** to the requester via the **Office 365 Outlook connector**

![LicensingAnalystDemoGIF-ezgif com-optimize-2](https://github.com/user-attachments/assets/a6d67af7-9519-4f88-8897-41d27170fc56)


**Screenshots:**
<img width="938" height="606" alt="Screenshot 2025-10-09 at 3 42 06 AM" src="https://github.com/user-attachments/assets/35e9fe17-e4d4-48e0-ad3e-2e33a51be3a7" />
<img width="701" height="591" alt="Screenshot 2025-10-09 at 3 13 03 AM" src="https://github.com/user-attachments/assets/50e05dd9-d70a-47f3-99af-b47276d5b798" />

---

### 3. Approval Workflow
Upon creation of a new record in Dataverse, a **multi-step Power Automate flow** is triggered:

1. **Manager Approval Workflow** — routes the request to the requester’s direct manager  
2. **Procurement Approval Workflow** — routes the request to the **Procurement Team** email address defined via an environment variable  

Once both approvals are received:
- The agent updates the Dataverse record
- A “Final Outcome” email is sent summarizing approvals

**Screenshots:**
<img width="1134" height="662" alt="Screenshot 2025-10-09 at 3 10 04 AM" src="https://github.com/user-attachments/assets/9f65985c-2ab0-44dd-b566-4572aa7c23ca" />
<img width="1199" height="656" alt="Screenshot 2025-10-09 at 3 10 26 AM" src="https://github.com/user-attachments/assets/a4f5a10e-1d13-4517-856f-c46054b2e337" />

---

### 4. License Provisioning
When the request has been approved by all parties, a provisioning flow executes to:
- Add the user to the appropriate **Entra ID group**
- Confirm successful license assignment via an **automated email notification**

**Screenshots:**

<img width="702" height="586" alt="Screenshot 2025-10-09 at 3 13 21 AM" src="https://github.com/user-attachments/assets/5de4a254-1ae0-4bb0-bc80-a23622c46d50" />
<img width="711" height="578" alt="Screenshot 2025-10-09 at 3 13 28 AM" src="https://github.com/user-attachments/assets/f25bfda6-dd6a-4a71-99a2-749650b18f15" />

---

## Lifecycle Summary

| Stage | Action | Output |
|-------|---------|--------|
| **Request Submission** | User submits a form or sends an email | Agent logs request in Dataverse |
| **License Evaluation** | Active Directory Analyst retrieves and rationalizes data | AI recommendation logged |
| **Approval Process** | Manager and Procurement Team review | Status updated in Dataverse |
| **Provisioning** | Entra ID Group assignment | License provisioned + confirmation email sent |
