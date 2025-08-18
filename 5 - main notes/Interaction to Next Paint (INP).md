2025/04/15  -  16:15
status: 

Tags: [[Performance]] 

Good responsiveness means that a page responds quickly to interactions. When a page responds to an interaction, the browser presents _visual feedback_ in the next frame that it paints. Visual feedback tells you if, for example, an item you add to an online shopping cart is indeed being added, whether a mobile navigation menu has opened, if a login form's contents are being authenticated by the server, and so forth.

Some interactions naturally take longer than others, but for especially complex interactions, it's important to quickly present some initial visual feedback to tell the user that something is happening. The next frame that the browser will paint is the earliest opportunity to do this.

INP is a metric that assesses a page's overall responsiveness to user interactions by observing the latency of all click, tap, and keyboard interactions that occur throughout the lifespan of a user's visit to a page. The final INP value is the longest interaction observed, ignoring outliers.

As the purposes of INP, **only the following interaction types are observed:**
- Clicking with a mouse.
- Tapping on a device with a touchscreen.
- Pressing a key on either a physical or onscreen keyboard.


# References
https://web.dev/articles/inp