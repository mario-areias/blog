+++
title = "Attack Trees, Agile and Threat Modeling"
date = 2019-02-22T20:47:42+10:00
draft = false
toc = false
categories = ["building"]
tags = ["appsec","threat-modeling","attack-trees"]
+++

I like threat models. I think when done right, they can really bring value to development teams. The problem is: it can go wrong very easily. There are two reasons why Threat Modeling is so hard. 

First reason: it is really hard to balance security X delivery. Too much security and nothing gets done. Too much delivery and we are shipping very insecure products. Get that balance correct is an eternal journey and the foundation of any security program.

Second reason: we, as industry, haven't figure out a good way to do threat modeling yet. We do have many different ways to do it, but we have very few experts who know them very well. Even then, they don't provide good and solid advice. Because there is none. It is still a very immature part of InfoSec and there are still lots of discovery on that field.

I have no ambition to solve the problem of Threat Modeling for our industry, but I can share what I have been using in the last year or so. It is been working very well for us, so hopefully it might be useful for some people too.

# Principles

Some time last year, we have decided to revamp the way we do threat model. We came up with a set of principles that really help drive us in a better outcome.

## Focus on the developers

Developers are the core of any development team. They build, fix and mitigate risks as they go. Security teams do not go very far without cooperation from developers. It is imperative the threat model solution we create has a strong focus on them. It means threat models should adapt to their flow and the reports/documents should be easily consumed by them.

## Security Teams are involved, but are not responsible

In today's world we hear a lot of "you build it, you run it". More recently people are adding "you secure it". Security is a responsibility of development teams. Security people are experts and advisors. They educate, consult and help identify/mitigate risks. Threat models should reflect that. Security people are involved, of course, but ultimately they are consultants. 

## Find balance of risk x time

Development teams have multiple, competing priorities at all times. Some of the priorities include security, of course. Finding the right balance of risk mitigation and disruption of developers's time is paramount to the success of Threat Modeling. Critical services are expected to have a more comprehensive and updated Threat Modeling. Low risk services do not need the same level of time investment.

# What we are NOT doing

Before I dive in what we are doing, I want to discuss what we are NOT doing.

## STRIDE

STRIDE is one of the most popular ways to do threat modeling. It is recommended by specialists and amateurs alike. Yet, we have chose NOT to do it. The reason being, in my opinion, STRIDE is focused to be driven and consumed by security people (which violates our first principle). It uses terms like Repudiation, Spoofing, Tampering. These are not terms all developers are familiar with. Also, at the end of the day, is mostly a checklist of potential attacks against a system. It is not a fun or challenge exercise.

## Automated Threat Modeling

I watched a few talks about how to automate threat modeling. I really put some effort into that, to understand how that would work at scale. The conclusion I have is: it won't. For two reasons mostly: 

1) There is no easy to automate threats, depending on the complexity a threat can require multiple layers of code to get done properly. Any automation that is too complex, it is quite prone to get flaky. This at scale, it is a recipe to get big, slow tests running, providing very value for anyone. 

2) In my mind, Threat Modeling is like architecture. One does not simply automate architecture. Architecture requires expertise, domain knowledge and a fair amount of thinking to be reasonably good. Threat modeling is the same, it only shines when the right people are involved, with the right amount of effort in place. One can't just simply automate thinking and a good conversation. Not yet anyway.

# What we are doing

So what are we doing then? We are using attack trees. There are a few things I like about attack trees.

1) It throws away the whole security jargon. We can adapt the vocabulary depending on the skill level of the attendees. If they know what privilege escalation is that is all good. We use that. If they don't and they are more familiar with "get admin access" we use that instead. The security people in the room know the concepts and the jargon, of course. However, a common vocabulary should be used when discussing with people with different levels of security expertise.

2) STRIDE is very oriented to digital threats. Attack trees are a lot more generic and is very easy to do an analogy with something more familiar to developers. That will be useful later on.

3) Attack trees are a great framework to make developers solve a problem: attack their own application. Developers ARE problem solvers by definition. Attack trees help them to go into a mindset they are already quite familiar with. Solve a problem. It turns out this problem is attack their own application. This is subtle but quite powerful and the main reason why I chose attack trees as opposed to STRIDE. Attack trees mindset is to solve a problem, STRIDE is to go through a checklist. While I believe checklists are quite important for many scenarios I believe it is the wrong mind set here. Checklists are useful for when people should not think, just follow procedures (like before a surgery or when checking airplane controls). Threat modeling is about thinking.

In summary, attack trees make developers think about security in their own terms. I believe it is a lot more powerful than go through a checklist of terms they most likely are not familiar with. The security team role in this process is to ask the hard questions and make sure all the basic controls are in place. As well as challenge developers go above and beyond, identifying different risks and bring general security expertise to the table.

## How to run a session

### Get the right people involved

