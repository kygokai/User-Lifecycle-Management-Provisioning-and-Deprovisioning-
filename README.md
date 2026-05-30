# User-Lifecycle-Management-Provisioning-and-Deprovisioning-
Joiner–Mover–Leaver (JML) workflows

Phase I. Environment Setup & Tools
1.	Creating Tenant (activate the Entra ID Premium P2 trial)
2.	Defining Lab Scenario:

•	Company: “TechHive”
•	Departments: HR, Sales and Help Desk.
•	Access To Applications: 
-HR needs access to Work Day. 
-Sales needs access to Salesforce.
-Help Desk needs access to ServiceNow.

3.	Setup Applications
•	Used Entra ID's pre-integrated gallery apps to simulate access.
•	In the properties of the apps, ensure Assignment required? is set to Yes.

<img width="975" height="403" alt="image" src="https://github.com/user-attachments/assets/5073341e-dceb-4d59-8957-03ace240deda" />

 
Phase II: Configuration & Policies (Dynamic Access)
1.	Creating Groups (Dynamic Security Groups)
 <img width="975" height="417" alt="image" src="https://github.com/user-attachments/assets/415a4db8-de91-4c89-891e-5ff9a4e11eb8" />
 <img width="975" height="435" alt="image" src="https://github.com/user-attachments/assets/9c194868-b466-4307-8bf0-59eee3be584e" />
 <img width="975" height="328" alt="image" src="https://github.com/user-attachments/assets/7dc2329f-08a3-4456-b3dd-07f5f678d8bb" />

2.	Assigning Apps Permissions to Groups
   <img width="975" height="435" alt="image" src="https://github.com/user-attachments/assets/dd9d975e-b852-4257-8f88-b591da0a944c" />
   <img width="975" height="438" alt="image" src="https://github.com/user-attachments/assets/b4c23fce-1170-422d-9cbb-a8670d2b90e8" />
   <img width="975" height="431" alt="image" src="https://github.com/user-attachments/assets/844e48d3-d361-4674-a1a0-cd99087d41c7" />


# Simulates a full Joiner, Mover, Leaver (JML) IAM workflow using Microsoft Graph

DESCRIPTION:
    1. Joiner: Creates a new user in the Sales department.
    2. Mover: Updates the user's department to HR (triggering dynamic group rules).
    3. Leaver: Disables the account and revokes all active sign-in sessions.

0. AUTHENTICATION & SETUP
```powershell
Set-ExecutionPolicy -ExecutionPolicy Bypass
EXPLANATION: Temporarily sets the script execution policy to Bypass, allowing all scripts to run without prompts or warnings.

Install-Module Microsoft.Graph -Scope CurrentUser -Repository PSGallery -Force
EXPLANATION: Installs the base Microsoft Graph PowerShell module for the current user from the PowerShell Gallery, forcing installation even if already present.

Install-Module Microsoft.Graph.Users -Scope CurrentUser -Force
EXPLANATION: Installs the Microsoft.Graph.Users module, which includes cmdlets for managing user accounts, scoped to the current user.

Connect-MgGraph -Scopes "Group.ReadWrite.All", "User.ReadWrite.All", “Directory.ReadWrite.All”, “User.RevokeSession.All”
EXPLANATION: Connects to Microsoft Graph with delegated permissions to read/write both users and groups.

# Define User Variables
$Domain = "sajemaph.onmicrosoft.com" 
$UPN = "lebronjames@$Domain"
$Password = "Welcome2026!"
$PasswordProfile = @{
    Password = $Password
    ForceChangePasswordNextSignIn = $true
}

1. THE JOINER WORKFLOW (Hire)
Write-Host "`n[JOINER] Creating new employee: Lebron James (Sales)" -ForegroundColor Green
Try {
    $NewUser = New-MgUser -DisplayName "Lebron James" `
        -UserPrincipalName $UPN `
        -MailNickname "lebron.james" `
        -PasswordProfile $PasswordProfile `
        -Department "Sales" `
        -JobTitle "Sales Associate" `
        -City "Manila" `
        -AccountEnabled:$true
} Catch {
    Write-Error "Failed to create user: $_" -ForegroundColor Red
}
   
<img width="269" height="288" alt="image" src="https://github.com/user-attachments/assets/6d6e569b-9cb9-418c-87f0-fefe70188acb" />
<img width="975" height="233" alt="image" src="https://github.com/user-attachments/assets/76166b32-f3c9-45b3-bc36-81297db98696" />


2. THE MOVER WORKFLOW (Transfer)
Write-Host "`n[MOVER] Transferring Lebron to HR..." -ForegroundColor Yellow
Try {
    Update-MgUser -UserId $NewUser.Id `
        -Department "HR" `
        -JobTitle "HR Manager"
    Write-Host "Success: Lebron’s attributes updated."
    Write-Host "-> Behind the scenes: Entra ID moves Lebron from PH-Sales' and adds them to PH-HR, granting WorkDay access."
} Catch {
    Write-Error "Failed to update user: $_" -ForegroundColor Red
}

 <img width="975" height="327" alt="image" src="https://github.com/user-attachments/assets/09e35c2d-01e2-485b-addf-390bba0b3f41" />


3.	THE LEAVER WORKFLOW (Resignation/Termination)
Write-Host "`n[LEAVER] Offboarding Lebron and revoking access..." -ForegroundColor Red

Try {
    # Step A: Disable the account so they cannot initiate new sign-ins
    Update-MgUser -UserId $NewUser.Id -AccountEnabled:$false
    Write-Host "Success: Account disabled."
    # Step B: Revoke active sessions (Kicks them out of any apps currently open in their browser)
    Revoke-MgUserSignInSession -UserId $NewUser.Id | Out-Null
    Write-Host "Success: All active sign-in sessions have been revoked."
    Write-Host "-> Lebron is completely locked out of the SajemaPH environment."
} Catch {
    Write-Error "Failed to offboard user: $_" -ForegroundColor Red
}
Write-Host "`nJML Lifecycle Simulation Complete." -ForegroundColor Cyan
   <img width="301" height="251" alt="image" src="https://github.com/user-attachments/assets/147b13ed-2da9-4ac6-a62a-1d41f95eac41" />
   <img width="975" height="446" alt="image" src="https://github.com/user-attachments/assets/0e2ce792-b376-4993-9f83-1a807523ce95" />
