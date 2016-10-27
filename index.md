---
layout: html5
---

Something will go here, no doubt

{% for crap in site.data.crap %}

* [![]({{ crap.thumb | prepend: site.data.aws.baseurl }}) {{crap.title}}]({{ crap.crap | prepend: site.data.aws.baseurl}})

{% endfor %}