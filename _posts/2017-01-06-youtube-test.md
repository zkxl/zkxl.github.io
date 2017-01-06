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



# Embed Youtube in jelyll powered GihubPage

1. Under you \_includes dir, create a "youtubePlayer.html".

```html
<iframe width="560" height="315" src="https://www.youtube.com/embed/{{ include.id }}" frameborder="0" allowfullscreen></iframe>
```

2. Under you \_posts dir, create a post with propriate jekyll front matter and include "youtubePlayer.html" where you want it to be.

{% highlight bash %}
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


{% include youtubePlayer.html id=page.youtubeID %}
{% endhighlight bash %}


__Make sure__: you push both files updated to your Github Repo.

* my thanks to the author: Adam Wade Harris and his [original post](http://www.adamwadeharris.com/how-to-easily-embed-youtube-videos-in-jekyll-sites-without-a-plugin/).
