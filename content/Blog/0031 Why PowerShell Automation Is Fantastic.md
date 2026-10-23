---
title: "Why PowerShell Automation is Amazing - Part One"
created: 2025-12-22
modified: 2025-12-22
tags: ["POWERSHELL"]
draft: true
---

## Introduction

I recently decided to build a custom PowerShell module which would allow me to quickly obtain all the information from the active directory environment without having to run a script over and over again.

## Requirements

The only requirements for the custom PowerShell module is to install `ActiveDirectory` module which will allow us to search through our active directory environment.

## Get-DomainAdmins

The `Get-DomainAdmins` will list all the users that are in `Domain Admins` group which will allow us to see all the domain administrators. It will show the attributes `Display Name`, `User Principal Name`, `Password Last Set`, and `Account Enabled` of the domain administrators.

```powershell
function Get-DomainAdministators {
    [PSCustomObject[]]$domainAdmins = @()

    Get-ADGroupMember "Domain Admins"  | Select-Object Name, SamAccountName | ForEach-Object {
        $domainAdmins = $_.SamAccountName
        Get-ADUser -Filter "SamAccountName -eq '$($_.SamAccountName)'" -Properties DisplayName, userPrincipalName, SamAccountName, PwdLastSet, Enabled | ForEach-Object {
            $usersInfo += [PSCustomObject]@{
                "Display Name" = $_.DisplayName
                "User Principal Name" = $_.userPrincipalName
                "Password Last Set" = [DateTime]::FromFileTime($_.PwdLastSet)
                "Account Enabled" = $_.Enabled
            }
        }
    }
 
    $domainAdmins | Format-Table
}
```

## Get-GroupMembers

The `Get-GroupMembers` will allow us to obtain a list of all the members of a group it will show informations such as `Display Name`, `User Principal Name`, `Password Last Set`, and `Account Enabled`. We can also search for accounts that are in `Active` and `Disabled` status.

```powershell
# Getting all group members
function Get-GroupMembers {
    param (
        [Parameter(Mandatory = $true)] $GroupName,
        [Parameter] [Boolean] [AllowNull()] $AccountStatus
    )

    [PSCustomObject[]]$members = @()

    if ($AccountStatus -eq $true -or $AccountStatus -eq $false) {
        Get-ADGroupMember $GroupName | Select-Object Name, SamAccountName | ForEach-Object {
            Get-ADUser -Filter "SamAccountName -eq '$($_.SamAccountName)'" -Properties DisplayName, userPrincipalName, SamAccountName, Enabled | ForEach-Object {
                if ($_.Enabled -eq [System.Convert]::ToBoolean($AccountStatus)) { 
                    $members += [PSCustomObject]@{
                        "Display Name" = $_.DisplayName
                        "User Principal Name" = $_.userPrincipalName
                        "Account Enabled" = $_.Enabled
                    }
                }
            }
        }
    }
    else {
        Get-ADGroupMember $GroupName | Select-Object Name, SamAccountName | ForEach-Object {
            Get-ADUser -Filter "SamAccountName -eq '$($_.SamAccountName)'" -Properties DisplayName, userPrincipalName, SamAccountName, Enabled | ForEach-Object {
                $members += [PSCustomObject]@{
                    "Display Name" = $_.DisplayName
                    "User Principal Name" = $_.userPrincipalName
                    "Account Enabled" = $_.Enabled
                }
            }
        }
    }

    $members | Format-Table
}
```

## Get-TotalGroupMembers

The `Get-TotalGroupMembers` enables us to obtain a number of total members of the group. It will also tell us the amount if `Active` and `Disabled` accounts that are inside the group.

