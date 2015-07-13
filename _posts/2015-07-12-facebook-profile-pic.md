---
layout: post
title:  Latest profile pic via Facebook Graph
tags: [persona]
---

The personal picture I have on the header for the website is being pulled from Facebook. The Graph API provides an easy means to linking to your current profile picture, so you can always have a fresh image on your personal website.  It is as simple as using the following markup: 

{% highlight html %}
<img src="http://graph.facebook.com/:your_id/picture/"></img>
{% endhighlight %}

To find your your Facebook id, use the [Facebook API Explorer](https://developers.facebook.com/tools/explorer) to hit the `/me` endpoint. You will get back something that looks like this: 

{% highlight json %}
{
  "id": "123123123",
  "name": "Walter Sobchak"
}
{% endhighlight %}

Just grab the number in the `id` field replace `:your_id` in the markup above. That's it! If you do not want the image to change dynamically, you can grab a link to your profile picture from the API Explorer by passing the parameters: `me?fields=id,name,picture`.
