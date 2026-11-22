---
created: 2026/09/12 20:18
updated: 2026-11-09 20:49
type: note
status: seed
topic:
tags:
  - "#performance"
  - "#browser"
aliases: []
---

### **1. Core Problem: The Main Thread Bottleneck**

- **Single-Threaded Constraint**: JavaScript runs on a single-threaded event loop where code execution, event handling, DOM manipulation, style calculations, layout (reflow), and paint operations all line up on the same main thread.
- **Frame Budget**: For smooth 60fps rendering, frames must be drawn every 16.6ms. Subtracting browser overhead leaves a practical developer budget of **~10ms per frame**.
- **Long Tasks & Jank**: Tasks taking longer than **50ms** block the main thread, causing frozen UI, delayed input handling, and stuttering animations ("jank").
- **Performance Metrics**: Core metrics like **[[Interaction to Next Paint (INP) | INP]]** (Interaction to Next Paint) and **TBT** (Total Blocking Time) explicitly measure how long the main thread remains blocked.

---

### **2. Family 1: Using the Main Thread Wisely (Intelligent Scheduling)**

#### **A. Splitting (Task Sizing)**

- **Concept**: Divide long synchronous tasks into smaller chunks and yield control back to the browser between chunks so rendering and user inputs can execute in the gaps. Yielding does not reduce overall computation time, but dramatically improves perceived responsiveness.
- **Yield Mechanisms**:
    - `setTimeout(resolve, 0)`: Yields control to the next task queue, though nested calls incur a minimum ~4ms delay overhead.
    - `MessageChannel` / `scheduler.yield()`: Schedules resumption ahead of lower-priority queued tasks without the `setTimeout` delay penalty.
    - `requestAnimationFrame` (rAF): Ideal when splitting work by time (e.g., budgeting ~5ms of work per frame) to sync execution with screen refreshes.

#### **B. Batching (Folding Fixed Costs)**

- **Concept**: Combine frequent, small operations into a single batch to avoid paying pipeline fixed costs repeatedly.
- **Event Handling**: Use **debounce** (delay execution until input pauses) or **throttle** (limit execution frequency) for high-frequency events like typing, scrolling, or resizing.
- **DOM Writes**: Batch DOM updates together or draw once per frame using requestAnimationFrame (see [[The heart of React]] and [[React fiber]]) rather than re-rendering on every incoming data tick.

#### **C. Prioritizing (Task Ordering)**

- **Concept**: Order tasks dynamically based on user urgency using a priority queue.
- **Idle-Until-Urgent Pattern**: Process background work sequentially during idle time, but bump a specific job to the front of the queue immediately if the user interacts with it.

#### **D. Deferring (Timing Management)**

- **Code Splitting**: Download and execute only the code required for the current screen during initial page load.
- **Off-Screen Deferral**: Use `IntersectionObserver` to render DOM elements (like long feed items) only when they approach the viewport.
- **Off-Screen Pause**: Stop continuously running work (such as carousels or live charts) whenever they scroll out of view.

---

### **3. Family 2: Not Using the Main Thread (Offloading & Eliminating Work)**

#### **A. Moving Work to the Compositor Thread**

- **Compositor Animations**: Properties like `transform` and `opacity` are handled directly on the compositor thread without triggering layout or paint on the main thread, keeping animations smooth even under heavy JS load.
- **FLIP Technique** (_First, Last, Invert, Play_): Force a single layout change up front, then let `transform` animate the movement on the compositor thread.
- **Avoid Layout Thrashing**: Separate and group DOM reads (`getBoundingClientRect`, `offsetWidth`) before DOM writes (`style.width`) to prevent repeated mid-frame reflows.

#### **B. Sending Work to Web Workers**

- **Off-Thread Computation**: Delegate heavy CPU-bound tasks with no DOM dependency (e.g., large JSON parsing, image processing like seam carving) to Web Worker threads.
- **Transferable Objects**: Use `ArrayBuffer` objects with `postMessage` to transfer memory ownership instantly without serialization copy overhead.

#### **C. Eliminating Work Altogether**

- **Dropping**: Discard obsolete backlogged data when inflow exceeds processing throughput (e.g., live logs).
- **Merging**: Collapse backlogged updates to apply only the final state (e.g., live ranking boards).
- **Skipping (Memoization)**: Cache and reuse results for identical inputs to avoid redundant calculations.

---


# References
https://kciter.so/posts/the-expensive-main-thread/en/