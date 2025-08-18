2024/11/14  -  00:01
status: 

Tags: [[Browser]] [[Performance]]

## Navigation and Response

- The first step in loading a webpage is navigation, where the browser locates the server hosting the webpage. This involves a DNS lookup to get the server’s IP address.

- Once located, the browser establishes a connection to the server through a TCP handshake. For secure connections over HTTPS, a TLS negotiation occurs, encrypting the communication channel. These steps involve several back-and-forth messages between the browser and server.

- After the connection is established, the browser sends an HTTP GET request, usually for an HTML file. The server responds with headers and the HTML content. The time it takes to receive the first byte of data is called Time to First Byte (TTFB).

- The TCP slow start algorithm manages the data transfer by gradually increasing the amount of data sent until the maximum bandwidth is determined. This prevents network congestion and optimizes the use of available bandwidth.

## Parsing

- Once the browser receives the HTML data, it starts parsing, converting the data into the DOM (Document Object Model) and CSSOM (CSS Object Model).

- The DOM tree represents the structure and content of the HTML document, while the CSSOM tree represents the style rules applied to the document.

 - While parsing, the browser's preload scanner identifies and requests high-priority resources like CSS, JavaScript, and web fonts in the background. This reduces delays caused by waiting for these resources to be downloaded.

 - JavaScript compilation also takes place during parsing, where the browser converts JavaScript code into a format it can execute.

- In addition, the browser builds an accessibility tree (AOM), a semantic version of the DOM used by assistive devices.

## Rendering

- The rendering process combines the DOM and CSSOM trees into a render tree, which is used to calculate the layout and appearance of the page.

- Styling involves applying the CSSOM rules to each visible node in the DOM tree, determining the computed styles for each element.

- Layout calculates the position and dimensions of each element on the page. Subsequent recalculations are referred to as reflows.

- Painting converts the layout information into actual pixels displayed on the screen. This process is sometimes called rasterization.

- To optimize rendering, the browser can divide the page into layers that are painted independently and then composited together. This can offload some of the work to the GPU, improving performance.

## Interactivity

Time to Interactive (TTI) measures the time it takes for the page to become fully interactive, meaning it can respond to user input within 50ms.

 - A major factor affecting TTI is the execution of JavaScript, which can block the main thread and delay interactivity. Developers should optimize JavaScript code and defer its execution where possible to improve TTI.

## Reflow and Repaint Timing

- Reflow is the process of calculating the layout of elements on the page, while repaint is the process of updating the pixels displayed on the screen based on the new layout.20

- Modern browsers are highly optimized and try to minimize the number of reflows and repaints. They often batch DOM manipulations together and perform a single reflow and repaint at the end

However, certain actions, like requesting layout information (e.g., using getBoundingClientRect()), can force the browser to perform a reflow immediately, even if it was planning to batch updates.

It's important to note that the specific implementation details of reflow and repaint handling can vary between different browsers. However, the general principles of minimizing reflows and repaints apply across all modern browsers.
# References

https://developer.mozilla.org/en-US/docs/Web/Performance/How_browsers_work

https://stackoverflow.com/questions/73736185/when-exactly-does-the-browser-repaint-and-reflow#:~:text=This%20process%20is%20sometimes%20also,a%20reflow%20was%20triggered%20previously.