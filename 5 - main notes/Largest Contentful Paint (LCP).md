2025/04/15  -  15:40
status: 

Tags: [[Performance]] 

LCP reports the render time of the largest [image, text block, or video](https://web.dev/articles/lcp#what-elements-are-considered) visible in the viewport, relative to when the user first navigated to the page. 

As specified in the [`Largest Contentful Paint API`](https://wicg.github.io/largest-contentful-paint/), the types of elements considered for `Largest Contentful Paint` are:
- `<img>` elements (the [first frame presentation time](https://chromium.googlesource.com/chromium/src/+/refs/heads/main/docs/speed/metrics_changelog/2023_08_lcp.md) is used for animated content such as GIFs or animated PNGs)
- `<image>` elements inside an `<svg>` element
- `<video>` elements (the poster image load time or [first frame presentation time](https://chromium.googlesource.com/chromium/src/+/refs/heads/main/docs/speed/metrics_changelog/2023_08_lcp.md) for videos is used—whichever is earlier)
- An element with a background image loaded using the [`url()`](https://developer.mozilla.org/docs/Web/CSS/url\(\)) function, (as opposed to a [CSS gradient](https://developer.mozilla.org/docs/Web/CSS/CSS_Images/Using_CSS_gradients))
- [Block-level](https://developer.mozilla.org/docs/Web/HTML/Block-level_elements) elements containing text nodes or other inline-level text element children.
As well as only considering some elements, LCP measurements use heuristics to exclude certain elements that users are likely to see as "`non-contentful`". For Chromium-based browsers, these include:
- Elements with an opacity of 0, that are invisible to the user
- Elements that cover the full viewport, that are likely considered as background rather than content
- Placeholder images or other images with a low entropy, that likely don't reflect the true content of the page
In addition to late-loading images and fonts, a page may add new elements to the DOM as new content becomes available. If any of these new elements is larger than the previous `largest contentful` element, a new `PerformanceEntry` will also be reported.
If the `largest contentful` element is removed from the viewport, or even from the DOM, it remains the `largest contentful` element unless a larger element is rendered.

# References
