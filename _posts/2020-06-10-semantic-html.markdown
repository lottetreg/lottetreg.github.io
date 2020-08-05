---
layout: post
title:  "Semantic HTML"
date:   2020-06-10
---

Today I learned about semantic HTML, which provides _meaning_ through the HTML elements as opposed to pure presentation.

Inside our React component we had a piece of JSX that represents a grid of logos, with the "logo" `<div>` elements placed inside a parent "grid" `<div>`.

{% highlight jsx %}
<div className="grid">
  {images?.map((image, index) => {
    return (
      <div className="logo" key={index}>
        <img alt={image.altText} src={image.path} />
      </div>
    )
  })}
</div>
{% endhighlight %}

It looks great in the browser, but it could still be improved. Because this use of `<div>` elements is not semantic, there is no way to tell by looking at these elements how they are related to each other. This can make it difficult for visually impaired users with screen readers to understand what's happening on the site. Neglecting to use semantic HTML can also have SEO repercussions, as search engines may have difficulty delivering the right pages for the right queries.

If we think about what this grid of logos really is, it is an unordered list of images. (If the order of the images mattered, for example if the images were part of a timeline and were in a chronological sequence, we would want to use an ordered list, or `<ol>`, instead.)

So let's change our code to use a `<ul>` element with `<li>` children, instead of `<div>` elements:


{% highlight jsx %}
<ul className="grid">
  {images?.map((image, index) => {
    return (
      <li className="logo" key={index}>
        <img alt={image.altText} src={image.path} />
      </li>
    )
  })}
</ul>
{% endhighlight %}

This is much more semantic, and will be more easily understood by screen readers; however, now it doesn't look right in the browser—the images are indented and have bullet points next to them. We have to reverse this default `<ul>` styling with CSS.

{% highlight css %}
.grid {
  list-style: none;
  margin-block-start: 0;
  margin-block-end: 0;
  padding-inline-start: 0;
}
{% endhighlight %}

Now the HTML is semantic, and it looks just like it did before in the browser, when we were using `<div>` elements.

Another important aspect of building sites for your visually impaired users (and for added the SEO benefits) is to have meaningful `alt` tag text on all of your images (except for those that are purely for decoration). This text should clearly and concisely describe the image, and if the image is inside an `<a>` tag, it should describe where the link goes.
