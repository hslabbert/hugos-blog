---
date: 2010-06-01 18:21:39+00:00
slug: gone-google-marketing-fail
title: '"Gone Google" Marketing Fail...'
tags:
- cloud
- fail
- google
- news
- web
---

Google is touting their Google Apps offering for business with a new post on the official Google Blog ([link](http://googleblog.blogspot.com/2010/06/take-test-drive-into-cloud.html)). The post presents a link to a gonegoogle.com site, which runs you through a calculator of how much cash you can save by "Going Google". The calculator seems to be based on industry averages for things like uptime, storage costs, remote access, etc., and presents a nice little summary poster at the end. An example is given from guest poster smartfurniture.com.

A nice little surprise comes up, though, when you click on the link to view the sample poster for Smart Furniture.

<!-- more -->

Bandwidth fail:

[![](http://justanothersysadmin.files.wordpress.com/2010/06/docs-bandwidth-fail.png)](http://justanothersysadmin.files.wordpress.com/2010/06/docs-bandwidth-fail.png)

Hmm...that's not good. And if we try to click **Download**?

[![](http://justanothersysadmin.files.wordpress.com/2010/06/docs-fail-virus-scan.png)](http://justanothersysadmin.files.wordpress.com/2010/06/docs-fail-virus-scan.png)

Not so good either...but wait! There is hope! Let's try to **Download anyway**:

[![](http://justanothersysadmin.files.wordpress.com/2010/06/docs-fail-forbidden.png)](http://justanothersysadmin.files.wordpress.com/2010/06/docs-fail-forbidden.png)

Oh...damn...

I've had a few different people check out the links from different networks and using different Google accounts, and all of them had the same result. With that being said, maybe we all downloaded too much docs content recently (?)

Now, don't get me wrong: I use a lot of Google products and services (Gmail; Docs; Android (HTC Magic running CyanogenMod 5.0.7DS); Chrome, as can be seen in the screenshots; etc.) and love them. This was just too funny to pass up! So, are you ready to Go Google? :)

UPDATE1: Google has changed the poster link to point to a dummy document ([link](https://docs.google.com/fileview?id=0B2Rsgoiin0I9NjAxYzM3YzAtMjExOC00ZDg1LTkxYWMtYmI1M2RmMzU5ZTYy&hl=en)) for _Acme Inc_. Nice one!

Just in case you were curious, the previous sample poster for Smart Furniture is still available ([link](http://docs.google.com/fileview?id=0B2Rsgoiin0I9NmFmNTIwOWEtNjdhMy00YWI2LWJmNmUtNDNlOGRkODcyNDBl&hl=en)). My guess is that, as per the initial warning page, posting the PDF for Smart Furniture resulting in them tripping over a bandwidth limit for non-Google format docs for the Smart Furniture Google Apps account. Google probably reset the counter for Smart Furniture, since the original sample doc from them is now accessible, and then generated the dummy Acme Inc. sample page, hosting it within a Google Docs account that is not under the same bandwidth restrictions (internal account?).

So, I guess this has become a "nothing to see here, move along!" post at this point, but the screenshots still tell the tale of a nice "oops!" on this campaign. If you share out a Google doc publicly, apparently you won't want it to become too popular lest you hit your cap.

UPDATE2: The links may have been updated to the Acme Inc. dummy company, but the images and text in the post still reference Smart Furniture. I'm just nitpicking at this point, though, since the numbers they plugged in for Acme Inc. match those from Smart Furniture; I guess I just like being a little pain in the ass ;)

UPDATE3: K, so they have it all set now. The docs in the sample paragraph just below the video link to the Acme Inc. docs, but Smart Furniture image does now link to an actual Smart Furniture poster. Interestingly, the initial link for the Smart Furniture poster was HTTP, whereas the new link is to same item but over HTTPS, which may indicate differences in bandwidth restrictions between SSL'd and non-SSL traffic for Google Docs, or it could just be fluke. Anyhow: Well done to Google for this wrapped up pretty quickly.

Bitcoin tip address for this post: 12xP41mYLfNkwbGZPiMNJ2CSmULcNqeTxb
