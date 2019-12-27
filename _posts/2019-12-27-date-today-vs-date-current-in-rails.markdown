---
layout: post
title:  "Date.today vs Date.current in Rails"
date:   2019-12-27
---

The other day I learned a valuable lesson when some of my tests in a Ruby on Rails application suddenly started failing, despite them passing earlier in the day and despite no changes to the code. The issue boiled down to my assumption that the following would always be true:

`Date.today + 1.day == Date.tomorrow`

However, when I ran my tests later in the evening, this comparison returned `false`.

It turns out that `Date#today` uses the system's time zone, which in my case was Eastern Standard Time (EST). `Date#tomorrow`, on the other hand, comes from [Rails' core extension to Ruby's `Date` class](https://api.rubyonrails.org/classes/Date.html), and it uses another core extension method, `Date#current` to retrieve tomorrow's date:

{% highlight ruby %}
# File activesupport/lib/active_support/core_ext/date/calculations.rb, line 43
def tomorrow
  ::Date.current.tomorrow
end
{% endhighlight %}

Looking at the source code of `Date#current`, we see that it uses `Time.zone` (if it exists) to determine today's date:

{% highlight ruby %}
# File activesupport/lib/active_support/core_ext/date/calculations.rb, line 48
def current
  ::Time.zone ? ::Time.zone.today : ::Date.today
end
{% endhighlight %}

In the application I was working in, `Time.zone` was set to Universal Time Coordinated (UTC), which was five hours ahead of my system's time.

It doesn't seem unreasonable to assume that `Date.today` and `Date.tomorrow` are intended to be used together; however, we've seen that these methods actually come from two different places and can potentially use two different time zones.

To avoid this, and to ensure you're always using the configured time zone instead of the system's, I reccommend using `Date.current` (instead of `Date.today`), along with the other [`Date` methods provided by Rails](https://api.rubyonrails.org/classes/Date.html).
