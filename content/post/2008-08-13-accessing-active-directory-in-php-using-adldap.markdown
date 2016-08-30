---
date: 2008-08-13 05:35:37+00:00
slug: accessing-active-directory-in-php-using-adldap
title: Accessing Active Directory in PHP using ADLDAP
categories:
- active directory
- howto
- network management
- php
- scripting
- server 2003
- web
- windows
- windows server
---

Lately, our company has started developing user web portals for our clients. The main goal is to provide a central reference point for common links (webmail, helpdesk, remote assistance links ... ), howto documents, and other files and resources. A secondary goal was to also allow user administrators to perform basic user management through a web interface. This would include things like disabling/creating/unlocking user accounts, resetting passwords, and modifying group memberships for access reasons. Myself and the other admin tasked with setting up this portal are most familiar with PHP, and so we went of looking for the best means of interfacing with Active Directory through PHP. <!-- more -->

Now, you can obviously use PHP's built-in LDAP support, provided PHP was built --with-ldap. If you're doing a lot of calls back and forth though, this can get pretty tedious pretty fast. It can also be quite intimidating to someone who is more of a sysadmin that a full-time developer (myself included!). So, abstracting away some of the complexity would be handy. I'm betting there are other systems out there, but for us, [adldap](http://adldap.sourceforge.net/) was the answer.

Adldap provides an easy-to-use interface for both querying and modifying Active Directory. This post is not meant to be a complete setup guide, but rather just an overview, so here's the quick summary:



	
  * Runs on Apache or IIS (although the documentation is a little thin on using IIS)

	
  * Might require some configuration in your environment to support secure LDAP queries over SSL

	
  * Incorporates into your PHP pages through a class definition file. Configure the settings in the provided adldap.php file to match your environment, include() it in your php page, initialize an instance of the class (`$adldap = new adLDAP();`), and you're ready to go.

	
  * Allows you to add custom functions by adding to the class definition file.


If some of that sounds scary, don't worry: It can sound more intimidating than it really is. If you have some reasonable PHP background, just go and check it out and get your feet wet. Like I said: Both myself and the other project contributor are not full-time developers, but we've thrown pretty useful AD integration into the portals. Heck, we've even started incorporating some AJAX on the site (with some help from [Prototype](http://www.prototypejs.org/)...but that's another post...) to work with adldap and make the thing pretty slick overall!

One thing that was getting at me a little bit was that I couldn't just throw a custom ldap query string into aldap and get the results. Part of the abstraction is that you get a set of functions for querying for specific types of information: user_info(), user_delete(), authenticate(), group_create(), etc, but no raw ldap query function. So I checked out how the thing is put together, and I bastardized some of the existing functions to suit my purposes. If you're familiar with ldap queries, add the following code into your aldap.php class definition file, and you've got yourself an easy way of performing custom ldap queries:

`function any_info($filter=NULL,$fields=NULL){
// Written by Hugo Slabbert (JustAnotherSysadmin - http://bamboo.slabnet.com/~hslabbert/blog) from other functions; defaults to root
if (!$this->_bind){ return (false); }
if ( $filter==NULL ){ $filter="(&(objectclass=domainDNS)(!(distinguishedname=" . $this->_base_dn . ")))"; }
if ($fields==NULL){ $fields=array("name","cn","displayname","dn"); }
$sr=ldap_search($this->_conn,$this->_base_dn,$filter,$fields);
$entries = ldap_get_entries($this->_conn, $sr);`

`return ($entries);
}`

If you do use the code snippet, I just ask that you keep the comments in tact. I don't mind sharing, but I'd like a little bit of the credit!

When you use the `any_info()` function above, it takes two parameters: your ldap query (`$filter`), and an array of which attributes you wish to return for the objects that match your query (`$fields`). If you leave the second parameter blank, the default attributes of name, cn (canonical name), display name, and dn (distinguished name) will be returned.

So, go ahead! Try it out! Let me know if it works for you and what doesn't. I do have two final recommendations, though:



	
  1. If you run into technical difficulties getting adldap running properly, you're probably better off going through their actual support (forums, documentation, etc.) than posting requests here. I've used the system, but I'm betting you will get way better support from the actual devs!

	
  2. Remember that the point of this tool is to enable access to your Active Directory through the a web interface. Secure your web app accordingly! Putting powerful tools on the web is great, but realize that you are doing just that: Putting POWERFUL tools on the web! You are the best judge (hopefully!) of what suits your organization.


Happy coding!

Bitcoin tip address for this post: 13344S6vTAmrM5De7DermysvN6UE3QDSzb