```powershell
# Getting Total Member Count
function Get-TotalGroupCount {
    param(
        [Parameter(Mandatory = $true)] [String] $GroupName
    )

    $activeMembers = 0
    $nonActiveMembers = 0
    $totalMembers= 0
    
    Get-ADGroupMember $GroupName | Select-Object Name, SamAccountName | ForEach-Object {
        Get-ADUser -Filter "SamAccountName -eq '$($_.SamAccountName)'" -Properties DisplayName, userPrincipalName, SamAccountName, Enabled | ForEach-Object {
            if ($_.Enabled -eq $True) {
                $activeMembers++
            }
            elseif ($_.Enabled -eq $False){
                $nonActiveMembers++ 
            }
            $totalMembers++
        }
    }

    Write-Host $GroupName "Active Member Count:" $activeMembers
    Write-Host $GroupName "Non Active Members Count: " $nonActiveMembers
    Write-Host $GroupName "Total Members:" $totalMembers
}
```

## Get-PNEUsers

The `Get-PNEUsers` will show a list of users with `PasswordNeverExpires` options on them. You can also use the parameter `-Location` to filter it based on their office as an example Oslo, Penang, and etc...

```powershell
function Get-PNEUsers {
    param(
        [String] $Location = "*"
    )

    [PSCustomObject[]]$users = @()

    if ($Location -ne "*")
    {
        Get-ADUser -Filter  "Office -eq '$($Location)'" -Properties DisplayName, userPrincipalName, Office, PwdLastSet, PasswordNeverExpires, Enabled | Select-Object DisplayName, userPrincipalName, Office, PwdLastSet, PasswordNeverExpires, Enabled | ForEach-Object {
            if ($_.PasswordNeverExpires -eq $true) {
                $users += [PSCustomObject]@{
                    "Display Name" = $_.DisplayName
                    "User Principal Name" = $_.userPrincipalName
                    "Password Last Set" = [DateTime]::FromFileTime($_.PwdLastSet)
                    "Office" = $_.Office
                    "Password Never Expires" = $_.PasswordNeverExpires
                    "Account Enabled" = $_.Enabled
                }
            }
        }
    }
    else
    {
        Get-ADUser -Filter  * -Properties DisplayName, userPrincipalName, Office, PwdLastSet, PasswordNeverExpires, Enabled | Select-Object DisplayName, userPrincipalName, Office, PwdLastSet, PasswordNeverExpires, Enabled | ForEach-Object {
            if ($_.PasswordNeverExpires -eq $true) {
                $users += [PSCustomObject]@{
                    "Display Name" = $_.DisplayName
                    "User Principal Name" = $_.userPrincipalName
                    "Password Last Set" = [DateTime]::FromFileTime($_.PwdLastSet)
                    "Office" = $_.Office
                    "Password Never Expires" = $_.PasswordNeverExpires
                    "Account Enabled" = $_.Enabled
                }
            }
        }
    }
    $users | Format-Table
}
```

## Future

I'm going to work on updating the PowerShell module with more functions as that will allow me to extend my understanding about PowerShell and also automate day-to-day tasks that I'm performing. Hopefully this article helped you with understanding why PowerShell is amazing.

## Full code

