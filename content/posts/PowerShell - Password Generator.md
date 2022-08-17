---
title: "PowerShell - Password Generator"
date: 2022-08-17T12:50:06+02:00
draft: false
---

This PowerShell function generates one or more "random" human-readable passwords.


```powershell
<#
    .SYNOPSIS
    Generates one or more "random" human-readable passwords

    .EXAMPLE
    New-Password -PassCount 10

    .NOTES
    Author:   Tom de Leeuw
    Website:  https://ucsystems.nl
#>
function New-Password {
    [CmdletBinding()]
    Param(
        # Path to file with words to use
        [ValidateScript({ Test-Path -Path $_ })]
        [Alias('Path', 'WordList', 'File', 'SourceFile')]
        [string] $WordListFile,
    
        # Amount of passwords to generate
        [int] $PassCount = 1,
    
        # Amount of words to use when generating password
        [int] $WordCount = 2,
    
        # Amount of numbers to use in password
        [int] $NumberCount = 4,

        # Amount of special characters to use
        [int] $CharCount = 1,
    
        # Range of numbers to use in password
        [array] $NumberRange = 1..9,
    
        # Special characters to use in password
        [array] $SpecialChars = '!,@,#,$,%' -split ','
    )

    begin {
        if ($PSBoundParameters.ContainsKey('WordListFile')) {
            try {
                $WordList = Get-Content -Path $WordListFile
            }
            catch {
                throw "No wordlist found! Verify if $WordList exists."
            }
        }
        else {
            try {
                $WordListURL = Invoke-WebRequest -Uri 'https://raw.githubusercontent.com/tomskovich/Public/main/src/Wordlists/WordList_Dutch.txt'
                $WordList = $WordListURL.Content.Trim().split("`n")
            }
            catch {
                throw "No wordlist found! Verify if $WordList exists."
            }
        }
        
        $Passwords = New-Object System.Collections.ArrayList
    }

    process {
        try {
            foreach ($i in 1..$PassCount) {
                # Get random word(s) from list, then title-case each word
                $RandomWords = -join (
                    Get-Random -InputObject $WordList -Count $WordCount).ForEach({
                        (Get-Culture).TextInfo.ToTitleCase($_)
                    }
                )
                # Generate random special character(s)
                $RandomChars = -join (Get-Random -InputObject $SpecialChars -Count $CharCount)
                # Generate random number
                $RandomNums = -join (Get-Random -InputObject $NumberRange -Count $NumberCount)
                # Join everything to create final password
                $Password = -join (
                    $RandomWords,
                    $RandomChars,
                    $RandomNums
                )
                [void]$Passwords.Add($Password)
            }
        }
        catch {
            Write-Error $_
        }
    }

    end {
        return $Passwords
    }
}
```