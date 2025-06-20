# Powershell-for-Microsoft-Teams

A collection of simple and effective PowerShell commands for managing Microsoft Teams using the official MicrosoftTeams PowerShell module.

These scripts are designed to be easy to use, adapt, and automate for day-to-day Teams administration tasks.

---

## Requirements
- Windows PowerShell 5.1+ or PowerShell Core 7+
- MicrosoftTeams module installed
## powershell module
```
Install-Module -Name MicrosoftTeams -Force
```
## connect to Microsoft Teams
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
