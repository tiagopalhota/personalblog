---
title: 'Azure Policy Manual Trigger'
date: 2023-07-06T16:30:50+01:00
draft: true
description: 'Reminds me on how to manual kick in the Azure Policy compliance checks'
thumbnail: '/images/azurepolicy.png'
---

### Have I told you how much I hate Azure Policy?

### Take your time, I will wait for you to be done whenever, Just keep me in the dark please

Yeah, great fun today, or not! I´ve been trying to enforce a few tags in resource groups so that a company I´m currently working for can cross charge different Business Units for their resource costs.
It just happens that whenever you make a change to the Azure Policy, it needs to replicate, do internal checks and whatever, and in the meantime your perfectly fine deny policy, is not working, just bypassed.

And you wonder what the hell did you do wrong? Then you delete the policy, check if it´s being bypassed again, and now it tells you is not compliant with a policy that it doesn't exist anymore. Great! What a way to waste my day!
I might be wrong, but it would be nice to have some feedback of when the policy is being applied or not, I just don't and can´t know all the internals of this thing.

Sorry for the rant 😄

Anyway, The post of today is to remind myself on how to manual trigger a compliance check, since Microsoft runs them automatically at a specific schedule that is still not clear to me(24h? don´t really know!)

{{% notice tip "Powershell Syntax" %}}
Start-AzPolicyComplianceScan
[-ResourceGroupName <String>]
[-AsJob]
[-PassThru]
[-DefaultProfile <IAzureContextContainer>]
[-WhatIf]
[-Confirm]
[<CommonParameters>]
{{% /notice %}}

For powershell, I like the fact that I can do this asynchronously like this

```powershell
$job = Start-AzPolicyComplianceScan

# To check the status, just output $job
$job
```

{{% notice tip "azure CLI Syntax" %}}
az policy state trigger-scan --resource-group
{{% /notice %}}
