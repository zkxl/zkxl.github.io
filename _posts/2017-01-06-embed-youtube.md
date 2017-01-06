---
layout: post
title:  embed youtube
date:   2017-01-06 00:48:00 -0800
categories: HOW-TO
youtubeID: iX5V1WpxxkY
tag: youtube
---


* content
{:toc}


{% include youtubePlayer.html id=page.youtubeID %}

---

1. Under you \_includes dir, create a "youtubePlayer.html".

{% highlight bash %}
echo '<iframe width="560" height="315" src="https://www.youtube.com/embed/{{ include.id }}" frameborder="0" allowfullscreen></iframe>' > youtubePlayer.html
{% endhighlight bash %}

2. Under you \_posts dir, create a post with propriate jekyll front matter and include "youtubePlayer.html" where you want it to be.


```
---
layout: post
title:  embed youtube
date:   2017-01-06 00:48:00 -0800
categories: HOW-TO
youtubeID: iX5V1WpxxkY --> put up the ID you wanted instead
tag: youtube
---


* content
{:toc}


{\% include youtubePlayer.html id=page.youtubeID %} 
//drop the backslash to include it.

```

__Make sure__: you push both files up to your Github Repo.


* my thanks to the author: Adam Wade Harris and his [original post](http://www.adamwadeharris.com/how-to-easily-embed-youtube-videos-in-jekyll-sites-without-a-plugin/).
