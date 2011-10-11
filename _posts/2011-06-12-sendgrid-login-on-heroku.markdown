---
layout: post
title: How to find your SendGrid login when using the Heroku addon
categories: [heroku, judy]
---

If you're using the Heroku SendGrid addon, they'll create account credentials for you. e.g. a username and password. Sometimes you want to either do some configuration or more frequently, log in to the SendGrid site to see stats and activity and stuff. 

    heroku config
    
and you'll see a bunch of config vars. The ones you're looking for are:

    SENDGRID_USERNAME
    SENDGRID_PASSWORD
    SENDGRID_DOMAIN
