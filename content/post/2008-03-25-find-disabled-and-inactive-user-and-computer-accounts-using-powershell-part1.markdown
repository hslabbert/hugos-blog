---
date: 2008-03-25 05:28:15+00:00
slug: find-disabled-and-inactive-user-and-computer-accounts-using-powershell-part1
title: Find Disabled and Inactive User and Computer Accounts using Powershell - Part
  I
categories:
- .net
- active directory
- adsi
- howto
- powershell
- scripting
- troubleshooting
- user management
- windows
- windows server
---

We'll start off with Inactive accounts first, and then work on the disabled accounts after that.

Active Directory in Server 2003 has a nice user/computer attribute called lastLogonTimeStamp that can help us keep track of inactive accounts. If you have ever tried to use that attribute, however, you might have come up with something like this...

<!-- more -->

`PoSH> $struserdn = "CN=Some User,OU=Users,OU=Corp,DC=yourdomain,DC=local"
PoSH> $adobjuser = [ADSI]"LDAP://$struserdn"
PoSH> $adobjuser
{CN=Some User,OU=Users,OU=Corp,DC=yourdomain,DC=local}`

`PoSH> $adobjuser.lastLogonTimetamp`

`PoSH>`

What the ...? Hmm...I'm sure I saw lastLogonTimestamp in the members for that object before:

`PoSH>$adobjuser | Get-Member`

`TypeName: System.DirectoryServices.DirectoryEntry
...
lastLogonTimestamp              Property   System.DirectoryServices.PropertyValueCollection lastLogonTimestamp {get;...
...
`
`PoSH>`

Right,  like I said! Then why doesn't it give me any information for that property? May it's blank? Highly unlikely, but let's check:

`PoSH> $adobjuser | Format-List *
...
lastLogonTimestamp              : {System.__ComObject}
...`

`PoSH>`

Great! What do I do with that thing? To be honest, working with ComObjects is not really my thing, but we still can get the information we want. For my purposes, I was searching through AD with a DirectorySearcher anyway, and the answer to my problems came from DirectorySearcher in the end. First, the code (credit to [Bahram's Blog](http://blogs.technet.com/bahramr/archive/2008/01/25/powershell-script-to-disable-inactive-accounts-in-active-directory.aspx)), and then we'll run through what we're actually doing.

`$currentDate = [System.DateTime]::Now
$currentDateUtc = $currentDate.ToUniversalTime()
``$lltstamplimit = $currentDateUtc.AddDays(- $NumDays)
$lltIntLimit = $lltstampLimit.ToFileTime()
$adobjroot = [adsi]''
``$objstalesearcher = New-Object System.DirectoryServices.DirectorySearcher($adobjroot)
``$objstalesearcher.filter = "(&(objectCategory=person)(objectClass=user)(lastLogonTimeStamp<=" + $lltIntLimit + "))"
``$resultstaleaccn = $objstalesearcher.findall() | sort path`

Alright, so before we get going, this snippet of code requires that we define an Int32 variable called `$numdays` before we run it. This will be used as the number of days since the account last logged on. So what exactly is going on here? The easiest way to both get a proper DateTime object and get a relative time for setting the period for inactive accounts is to get the current date and time:

`$currentDate = [System.DateTime]::Now`

As far as I can see, you could also use `$currentdate = Get-Time` , so it's your choice, really.

Active Directory stores times in UTC format, so we can use the ToUniversalTime() method available on the System.DateTime class:

`$currentDateUtc = $currentDate.ToUniversalTime()`

This is where the `$numdays` variable comes in. Let's say your organization has a policy in effect that accounts that have been inactive for 60 must be disabled. If the lastLogonTimeStamp attribute is 60 days less than the current Date/Time, then we know that account has not been logged onto for the last 60 days. So, we will want to search for accounts that have a value 60 days less than today in their lastLogonTimeStamp attribute. To make that reference point, we can do some simple math:

`$lltstamplimit = $currentDateUtc.AddDays(- $NumDays)`

This assumes that we have run `$NumDays = 60` somewhere before this last command, and effectively saves our last logon time stamp limit ($lltstamplimit) as a System.DateTime value 60 days before now.

`$lltIntLimit = $lltstampLimit.ToFileTime()` converts the `$lltstamplimit` to an Int64 value. Why? You've got me! Please let me know if you have the goods on this, because I'm still blissfully unaware of the purpose/function of Int64 values.

The rest is fairly straight-forward. We set up a directory searcher and define the filter as a user object the lastLogonTimeStamp less than the limit we defined in `$lltIntLimit`.

`$adobjroot = [adsi]''
$objstalesearcher = New-Object System.DirectoryServices.DirectorySearcher($adobjroot)
$objstalesearcher.filter = "(&(objectCategory=person)(objectClass=user)(lastLogonTimeStamp<=" + $lltIntLimit + "))"
`

In the name of tidiness and efficiency, I would recommend restricting the search root to your user containers with something like `$objstalesearcher.searchroot = "LDAP://$searchrootdn"` where $searchrootdn is a distinguished name as a string.

All that remains at that point is to collect the results:

`$resultstaleaccn = $objstalesearcher.findall() | sort path`

For my own purposes I sort the results by their LDAP path, but you obviously don't have to. Remember that your result will be saved as a System.DirectoryServices.SearchResult object, so you will probably have to use `$_.Properties.[SomeProperty]` to get the information you're looking for.  From there, you can do with the results as you please.

Oh, and if you're looking for inactive computer  accounts instead of user accounts, switch both the objectCategory and objectClass values in your DS searcher filter to _computer._

To learn how to find _disabled_ accounts rather than inactive ones, head over to Part II.
