---
layout:       post
uuid:         59a009a0-ff02-0135-2e86-745c898ed35b
categories:   HTML
tags:         [javascript, emberjs, html]
title:        'Not so simple simple "see more" link'
date:         2018-02-28
author:
  name:       Hunter
  twitter:    hefoxed
  github:     hefox
feature_img:  null
sitemap:
  lastmod:    2018-02-28T14:12:12
  priority:   0.5
  changefreq: monthly
  exclude:    'no'
---

I needed a 'see more' link and my usual go-tos wouldn't work, so with some fancy CSS, I made my own.

### The Task

I needed to do a fairly common task -- show a snippet of text that then expand to show the full text on clicking.

Modern CSS is great compared to when I first started back in the early 2000s.  One of the nifty properties now is `text-overflow`, which can easily be used to truncate text.  Combined with a bit of JavaScript this makes for a quick and easy expand functionality.

All you have to do is:

{% highlight html %}
<div class="truncatable truncate">
  Text that can be very long that you want to truncate
</div>
{% endhighlight %}

{% highlight css %}
.truncatable {
  width: 100px;
  overflow: hidden;
}
.truncate {
  white-space: nowrap;
  text-overflow: ellipsis;
}
{% endhighlight %}

Then add a bit of JavaScript that removes `truncate class` on click.  [Try it out!](http://jsbin.com/bemoqoy/3/edit?html,css,js,output)!

It was working brilliantly, but after some internal testing we received feedback that we wanted hide or show multiple rows instead of just one, but the `text-overflow` property only works for a single line!

### I considered various options

First, I considered JavaScript. I’m using [EmberJs](http://emberjs.com/) as our frontend framework, so I could have made separate variables for truncated text and non-truncated text, switching between them in a click handler. However, I need to support more than simple text; the content includes links and HTML rendered via handlebar templates.

I then considered [ember-truncate](https://www.npmjs.com/package/ember-truncate), but as we don’t have ember-cli yet, it was not simplistic to add the component.  And, looking at the code, the implementation is one I considered, but would prefer not to use as ember (like many modern frontend frameworks) can be a bit iffy with manual DOM manipulation. While ember-truncate is an open source add-on, that doesn’t necessarily mean it’ll work well in my project. Also, the final nail in the coffin, I have to do the same formatting in non-ember applications so a pure CSS/HTML/JavaScript solution is the most ideal.

I then discovered a [nifty technique](http://hackingui.com/front-end/a-pure-css-solution-for-multiline-text-truncation/) that uses `&before`/`&after` to add an ellipsis if the content will overflow. However, I didn't quite get it to work, and it relied on justified text, which I didn’t want.  Finally, the ideal solution would add a “See more" link under the truncated text instead.

### I adapted that technique into my own

One important part of that technique is to control how many lines appear using `line-height` to a multiple of the text height using `em`. For example, `max-height: 3em` with `line-height` of '1.5em' will show 2 lines, `max-height: 6` will be 4 lines, `max-height: 7.5` will be 5 lines, and so on. Using that, I can position something at specific line height for it to show on that line.

Say I want 5 lines. I set the parent div (`.truncatable`) to be:
- relative position so that can absolute position items in it. Absolute positioning positions by the most recent ancestor with non-static positioning (for example, a parent with `position: relative`)
- A specific line-height (e.g. `line-height: 1.5em`)
- Overflow to hide so does not show (`overflow: hidden`)
- When `truncate` class is present, set max-height to the number of lines I want to show times the line-height (so for five lines and 1.5 line height, `max-height: 7.5`)

Then I add a new div, `.see-more` with a link in it. When the parent div is being truncated (`.truncate`), I want this new div to
- be absolutely positioned so I can place it specifically (`position: absolute`) at the last line of the visible area, so line height (1.5) times amount of rows (5) minus 1 -- which for this example is (1.5 * (5 - 1)) = 6em (`top: 6em`)
- have 100% width since the absolute positioning removes the usual width behavior (`width: 100%`)
- set background colour to that of the parent div. This is positioned over the visible text, so I “hide" the text below it. Since it’s has line-height the way it is, the full line is hidden.

There’s three cases to examine to understand this technique:

Area is less than 4 lines: The see-more div does not show because it’s positioned in an area of the div that is not visible since the text is too short
Area is more than 5 lines: The see-more div shows at the bottom as desired. Clicking it removes the truncate class and see more link is now at the bottom as it’s following normal positioning  (as it’s only absolutely positioned when the truncate class is on the parent).
Area is exactly 5 lines: The see-more appears, and when clicked disappears to show one line -- this is less then ideal but for my use case, isn’t the usual case enough to care.

{% highlight html %}
<div class="truncatable truncate">
  Text that can be very long that you want to truncate
  <div class="see-more">
    <a>See more/less</a>
  </div>
</div>
{% endhighlight %}

{% highlight css %}
.truncatable {
  line-height: 1.5em;
  position: relative;
  width: 100px;
  overflow: hidden;
  background-color: white;
}
.truncatable.truncate {
  max-height: 7.5em;
}
.truncate.see-more {
  top: 6em;
  width: 100%;
  position: absolute;
  background-color: white;
}
{% endhighlight %}

[Try it out!](http://jsbin.com/zusahi/edit?html,css,js,output)
