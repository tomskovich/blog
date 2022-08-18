---
title: "PowerShell - Password Generator"
date: 2022-08-17T12:50:06+02:00
tags: ['Powershell', 'Function']
draft: false
---

This PowerShell function generates one or more "random" human-readable passwords, with some options for complexity and language.

By default, this function will use a wordlist hosted on my [GitHub](https://github.com/tomskovich/).
If you'd like to use your own (offline) wordlist, use the "-WordlistFile <path>" parameter, or change the URL in the function.

Some examples:

```powershell
New-Password -Count 2 -NumberCount 5 -CharacterCount 3
```
```txt
# OUTPUT
BridgeMarry#@$32968
SilverSkill@#%83791
```
```powershell
New-Password -WordListFile 'C:\Temp\MyOwnCustomWordlist.txt'
```
```txt
# OUTPUT
CustomWords#7638
```

See below for the full function, or directly on my [GitHub](https://github.com/tomskovich/Public/blob/main/PowerShell/Functions/New-Password.ps1).

```powershell
<#
    .SYNOPSIS
    Generates one or more "random" human-readable passwords. Default language is English.

    .LINK
    https://tech-tom.com/posts/powershell-password-generator/

    .EXAMPLE
    New-Password -Count 10

    .NOTES
    Author:   Tom de Leeuw
    Website:  https://tech-tom.com / https://ucsystems.nl
#>
function New-Password {
    [CmdletBinding(DefaultParameterSetName = 'Default')]
    Param(
        # Language to use
        [Parameter(ParameterSetName = 'Default')]
        [ValidateSet('NL', 'EN')]
        [ValidateNotNullOrEmpty()]
        [string] $Language = 'EN',

        # URL to get wordlist from
        [Parameter(ParameterSetName = 'Default')]
        [ValidateNotNullOrEmpty()]
        [string] $URL = "https://raw.githubusercontent.com/tomskovich/Public/main/src/Wordlists/$($Language).txt",

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

        # Amount of special characters to use
        [Alias('CharCount')]
        [int] $CharacterCount = 1,

        # Range of numbers to use in password
        [array] $NumberRange = 1..9,

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
    } # end of "Begin" block

    process {
        foreach ($i in 1..$Count) {
            # Get random word(s) from list, then title-case each word
            $RandomWords = -join (
                Get-Random -InputObject $WordList -Count $WordCount).ForEach({
                    (Get-Culture).TextInfo.ToTitleCase($_)
                }
            )
            # Generate random special character(s)
            $RandomCharacters = -join (Get-Random -InputObject $Characters -Count $CharacterCount)
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
        } # end Foreach
    } # end of "Process" block

    end {
        return $Passwords
    } # end of "End" block
}
```