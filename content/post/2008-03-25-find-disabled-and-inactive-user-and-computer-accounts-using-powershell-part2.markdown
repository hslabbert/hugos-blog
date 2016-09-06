---
date: 2008-03-25 05:28:20+00:00
slug: find-disabled-and-inactive-user-and-computer-accounts-using-powershell-part2
title: Find Disabled and Inactive User and Computer Accounts using Powershell - Part
  II
tags:
- .net
- active directory
- adsi
- howto
- powershell
- scripting
- server 2003
- user management
- windows
- windows server
---

Part I demonstrated how to find aged or inactive accounts, and in Part II we will look at another lingering account type: disabled accounts.

Like inactive accounts, Directory Searchers also come in handy for disabled accounts. We can also, however, read an Active Directory account's status directly from a hidden attribute on the ADSI object. Let's start with the Directory Searcher method. This entry also draws from [Bahram’s Blog](http://blogs.technet.com/bahramr/archive/2008/01/25/powershell-script-to-disable-inactive-accounts-in-active-directory.aspx). The code:

`$adobjroot = [adsi]''
$objdisabsearcher = New-Object System.DirectoryServices.DirectorySearcher($adobjroot)
$objdisabsearcher.filter = "(&(objectCategory=person)(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=2))"
$resultdisabaccn = $objdisabsearcher.findall() | sort path`

<!-- more -->

That was a lot easier! The `$adobjroot` line gives us an ADSI object for the root of our domain. The second line creates a new Directory Services searcher, and then we add our filter.

As with Part I, setting objectCategory to `person` and objectClass to `user` sets up our filter to search for user accounts; switch both of those to `computer` to search for computer accounts instead.

The userAccountControl portion is a bit of weird number, though, isn't it?! After some digging, I was able to determine that the `:1.2.840.113556.1.4.803:` is the attribute ID for the Last-Logon-Timestamp Attribute (found on an [MSDN article](http://msdn2.microsoft.com/en-us/library/ms676824.aspx) linked from [Bahram's Blog](http://blogs.technet.com/bahramr/archive/2008/01/25/powershell-script-to-disable-inactive-accounts-in-active-directory.aspx)). Specifying that value in the directory searcher filter queries for the value of that specific attribute stored in the userAccountControl property, rather than userAccountControl as a whole. Stephen Looney actually corrected this for me, as I was somewhat off the mark in my deduction. The string is not a selection filter, as I had supposed, but more akin to a bitwise OR operator. See his comment below for clarification.

The last line of the code simply collects our searcher results in System.DirectoryServices.SearchResult collection

The alternative method to determine an account's status is to check a hidden attribute on the ADSI object itself. For this you will need the Distinguished Name of a user or computer account:

`$struserdn = "CN=Some User,OU=Users,OU=Corp,DC=yourdomain,DC=com"`

Set up an ADSI object for that account:

`$adobjuser = [ADSI]"LDAP://$struserdn"`

And access the hidden method:

`$adobjuser.PsBase.InvokeGet("AccountDisabled")`

This will return $true if the account is disabled, and $false if the account is not disabled (i.e. it is enabled). It's not as glamorous as the directory searcher method, but I think both have their place.

Also, feel free to play around with the lastLogonTimeStamp and UserAccountControl attributes in the directory searcher. For instance, to find only disabled accounts that have been inactive for a certain number of days, you could use an LDAP string like:

`$objsearcher.filter = ``(&(objectCategory=person)(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=2)``(lastLogonTimeStamp<=" + $lltIntLimit + ")``)"`

Or switch it up for only enabled accounts inactive for a certain period by flipping the userAccountControl portion around be negative:

`$objsearcher.filter = ``(&(objectCategory=person)(objectClass=user)(!(userAccountControl:1.2.840.113556.1.4.803:=2)``)(lastLogonTimeStamp<=" + $lltIntLimit + ")``)"`

I hope this has been helpful. As always, comments, corrections, additions, or questions are appreciated.

Bitcoin tip address for this post: 192yT6362K6BgLpm8B2xhcCVZtrnwQkjek
