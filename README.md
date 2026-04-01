# Azure Logic App – Reservation Expiry Notification

## Overview

This Logic App provides automated monitoring and reporting of Azure Reservations that are:

- Expiring soon (within a configurable future window)
- Recently expired (within a configurable past window)

It runs on a scheduled basis and sends a formatted HTML email report with reservation details.

---

## Key Capabilities

### 1. Scheduled Execution
- Runs on a configurable schedule (default: weekly, Monday at 8 AM)
- Supports:
  - Custom frequency (Day / Week / Month)
  - Time zone
  - Specific days and times

---

### 2. Azure Reservation Retrieval
- Calls Azure Management API:
  GET /providers/Microsoft.Capacity/reservationOrders
- Uses Managed Identity authentication
- No credentials stored in the Logic App

---

### 3. Intelligent Filtering

Filters reservations based on:

- Expiring soon (within next X days)
- Recently expired (within last Y days)

---

### 4. Data Transformation

Extracts and formats:
- Reservation ID
- Display Name
- Created Date
- Expiration Date
- Term
- Status

---

### 5. HTML Email Report

- Styled HTML table
- Easy-to-read format
- Includes reservation summary and details
<img width="1707" height="566" alt="image" src="https://github.com/user-attachments/assets/99bd87b3-40ec-44a0-a068-2286fc02c66a" />
---

### 6. Email Delivery

- Uses Office 365 connector
- Sends to configured recipients
- Subject is customizable

---

## Architecture Flow

Recurrence Trigger  
→ HTTP Call (Azure Reservations API)  
→ Parse JSON  
→ Filter Expiring / Expired  
→ Transform Data  
→ Build HTML Table  
→ Send Email  

---

## Prerequisites

- Managed Identity with reservation read access
- Office 365 connection (must be authorized after deployment)

---

## Deployment

Deploy via ARM template using PowerShell.

---

## Post-Deployment

Re-authorize the Office 365 connection in Azure Portal.

---

## Benefits

- Proactive cost management
- Prevents missed renewals
- Automated reporting
- Reusable deployment

---

## Summary

This Logic App acts as a lightweight FinOps automation tool for tracking Azure reservation expirations.


# Step-by-Step Setup Guide  
## Azure Logic App – Reservation Expiry Notifications

This guide walks through how to fully set up the Logic App:

1. Create Resource Group  
2. Create Managed Identity  
3. Grant RBAC permissions  
4. Deploy Office 365 connection (ARM)  
5. Deploy Logic App (ARM)  
6. Configure Managed Identity in HTTP action  
7. Re-authorize Office 365 connection  
8. Test the workflow  

---

## 1. Prerequisites

- Azure subscription
- PowerShell with Az module installed
- Permission to:
  - Create resources
  - Assign RBAC roles
- Office 365 account for email sending

---

## 2. Create Resource Group

```powershell
$subscriptionId = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
$resourceGroup  = "rg-logicapps"
$location       = "eastus"

Connect-AzAccount
Set-AzContext -SubscriptionId $subscriptionId

New-AzResourceGroup -Name $resourceGroup -Location $location
```

---

## 3. Create User-Assigned Managed Identity

```powershell
$miName = "mi-logicapp-reservations"

$mi = New-AzUserAssignedIdentity `
  -ResourceGroupName $resourceGroup `
  -Name $miName `
  -Location $location

$managedIdentityResourceId = $mi.Id
$managedIdentityPrincipalId = $mi.PrincipalId

Write-Output $managedIdentityResourceId
Write-Output $managedIdentityPrincipalId
```

---

## 4. Grant RBAC to Managed Identity

Recommended Role:
- Reservations Reader

---

## 5. Deploy Office 365 API Connection (ARM)

No need to create an API connection as this time.

---

## 6. Deploy Logic App via ARM

```powershell
New-AzResourceGroupDeployment `
  -ResourceGroupName $resourceGroup `
  -TemplateFile ".\logicapp-reservation-alerts-arm.json" `
  -TemplateParameterFile ".\logicapp-reservation-alerts-arm.parameters.json"
```

---

## 7. Configure Managed Identity in Logic App HTTP Action

- Open Logic App Designer
- Select HTTP action
- Authentication → Managed Identity
- Choose your identity
<img width="562" height="698" alt="image" src="https://github.com/user-attachments/assets/2a65ece9-ad3e-495a-987e-4e7ede60c334" />
---

## 8. Re-Authorize Office 365 Connection

- Go to Azure Portal → Connections
- Select office365
- Click "Fix connection"
<img width="553" height="722" alt="image" src="https://github.com/user-attachments/assets/b56a21cd-31cd-4847-8f3f-8f9a05dfb84d" />

---

## 9. Test the Logic App

- Click Run Trigger
- Or wait for schedule

---

## Summary

This setup provides automated monitoring of Azure reservations using secure managed identity and email notifications.
