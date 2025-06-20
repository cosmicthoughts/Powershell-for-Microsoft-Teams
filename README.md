# Powershell-for-Microsoft-Teams

A collection of simple and effective PowerShell commands for managing Microsoft Teams using the official MicrosoftTeams PowerShell module.

These scripts are designed to be easy to use, adapt, and automate for day-to-day Teams administration tasks.

---

## Requirements
- Windows PowerShell 5.1+ or PowerShell Core 7+
- MicrosoftTeams module installed
## Install powershell module
``` powershell
Install-Module -Name MicrosoftTeams -Force
```
## Connect to Microsoft Teams
``` powershell
Connect-MicrosoftTeams
```
## List all Teams
``` powershell
Get-Team | Select DisplayName, GroupId, Visibility
```
## List All Users in a Specific Team
``` powershell
# Replace with your Team's GroupId
$groupId = "<your-group-id>"

Get-TeamUser -GroupId $groupId | Select User, Role
```
## List All Teams with Owners
``` powershell
$teams = Get-Team
foreach ($team in $teams) {
    Write-Host "Team: $($team.DisplayName)"
    Get-TeamUser -GroupId $team.GroupId | Where-Object {$_.Role -eq "Owner"} | Select -ExpandProperty User
    Write-Host ""
}
```
## Remove a user from all teams
``` powershell
# Replace with the target user's UPN
$user = "user@domain.com"

$teams = Get-Team
foreach ($team in $teams) {
    $members = Get-TeamUser -GroupId $team.GroupId
    if ($members.User -contains $user) {
        Remove-TeamUser -GroupId $team.GroupId -User $user -Confirm:$false
        Write-Host "Removed $user from $($team.DisplayName)"
    }
}
```
## Add a User to a Team
``` powershell
# Replace with Team GroupId and user UPN
Add-TeamUser -GroupId "<group-id>" -User "user@domain.com"
```
# Microsoft Teams Policies
## List all messaging policies
``` powershell
Get-CsTeamsMessagingPolicy
```
## List all meeting policies
``` powershell
Get-CsTeamsMeetingPolicy
```
## List all Calling Policies
``` powershell
Get-CsTeamsCallingPolicy
```
### List all App Setup Policies
``` powershell
Get-CsTeamsAppSetupPolicy
```
## List all App Permission Policies
``` powershell
Get-CsTeamsAppPermissionPolicy
```
## List all Meeting Broadcast (Live Events) Policies
``` powershell
Get-CsTeamsMeetingBroadcastPolicy
```
## List all Emergency Calling Policies
``` powershell
Get-CsTeamsEmergencyCallingPolicy
```
## List all Emergency Call Routing Policies
``` powershell
Get-CsTeamsEmergencyCallRoutingPolicy
```
## List all Calling Line Identify Policies
``` powershell
Get-CsCallingLineIdentity
```
## List all Teams Upgrade Policies
``` powershell
Get-CsTeamsUpgradePolicy
```
## Export all teams policies and all properties
``` powershell
<#
.SYNOPSIS
Retrieves all Microsoft Teams policies and exports every property to CSV.

.DESCRIPTION
This script gathers every Teams policy type and exports the full object (all properties) into individual CSV files. 
Useful for full audits or documentation of tenant configuration.

.PARAMETER ExportPath
Folder path where CSVs will be saved.

.EXAMPLE
.\List-AllTeamsPolicies.ps1 -ExportPath "C:\TeamsPolicies\"
#>

param (
    [Parameter(Mandatory = $true)]
    [string]$ExportPath
)

function Export-IfPath {
    param (
        [string]$Name,
        [object]$Data
    )
    if ($ExportPath) {
        $fullPath = Join-Path -Path $ExportPath -ChildPath "$Name.csv"
        $Data | Export-Csv -Path $fullPath -NoTypeInformation
        Write-Host "Exported $Name to $fullPath" -ForegroundColor Green
    }
}

# Create folder if it doesn't exist
if (-not (Test-Path $ExportPath)) {
    New-Item -Path $ExportPath -ItemType Directory | Out-Null
}

# --- Export all policy types ---

Export-IfPath -Name "MessagingPolicies"             -Data (Get-CsTeamsMessagingPolicy)
Export-IfPath -Name "MeetingPolicies"               -Data (Get-CsTeamsMeetingPolicy)
Export-IfPath -Name "CallingPolicies"               -Data (Get-CsTeamsCallingPolicy)
Export-IfPath -Name "AppSetupPolicies"              -Data (Get-CsTeamsAppSetupPolicy)
Export-IfPath -Name "AppPermissionPolicies"         -Data (Get-CsTeamsAppPermissionPolicy)
Export-IfPath -Name "MeetingBroadcastPolicies"      -Data (Get-CsTeamsMeetingBroadcastPolicy)
Export-IfPath -Name "EmergencyCallingPolicies"      -Data (Get-CsTeamsEmergencyCallingPolicy)
Export-IfPath -Name "EmergencyRoutingPolicies"      -Data (Get-CsTeamsEmergencyCallRoutingPolicy)
Export-IfPath -Name "CallingLineIdentityPolicies"   -Data (Get-CsCallingLineIdentity)
Export-IfPath -Name "UpgradePolicies"               -Data (Get-CsTeamsUpgradePolicy)
```



