# Backdrop-filter Explainer

## Introduction
Backdrop-filter is a CSS property that applies one or more filters to the "backdrop" of an element. The "backdrop" basically<sup>1</sup> means all of the painted content that lies behind the element. This allows designers to construct "frosted glass" dialog boxes, video overlays, translucent navigation headers, and more.

The backdrop-filter feature is easy to use. One simply needs to create an element with a partially transparent background, and apply the `backdrop-filter: {filters};` style to that element. The specified filters will be applied to the painted content **behind** the element, prior to drawing the element itself.

<sub><sup>1</sup> See the detailed spec for the precise definition of the "backdrop". It is defined as almost everything behind the page, except in some circumstances where a DOM ancestor of the backdrop-filtered element contains filters or opacity or a few other things.</sub>

## Why is this needed? Who wants it?
Ever since Webkit shipped a prefixed version of this feature [in 2015](https://webkit.org/blog/3632/introducing-backdrop-filters), developers have been asking for Chromium to implement it. The [main Chromium feature tracking bug](https://crbug.com/497522) has **573** stars as of May 20, 2019. A number of web design blogs highlight this "cool" feature, and bemoan the lack of Chromium support. It is clear from the comments on [the bug](https://crbug.com/497522), and in the general discussions around the web, that this feature is highly desired by designers. We should make it available to them.

Here are a few such blog posts:
* https://ferdychristant.com/please-help-make-backdrop-filter-a-reality-f81805ba3d52
* https://css-tricks.com/the-backdrop-filter-css-property
* https://webdesign.tutsplus.com/tutorials/css-backdrop-filters--cms-27314
* https://www.reddit.com/r/webdev/comments/3zgpnb/we_figured_out_a_way_to_recreate_the_blurred
* https://stackoverflow.com/questions/38145368/css-workaround-to-backdrop-filter
* https://www.sitepoint.com/create-stunning-image-effects-with-css-backdrop-filter

Some quotes from the [Chromium bug](https://crbug.com/497522):
* "I'd love to see this feature this year. It will change the look of the internet." - xxephis@gmail.com, Jul 13 2017
* "Backdrop blurs date as far back as Windows 7, aero theme. Not that it matters, just to agree it's about time we get this on the web. Gimmick or not, sometimes the web needs something exciting and beautiful." - ferdy.christant@gmail.com, Jul 13 2017
* "this is not some "nice to have" it really is a significant ability to filter the things below a div in a z-organized stack of elements. I will add that Android material design, ios/macos, and windows 10 fluent design all provide this kind of functionality." - onlineventures@gmail.com, Jul 25 2017
* "Computationally blur is expensive. Hopefully Chromium developers are taking the time to carefully consider how to maintain good user experience (in terms of performance and battery life)." - henrikhelmers@gmail.com, Jul 26 2017
* "Hugely useful feature." - gareth@cityseer.io, Aug 14 2017
* "my most expected feature of this year!" - d2phap@gmail.com, Aug 29 **2017**
* "The blur-behind paradigm has been crucial for Apple UIs for a very long time and it would be really great to have simple parity in this regard on the Web." - ahogan@zendesk.com, Sep 20 2017
* "I definitely support adding this. Our designers are asking for it, and it'd help us out a lot!" - layton.miller@medchatapp.com, May 3 2018
* "I use this feature too and would appreciate it being prioritized." - and.mikhaylov@gmail.com, May 31 2018
* ... and over 150 more ...

## Goals
One of the primary reasons why this feature has not been delivered for the last few years centers around the need for a concrete specification. Several details were historically ambiguous, leading to differences in implementation, and some performance issues. The goals of this implementation are therefore:
* A clean, unambiguous definition of how the feature works, and what the output should look like.
* A performant implementation that does not excessively slow down sites that use it.

Note that this feature can be very performance-demanding. This is part of the reason why there are no effective polyfills - none can be built that are fast enough. See, for example, the comments on the [Reddit](https://www.reddit.com/r/webdev/comments/3zgpnb/we_figured_out_a_way_to_recreate_the_blurred) and [StackOverflow](https://stackoverflow.com/questions/38145368/css-workaround-to-backdrop-filter) posts - while polyfills are provided, they are extremely limited in terms of what they can filter, and all are too slow for production use.

## Where can this be used?
There are several often-cited examples for the use of this feature, including translucent dialog boxes, navigation headers, and other cool effects.

Here are a few examples from around the web:
* https://css-tricks.com/wp-content/uploads/2018/05/backdrop-demo.mp4 ([Live site here](https://codepen.io/robinrendle/full/LmzLEL))
* https://youtu.be/MWHBpReeAJ0
* https://www.apple.com (see the translucent navigation bar at the top)

However, as with all design-related features, it is likely that many cool new uses of this feature will only be discovered after this feature is shipped and usable on production sites.

## Solution
The backdrop-filter CSS property solves the problems described above, and meets the goals. It gives web authors an easy way to achieve the effects they want, without going to great lengths in HTML/CSS/JS to achieve them, and without incurring a significant performance penalty.

The implementation of this feature in Chromium is complex, but not overly so, due to the adoption of reasonable spec language defining the "Backdrop Root". The backdrop-filter style is plumbed through to the `cc::EffectNode` for the element, and on to the `viz::RenderPass`. In the renderer, when this renderpass is encountered:

 1. A readback of the destination graphics texture is performed. This contains the [Backdrop Image](https://drafts.fxtf.org/filters-2/#BackdropRoot).
 2. The filters are applied to this readback.
 3. The filtered image is clipped to the rounded-corner rect of the element's border.
 4. This image is drawn into the texture for the backdrop-filtered element, before any other contents.
 5. The rest of the backdrop-filtered element and children are rendered.

Because of the way the Backdrop Root is defined (see [the spec](https://drafts.fxtf.org/filters-2/#BackdropRoot)), at step #1, only the nearest ancestor render surface is needed. No additional rasterization needs to take place, and this leads to a performant implementation.

## Example code
Using backdrop-filter is easy. Assuming you want a dialog box that has a "frosted glass" look:
```
<body>
  <p>...page content here...</p>
  <div id="modal-dialog">
     <p>...dialog box content here...</p>
  </div>
</body>
```
...simply apply the backdrop-filter style to the element:
```
<style>
#modal-dialog {
  backdrop-filter:blur(10px);
}
</style>
```

[Here is a live demo](https://mfreed7.github.io/backdrop-filter-feature/examples/scrollable.html) showing the filtering effect applied to a dialog over a scrollable image.


## The specification
The [specification](https://drafts.fxtf.org/filter-effects-2/#BackdropFilterProperty) was [approved by the CSSWG](https://github.com/w3c/fxtf-drafts/issues/53#issuecomment-467152004) at the CSSWG SF F2F, on Feb 25, 2019. It contains much more detail than this page, including a large motivation section detailing the need for some of the restrictions included in the spec.

## Chrome Status Entry
The Chrome Platform Status entry is [here](https://www.chromestatus.com/features/5679432723333120).

## Alternatives
The primary debate with respect to backdrop-filter lies in the specification of what exactly constitutes the "backdrop". There was significant debate about this, [spanning three years](https://github.com/w3c/fxtf-drafts/issues/53), with positions ranging from "it should filter everything on the page" to "it should only filter up to the containing stacking context". In the end, the solution described in the [spec](https://drafts.fxtf.org/filter-effects-2/#BackdropFilterProperty) falls somewhere in the middle. It offers as much of the flexibility of the "it should filter everything" camp as possible, without incurring the intractability and performance problems of including filters, opacity, and masks. See [the Motivation section](https://drafts.fxtf.org/filter-effects-2/#BackdropRootMotivation) of the spec for a much more detailed explanation of the trade-offs, including examples.

## Implementation
This feature, as described in the spec, is implemented in Chromium as of M76.