This is step 0. Without the right people in the room, there is no chance to get a positive outcome. This most likely involves getting the whole development team in the room, the security people more involved with that team and whatever experts are necessary to be there. For example, if a product is going to the cloud and the development team does not have this expertise, bring in somebody who does it. I can't emphasize this enough. The session is only as good as the people in the room. Having said that, limit the room to about 10 people in total. More people than that will make the facilitator's life quite hard. 

If the right people are not involved or in the room, it is better to cancel the session altogether and do it another time.

### Get familiar with the schedule

We run 1h30 sessions. There are two type of sessions. The initial sessions and the follow up sessions. Let's focus more on the initial session, shall we?

#### Product introduction

This is a 5 minutes introduction to talk about the product being threat modeled. Useful for people not familiar with what the business drive is for that product.

#### Attack Trees introduction

This is a 5 minutes introduction to attack trees. People can learn in different ways. Some people learn by visualising, other by hearing and others by doing. We do all 3 in this mini session. This part consists in explain what an attack tree is (by both speaking and drawing in a board) following to a quick example in how to do it. I tested many different examples, the one I have choose as my default one is a physical banking branch. The goal being how to get the cash. It is a fun example, who puts people in the right mindset. Promise is only for science ~~and not actually building a database of ideas in how to rob a bank~~. This will work as an ice breaker as well as to explain how attack trees work.

#### Describe architecture of the product

Get somebody familiar with the architecture to explain what they intend to build. Go deep in details about the feature being developed. Be careful with scope here. Focus on what the team is building rather than the whole architecture. If a team is building something in AWS, you don't want to dive in how AWS set up certs in CloudFront. As long as the certs are properly setup, there is no much else to discuss. Focus on the details of what the group involved have autonomy to fix. If there are questions about how other teams interact with the architecture, make a note of that and move on. This usually takes 15-20 minutes.

#### Choose a goal

Given the current architecture, make the development team choose a goal an attacker would choose. This is the first attack tree, so don't need to worry too much about it. As long as the goal is relevant, any goal works (don't forget there are follow up sessions, yeah?). Let the team brainstorm for a bit, but choose one quickly. 5 minutes should be enough for this.

#### Build an attack tree

Hopefully with the example previously explained, the team understand how attack trees work. Now it is time to build the tree. Again, be careful with scope. Make notes of questions for different teams in the organisation, but focus on what that team is doing. Remember, focus on the developers! It should about what they are building not what other people are building. This should take around 30-40 minutes and it is the main part of the meeting. If this part goes well, the meeting was successful!

#### Action items and follow ups

Now wrap up the discussion to capture points of concern, further investigation and identified risks.

## When to run a session

When a big business feature is about to start to be implemented. This is intentionally a generic answer. Some companies call those features Epics, others just group them as stories. Regardless what they are called, threat models only make sense for not so simple features and not so complex too. A bug fix or change on the UI will hardly be of significance from a threat model perspective. Adding 2FA to your application definitely is! Get all your services on prem and migrate them to the cloud is too complex for one session! Break that up and make multiple sessions instead.

It is really hard to define a size here, it is very contextual based. However, after running one or two sessions will be easy to identify the ideal size of a feature to be threat modeled. Also, make sure you run that BEFORE any code is written but AFTER some architecture has been decided. It is a sweet spot where is easy to change architecture if any risks are identified and not too early where the architecture is likely to change a lot.


### Lessons Learned

## Tips and Tricks

As discussed already, facilitation and scope are paramount for these sessions. It is very easy for people discuss very interesting things that are not related to the product being developed. Also, encourage security people to speak up and ask hard questions. That will make developers think and maybe identify yet more risks. Developers bring the architecture expertise, security teams bring...security expertise. Both working together build very good threat models.

## Measuring benefits

Some benefits is easy to measure. If threat models are done correctly, less security issues should be shipped to production and less pen testing findings should come up in the reports. Some other benefits are not easily measured. For example, developers talking more about security, researching topics and asking for advice more often. That really helps and warms my heart every time it does. However, this is quite hard to measure. Make the organisation think more about security is really hard goal to achieve. But I really believe that very well facilitated threat model sessions are one of the ways to get there.

## It is a conversation starter

For some companies, threat modeling should be done methodically and have a very big comprehensive documents with all threats identified. For us, it is a conversation starter, although still properly documented, we have no ambition to cover all threats in a few sessions (or at all tbh). Threat modeling for us is a process. A journey. It won't be solved in a single session. Rather, it will be discussed offline, stand up, on a coffee break. A threat modeling session helps to get the conversation started, but the work definitely does not finish there.


# Counter point

Months and months after we have implemented our way to do threat modeling, I saw this document from ThoughtWorks about how they do Threat Modeling. There are lots of similarities, which is a good thing. But they use STRIDE, so it is a good document in case you want to see a different perspective.

https://thoughtworksinc.github.io/sensible-security-conversations/materials/Sensible_Agile_Threat_Modelling_Workshop_Guide.pdf
