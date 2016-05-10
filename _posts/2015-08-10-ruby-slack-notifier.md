---
layout: post
title:  Slack Notifications from a Rails app
tags: [professional]
---

With the upcoming [University of Minnesota Campus Codefest](http://umn.campuscodefest.org/events/8-campus-codefest-2015), I had a need for a "backroom" chat and live event feed to increase day of event engagement. I had played around with a prototype, reinventing the comment and recent activity feed wheel, when another developer had a simple suggestion... Why not just use Slack? I dove in and had something deployed a few hours later.

We already had a Slack team, so that part was easy. I first created a new [Incoming Webhook integration](https://api.slack.com/incoming-webhooks). Next, Ifound the [Slack Notifier](https://github.com/stevenosloan/slack-notifier) gem, which is a simple wrapper for the webhooks maintained by Steven Sloan.

To integrate with my app, I created a concern to mix in to the models I wanted to treat as first class citizens, worthy of a Slack notification:

{% gist pete2786/80f3eb56e6168eafd949 slack_notifiable.rb %}

I then include the concern in each model overroad the `slack_message` method in each model. For example:

{% gist pete2786/80f3eb56e6168eafd949 project_comment.rb %}

My application has a multi-tenant architecture, so I simply stuck an attribute on the tenant model to add a webhook URL, then deployed the application. In the future, I'd move the notification processing to a background task in the mixin.

Overall, I met the original need for a backroom chat and saved myself the headache of another custom solution when Slack met the requirements.
