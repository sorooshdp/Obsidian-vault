2025/04/12  -  10:16
status: 

Tags: [[Performance]]  

## Largest Contentful Paint (LCP)

Largest Contentful Paint measures perceived load speed and marks the point in the page load timeline when the page's main content has likely loaded. In simpler terms, LCP tracks how quickly the largest visible content element (usually an image, video, or large text block) appears on the screen.

Thresholds:
- Good: ≤2500 ms
- Needs Improvement: 2500-4000 ms
- Poor: >4000 ms

LCP directly correlates with user perception of site speed. When users see the main content quickly, they perceive the site as fast and responsive, even if other elements are still loading. Conversely, slow LCP creates the impression of a sluggish site, regardless of how quickly other aspects load.

**Common LCP Elements:**
- Large hero images
- Banner images
- Large text blocks
- Video thumbnails
- Above-the-fold content
## Interaction to Next Paint (INP)

Interaction to Next Paint measures responsiveness and quantifies the experience users feel when trying to interact with the page[2](https://web.dev/articles/defining-core-web-vitals-thresholds). INP evaluates how quickly a page responds to user interactions such as clicks, taps, and keyboard inputs. This metric has replaced the older First Input Delay (FID) metric because it more comprehensively measures the entire interaction latency rather than just the initial delay.

**Thresholds:**
- Good: ≤200 ms
- Needs Improvement: 200-500 ms
- Poor: >500 ms[2](https://web.dev/articles/defining-core-web-vitals-thresholds)

INP is particularly important because it directly measures the frustration users feel when a site appears unresponsive. When users click a button or tap a link, they expect immediate feedback; any delay creates confusion and annoyance.
The measurement of INP considers the full duration from when a user initiates an interaction until the browser renders the next frame that shows the result of that interaction. This includes processing time, rendering time, and any other delays that affect the perceived responsiveness of the page.

## Cumulative Layout Shift (CLS)
Cumulative Layout Shift measures visual stability and quantifies the amount of unexpected layout shift of visible page content[2](https://web.dev/articles/defining-core-web-vitals-thresholds). CLS tracks how much the visible elements on a page move around as the page loads, capturing the frustration users experience when content shifts unexpectedly.

**Thresholds:**
- Good: ≤0.1
- Needs Improvement: 0.1-0.25
- Poor: >0.25[2](https://web.dev/articles/defining-core-web-vitals-thresholds)

CLS is calculated based on both the size of the unstable element and the distance it moves. For instance, a small button shifting slightly receives a lower CLS score than a large image that jumps several hundred pixels.
Layout shifts typically occur when resources load asynchronously or DOM elements are dynamically added to the page above existing content, such as ads, embeds, or images without predefined dimensions.

## Measuring and Interpreting Core Web Vitals

To classify the overall performance of a page or site, Core Web Vitals use the 75th percentile value of all page views to that page or site[2](https://web.dev/articles/defining-core-web-vitals-thresholds). This means:

- If at least 75 percent of page views meet the "good" threshold, the site is classified as having "good" performance for that metric.
- If at least 25 percent of page views meet the "poor" threshold, the site is classified as having "poor" performance[2](https://web.dev/articles/defining-core-web-vitals-thresholds).

## Hydration: A Critical Performance Factor

Hydration is a process that restores server-side rendered applications on the client, including reusing the server-rendered DOM structures, persisting application state, and transferring application data that was already retrieved by the server[3](https://angular.dev/guide/hydration). This process is crucial for modern frontend frameworks that implement server-side rendering (SSR).
## Why Hydration Matters for Core Web Vitals

Hydration significantly impacts Core Web Vitals metrics in several ways:
1. **Improved LCP**: By reusing existing DOM elements rather than recreating them, hydration can significantly improve LCP by making content visible faster.
2. **Better INP**: Proper hydration enables faster interactivity by avoiding the need to re-render elements before they become interactive.
3. **Reduced CLS**: Without hydration, server-side rendered applications will destroy and re-render the application's DOM, which may result in visible UI flicker and layout shifts[3](https://angular.dev/guide/hydration).

## Optimizing LCP
1. **Minimize Main Thread Work**: It's imperative to minimize main thread work and avoid nested resources. This requires senior engineering expertise and often substantial code rewriting[1](https://www.rebelmouse.com/how-to-improve-core-web-vitals).
2. **Optimize Critical Rendering Path**: Ensure critical CSS is inlined and non-critical resources are deferred.
3. **Optimize Images**:
    - Use modern image formats (WebP, AVIF)
    - Implement responsive images with appropriate sizes
    - Lazy load images below the fold
    - Preload critical images[1](https://www.rebelmouse.com/how-to-improve-core-web-vitals)
4. **Implement Preconnect and Preload**: Use resource hints to establish early connections and load critical resources sooner.
5. **Use Server-Side Rendering or Static Generation**: These approaches can significantly improve LCP by delivering pre-rendered content to users.

## Optimizing INP
1. **Reduce JavaScript Execution Time**: Optimize JavaScript execution, especially during hydration phases[4](https://dev.to/noriste/how-preply-improved-inp-on-a-nextjs-application-without-react-server-components-and-app-router-j8c).
2. **Virtualize Long Lists**: For pages with extensive lists, implement virtualization to reduce the number of DOM nodes and improve interactivity[4](https://dev.to/noriste/how-preply-improved-inp-on-a-nextjs-application-without-react-server-components-and-app-router-j8c).
3. **Leverage React 18 Features**: Upgrading to React 18 can significantly improve INP (one company saw a 140ms decrease) due to improved concurrent rendering and hydration processes[4](https://dev.to/noriste/how-preply-improved-inp-on-a-nextjs-application-without-react-server-components-and-app-router-j8c).
4. **Implement Event Delegation**: Use event delegation where appropriate to reduce the number of event listeners and improve interaction responsiveness.
5. **Optimize Third-Party Scripts**: Audit and optimize third-party scripts, which often contribute significantly to poor interactivity.
## Optimizing CLS
1. **Set Explicit Dimensions for Media**: Always specify width and height attributes for images, videos, and embeds to reserve space during loading.
2. **Avoid Inserting Content Above Existing Content**: When dynamically adding content, add it below existing content to prevent shifts.
3. **Use CSS aspect-ratio Property**: For responsive elements, use the aspect-ratio property to maintain proportions during loading.
4. **Reserve Space for Ads and Embeds**: Always allocate specific dimensions for ads, embeds, and iframes.
5. **Use Font Display Settings**: Implement proper font display settings to prevent text shifts during font loading.
## Optimizing Hydration
1. **Implement Progressive Hydration**: Hydrate the most critical parts of the page first and defer less important sections.
2. **Leverage Framework-Specific Features**:
    - For React, use React 18's improved hydration features and consider React Server Components for static content[4](https://dev.to/noriste/how-preply-improved-inp-on-a-nextjs-application-without-react-server-components-and-app-router-j8c)
    - For Angular, enable hydration through `provideClientHydration`[3](https://angular.dev/guide/hydration)
3. **Manage Third-Party JavaScript**: Implement a progressive web app approach to third-party libraries, loading ads and other resources after critical elements[1](https://www.rebelmouse.com/how-to-improve-core-web-vitals).
4. **Use Passive Listeners**: Implement passive event listeners to improve scrolling performance and interactivity[1](https://www.rebelmouse.com/how-to-improve-core-web-vitals).
5. **Implement Smart Caching Rules**: Define effective caching policies at both the CDN and origin levels to improve performance[1](https://www.rebelmouse.com/how-to-improve-core-web-vitals).

## Advanced Optimization Techniques

## Ultra-Light JavaScript Applications
Popular frameworks like React.js and Bootstrap speed up development but require significant overhead. Consider custom-writing only what you need and continuously optimizing your code base. RebelMouse reports getting their entire library down to 15 KB through this approach[1](https://www.rebelmouse.com/how-to-improve-core-web-vitals).
## Progressive Web App Techniques
1. **Smart Loading Strategies**: Load images ahead of time for certain screen sizes depending on network speeds[1](https://www.rebelmouse.com/how-to-improve-core-web-vitals).
2. **Font Management**: Preload fonts to avoid rendering delays and layout shifts[1](https://www.rebelmouse.com/how-to-improve-core-web-vitals).
3. **Service Worker Caching**: Implement service workers to cache resources and provide offline functionality.
## Performance Monitoring and Analysis
1. **Implement Real User Monitoring (RUM)**: Use the web-vitals library to collect Core Web Vitals data in real-time from actual users[4](https://dev.to/noriste/how-preply-improved-inp-on-a-nextjs-application-without-react-server-components-and-app-router-j8c).
2. **Create Performance Dashboards**: Develop dashboards to visualize performance data and identify problem areas[4](https://dev.to/noriste/how-preply-improved-inp-on-a-nextjs-application-without-react-server-components-and-app-router-j8c).
3. **Intersect Multiple Data Sources**: Combine data from various sources like heat maps, browser dev tools, and performance monitoring to gain comprehensive insights[4](https://dev.to/noriste/how-preply-improved-inp-on-a-nextjs-application-without-react-server-components-and-app-router-j8c).

# References
