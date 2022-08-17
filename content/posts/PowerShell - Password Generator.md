---
title: "PowerShell - Password Generator"
date: 2022-08-17T12:50:06+02:00
draft: false
---

This PowerShell function enerates one or more "random" human-readable passwords, with some options for complexity and language.

By default, this function will use a wordlist hosted on my GitHub.
If you'd like to use your own (offline) wordlist, use the "-WordlistFile" parameter.

Some examples:

```powershell
# INPUT
New-Password
# OUTPUT
PerfectStory!4985

# INPUT
New-Password -Language NL -Count 2
# OUTPUT
ZaakGevangenis#7638
VerliezenPopulair@1683
```

See below for the full function:

```powershell
<#
    .SYNOPSIS
    Generates one or more "random" human-readable passwords

    .EXAMPLE
    New-Password -Count 10

    .NOTES
    Author:   Tom de Leeuw
    Website:  https://ucsystems.nl
#>
function New-Password {
    [CmdletBinding(DefaultParameterSetName = 'Default')]
    Param(
        # Language to use
        [Parameter(ParameterSetName = 'Default')]
        [ValidateSet('NL', 'EN')]
        [string] $Language = 'EN',

        # Path to file with words to use
        [Parameter(ParameterSetName = 'Custom')]
        [ValidateScript({ Test-Path -Path $_ })]
        [Alias('Path', 'WordList', 'File', 'SourceFile')]
        [string] $WordListFile,

        # Amount of passwords to generate
        [Alias('PasswordCount', 'PassCount')]
        [int] $Count = 1,
    
        # Amount of words to use when generating password
        [int] $WordCount = 2,
    
        # Amount of numbers to use in password
        [int] $NumberCount = 4,

        # Range of numbers to use in password
        [array] $NumberRange = 1..9,

        # Amount of special characters to use
        [int] $CharCount = 1,
        
        # Special characters to use in password
        [array] $Characters = '!,@,#,$,%' -split ','
    )

    begin {
        # Parameter validation
        if ($WordListFile) {
            try {
                $WordList = Get-Content -Path $WordListFile
            }
            catch {
                throw "No wordlist found! Verify if $WordList exists."
            }
        }
        if ($Language) {
            $URL = "https://raw.githubusercontent.com/tomskovich/Public/main/src/Wordlists/$($Language).txt"
            try {
                $Request  = Invoke-WebRequest -Uri $URL
                $WordList = $Request.Content.Trim().split("`n")
            }
            catch {
                throw "Error getting wordlist from $URL"
            }
        }

        # Create arraylist for output 
        $Passwords = New-Object System.Collections.ArrayList
    }

    process {
        try {
            foreach ($i in 1..$Count) {
                # Get random word(s) from list, then title-case each word
                $RandomWords = -join (
                    Get-Random -InputObject $WordList -Count $WordCount).ForEach({
                        (Get-Culture).TextInfo.ToTitleCase($_)
                    }
                )
                # Generate random special character(s)
                $RandomCharacters = -join (Get-Random -InputObject $Characters -Count $CharCount)
                # Generate random number
                $RandomNumbers = -join (Get-Random -InputObject $NumberRange -Count $NumberCount)
                # Join everything to create final password
                $Password = -join (
                    $RandomWords,
                    $RandomCharacters,
                    $RandomNumbers
                )
                # Add password to collection but hide output
                [void] $Passwords.Add($Password)
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