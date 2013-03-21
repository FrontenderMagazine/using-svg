# Using SVG

SVG is an image format for vector graphics. It literally means **Scalable Vector
Graphics**. Basically, what you work with in Adobe Illustrator. You can use SVG
on the web pretty easily, but there is plenty you should know.

## Why use SVG at all?

* Small file sizes that compress well
* Scales to any size without losing clarity (except very tiny)
* Looks great on retina displays
* Design control like interactivity and filters

## Getting some SVG to work with

Design something in Adobe Illustrator. Here's a Kiwi bird standing on an oval.

![Kiwi bird][A Kiwi bird standing on an oval]

Notice the artboard is cropped up right agains the edges of the design. Canvas
matters in SVG just like it would in PNG or JPG.

You can save the file directly from Adobe Illustrator as an SVG file.

![Saving][Save the file Adobe Illustrator as an SVG file]

As you save it, you'll get another dialog for SVG Options. I honestly don't know
much about all this. There is a whole spec for [SVG Profiles][1]. I find SVG 1.1
works fine.

![SVG Options][As you save the file, you'll get a dialog for SVG Options]

The interesting part here is that you can either press OK and save the file, or
press "SVG Code..." and it will open TextEdit (on a Mac anyway) with the SVG
code in it.

![SVG code][TextEdit with the SVG code in it]

Both can be useful.

## Using SVG as an `<img>`

If I save the SVG to a file, I can use it directly in an `<img>` tag.

    <img src="kiwi.svg" alt="Kiwi standing on oval">

In Illustrator, our artboard was 612px ✕ 502px.

![Artboard][Size of our artboard in Illustrator]

That's exactly how big the image will on the page, left to itself. You can
change the size of it though just by selecting the `img` and changing its
`width` or `height`, again like you could a PNG or JPG. Here's [an example][2]
of that:

<pre class="codepen" data-height="300" data-type="result" data-href="lCEux" data-user="chriscoyier" data-safe="true"><code></code><a href="http://codepen.io/chriscoyier/pen/lCEux">Check out this Pen!</a></pre>

### Browser support

Using it this way has its own set of specific [browser support][3]. Essentially:
it works everywhere except IE 8 and down and Android 2.3 and down.

If you'd like to use SVG, but also neeed to support these browsers that don't
support using SVG in this way, you have options. I've covered different
techniques in [different][4] [workshops][5] [I've done][6].

One way is to test for support with Modernizr and swap out the `src` of the
image:

    if (!Modernizr.svg) {
      $(".logo img").attr("src", "images/logo.png");
    }

David Bushell has a really [simple alternative][7], if you're OK with JavaScript
in the markup:

    <img src="image.svg" onerror="this.onerror=null; this.src='image.png'">

[SVGeezy][8] can also help. We'll cover more fallback techniques as this article
progresses.

## Using SVG as a `background-image`

Similarly easy to using SVG as an `img`, you can use it in CSS as a `background-
image`.

    <a href="/" class="logo">
      Kiwi Corp
    </a>

    .logo {
      display: block;
      text-indent: -9999px;
      width: 100px;
      height: 82px;
      background: url(kiwi.svg);
      background-size: 100px 82px;
    }

Notice we set the `background-size` to the size of the logo element. We have to
do that otherwise we'll just see a bit of the upper left of our much larger
original SVG image. These numbers are aspect-ratio aware of the original size.
But you could use a `background-size` keywords like `contain` if you want to
make sure the image will fit and can't know the parent image will be of the
exact right size.

### Browser support

Using SVG as background-image has its own special set of [browser support][9],
but it's essentially the same as using SVG as img. The only problem browsers are
IE 8 and down and Android 2.3 and down.

Modernizr can help us here, and in a more efficient way than using img. If we
replace the background-image with a supported format, only one HTTP request will
be made instead of two. Modernizr adds a class name of "no-svg" to the html
element if it doesn't support SVG, so we use that:

    .main-header {
      background: url(logo.svg) no-repeat top left;
      background-size: contain;
    }

    .no-svg .main-header {
        background-image: url(logo.png);
    }

## The problem with both `<img>` and `background-image`...

Is that you don't get to control the innards of the SVG with CSS like you can
with the following two ways. Read on!

## Using "inline" SVG

Remember how you can grab the SVG code right from Illustrator while saving if
you want? (You can also just open the SVG file in a text editor and grab that
code.) You can drop that code right into an HTML document and the SVG image will
show up just the same as if you put it in an img.

    <body>
    
       <!-- paste in SVG code, image shows up!  -->
    
    </body>

This can be nice because the image comes over right in the document and doesn't
need to make an additional HTTP request. In other words, it has the same
advantages as using a [Data URI][10]. It has the same disadvantages too. A
potentially "bloated" document, a big chunk of crap right in the document you're
trying to author, and inability to cache.

If you're using a back end language that can go fetch the file and insert it, at
least you can clean up the authoring experience. Like:

    <?php include("kiwi.svg"); ?>

### Optimize it first

Likely not a huge shocker, but the SVG that Adobe Illustrator gives you isn't
particularly optimized. It has a DOCTYPE and generator notes and all that junk.
SVG is already pretty small, but why not do all we can? Peter Collingridge has
an online [SVG Optimiser][11] tool. Upload the old, download the new. In [Kyle
Foster's video][12], he even takes it the extra mile and removes line breaks
after this optimization.

If you're even more hardcore, here is a [Node JS tool][13] for doing it yourself.

### Now you can control with CSS!

See how the SVG looks a lot like HTML? That's because they are both essentially
XML (named tags with angle brackets with stuff inside). In our design, we have
two elements that make up the design, an `<ellipse>` and an `<path>`. We can
jump into the code and give them class names, just like any other HTML element
can have.

    <svg ...>
      <ellipse class="ground" .../>
      <path class="kiwi" .../>
    </svg>

Now in any CSS on this page we can control those individual elements with
special SVG CSS. This doesn't have to be CSS embedded in the SVG itself, it can
be anywhere, even in our global stylesheet `<link>`ed up. Note that SVG elements
have a special set of CSS properties that work on them. For instance, it's not
`background-color`, it's `fill`. You can use normal stuff like :hover though.

    .kiwi {
      fill: #94d31b; 
    }
    .kiwi:hover {
      fill: #ace63c; 
    }

Even cooler, SVG has all these fancy filters. For instance blurring. Chuck a
filter in your `<svg>`:

    <svg ...>
      ...
      <filter id="pictureFilter" >
        <feGaussianBlur stdDeviation="5" />
      </filter> 
    </svg>

Then you can apply that in your CSS as needed:

    .ground:hover {
      filter: url(#pictureFilter);
    }

Here's [an example][14] of all that:

<pre class="codepen" data-height="300" data-type="result" data-href="evcBu" data-user="chriscoyier" data-safe="true"><code></code><a href="http://codepen.io/chriscoyier/pen/evcBu">Check out this Pen!</a></pre>

* [More on applying filters to SVG][15]
* [The best list I could find on SVG-specific CSS properties][16] (specific to 
Opera)
* [SVG filter effects playground][17] (from Microsoft)

## Browser support

Inline SVG has it's own [set of browser support][18], but again, it's
essentially only an issue in IE 8 and down and Android 2.3 and down<a
href="#note-1" id="ref-1" class="reference">1</a>.

One way to handle fallbacks for this type of SVG is:

    <svg> ... </svg>
    <div class="fallback"></div>

Then use Modernizr again:

    .logo-fallback { 
      display: none;
      /* Make sure it's the same size as the SVG takes up */
    }
    .no-svg .logo-fallback { 
      background-image: url(logo.png); 
    }

## Using SVG as an `<object>`

If "inline" SVG just isn't your jam (remember it does have some legit drawbacks
like being hard to cache), you can link to an SVG file and retain the ability to
affect its parts with CSS by using `<object>`.

    <object type="image/svg+xml" data="kiwi.svg" class="logo">
      Kiwi Logo <!-- fallback image in CSS -->
    </object>

For the fallback, Modernizr detection will work fine here:

    .no-svg .logo {
      width: 200px;
      height: 164px;
      background-image: url(kiwi.png);
    }

This will work great with caching and actually has *deeper* support than using
it any other way. But, if you want the CSS stuff to work, you can't use an
external stylesheet or `<style>` on the document, you need to use a `<style>`
element inside the SVG file itself.

    <svg ...>
      <style>
        /* SVG specific fancy CSS styling here */
      </style>
      ...
    </svg>

### External stylesheets for `<object>` SVG

SVG has a way to declare an external stylesheet, which can be nice for authoring
and caching and whatnot. This only works with `<object>` embedding of SVG files
as far as I've tested. You'll need to put this in the SVG file above the opening
`<svg>` tag.

    <?xml-stylesheet type="text/css" href="svg.css" ?>

If you put that in your HTML, the page will barf and not even try to render. If
you link up an SVG file that has that in it as an `<img>` or `background-image`,
it won't barf, but it won't work (the SVG will still render though).

## Data URI's for SVG

A way to shrink SVG's even smaller is to convert them into Data URI's.
Mobilefish.com has an [online conversion tool][19] for that. Simply paste in the
contents of your SVG file and fill out the form and it will display the results
in a textarea for you to copy. Remember to remove line breaks in the data it
gives you back. It looks like pure gibberish:

![Data URI][The results of SVG convertion into Data URI]

You can use that anywhere we've talked about so far (except inline `<svg>`
because that just doesn't make sense) Just put the gibberish where it says
[data] in these examples.

### As `<img>`

    <img src="data:image/svg+xml;base64,[data]>

### As CSS

    .logo {
      background: url(data:image/svg+xml;base64,[data]);
    }

### As `<object>`

    <object type="image/svg+xml" data="data:image/svg+xml;base64,[data]>
      fallback
    </object>

And yep, if you have an embedded `<style>` in your SVG before you base64 it, [it
will work][20] if you use it as an `<object>` still!

For the hardcore, Filament group has [grunticon][21] for automating the process.

Command line thingy for base64ing SVG:

<blockquote class="twitter-tweet"><p>@<a href="https://twitter.com/chriscoyier">chriscoyier</a> @<a href="https://twitter.com/hkfoster">hkfoster</a> maybe you could take a shortcut with &gt;&gt;&gt; echo -n `cat logo.svg` | base64 | pbcopy</p>&mdash; Benny Schudel (@bennyschudel) <a href="https://twitter.com/bennyschudel/status/307963605998006273">March 2, 2013</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

Or alternatively [Mathias Bynens has some techniques][22]:

> Use `openssl base64 < path/to/file.png | tr -d '\n' | pbcopy` or 
`cat path/to/file.png | openssl base64 | tr -d '\n' | pbcopy` to skip writing to
a file and just copy the base64-encoded output to the clipboard without the line
breaks.

## Related Stuff!

* David Bushell: [A Primer to Front-end SVG Hacking][23]
* David Bushell: [Resolution Independence With SVG][24]
* [MDN on SVG][25]
* Browser support for [a variety of different SVG related things][26].
* Peter Gasston: [Better SVG Sprites With Fragment Identifiers][27]
* simuari: [SVG Stacks][28]
* [SVG.js][29] - "A lightweight library for manipulating and animating SVG."
* Emmet has [an awesome way][30] to get a data URI from an SVG right from your 
code editor.
* Compass [has a helper][31] for data URIs too.
* Adobe: [Styling SVG][32]
* Andrew J. Baker: [Taming the SVG Beast][33]
* Illustrator alternatives: [Inkscape][34], [Sketch][35]
* Krister Kari: [Dealing with SVG images in mobile browsers][36]

Kyle Foster's [An Optimized SVG Workflow][37], which is worth an embed:

<iframe width="480" height="360" src="http://www.youtube.com/embed/iVzW3XuOm7E?rel=0" frameborder="0" allowfullscreen></iframe>

... and the [follow up][38] + [slides][39].

---

### Notes

<a href="#ref-1" id="note-1" class="note">1.</a> And speaking of Android 2.3 
browser, [this][40]. But if you absolutely have to support the native browser, 
[this][41].

[1]: http://www.w3.org/TR/SVGMobile/
[2]: http://codepen.io/chriscoyier/pen/evcBu
[3]: http://caniuse.com/#feat=svg-img
[4]: http://css-tricks.com/workshop-notes-webstock-13/
[5]: http://css-tricks.com/workshop-notes-from-incontrol-hawaii/
[6]: http://css-tricks.com/deseret-digital-workshop/
[7]: http://dbushell.com/2013/02/04/a-primer-to-front-end-svg-hacking/
[8]: http://benhowdle.im/svgeezy/
[9]: http://caniuse.com/#feat=svg-css
[10]: http://css-tricks.com/data-uris/
[11]: http://petercollingridge.appspot.com/svg_optimiser
[12]: http://www.youtube.com/watch?v=iVzW3XuOm7E&feature=youtu.be
[13]: https://github.com/svg/svgo
[14]: http://codepen.io/chriscoyier/pen/evcBu
[15]: https://developer.mozilla.org/en-US/docs/Applying_SVG_effects_to_HTML_content
[16]: http://www.opera.com/docs/specs/presto25/svg/cssproperties/
[17]: http://ie.microsoft.com/testdrive/graphics/hands-on-css3/hands-on_svg-filter-effects.htm
[18]: http://caniuse.com/#feat=svg-html5
[19]: http://www.mobilefish.com/services/base64/base64.php
[20]: http://codepen.io/chriscoyier/pen/ioCjk
[21]: https://github.com/filamentgroup/grunticon
[22]: http://superuser.com/questions/120796/os-x-base64-encode-via-command-line#comment280484_120815
[23]: http://dbushell.com/2013/02/04/a-primer-to-front-end-svg-hacking/
[24]: http://coding.smashingmagazine.com/2012/01/16/resolution-independence-with-svg/
[25]: https://developer.mozilla.org/en-US/docs/SVG
[26]: http://caniuse.com/#search=svg
[27]: http://www.broken-links.com/2012/08/14/better-svg-sprites-with-fragment-identifiers/
[28]: http://simurai.com/post/20251013889/svg-stacks
[29]: http://svgjs.com/
[30]: http://docs.emmet.io/actions/base64/
[31]: http://compass-style.org/reference/compass/helpers/inline-data/
[32]: http://blogs.adobe.com/webplatform/2013/01/08/svg-styling/
[33]: http://buildnewgames.com/taming-the-svg-beast/
[34]: http://inkscape.org/
[35]: http://www.bohemiancoding.com/sketch/#4
[36]: http://kristerkari.github.com/adventures-in-webkit-land/blog/2013/03/08/dealing-with-svg-images-in-mobile-browsers/
[37]: http://www.youtube.com/watch?v=iVzW3XuOm7E&feature=youtu.be
[38]: http://www.youtube.com/watch?v=1AdX8odLC8M&feature=youtu.be
[39]: http://kylefoster.me/svg-slides/
[40]: https://twitter.com/paul_irish/status/309037258638512129
[41]: http://www.kendoui.com/blogs/teamblog/posts/12-02-17/using_svg_on_android_2_x_and_kendo_ui_dataviz.aspx

[A Kiwi bird standing on an oval]: img/kiwi.png?raw=true&amp;repo=using-svg
[Save the file Adobe Illustrator as an SVG file]: img/save-as-svg.png?raw=true&amp;repo=using-svg
[As you save the file, you'll get a dialog for SVG Options]: img/svg-options.png?raw=true&amp;repo=using-svg
[TextEdit with the SVG code in it]: img/svg-code.png?raw=true&amp;repo=using-svg
[Size of our artboard in Illustrator]: img/artboard.png?raw=true&amp;repo=using-svg 
[The results of SVG convertion into Data URI]: img/base64-data.png?raw=true&amp;repo=using-svg