## List all policies assigned to a user
``` powershell
<#
.SYNOPSIS
Retrieves all Teams-related policies assigned to a specific user.

.DESCRIPTION
This script queries the full set of Teams policies assigned to a given user account, using Get-CsOnlineUser. 
Useful for auditing or troubleshooting permission and feature access in Microsoft Teams.

.PARAMETER UserPrincipalName
The UPN (email) of the user whose policy assignments you want to view.

.EXAMPLE
.\Get-TeamsUserPolicies.ps1 -UserPrincipalName "user@domain.com"
#>

param (
    [Parameter(Mandatory = $true)]
    [string]$UserPrincipalName
)

$user = Get-CsOnlineUser -Identity $UserPrincipalName

if (-not $user) {
    Write-Error "User not found: $UserPrincipalName"
    exit
}

[PSCustomObject]@{
    DisplayName                       = $user.DisplayName
    UserPrincipalName                 = $user.UserPrincipalName
    TeamsMessagingPolicy              = $user.TeamsMessagingPolicy
    TeamsMeetingPolicy                = $user.TeamsMeetingPolicy
    TeamsCallingPolicy                = $user.TeamsCallingPolicy
    TeamsAppSetupPolicy               = $user.TeamsAppSetupPolicy
    TeamsAppPermissionPolicy          = $user.TeamsAppPermissionPolicy
    TeamsMeetingBroadcastPolicy       = $user.TeamsMeetingBroadcastPolicy
    TeamsEmergencyCallingPolicy       = $user.TeamsEmergencyCallingPolicy
    TeamsEmergencyCallRoutingPolicy   = $user.TeamsEmergencyCallRoutingPolicy
    CallingLineIdentityPolicy         = $user.CallingLineIdentityPolicy
    TeamsUpgradePolicy                = $user.TeamsUpgradePolicy
}
```
# Telephony
## List all Users enabled for Teams Phone
``` powershell
Get-CsPhoneNumberAssignment | Where-Object {$_.PstnAssignmentStatus -eq "Assigned"}
```
## Assign a phone number to user
``` powershell
Set-CsPhoneNumberAssignment -Identity "user@domain.com" -PhoneNumber "+441234567890" -PhoneNumberType DirectRouting
```
### For calling plan users
``` powershell
Set-CsPhoneNumberAssignment -Identity "user@domain.com" -PhoneNumber "+441234567890" -PhoneNumberType CallingPlan
```
## Remove a phone number from a user
``` powershell
Remove-CsPhoneNumberAssignment -Identity "user@domain.com" -PhoneNumber "+441234567890" -PhoneNumberType DirectRouting
```
## Assign an Emergency location
``` powershell
Set-CsPhoneNumberAssignment -Identity "user@domain.com" -PhoneNumber "+441234567890" -PhoneNumberType DirectRouting -EmergencyLocationId "your-location-id"
```
## List all Emergency locations
``` powershell
Get-CsOnlineLisLocation
```
## View unassigned phone numbers
``` powershell
Get-CsPhoneNumber -TelephoneNumberStatus Unassigned
```
## List all Voice Routing Policies
``` powershell
Get-CsOnlineVoiceRoutingPolicy
```
## Assign a Voice Routing Policy to a user
``` powershell
Grant-CsOnlineVoiceRoutingPolicy -Identity "user@domain.com" -PolicyName "UK-DirectRouting"
```
## List normalization rules and dial plans
``` powershell
Get-CsTenantDialPlan
Get-CsVoiceNormalizationRule
```
## Assign a Calling Policy to a user
``` powershell
Grant-CsTeamsCallingPolicy -Identity "user@domain.com" -PolicyName "InternationalCalling"
```
## Export Teams Calling Configuration for all users
``` powershell
<#
.SYNOPSIS
Exports Teams Phone System calling configuration for all users with a LineURI assigned.

.DESCRIPTION
Gathers and exports user-level Teams calling settings such as phone number, LineURI, calling policies, dial plans, and emergency location.

.OUTPUT
Exports to TeamsPhoneUsers.csv in the current directory.

.EXAMPLE
.\Export-TeamsCallingConfig.ps1
#>

# Optional: Connect if not already connected
# Connect-MicrosoftTeams

$users = Get-CsOnlineUser | Where-Object { $_.LineURI -like "tel:*" }

$results = foreach ($user in $users) {
    [PSCustomObject]@{
        DisplayName              = $user.DisplayName
        UserPrincipalName        = $user.UserPrincipalName
        LineURI                  = $user.LineURI
        OnPremLineURI            = $user.OnPremLineURI
        UsageLocation            = $user.UsageLocation
        TeamsCallingPolicy       = $user.TeamsCallingPolicy
        OnlineVoiceRoutingPolicy = $user.OnlineVoiceRoutingPolicy
        TenantDialPlan           = $user.TenantDialPlan
        EmergencyLocation        = $user.E911Location
        CallingLineIdentity      = $user.CallingLineIdentity
        TeamsUpgradePolicy       = $user.TeamsUpgradePolicy
    }
}

$results | Export-Csv -Path ".\TeamsPhoneUsers.csv" -NoTypeInformation -Encoding UTF8
Write-Host "Export complete: TeamsPhoneUsers.csv" -ForegroundColor Green
```