```powershell
# Allow function imports
$MaximumFunctionCount = 8192

# Importing ActiveDirectory Module
Import-Module ActiveDirectory

# Displays all users with PasswordNeverExpires
function Get-PNEUsers {
    param(
        [String] $Location = "*"
    )

    [PSCustomObject[]]$users = @()

    if ($Location -ne "*")
    {
        Get-ADUser -Filter  "Office -eq '$($Location)'" -Properties DisplayName, userPrincipalName, Office, PwdLastSet, PasswordNeverExpires, Enabled | Select-Object DisplayName, userPrincipalName, Office, PwdLastSet, PasswordNeverExpires, Enabled | ForEach-Object {
            if ($_.PasswordNeverExpires -eq $true) {
                $users += [PSCustomObject]@{
                    "Display Name" = $_.DisplayName
                    "User Principal Name" = $_.userPrincipalName
                    "Password Last Set" = [DateTime]::FromFileTime($_.PwdLastSet)
                    "Office" = $_.Office
                    "Password Never Expires" = $_.PasswordNeverExpires
                    "Account Enabled" = $_.Enabled
                }
            }
        }
    }
    else
    {
        Get-ADUser -Filter  * -Properties DisplayName, userPrincipalName, Office, PwdLastSet, PasswordNeverExpires, Enabled | Select-Object DisplayName, userPrincipalName, Office, PwdLastSet, PasswordNeverExpires, Enabled | ForEach-Object {
            if ($_.PasswordNeverExpires -eq $true) {
                $users += [PSCustomObject]@{
                    "Display Name" = $_.DisplayName
                    "User Principal Name" = $_.userPrincipalName
                    "Password Last Set" = [DateTime]::FromFileTime($_.PwdLastSet)
                    "Office" = $_.Office
                    "Password Never Expires" = $_.PasswordNeverExpires
                    "Account Enabled" = $_.Enabled
                }
            }
        }
    }
    $users | Format-Table
}

# Displays all the domain administrators
function Get-Domains {
    [PSCustomObject[]]$domainAdmins = @()

    Get-ADGroupMember "Domain Admins"  | Select-Object Name, SamAccountName | ForEach-Object {
        $samAccountName = $_.SamAccountName
        Get-ADUser -Filter "SamAccountName -eq '$($_.SamAccountName)'" -Properties DisplayName, userPrincipalName, SamAccountName, PwdLastSet, Enabled | ForEach-Object {
            $domainAdmins += [PSCustomObject]@{
                "Display Name" = $_.DisplayName
                "User Principal Name" = $_.userPrincipalName
                "Password Last Set" = [DateTime]::FromFileTime($_.PwdLastSet)
                "Account Enabled" = $_.Enabled
            }
        }
    }

    $domainAdmins | Format-Table
}

# Getting all group members
function Get-GroupMembers {
    param (
        [Parameter(Mandatory = $true)] $GroupName,
        [Parameter] [Boolean] [AllowNull()] $AccountStatus
    )

    [PSCustomObject[]]$members = @()

    if ($AccountStatus -eq $true -or $AccountStatus -eq $false) {
        Get-ADGroupMember $GroupName | Select-Object Name, SamAccountName | ForEach-Object {
            Get-ADUser -Filter "SamAccountName -eq '$($_.SamAccountName)'" -Properties DisplayName, userPrincipalName, SamAccountName, Enabled | ForEach-Object {
                if ($_.Enabled -eq [System.Convert]::ToBoolean($AccountStatus)) { 
                    $members += [PSCustomObject]@{
                        "Display Name" = $_.DisplayName
                        "User Principal Name" = $_.userPrincipalName
                        "Account Enabled" = $_.Enabled
                    }
                }
            }
        }
    }
    else {
        Get-ADGroupMember $GroupName | Select-Object Name, SamAccountName | ForEach-Object {
            Get-ADUser -Filter "SamAccountName -eq '$($_.SamAccountName)'" -Properties DisplayName, userPrincipalName, SamAccountName, Enabled | ForEach-Object {
                $members += [PSCustomObject]@{
                    "Display Name" = $_.DisplayName
                    "User Principal Name" = $_.userPrincipalName
                    "Account Enabled" = $_.Enabled
                }
            }
        }
    }

    $members | Format-Table
}

# Getting Total Member Count
function Get-TotalGroupCount {
    param(
        [Parameter(Mandatory = $true)] [String] $GroupName
    )

    $activeMembers = 0
    $nonActiveMembers = 0
    $totalMembers= 0
    
    Get-ADGroupMember $GroupName | Select-Object Name, SamAccountName | ForEach-Object {
        Get-ADUser -Filter "SamAccountName -eq '$($_.SamAccountName)'" -Properties DisplayName, userPrincipalName, SamAccountName, Enabled | ForEach-Object {
            if ($_.Enabled -eq $True) {
                $activeMembers++
            }
            elseif ($_.Enabled -eq $False){
                $nonActiveMembers++ 
            }
            $totalMembers++
        }
    }

    Write-Host $GroupName "Active Member Count:" $activeMembers
    Write-Host $GroupName "Non Active Members Count: " $nonActiveMembers
    Write-Host $GroupName "Total Members:" $totalMembers
}
```