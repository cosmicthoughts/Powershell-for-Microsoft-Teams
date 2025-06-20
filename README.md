# Powershell-for-Microsoft-Teams

A collection of simple and effective PowerShell commands for managing Microsoft Teams using the official MicrosoftTeams PowerShell module.

These scripts are designed to be easy to use, adapt, and automate for day-to-day Teams administration tasks.

---

## Requirements
- Windows PowerShell 5.1+ or PowerShell Core 7+
- MicrosoftTeams module installed
## Install powershell module
```
Install-Module -Name MicrosoftTeams -Force
```
## Connect to Microsoft Teams
```
Connect-MicrosoftTeams
```
## List all Teams
```
Get-Team | Select DisplayName, GroupId, Visibility
```
## List All Users in a Specific Team
```
# Replace with your Team's GroupId
$groupId = "<your-group-id>"

Get-TeamUser -GroupId $groupId | Select User, Role
```
## List All Teams with Owners
```
$teams = Get-Team
foreach ($team in $teams) {
    Write-Host "Team: $($team.DisplayName)"
    Get-TeamUser -GroupId $team.GroupId | Where-Object {$_.Role -eq "Owner"} | Select -ExpandProperty User
    Write-Host ""
}
```
## Remove a user from all teams
```
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
```
# Replace with Team GroupId and user UPN
Add-TeamUser -GroupId "<group-id>" -User "user@domain.com"
```
# Microsoft Teams Policies
## List all messaging policies
```
Get-CsTeamsMessagingPolicy
```
## List all meeting policies
```
Get-CsTeamsMeetingPolicy
```

## List all policies assigned to a user
```
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
