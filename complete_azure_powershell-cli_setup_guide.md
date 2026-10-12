# Complete Azure PowerShell Setup & Non-Interactive Login Guide

This guide covers everything from installing the `Az` module as an Administrator to setting up a Service Principal, configuring environment variables, and creating a streamlined custom function for passwordless, non-interactive logins.

---

## Phase 1: PowerShell Module Setup

Before interacting with Azure, ensure your environment has the correct PowerShell module installed and updated. *(Note: Run these commands as **Administrator**).*

### 1. Check PowerShell Version
Verify your current PowerShell version:
```powershell
$PSVersionTable
```

### 2. Install or Update the Az Module
If the `Az` module is not installed, install it from the gallery:
```powershell
Install-Module -Name Az -AllowClobber -Repository PSGallery -Force
```

If it is already installed, update it to the latest version:
```powershell
Update-Module -Name Az -AllowClobber -Repository PSGallery
```

### 3. Confirm Installed Version
Verify the active version against powershellgallery.com:
```powershell
Get-InstalledModule -Name Az -AllVersions | Select-Object -Property Name, Version
```

---

## Phase 2: Azure Service Principal Setup

To enable non-interactive scripting without manual browser prompts, create an app registration and grant it permissions.

### 1. Create an App Registration
1. Log in to the Azure Portal.
2. Search for Microsoft Entra ID and navigate to App registrations > New registration.
3. Name: Enter a recognizable name (e.g., `PowerShell-Logon`).
4. Supported account types: Select Accounts in this organizational directory only (Single tenant).
5. Redirect URI: Leave blank, then click Register.
> Save: Copy your Application (client) ID and Directory (tenant) ID from the Overview page.

### 2. Generate a Client Secret
1. In your App Registration, go to Certificates & secrets > Client secrets > New client secret.
2. Add a description and expiration, then click Add.
> Save: Immediately copy the secret Value. It is only shown once.

### 3. Assign Permissions in IAM
1. Navigate to Subscriptions in the Azure Portal and select your subscription.
2. Go to Access control (IAM) > + Add > Add role assignment.
3. Under the Privileged administrator roles, choose a role (e.g., `Owner`).
4. Under Members, select User, group, or service principal, click + Select members, use the search feature to find your app registration, select it, and click Review + assign.

---

## Phase 3: Environment Variables & Profile Integration

Store your credentials securely at the user level and set up a custom profile function for instantaneous logins.

### 1. Set User-Level Environment Variables
Open PowerShell and run:
```powershell
[System.Environment]::SetEnvironmentVariable("AZURE_TENANT_ID", "<tenant id guid>", "User")
[System.Environment]::SetEnvironmentVariable("AZURE_CLIENT_ID", "<application id>", "User")
[System.Environment]::SetEnvironmentVariable("AZURE_CLIENT_SECRET", "<secret value>", "User")
```

### 2. Add the Login Helper Function
Add to your private profile file:
```powershell
function Connect-AzSp {
    $sec = ConvertTo-SecureString $env:AZURE_CLIENT_SECRET -AsPlainText -Force
    $cred = New-Object System.Management.Automation.PSCredential($env:AZURE_CLIENT_ID, $sec)
    Connect-AzAccount -ServicePrincipal -TenantId $env:AZURE_TENANT_ID -Credential $cred
}
```

---

## Phase 4: Daily Usage (PowerShell)
* Login: `Connect-AzSp`
* Verify: `Get-AzContext`
* Logout: `Disconnect-AzAccount`

---

## Phase 5: WSL Ubuntu CLI Setup

### 1. Install Azure CLI on WSL Ubuntu
```bash
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```

### 2. Configure Environment Variables in ~/.bashrc
Add to the end of `~/.bashrc`:
```bash
export AZURE_TENANT_ID="<tenant id guid>"
export AZURE_CLIENT_ID="application id"
export AZURE_CLIENT_SECRET="<secret value>"

azsp() {
    az login --service-principal -u "$AZURE_CLIENT_ID" -p "$AZURE_CLIENT_SECRET" --tenant "$AZURE_TENANT_ID"
}
```

### 3. Apply and Verify
```bash
source ~/.bashrc
azsp
az account show
```