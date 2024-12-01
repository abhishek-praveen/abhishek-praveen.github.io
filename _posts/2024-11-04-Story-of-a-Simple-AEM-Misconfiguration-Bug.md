---
title: Story of a Simple AEM Misconfiguration Bug 
date: 2024-11-04 13:00:48 +0530
categories: [Web Security]
tags: [AEM Misconfiguration,Information Disclosure,Hall of Fame]
---

This is a small blog post on how the habit of skimming through medium blogs daily led me to stumble upon a simple vulnerability that got me my first hall of fame. This finding was in fact a year back, but I wasn't able to document it during that time. So let's see what this vulnerability is all about and how I was able to exploit it.

During the initial hours of a Monday morning in February 2023, I was casually scrolling through the medium writes for the day, which medium sends you directly into your inbox. That's when the title “Adobe Experience (AEM) Information Disclosure Vulnerability”  by the Security  Researcher [Fat Selimi](https://x.com/fattselimi) caught my attention. Upon reading it fully, I was surprised on how easy the exploitation of this bug was, which motivated me to dig deeper into it. The original blog post which I read seems to unavailable right now, but a tweet by the same researcher summarizes the discovery of this vulnerability.

<!-- <img src="assets/posts/post1/aem_tweet.jpg" alt="Tweet about the vulnerability" width="400" /> -->

<blockquote class="twitter-tweet">
    <p lang="en" dir="ltr">Tweet content here...</p>
    &mdash; Intigriti (@intigriti) <a href="https://twitter.com/intigriti/status/1611704251096383489">January 7, 2023</a>
</blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<br>Now, let’s look into the technical side of the bug.

## Vulnerability

The vulnerability was **Sensitive Data Exposure due to Adobe Experience Manager Misconfiguration**. Let me further break this down into simpler terms.

**Adobe Experience Manager (AEM)** is a software solution that is a combination of Content Management System (CMS) and Digital Asset Management (DAM) system that helps companies to manage digital assets across multiple channels, apps, and sites. We are interested in the latter use-case, as that is where the vulnerability lies. The DAM system is called as **Adobe Experience Manager Assets**, and it is a cloud based system that stores and organizes all the digital files. While misconfigured, this software can be exploited to gain sensitive data. 

But how do we know that the given website is using AEM ?

## Discovery

As hinted in the twitter post, google dorking can be used to find if our target is using AEM to manage content. 

```
site:target.* inurl:'/content/dam' ext:txt
```
This lists all the webpages that uses AEM to render a txt file in the target site 

While testing for this vulnerability I didn't have a particular target site in mind, so I tweaked the Google dork a bit to see which all websites are using AEM and the dork is as follows

```
site:*.*  inurl:'/content/dam' ext:pdf 
```
This lists all the sites which uses AEM to render a PDF file. It took me no time to find websites that are using this software, but now I need to make sure that these websites have a vulnerability disclosure program before diving deeper into the same. 


[YesWeHack VDP Finder](https://github.com/yeswehack/yeswehack_vdp_finder) extension to the rescue! This is a simple browser extension that  checks if the visited site has a vulnerability disclosure program. Thus, I was able to find the associated responsible disclosure program for the website that I was testing.

## Exploitation 

This was the least complicated stage. But before moving on to that, please keep in mind that this bug can be rated at a low severity or even out-of-scope, depending upon the bug bounty program's rules. So it's always suggested to try to escalate the issue to high or medium severity before reporting this as it is. The bug which I found was in a Vulnerability Disclosure Program, so considering the time and skill required for the same, I had to stop at the initial phase of exploitation. 
 

Anyways providing you with an overview of the steps involved for better clarity.


1. Let's call our vulnerable website `www.example.com` , and it renders a PDF at `https://www.example.com/content/dam/example.pdf/`

2. Append `.children.json` which modifies the URL to 
`https://www.example.com/content/dam/example.pdf/.children.json`

3. A JSON response will be returned which contains a data field `jcr:lastModifiedBy` that has sensitives details such as Email ID / Name / Numerical IDs of the last person who modified the given file.

An example of such a JSON response is shown below

```json
 [
    {
        "id": "jcr:content",
        "uri": "/content/dam/redacted/resources/example.pdf/jcr:content",
        "jcr:primaryType": "dam:AssetContent",
        "jcr:mixinTypes": [
            "cq:ReplicationStatus"
        ],
        "cq:lastReplicationAction": "Activate",
        "cq:lastReplicatedBy": "user@company.com",
        "jcr:lastModifiedBy": "user@company.com",
        "restored": true,
        "cq:lastReplicated": "2015-09-17T11:56:18.736-05:00",
        "cq:name": "example.pdf",
        "jcr:lastModified": "2015-09-17T11:31:45.007-05:00",
        "cq:parentPath": "/content/dam/redacted/resources/messages",
        "cq:versionCreator": "user"
    }

]
```

I was able to view the internal email ID of the employee who last updated the file. This being a PII (Personally Identifiable Information)  was sufficient to show the impact of the vulnerability.

Even though being a very minimalistic bug, it has significant impact. Since the disclosed data is an email ID, bad actors can misuse this to carry out phishing and social engineering attacks targeting the specific employee or the company. Furthermore, there is a risk of damage to reputation and loss of confidentiality.

A simple fix to the vulnerability is to filter out the .json endpoints and whitelist only the ones that should be allowed.

## Conclusion

This vulnerability demonstrates that even a minor misconfiguration in softwares with good security posture can have significant impact on the users. This was a very special bug to me as it helped me get my first Hall of Fame.  Although I had to wait for months before the bug was disclosed, the end result was sweet. 

Thank you all for taking the time to read this. Please do not hesitate to [contact me](https://x.com/_p3g4sus) with any suggestions or criticisms. See you !!
