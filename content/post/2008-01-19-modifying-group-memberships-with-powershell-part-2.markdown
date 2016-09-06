---
date: 2008-01-19 23:30:35+00:00
slug: modifying-group-memberships-with-powershell-part-2
title: Modifying Group Memberships with Powershell, Part II
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

I had hoped to put this all in one post, but the thing would have gone on forever! Part I covered some basics in copying group memberships to an Active Directory user from another user, such as a template account, using Powershell. Part II will delve into my misadventures in gaining more control of user group memberships, including removing users from a group either by editing the group's attributes or editing the user's attributes. I was also looking for a way to change dial-in permissions on user accounts, and that will be covered by a similar strategy.








While these examples should be less dependent on the MS Exchange 2007 snap-in for Powershell and [Powershell Community Extensions](http://www.codeplex.com/PowerShellCX), please note that I have not checked through the code samples to confirm what is purely Powershell and what requires those snap-ins.




<!-- more -->








So in Part I we did the following to add a user to a group:









	
  * Connected to the ADSI object for our group using `[adsi]"LDAP://[group's distinguished name]"` and stored as _$fqgroup_.

	
  * Added the user to the group using `$fqgroup.member.add("[user's distinguished name]")`

	
  * Applied the changes user `$fqgroup.setinfo()`







The problem for me came when I tried to remove group memberships. I don't have a huge scripting background; I mostly dabble in batches, and PHP, and have avoided VBS whenever I can. Powershell is my first real foray into extensive automation, and I try to keep things simple where I can and use provided cmdlets, snap-ins, and wrappers. So, when I wanted to remove group memberships, I checked out what the Exchange 2007 snap-in for Powershell had to offer.




Wouldn't you know it? Exchange has some membership-manipulating options available for us:




`Get-Group 'Test Group' | Get-Member | Where-Object { $_.name -match 'members' } | Format-List Name,Membertype,Definition`




returns:








`Name : get_Members
MemberType : Method
Definition : Microsoft.Exchange.Data.MultiValuedProperty`1[[Microsoft.Exchange.
Data.Directory.ADObjectId, Microsoft.Exchange.Data.Directory, Version=8.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35]] get_Members()
``
Name : set_Members
MemberType : Method
Definition : System.Void set_Members(MultiValuedProperty`1 value)
``
Name : Members
MemberType : Property
Definition : Microsoft.Exchange.Data.MultiValuedProperty`1[[Microsoft.Exchange.
Data.Directory.ADObjectId, Microsoft.Exchange.Data.Directory, Version=8.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35]] Members {get;set;}`








The only reasons I threw in the _Where-Object_ and _Format-List_ cmdlets were because I didn't want to show every single member, and Powershell will by default output _Get-Member_ to a table and cut off the _Definition_ column that I'm interested in here.








Now, notice that the property _Members** **_(the last one in our list) specifically states {get;set;} at the end of its definition. This tells us that we should be able to both retrieve the members property (get) and make changes to it (set). Conveniently, we also have two methods for those tasks: _get_Members_ for retrieving the _Members_ property and _set_Members_ for making changes to it. It's weird, though, that the definition for _get_Members_ has a whole bunch of .Net information about Exchange going on in there, while _set_Members_ simply references _System.Void_, isn't it? It's also a little weird that the _Set-Group_ cmdlet doesn't seem to have any options that reference membership (check `Get-Help Get-Group -detailed`)








Let's try it, though:








Using Active Directory Users and Computers (ADUC), create a new security group called _Powershell TestGroup_, and add some users to it. I will use the example of User1 and User2. Then try the following code:








`$group = Get-Group 'Powershell TestGroup'
$group.Members`








This should display a list of your group members showing their Rdn, Parent, Depth, DistinguishedName, DomainId, ObjectGuid, and Name properties. Good so far; let's try the get_Members method:








`$group.get_Members()`








Still good right? You get the same list. Alright, now let's save a new list of users that doesn't include User1, and then to our _set_Members_ method:








`$newgroupmembers = $group.get_Members() | Where-Object { $_.name -notmatch "User1" }`








$newmembers will now include all of the group members except for User1. You can verify this by just typing `$newgroupmembers | Sort | Format-Table Name` at the console. Alright, so we have our new list of users. Remember that we didn't make the array $newmembers up ourselves but got the list using the get_Members() method, so we're free of syntax errors here. Let's try _set_Members_ then:








`$group.set_Members($newgroupmembers)`








Hmm...that didn't go so well. All of sudden Powershell gives us a bunch of error output:








`Cannot convert argument "0", with value: "System.Object[]", for "set_Members" to type "Microsoft.Exchange.Data.MultiValuedProperty`1[Microsoft.Exchange.Data.Directory.ADObjectId]": "Cannot convert value "System.Object[]" to type "Microsoft.Exchange.Data.MultiValuedProperty`1[Microsoft.Exchange.Data.Directory.ADObjectId]". Error: "Conversion from System.Management.Automation.PSObject to Microsoft.Exchange.Data.Directory.ADObjectId has not been implemented.""t line:1 char:19
+ $group.set_members( <<<< $newgroupmembers)`








Take note of the second to last line:








<blockquote>Conversion from System.Management.Automation.PSObject to Microsoft.Exchange.Data.Directory.ADObjectId has not been implemented.</blockquote>







Now, I'm not a .Net developer and I don't claim to be _nearly _as intimately familiar with Powershell as some of the other guys out there ([The Powershell Guy](http://thepowershellguy.com/blogs/posh/) for instance), but this seems to be one of those little things that the Exchange team just didn't get around to implementing. Maybe I'm wrong, I don't know. But I do know that I needed a way around this road block.








So, off I went through the wild, wild web, looking for people much smarter than myself who might be able to get this thing up and running. I have to give credit to these folks, and you might be able to get more useful information from them, so I will include links to other useful articles/blogs below. Keeping right on though, here's what we need to do:








`
$group = Get-Group "Powershell TestGroup"
$username = "User1"
$user = get-user $username
$userdn = $user.distinguishedName
$newgroupmembers = $group.members | Where-Object { $_.name -notmatch "$username" }
$groupdn = $group.distinguishedName
$fqgroup = [adsi]"LDAP://$groupdn"
$fqgroup.Member.Remove($userdn)
$fqgroup.setInfo()
`








After all that searching and tests, it was actually pretty simple! Here's what we did:








I did cheat a bit (like I said, I like to keep things simple when I can) and used the Exchange _Get-Group_ cmdlet for our $group variable, and I also used the Exchange _Get-User_ cmdlet for our $user variable, but you could use DirectoryServices.DirectorySearcher instead as in [this post](http://janssenjones.typepad.com/janssenjonescom/2007/01/powershell_and_.html). I borrowed some of my info from that post as well, so credit is owed there. But I digress.








The _$newgroupmembers_ line is similar to the one we used earlier with our _set_Members_ example that failed. You can use either $group.members or $group.get_Members() interchangeably. All we're doing is creating an array of group members that does not contain the user that we want to remove from the group.








With _$fqgroup_ we are again just connecting to the group using the Powershell ADSI wrapper. In the end, all that needs to be done here is to use the _remove()_ method of the Member property of our _$fqgroup_. The change is then applied using _setinfo()._








Now, remember how we found that methods weren't displayed through Get-Member for ADSI objects? Try this one out for size:








`$fqgroup.member | Get-Member`











Here we see methods available! But for `$fqgroup.member | Get-Member`? Nope, nothing! Weird...




You should also theoretically be able to modify the user account directly and change the _memberOf_ attribute, but I have had some difficulty with this. I will update the post if I figure out what the problem was there.




Anyway, there you have it. I owe a tremendous amount of thanks to several other IT bloggers for being able to post this:








[Benp’s Basic Guide to Managing Active Directory Objects with PowerShell](http://blogs.technet.com/benp/archive/2007/03/05/benp-s-basic-guide-to-managing-active-directory-objects-with-powershell.aspx)
[How to get DL membership in Exchange 2007
](http://www.viveksharma.com/techlog/2006/10/22/how-to-get-dl-membership-in-exchange-2007/)[Managing group membership in Active Directory with PowerShell (Part 1)
](http://www.leadfollowmove.com/archives/powershell/managing-group-membership-in-active-directory-with-powershell-part-1)[Managing group membership in Active Directory with PowerShell (Part 2)](http://www.leadfollowmove.com/archives/powershell/managing-group-membership-in-active-directory-with-powershell-part-2)
[Powershell and ActiveDirectory - Modify-Group-Membership
](http://janssenjones.typepad.com/janssenjonescom/2007/01/powershell_and_.html)[PowerShell script to list user group membership in Active Directory
Accessing AD with PowerShell](http://richardsiddaway.spaces.live.com/blog/cns!43CFA46A74CF3E96!241.entry)




Bitcoin tip address for this post: 1DMybxZn2eUfKndJsn56Xq7BUoZSFvFfeb
