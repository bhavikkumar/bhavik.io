---
layout: post
title:  "Google Analytics with Jekyll"
date:   2016-05-22 17:30:00 +1200
---
Google analytics is easy to setup from the instructions which are provided by Google themselves. However, there are some quirks if you set it up with default settings. Mainly around if people are viewing the site via services such as Google translate<sup>[[1]](http://veithen.github.io/2015/01/05/jekyll-improving-ga-data-quality.html)</sup>.

The following should allow you to setup analystics without getting strange results when looking at the traffic in Google Analytics.

Once you have the Google analytics setup, Google says `paste your tracking code snippet (unaltered, in its entirety) before the closing </head> tag on every web page on your site you wish to track`. 

This would be error prone if we had to put it into every page we create so we are going to create a new file called `analytics.html` in the `_includes` folder in the Jekyll project. We don't want to be posting the actual snippet from Google analytics because this will cause us problems later on, instead we want to use the following:
{% highlight js %}{% raw %}
<script>
    (function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
    (i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
    m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
    })(window,document,'script','https://www.google-analytics.com/analytics.js','ga');

    ga('create', '{{ site.google_analytics }}', 'auto');
    ga('send', 'pageview', {
    'page': '{{ page.url }}',
    'title': '{{ page.title | replace: "'", "\\'" }}'
    });
</script>
{% endraw %}{% endhighlight %}

Don't forget to add your tracking code to `_config.yml` as `google_analytics: UA-XXXXXXXX-X`.

Now we need to have the `analytics.html` included on all our pages, this can be done by adding it to `head.html`, if you are using the default layout, otherwise put the following snippet in the appropriate location<sup>[[2]](https://michaelsoolee.com/google-analytics-jekyll/)</sup>.

{% highlight html %}{% raw %}
    {% if jekyll.environment == 'production' %}
        {% include analytics.html %}
    {% endif %}
    </head>
{% endraw %}{% endhighlight %}

You may have noticed that I've wrapped mine in a if statement, this is so that when testing on localhost I don't get analytics sent from my testing.

The final step is to build the site with the production environment variable, the default for Jekyll is development. Since I am build my blog on Windows 10 and use powershell, the following command will get it to build with the analytics included `$env:JEKYLL_ENV="production"; jekyll build`