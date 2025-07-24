---
layout: post
title:  "A New Player Has Arrived - W200"
category : Certifications
tags :  Certifications ActiveDirectory Review
---
Today I want to talk about something different than I "usually" do. The popularity of penetration testing, specifically penetration testing *certifications*, has exploded over the last couple of years.

![](assets/attachment/ca2dd14216bc6dcd1d1a8bea82164850.png){: width="550"}

> That chart is missing some newer certifications, but it should still give you a  general overview and a "feeling" for how many certs are out there: [https://pauljerimy.com/security-certification-roadmap/](https://pauljerimy.com/security-certification-roadmap/)

Active Directory (*AD*) became one of the more sought-after topics, especially in combination with red teaming. So in all that mess..how do you choose which AD certification to go for? Everyone knows the big players like OffSec, Altered Security, HackTheBox, etc. but what if I told you there is a much more affordable and qualitatively better alternative to most of them. 

Let me introduce you to **Async Security Labs**.

![](assets/attachment/948b92a681d7bdd363cf587e6f3ad4ed.png)

> I am not sponsored by Async Security or any other vendor in any way. They didn't ask me to write this post. I just wanted to write this blog post because I genuinely believe that what Async Security offers is incredibly good.

> This review is purely my subjective point of view. At the end of the day, you are responsible what you want to purchase or not.

## Who ?

Async Security Labs is a fairly new company based in Singapore. They offer Penetration testing & Red Teaming services and of course, a course called *W200 - Introduction to Active Directory Penetration Testing*.

> Website: [https://async.sg/](https://async.sg/)

One of the founding members is [@Gatari](https://www.linkedin.com/in/zavierlee-sg/), who's quite known in the cyber security community. He, along with his colleagues, designed both the course and the labs. Knowing that, and how skilled these guys are in AD exploitation, I immediately wanted to try it out. 

Even though it's called "Introduction to AD", don't let that fool you. I've got a few certifications under my belt, and I work as a penetration tester, but I *still* learned a lot of new things. 

## What does it cover ?

> TLDR: you can find the Syllabus here: [https://shop.async.sg/pages/w200-syllabus-active-directory-penetration-testing](https://shop.async.sg/pages/w200-syllabus-active-directory-penetration-testing)

The W200 course walks you through a full Active Directory attack path, from the first initial access all the way to full domain and forest compromise. One big highlight of the course was **Linux AD**, which is not often covered by other vendors. This isn't just PowerPoint theory. Every technique is demonstrated in the lab, step-by-step. 

While the course is meant to be introductory, it doesn’t shy away from going deep where it matters (e.g. Kerberos) and forces you to try out things in a lab environment (your own home lab or if you have purchased lab access, their lab which is frankly good). You're gently guided, but some sections will challenge you.

You can technically rush through the course in a few days, but that's missing the entire point of the course. W200 encourages you to explore things deeply, to *understand why something works*, and not just blindly follow steps. For example, the **Kerberos** section is much more in-depth than I expected, forcing me to open WireShark and analyze the ticket exchange process to understand it at a *protocol level*. I thought I understood Kerberos pretty well before, turns out I didn't.

Another unique aspect is that the course teaches you AD exploitation mainly *from* a Linux attacker machine, referencing here and there windows alternatives. You'll also learn how to attack *Linux AD joined* machines, which was frankly a newish topic for me.  

![](assets/attachment/df24ff01f95c62e7e2c60e4f822300cc.png)

Some classic topics like *Windows Privilege Escalation* are also included and are well written. If you've already got some prior experience, you might already know most of it.

The course wraps up with some best practices to harden your AD, which I found extremely refreshing and is as important as the offensive part itself. This is often missed in other pentesting / red team courses. 

One final thing I appreciated: the course emphasizes that this is *not* a CTF. You're taught to properly approach assessments in a professional way. 

## Pricing

As you can see, the course content is really dense and teaches you a lot of concepts that other vendors simply don't cover. The pricing for this course is *extremely* competitive.

![](assets/attachment/0f44dfe6e90c3cc730b2950bb48d83e3.png)

The prices are in SGD (Singapore Dollar), so if you convert them to USD, that's about `$78.22`! If you're based in the EU, that would be even less: `66.76€`. I know no other vendors that offers such competitive prices. Even if the price goes up to $100 / 100€.. I’d still consider it one of the **best value-for-money** AD courses available.

> HackTheBox does offer a student license for Tier 1 & 2 modules, which costs around 7€ (~$8.20 USD) every month and is also an incredible value.

An important thing to mention: **YOU HAVE LIFETIME ACCESS TO THE COURSE MATERIALS**. That includes all future updates (which are frequent!).

## Community listener

I also quickly wanted to mention something that is important to me: listening to feedback from the community. I truly believe that any company offering courses should actively listen to what their learners are saying, and make changes when necessary. 

Another W200 learner posted some feedback in the Discord server, which imo was really solid and valuable.

![](assets/attachment/0676665836a4dd3b9e5e3221c9dc8135.png)

And Async Security reacted in the best way possible: listening and taking it to heart:

![](assets/attachment/07575ed75b6160737bcd719fe1e1a89a.png)

Which translated into this just a few days later:

![](assets/attachment/69650ccf7c2099a869d02bcdd87da017.png)

Now sure, you could argue that it's easier for a smaller company with a smaller community to react this quickly. And that's true. But I genuinely hope Async Security keeps that same energy as they grow. So far, it’s a great sign.

## W200 vs. OSCP / CRTP / CRTE / CAPE

TLDR High level overview:

| Course / Training | Price   | Focus                                           | Difficulty | Lab Quality     | Realism | Best For                        |
| ----------------- | ------- | ----------------------------------------------- | ---------- | --------------- | ------- | ------------------------------- |
| **W200**          | ~€66    | AD Exploitation                                 | Medium     | Good            | High    | AD beginners/intermediates      |
| **OSCP**          | ~€1700+ | General Pentesting                              | Medium     | Decent (varies) | Low     | Entry-level pentesters          |
| **CRTP**          | ~€212   | Basic AD Exploitation & Basic Red Team concepts | Low        | Okay            | Medium  | AD beginners                    |
| **CRTE**          | ~€254   | AD Exploitation & Basic Red Team concepts       | Medium     | Very Good       | Medium  | Post-CRTP, Cross-Forest attacks |
| **CAPE**          | ~€700   | Advanced AD Exploitation                        | Very High  | N/A             | High    | Experienced penetration testers |

> I always took the *cheapest* options available, but I'm not sure about CAPE though.

### W200 vs. OSCP

The first and most obvious difference when you compare these two courses is the *price*. It's not even remotely comparable, and the clear winner here is W200. Having done the OSCP myself, I can confidently say this: **you will be nowhere near ready** for an internal penetration test after completing OSCP. It barely scratches the surface when it comes to AD, and doesn't teach you any of the core AD attacks or structures you need to understand. The W200, on the other hand, builds a solid understanding of **how AD works**, **how it breaks**, and **how you can abuse it** from the ground up. 

One thing I want to mention that the OSCP *does* offer and W200 isn't, is a (small) web exploitation part. Yes, the W200 is a fully AD course and doesn't cover any web attacks. But to be honest, it's not like the web part in the OSCP is worth anything. 

In terms of **price-to-content ratio**, it's hard to justify OffSec's side, which makes the W200 a clear winner in this match-up.

For me, the **Winner** is all the way W200.

### W200 vs. CRTP

This one's a bit more interesting to compare, since both courses focus on AD. CRTP does a solid job of introducing you to AD. It does go through a variety of AD attacks and principles for AD exploitation. Both, W200 & CRTP, offers valuable content in AD, but the main difference for me lies in how *deep* you want to go into each topic. W200 offers you more *technical depth* and encourages you to explore beyond the material. 

CRTP, on the other hand, offers an option to use *Covenant* as a C2 framework, which is nice as an intro C2 framework. W200 is purely from a **penetration testing** perspective, and does not teach you any red team concepts or tooling. The focus slightly differs as you can see. 

Another big difference is the **Linux AD** component in the W200, which is completely missing in the CRTP. Most attacks in W200 are executed from a Linux machine, which reflects most of internal pentests anyway. In contrast, CRTP is only windows based, meaning that you perform all attacks *from* a windows machine. Also, attacks on *Linux AD joined machines* are not covered at all in the CRTP.

So what should I choose ?

![](assets/attachment/0a863e4b42b1a3e9ba6270eae397a28a.png)

- If you want a *deep* dive into Active Directory attacks & exploitation: **W200**
- If you want *some* Active Directory attacks and a bit of guidance with a C2: **CRTP**

For me, the **Winner** is W200.

### W200 vs. CRTE

I won't write too much here, because what I said about the CRTP applies to CRTE aswell. CRTE gives you *some* additional insights into evasion, briefly touches on EDRs and focuses more on cross-forest attacks. Those are the main additions worth noting, but I wouldn't say you get more "red team" content compared to CRTP.

Considering the minor changes between the two (and the price), I would recommend you to directly take the CRTE and skip the CRTP. The lab in the CRTE is very well designed which makes it a very good practice ground. 

Compared to **W200**, I’d say the **lab quality and lab size** in CRTE is superior. I really enjoyed spending time there to practice my AD skills.

However, when it comes to **course content quality**, the W200 is superior. Since the lab in the CRTE is really good, I would recommend you to take the W200 course and then chain it with the CRTE. After that, you should have a really good foundation in AD.

> If Async Security steps up their game with the lab environment, then I’d say skip the CRTE entirely and go with **W200 only**. But at the time of writing this post, that’s not yet the case.

For me, the *Winners* are both: CRTE & W200.

### W200 vs. CAPE

When I say "CAPE", I refer to the *Active Directory Penetration Tester* learning path available on HackTheBox. 

CAPE is hard. Like, really hard. It’s one of the most challenging Active Directory certifications on the market today. The skill assessments are tough and the attacks you're required to perform are really advanced. It’s clearly designed for people who have already a solid background in AD (exploitation). If you want a review about the *exam*, I recommend reading the following blog posts:

- [https://gatari.dev/posts/cape-experience/](https://gatari.dev/posts/cape-experience/)
- [https://lorenzomeacci.com/htb-cape-review](https://lorenzomeacci.com/htb-cape-review)
- [https://medium.com/@0xs4m/htb-cape-review-9e333dc4740e](https://medium.com/@0xs4m/htb-cape-review-9e333dc4740e)

Comparing W200 directly to CAPE doesn’t make a lot of sense, because the focus is completely different. 

However, If you're planning to go for CAPE, W200 might be exactly what you need as a warm-up. In fact, some chapters in W200 actually go deeper into certain topics than the CAPE material (yes you read that right!). For me it feels like the W200 could be a great course to take *before* diving into the CAPE path.

Winners are both I guess ? :p

## Conclusion


There are a lot of Active Directory courses and certifications out there — some great, some outdated (`*cough*` OffSec), and some way overpriced (`*cough x2*` OffSec). But **W200** really surprised me. For the price, the structure, and the depth of the material, it's one of the best investments you can make if you're serious about AD exploitation.

Combine it with other courses like **CRTE** or **CAPE** and you’ve got strong, realistic pentesting skills.

Big respect to the *Async Security team* for putting out something this good at a price point this fair (how are you guys even making profit ?). Looking forward to seeing what they release next.

Thanks for reading! :)

## Thanks

Thanks to Tuuli for proofreading! ^^
