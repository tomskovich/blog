---
title: "PowerShell - OpenProvider API wrapper"
date: 2022-08-19T10:40:06+02:00
tags: ['Powershell', 'Module', 'API', 'DNS']
draft: false
---
## INTRODUCTION
`IMPORTANT: This module is still a work in progress!`

At my current job, we use [OpenProvider](https://openprovider.com) as our main domain registrar and DNS provider.

I initially wrote this module for personal convenience, but I've also integrated some of the functions in our [PowerShell Universal](https://ironmansoftware.com/powershell-universal) dashboards.

Lately, OpenProvider has had multiple DNS outages. This has forced us to migrate some of our bigger clients' DNS-zones to Azure.

To automate this process, I added the `Sync-OPDnsToAzure` function.


I'll showcase some of the included functions below. The full module can be found on my [GitHub](https://github.com/tomskovich/Public).

### Get-OPBearerToken

At its current state, Openprovider's API only supports Bearer Authentication which involves acquiring security tokens that are then passed in a request. 

OpenProvider Bearer Tokens are valid for 24 hours. This function requests a new Bearer Token and saves it in a script variable. If the token is created less than 24 hours ago, the token will be reused.

```powershell
PS > Get-OPBearerToken
```
```txt
Enter username/password for OpenProvider API authentication:
User: USERNAMEHERE
Password for user usernamehere: *******

Token                            CreationTime
-----                            ------------
XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX 19/08/2022 10:06:09
```

### Get-OPTransferToken

Retrieves domain transfer key (EPP token) for one or multiple domains.


```powershell
PS > Get-OPTransferToken -Domains 'tech-tom.com', 'liontech.nl'
```
[Get-OPTransferToken](/get-optransfertoken_example.png)

### Sync-OPDnsToAzure

Copies DNS zone(s) from OpenProvider to Azure. Also creates DNS zone in Azure if it does not exist.


When the "-Migrate" parameter is used, you'll be given the option to change the NameServers immediately after copying the zone as well.


```powershell
PS > Sync-OPDnsToAzure -domain 'tech-tom.com'
```
```md
Connected to Azure. Continuing...
Account has permissions for Resource Group DNS. Continuing...
Getting OpenProvider DNS zone for domain: tech-tom.com
Getting OpenProvider NameServer group for domain: tech-tom.com
=== Received the following domain information from OpenProvider. Please verify if this domain is correct.

ID          : XXXXXXX
Domain      : tech-tom.com
NameServers : {ns1.openprovider.nl, ns2.openprovider.be, ns3.openprovider.eu}
NSGroup     : dns-openprovider
Owner       : Tom de Leeuw

Press Enter to continue...:
```