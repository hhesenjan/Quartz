---
title: "Finding Users With Password Never Expires In Active Directory"
created: 2025-11-27
modified: 2025-11-27
tags: ["AD", "POWERSHELL"]
draft: false
---

## Introduction

Many organizations nowadays has password policy to change the password frequently within a year period. However, there might be specific users who has option `PasswordNeverExpires` enabled which excludes them from the password policy. In this article I'll go through finding these users and disabling the feature with minimal efforts.

## Export Script

The `ActiveDirectory` module from Microsoft enables us to enumerate through all our users in Active Directory environment using PowerShell. I made the following script to find all the users who has `PasswordNeverExpires` enabled on them and export them into a CSV file.

```powershell hlt="<PATH>"
# Allowing high import of functions
$MaximumFunctionCount = 8192

# Relative Paths
$bergenCSV = ".\Bergen_NoPasswordExpiry.csv"
$osloCSV =  ".\Oslo_NoPasswordExpiry.csv"
$offshoreCSV =  ".\Offshore_NoPasswordExpiry.csv"
$othersEnabledAccounts =  ".\Others_EnabledAccounts_NoPasswordExpiry.csv"
$othersDisbaledAccounts =  ".\Others_DisbaledAccounts_NoPasswordExpiry.csv"

# Importing ActiveDirectory library
Import-Module ActiveDirectory

# Getting all users
Get-ADUser -Filter * -Properties DisplayName, userPrincipalName, Office, PwdLastSet, PasswordNeverExpires, Enabled | Select-Object DisplayName, userPrincipalName, Office, PwdLastSet, PasswordNeverExpires, Enabled | ForEach-Object {
    # Creating variables with user data
    $displayName = $_.DisplayName
    $userPrincipalName = $_.userPrincipalName
    $officeLocation = $_.Office
    $pwdLastSet = [DateTime]::FromFileTime($_.PwdLastSet)
    $passwordNeverExpires = $_.PasswordNeverExpires
    $accountEnabled = $_.Enabled

    # Creating a PowerShell Object with all attributes enumerated through Get-ADUser function
    $userData = [PSCustomObject]@{
        displayName = $displayName
        userPrincipalName = $userPrincipalName
        PasswordLastSet = $pwdLastSet
        officeLocation = $officeLocation
        accountEnabled = $accountEnabled
        passwordNeverExpires = $passwordNeverExpires
    }

    # If Office equals Oslo then append user data to Oslo_NoPasswordExpiry.csv file
    if ($officeLocation -eq "Oslo" -and $passwordNeverExpires -eq $true) {            
        $userData | Export-Csv -Path $osloCSV -Append -NoTypeInformation -Encoding UTF8
    }
    # If Office equals Bergen then append user data to Bergen_NoPasswordExpiry.csv file
    elseif($officeLocation -eq "Bergen" -and $passwordNeverExpires -eq $true) {
        $userData | Export-Csv -Path $bergenCSV -Append -NoTypeInformation -Encoding UTF8
    }
    # If Office equals Offshore then append user data to Offshore_NoPasswordExpiry.csv file
    elseif($officeLocation -eq "Offshore" -and $passwordNeverExpires -eq $true) {            
        $userData | Export-Csv -Path $offshoreCSV -Append -NoTypeInformation -Encoding UTF8
    }
    # Any other users...
    else {
        # If their account is enabled append user data to Others_EnabledAccounts_NoPasswordExpiry.csv file
        if ($accountEnabled -eq $true -and $passwordNeverExpires -eq $true) {
            $userData | Export-Csv -Path $othersEnabledAccounts -Append -NoTypeInformation -Encoding UTF8
        }
        # Otherwise append the user data to Others_DisbalededAccounts_NoPasswordExpiry.csv file
        else {
            $userData | Export-Csv -Path $othersDisbaledAccounts -Append -NoTypeInformation -Encoding UTF8
        }
    }
}
```

The PowerShell script will enumerate through all the users inside the Active Directory and to find the users with `PasswordNeverExpires` configured on them. It will then export these users into a CSV file with their display name, user principal name, and other informations.

## Disable Script

We can also automate the process of disabling `PasswordNeverExpires` on all the users inside the Active Directory using the following PowerShell script. It's important to note that you will need to be a domain administrator to use the following PowerShell script.

```powershell
# Allowing high import of functions
$MaximumFunctionCount = 8192

# Importing ActiveDirectory library
Import-Module ActiveDirectory

# Getting all users
Get-ADUser -Filter * -Properties DisplayName, userPrincipalName, Office, PwdLastSet, PasswordNeverExpires, Enabled | Select-Object DisplayName, userPrincipalName, Office, PwdLastSet, PasswordNeverExpires, Enabled | ForEach-Object {
    # Creating variables with user data
    $displayName = $_.DisplayName
    $userPrincipalName = $_.userPrincipalName
    $officeLocation = $_.Office
    $pwdLastSet = [DateTime]::FromFileTime($_.PwdLastSet)
    $passwordNeverExpires = $_.PasswordNeverExpires
    $accountEnabled = $_.Enabled

    # Creating a PowerShell Object with all attributes enumerated through Get-ADUser function
    $userData = [PSCustomObject]@{
        displayName = $displayName
        userPrincipalName = $userPrincipalName
        PasswordLastSet = $pwdLastSet
        officeLocation = $officeLocation
        accountEnabled = $accountEnabled
        passwordNeverExpires = $passwordNeverExpires
    }

    # If Office equals Oslo then append user data to Oslo_NoPasswordExpiry.csv file
    if ($passwordNeverExpires -eq $true) {            
        Set-ADUser -Identity $userPrincipalName -PasswordNeverExpires $false
    }
}
```

The PowerShell script will automatically enumerate through all the users and disable the `PasswordNeverExpires` on all the accounts that has `PasswordNeverExpires` on them.

## Conclusion

We should avoid using `PasswordNeverExpires` on user accounts because the user password might not be changed for years. The password could aslo be leaked in a data brach were threat actors can use it to lateral move through our environment. If we do use `PasswordNeverExpires` option we should ensure that the password is 15 character long with special characters on them.