---
layout : layout
title : anthonyvscode
---

<ul id="archive">
    {% for post in site.posts %}
		<li>
			<a href="{{ post.url }}">{{ post.title }}</a><span data-disqus-identifier="{{ post.url }}"></span>
			<span class="date">{{ post.date | date: "%d %B, %Y" }}</span>
		</li>
    {% endfor %}
</ul>


<script type="text/javascript">
/* * * CONFIGURATION VARIABLES: EDIT BEFORE PASTING INTO YOUR WEBPAGE * * */
var disqus_shortname = 'anthonyvscode'; // required: replace example with your forum shortname

/* * * DON'T EDIT BELOW THIS LINE * * */
(function () {
    var s = document.createElement('script'); s.async = true;
    s.type = 'text/javascript';
    s.src = 'http://' + disqus_shortname + '.disqus.com/count.js';
    (document.getElementsByTagName('HEAD')[0] || document.getElementsByTagName('BODY')[0]).appendChild(s);
}());
</script>