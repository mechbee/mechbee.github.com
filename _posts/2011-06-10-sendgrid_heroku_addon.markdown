---
layout: post
title: "Why we don't use the Heroku SendGrid addon"
category: heroku
author: judy
---

**TL;DR**: When you use Heroku's SendGrid add-on, your account is a sub-user of the main Heroku account and  doesn't get manually provisioned or tiered based on goodness like solo SendGrid accounts. This means that the IP group your in is more likely to be populated with miscreants whose bad behavior decreases your own deliverability.  So if you're not a miscreant and don't want to get lumped in with other miscreants, you should just sign up for a normal SendGrid account&#8212;the [pricing/plans](http://sendgrid.com/pricing.html) are the same or better anyways.

#### Using SendGrid via the Heroku addon is brain-dead easy

Using a normal SendGrid account with a Rails app is easy&#8212;sign up for an account and modify the <a href="http://edgeguides.rubyonrails.org/action_mailer_basics.html#example-action-mailer-configuration">smtp config for action mailer</a>. You know what's easier though? Clicking a button. And that's how the <a href="http://devcenter.heroku.com/articles/sendgrid">Heroku SendGrid add-on</a> works: click a button and SendGrid starts delivering your app's emails. With no discernible difference in features or price, we picked the easiest option and activated the Heroku SendGrid add-on. Ticket complete. 

But then people started complaining that they hadn't received their confirmation email or password reset email in 6, 12, 24 hours. What was going on?

At this point, we have to pause to commend SendGrid support. We were at a Hackathon for <a href="http://www.internetweekny.com/">New York Internet Week</a> and mentioned to a SendGrid engineer that we had noticed some problems with deliverability. He said he'd inquire with his compatriots and honestly we didn't really believe him since a) we said it in a joking tone, b) they were visiting New York from Cali and who wants to remember complaints from randoms on another coast, and c) we are tiny fish in their vast ocean, paying exactly nothing. But he did and they were so responsive and solicitous, we were astonished they could get any work done in between their many detailed, specific emails.

#### Sharing with sketchballs makes you look sketchy

So what was going on with our emails? We signed in to Sendgrid with the account credentials Heroku had created for us (use *heroku config* to get your username and password) and under the email activity tab, found that a number of emails had been "deferred" with these errors:

    reason: 452 4.2.2 The email account that you tried to reach is over quota. Please direct the recipient to http://mail.google.com/support/bin/answer.py?answer=6558 z5si15616573yhh.117 (throttled)

    reason: 450 4.2.1 The user you are trying to contact is receiving mail at a rate that prevents additional messages from being delivered. Please resend your message at a later time. If the user is able to receive mail at that time, your message will be delivered. For more information, please visit http://mail.google.com/support/bin/answer.py?answer=6592 w4si8027019anc.83 (throttled)

    reason: 421 4.7.0 [TS01] Messages from 67.228.50.54 temporarily deferred due to user complaints - 4.16.55.1; see http://postmaster.yahoo.com/421-ts01.html (throttled)

    reason: 421 4.7.0 [TS01] Messages from 67.228.50.55 temporarily deferred due to user complaints - 4.16.55.1; see http://postmaster.yahoo.com/421-ts01.html (throttled)</pre>

From the help links in the first two errors, it looks like the problem is with the recipient. But based on the latter two errors, it looks like the problem is with emails coming from the ip addresses. Everyone using the Heroku SendGrid addon gets added as a subuser to the master Heroku account and shares an IP group. This means that if other subusers are up to no good, efforts to curb their sketchy email will affect our legitimate email as well. The obvious solution is to switch to one of the SendGrid plans that include a dedicated IP address but we preferred the price of our current plan (free).

#### If you must share, pick a good neighborhood

What we discovered: all normal SendGrid accounts get manually provisioned and those without dedicated IPs get sorted into tiers based on goodness. Accounts that get added through Heroku, since they are subusers of the Heroku account, don't go through either of these filters.

From our favorite SendGrid engineer: 

> ...Our Bronze and Free packages are generally a bit better than Heroku. One reason for this is; Free or Bronze accounts are tiered IP groups based on reputation, so if you guys are not bouncing a lot of emails and don't have many complaints you will filter into the highest tier of IPs which will have better delivery rates than the lower tiers. I believe we have four tiers total, so that gives some better granularity to pair better customers with better customers rather than lumping you all together.
>
> This tiering doesn't happen on Heroku accounts, so everyone is all together, good, bad, or ugly...
> 
> ...the fact that there is an increased complaint rate on the Heroku IP group means someone is not behaving themselves and is probably sending out either something resembling spam, or to a list that they didn't _really_ opt in. Either way, this bad for all the rest of the upstanding citizens on the Heroku IP Group. So, we will need to look into this and see if we can find an eliminate the guilty party. Heroku accounts unfortunately are a bit more prone to this, as the accounts are subject to Heroku on-boarding procedures, and thus get to skip the SendGrid manual provisioning process... 

Heroku is great for many reasons, one of them being the addons ecosystem. Just don't use the Heroku SendGrid add-on to add SendGrid to your Rails app. We've discovered that it's well worth the extra 10 minutes required to sign up for your own account; we've since switched to a normal SendGrid account and have noticed significantly fewer problems.