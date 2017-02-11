---
date: 2008-01-19 19:03:40+00:00
slug: modifying-group-memberships-with-powershell-part-1
title: Modifying Group Memberships with Powershell, Part I
tags:
- .net
- active directory
- adsi
- groups
- howto
- network management
- powershell
- scripting
---

I recently had to spend _hours_ figuring out how to properly modify Active Directory group memberships using Powershell. Some of the .Net methods have not yet been implemented, so I had to get a bit tricky with it. I could find the various bits of information I needed in various places, so I hope that collecting them here in one place is of some use to others.

The scenario was that I needed to disable user accounts in a Windows Server 2003 Active Directory environment running with Exchange 2007. We have a fairly customized, hosted Exchange environment, and so disabling a user is not just a simple matter and right-clicking and disabling the account in Active Directory Users and Computers (ADUC); we have a 2-page doc for the process to catch everything from removing group memberships to setting up email forwarding or restrictions, changing dial-in permissions, changing NTFS permissions on profile directories, etc.

<!-- more -->

Anyway, I had already dabbled in modifying group memberships in our user creation script (still a bit clunky, but it gets the job done) by copying group memberships from a template account. It goes something like this (please note that several of these commands require the Exchange 2007 snap-in for Powershell, and some also the [Powershell Community Extensions](http://www.codeplex.com/PowerShellCX)  snap-in):

```
$templaccn = Get-Mailbox | Where-Object { $_.name -match [template account name] }
$newuser = Get-User [new user name]
$filterid = ( Get-User $templaccn.name ).identity
$groups = Get-Group -filter { Members -eq $filterid }
$groups | Foreach-Object {
    $groupdn = $_.DistinguishedName
    $fqgroup = [ADSI]("LDAP://$groupdn")
    $membercheck = ($fqgroup.member | Where-Object { $_ -eq $newuser})
    if ( $membercheck.length -ge 1)
    {
        Write-Host "User is already a member of" $_.name "`b. No group addition made. `n"
    }
    else
    {
        $fqgroup.member.add("$newuserdn")
        $fqgroup.setinfo()
    }
}
```

So here's the run down:

The opening _Get-Mailbox_ line uses one of the Exchange 2007 snap-ins to get the mailbox object for our template account and saves it to the _$templaccn_ variable; substitute `[template account name]` with the name of your template account. I also set the _$newuser_ variable to the name of the user who's group membership we will be modifying. This is just because the script is used for user creation, so you might want to change the variable name to something like _$user_ or _$moduser_; just be sure to change the variable throughout your code!

The next two lines are used in conjunction with each other to find all groups of which the template account is a member. You could also replace these two lines with `Get-Group | Where-Object { $_.Members -match $templaccn }`, but that first collects _ALL_ the groups in your environment and then runs them through a filter of `Where-Object`. I have found the filterid way to be much quicker.

For each group, we then first save the distinguished name to the _$groupdn _variable, then use the Powershell ADSI wrapper to store its full ADSI object as _$fqgroup_. I usually prefix _fq_ to variables for full ADSI objecs to denote their type, but you can obviously use something like _adObj** **_whatever convention you like. The _member_ property for the group is a multi-valued property that contains the groups members, so using _Where-Object_ we effectively set a value to _$membercheck_ only if our user is a member of that group. As I was using this to create new users, this was probably not completely necessary, but it was good practice anyway.

The _if_ statement there just throws out that the user is already a member of the group if _$membercheck_ had any value set, otherwise it proceeds to adding the user to the group.

We then use the _add()_  method of the member property of the ADSI object for our group, supplying our user's distinguished name as an argument, and use _setInfo()_ to apply the changes to the ADSI object.

Now, the reason this was tough to do in the first place, and why I ended up later having so much trouble with removing group memberships and making other modifications, is because the _$fqgroup_ ADSI object does not display any methods! You can read all the properties you want, but for some reason the Powershell design team thought it would be a good idea to hide the methods, even though they are there. If you don't believe me, try this:

```
$domroot = [adsi]''

distinguishedName
-----------------
{DC=test,DC=local}
```

Try just typing _$domroot_. It should return something like:

```
[PS] C:\>$domroot

distinguishedName
-----------------
{DC=test,DC=local}
```

Alright, now try using Get-Member to get some info on this ADSI object: `$domroot | Get-Member`

All of those seem to be Properties, right, with no methods? Hmm... For more info on this, check out [Benp’s Basic Guide to Managing Active Directory Objects with PowerShell](http://blogs.technet.com/benp/archive/2007/03/05/benp-s-basic-guide-to-managing-active-directory-objects-with-powershell.aspx) as well as ["Invisible" methods for ADSI?](http://pathologicalscripter.wordpress.com/2006/09/28/invisible-methods-for-adsi/) from the Pathological Scripter.

Anyway, all this to show that it doesn't seem to be as straightforward as we might have hoped.

So, that seems to have covered _adding_ users to groups...now what if I want to _remove_ a user's group memberships? Part II of this post will cover that...
