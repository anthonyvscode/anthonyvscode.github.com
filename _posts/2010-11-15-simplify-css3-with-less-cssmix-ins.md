---
title: "Simplify CSS3 with LESS CSS Mixins"
date: 2010-11-15
layout: post
---		                
<p><a title="LESS CSS" href="http://www.dotlesscss.org/" target="_blank">Dotless CSS</a> is one of the greatest libraries for asp.net development ever, and has very quickly become a must-include base library for all my new projects. It organises my CSS into a structure I could set my watch to, and boasts an impressive set of features&#8230; but by far the most useful area of LESS CSS is its &#8220;mix-in&#8221; abilities, allowing you to create snippets of CSS. Adding it to your Visual Studio project is a piece of cake, with the 2 best solutions ive come across being <a href="http://chirpy.codeplex.com/" target="_blank">Chirpy</a> (If you want your CSS minified loaded in the IDE in realtime so you can take advantage of intellisense) or my personal favourite, <a href="https://github.com/jetheredge/SquishIt" target="_blank">Squishit</a>, which handles combining and minimization of your .less and .css (javascript, too) server side.</p>
<p>With the release of IE9, all major browsers now support CSS3 (phew) so finally, the world is our canvas and we can start enjoying the fruits that it has on offer. Even with a few hacks, support for IE6+ is also possible for some CSS3 attributes.</p>
<p><strong>The annoying part: </strong>each browser has its own set of attributes to deal with to get to the end result.</p>
<p><strong>The solution:</strong> LESS CSS Mix-ins. Here are some of my favourite CSS3 functions LESS-ified, one call and all the attributes are put into their right places. You no longer need to remember every browser specific attribute, and if you think of more, just add it to the function and it will appear everywhere.</p>
<pre class="prettyprint" title="">

/*
	Developed by anthonyvscode.com
	This solution is to be used in the dotless implemention of less css: http://www.dotlesscss.org/
*/

/*Border Radius Functions*/
.border_radius(@radius: 10px)
{
	.border_radius(@radius, @radius);
}

.border_radius(@radius_top, @radius_bottom)
{
	.border_radius(@radius_top, @radius_top, @radius_bottom, @radius_bottom);
}

.border_radius(@radius_top_left, @radius_top_right, @radius_bottom_right, @radius_bottom_left)
{
	-webkit-border-radius:@radius_top_left @radius_top_right @radius_bottom_right @radius_bottom_left;
	-moz-border-radius:@radius_top_left @radius_top_right @radius_bottom_right @radius_bottom_left;
	border-radius:@radius_top_left @radius_top_right @radius_bottom_right @radius_bottom_left;
}

/*Shadows*/
.box_shadow(@shadow_x:3px, @shadow_y:3px, @shadow_rad:3px, @shadow_color:#888)
{
	box-shadow: @shadow_x @shadow_y @shadow_rad @shadow_color;
	-webkit-box-shadow:@shadow_x @shadow_y @shadow_rad @shadow_color;
	-moz-box-shadow:@shadow_x @shadow_y @shadow_rad @shadow_color;
}

.text_shadow(@shadow_color:#fff)
{
	text-shadow:0 1px 0 @shadow_col;
}

.opacity(@op:100)
{
	filter:alpha(opacity=@op);
	-moz-opacity:@op/100;
	-khtml-opacity:@op/100;
	opacity:@op/100;
}

.background_gradient(@from:#000, @to:#EEE)
{
	background: @from;
	background-image: -webkit-gradient(linear, left top, left bottom, from(@from), to(@to));
	background-image: -moz-linear-gradient(top, @from, @to);
	filter: formatstring(&quot;progid:DXImageTransform.Microsoft.gradient(startColorstr='{0}', endColorstr='{1}')&quot;, @from, @to); /* IE6,IE7 */
	-ms-filter: formatstring(&quot;\&quot;progid:DXImageTransform.Microsoft.gradient(startColorStr='{0}', EndColorStr='{1}')\&quot;&quot;, @from, @to); /* IE8 */
}

.transition(@range: all, @time: 1s, @ease: ease-in-out)
{
	-moz-transition: @range @time @ease;
	-webkit-transition: @range @time @ease;
	-o-transition: @range @time @ease;
	transition: @range @time @ease;
}

/*Transformations*/
.skew(@angle_x:35, @angle_y:0)
{
	-webkit-transform: skew(formatstring(&quot;{0}deg&quot;, @angle_x), formatstring(&quot;{0}deg&quot;, @angle_y));
	-moz-transform: skew(formatstring(&quot;{0}deg&quot;, @angle_x), formatstring(&quot;{0}deg&quot;, @angle_y));
	-o-transform: skew(formatstring(&quot;{0}deg&quot;, @angle_x), formatstring(&quot;{0}deg&quot;, @angle_y));
	-ms-transform: skew(formatstring(&quot;{0}deg&quot;, @angle_x), formatstring(&quot;{0}deg&quot;, @angle_y));
	transform: skew(formatstring(&quot;{0}deg&quot;, @angle_x), formatstring(&quot;{0}deg&quot;, @angle_y));
}

.scale(@scale_x: 1)
{
	-webkit-transform: scale(@scale_x);
	-moz-transform: scale(@scale_x);
	-o-transform: scale(@scale_x);
	-ms-transform: scale(@scale_x);
	transform: scale(@scale_x);
}

.rotate(@deg:35)
{
	-webkit-transform: rotate(formatstring(&quot;{0}deg&quot;, @deg));
	-moz-transform: rotate(formatstring(&quot;{0}deg&quot;, @deg));
	-o-transform: rotate(formatstring(&quot;{0}deg&quot;, @deg));
	-ms-transform: rotate(formatstring(&quot;{0}deg&quot;, @deg));
	transform: rotate(formatstring(&quot;{0}deg&quot;, @deg));
}

.translate(@move_x: 10px, @move_y: 10px)
{
	-webkit-transform: translate(@move_x, @move_y);
	-moz-transform: translate(@move_x, @move_y);
	-o-transform: translate(@move_x, @move_y);
	-ms-transform:translate(@move_x, @move_y);
	transform: translate(@move_x, @move_y);
}
</pre>
<p>Use and abuse these mix-ins all you want, if I&#8217;ve missed anything (I probably have) just let me know and I will edit it into the post.</p>