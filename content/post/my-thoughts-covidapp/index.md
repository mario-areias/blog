+++
title = "My thoughts on COVIDSafe app"
date = 2020-04-28T8:20:00+10:00
description = "Another analysis of COVIDSafe app, going through a few thoughts and describing weaknesses"
draft = false
toc = false
categories = ["breaking"]
tags = ["appsec", "covid", "reverse-engineering"]
+++


Australia [has released a new app](https://www.abc.net.au/news/2020-04-26/coronavirus-tracing-app-covidsafe-australia-covid-19-data/12186068) to help fighting off Covid-19. Even before this app was released it sparked lots of discussions and now many people, myself included, are reverse engineering the app and trying to understand its shortcomings and risks.

<p align="center"> 
    <img src="/post/my-thoughts-covidapp/images/covid-safe-logo.png" alt="COVIDSafe logo"/> 
</p>

<!--more-->

It is very early stages and the app is likely to evolve, fix bugs and implement new features before the pandemic ends. COVIDSafe is not the only option, there are other alternatives that have better traction with experts and are [being adopted by other countries](https://www.reuters.com/article/us-health-coronavirus-europe-tech/germany-flips-on-smartphone-contact-tracing-backs-apple-and-google-idUSKCN22807J).

For now, that's not an option we are considering in Australia so we are digging deep in the COVIDSafe App.

## A note of caution

Be careful with any analysis regarding this app (including mine!), we are all people working in fulltime jobs, trying to do something good in this pandemic. There are many people out there trying to help and working together to make this app as safe and sound as possible. There are twitter threads, blog posts and even a discord server full of people doing reverse engineering. You can find a good summary of their [findings here](https://docs.google.com/document/d/17GuApb1fG3Bn0_DVgDQgrtnd_QO3foBl7NVb8vaWeKc/preview#).

Also, be careful with hot takes out there, being from CEOs or influencer who don't have expertise to say if the app is safe or not ([halo effect](https://en.wikipedia.org/wiki/Halo_effect) is a powerful thing). Look for advice from experts who _actually_ spent time analising the app and look for evidence.

## My thoughts

First, I want to address the fact that some people are concerned about the privacy implications of the app. I strong believe this is a valid concern, story is full of examples of what happens when we trust the Government blindly.

The narrative that some people are saying "but you give your data to Google and Facebook" is a bad one, particularly coming from security professionals. Different people have different threat models and some of them include Government. Activists, politicians and even journalists have been targeted by Governments before. Education and evidence is the best way to ease people concerns and as security professional that should be our approach, anything related to 

Having said that, I spent around 6-8 hours looking into COVIDSafe Android App (no, that's enough for a deep analysis) and I didn't see any big problems into it, at least, not for the users themselves. The app is far from be completely private or without problems, but it does take privacy seriously to an certain extent. 

That doesn't mean there are no shortcomings, they exist and I elaborate on them in the next section (which will be a bit more technical), including lack of transparency in the design and implementation of this app. 

But I appreciate the work done here, don't forget the people developing this app are also in the pandemic and have their own problems too. They did the best they could in a tight schedule and difficult situation. My intent with this post is to help highlight possible problems (and solutions) and to make COVIDSafe even better.

## Shortcomings

> This is all related to the Android App. I haven't looked at the iOS app yet

COVIDSafe is simple, but it has many moving parts, including servers in AWS, heavily bluetooth usage and upload of personal information (such as phone numbers).

I have been digging deep into the [Android App](https://github.com/ghuntley/COVIDSafe_1.0.11.apk), but there are other source codes to look at. The opentrace community (the open source implementation of the protocol that COVIDSafe uses) [has released their code](https://github.com/opentrace-community/opentrace-android). Also, there is a white paper wrote by BlueTrace (the company who created the protocol) that you can find [here](https://bluetrace.io/static/bluetrace_whitepaper-938063656596c104632def383eb33b3c.pdf).

By comparing the opentrace code with the COVIDSafe codebase I noticed they used different protocol versions. Opentrace source code (and the whitepaper) use the version `2` of the protocol, while COVIDSafe is using version `1`.

![Print screen of COVIDSafe decompiled code showing up the version as one>](/post/my-thoughts-covidapp/images/protocol_version.png)

From looking at the app, is not really trivial to figure out what is the difference in the protocols and I couldn't find any papers regarding the version `1`.

The biggest problem I managed to find is the quality of the data stored in the app (and eventually uploaded to the cloud). One of the privacy protections COVIDSafe implements is the generation of a `tempId` (a temporary ID that is generated in the server to avoid user tracking), according to the whitepaper `tempId` is the conjuction of `UserID`, `start time` and `expire time` (encrypted in the same blob) plus IV and Authentication tag.

The Android app is clueless about this format, since the server encrypts this information using a symetrical algorithm `AES-256-GCM`, the app can't verify the `tempId` authenticity and therefore its trustworthiness. From the app perspective, `tempId` is just a bunch of random characters. The fact the bluetooh handshake has no authentication - besides checking a hardcoded UUID - makes replay attack very easy to pull out.

An attacker could create an Android app that implements a similar protocol but send a bunch of garbage as `tempId` and use this app while in proximity with other people. These other people would receive messages from the attacker and store fake `tempId`s in their phones. If this data is uploaded to the AWS server they will be unusable and potentially damaging for the server side code.

Another possible attack is to capture other `tempId`s belonging to other people and replay as if these were the ids from the attacker. This attack is valid for up to 2 hours (the time configured by the Australian server to generate new ids) and means an attacker can impersonate someone and could cause the person who had their `tempId` replayed to be falsely notified they were in contact with someone infected. 

These attacks are small in scope and unlikely to cause major harm, but done in scale (either by group of people or automation) it could potentially damage the data used by the Government and therefore causing a failure to the whole experiment.

Also, the whole protocol _integrity_ is based on the encryption key not leaking. If a malicious person gets hold of it, can decrypt `tempId`s and get the real user ids. She can then forge new valid `tempIds` (they will be accepted by the server) and basically impersonates anyone who she managed to get `tempIds` from.

The app has other problems, of course, but the ones above are the ones who I think can cause big impact and the ones I would fix first.

## Possible improvements

There are ways to improve this protocol: using asymmetric encryption would enable clients to validate if the `tempId` is valid and mitigating the risk to accept _anything_ as `tempId`. To be frank, in the whitepaper it says the BlueTrace developers did play around with asymmetric encryption but they had performance issues on the devices. The approach they have tried is quite good, but they could improve by using something more simple. 

The server could control a keypair and make the `private` key confidential (as it is always the case), then the server could sign the `tempId`s using the private key and the Android app could verify using the `public` key. That would avoid the problem for the apps to accept anything as `tempId`. This approach could work on top of the current approach, so in order to generate a valid `tempId` an attacker would require the symetric key _and_ the private key from the keypair. Both keys could be managed differently and even being accessible in different services. It would improve the security in depth posture. 

Generate `tempIds` more often would reduce the risk of replay attacks, the whitepaper suggests to generate them every 15 minutes, which seems a reasonable option. COVIDSafe is generating every 2 hours and that grealy increases the risk for replay attacks, I would advise to follow the recommendation and use 15 minutes instead.

The app was clearly designed to protect the users information to be leaked and this is a really good goal to have. However, the whole point of this app is to provide authorities with good and accurate data and I personally feel the design of the app should have focused more in the integrity of the data as well.

Having the community participating in this process from day one, would be ideal too. Australia has many good professionals that can and _are_ contributing to the discussion.

## My recommendation

I do recommend install the app. There is no evidence that the Government is tracking users location, they only track who you were in contact with and only upload this information if the user requests. Now, there are ramifications regarding legislation and who can read this data, but this is outside my expertise so I won't comment on it. However, there _are_ very smart people looking at this problem, so I am not too concerned.

It makes me happy to see the amount of people who are jumping straight in and putting countless hours in reverse engineer this app. A collective and transparent effort like this is the way to go for an app that is so important like COVIDSafe and I hope the next steps for it, can bring researchers and Government working together to take this app to the next level. 




