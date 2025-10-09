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
