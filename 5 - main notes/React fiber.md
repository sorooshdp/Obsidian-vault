2025/08/06  -  21:21
status: 

Tags: [[react]] [[Performance]]

React Fiber is a complete rewrite of React's core reconciler, which has been the default reconciler since React version 16. It was developed to fix long-standing issues and provide new opportunities for future features.

• Definition and Core Idea:
    ◦ A Fiber in React is a plain JavaScript object with properties.
    ◦ The core underlying idea is that a Fiber represents a unit of work.
    ◦ React processes these Fibers, resulting in `finishedWork`, which is then "committed" to make visible changes in the DOM.
• Main Goals and Benefits:
    ◦ Fiber primarily focuses on animations and responsiveness.
    ◦ It allows React to split work into chunks and prioritize tasks.
    ◦ Unlike the old synchronous Stack reconciler, Fiber is asynchronous, enabling it to pause, resume, reuse, or abort work as needed. This helps prevent delays, for example, when typing into a text field while a background network request renders elements.
    ◦ While it brings significant performance increases, its main purpose is to improve the fundamental way React works, making it smarter and significantly easier to add new features.
• Two Phases of Work: React processes Fibers in two distinct phases:
    ◦ Render Phase:
        ▪ This phase is asynchronous and involves processing Fibers behind the scenes, not visible to the user.
        ▪ During this phase, React can prioritize tasks, pause work, or even discard it.
        ▪ Internal functions like `beginWork()` and `completeWork()` are called to process Fibers.
    ◦ Commit Phase:
        ▪ This phase is synchronous and cannot be interrupted.
        ▪ The `commitWork()` function is called, and this is where changes are reflected on the screen, specifically by applying "Effects".
• Fiber Properties and Relationships:
    ◦ A Fiber has a 1 to 1 relationship with "something", which can be a React component instance, a DOM node, or other entities.
    ◦ The tag property stores the type of this "something" (e.g., `FunctionComponent`, `ClassComponent`, `HostComponent` for div or paragraph in web).
    ◦ The `stateNode` property holds the reference to the "thing" itself, allowing React to access associated state.
    ◦ Fibers are often created from React elements and share properties like type and key. However, a crucial difference is that while React elements are recreated every time, Fibers are reused as often as possible after their initial mount.
    ◦ Fibers form a tree structure, similar to React elements, with relationships defined by three properties:
        ▪ child: Reference to the first child Fiber (e.g., h1 Fiber for a div Fiber).
        ▪ sibling: Reference to the next sibling Fiber (e.g., h2 Fiber is sibling of h1 Fiber).
        ▪ return: Reference to the parent Fiber (e.g., div Fiber is the parent of heading Fibers).
• Concept of "Work" and Time Slicing:
    ◦ Work includes changing state, calling lifecycle functions, or updates leading to DOM changes.
    ◦ React can split work into chunks using a feature called time slicing.
    ◦ It uses `requestAnimationFrame()` for high-priority work (like animations) and `requestIdleCallback()` for low-priority work (like network requests), scheduling them to be called during appropriate browser periods.
• Two Fiber Trees (Double Buffering):
    ◦ React maintains two Fiber trees: the current tree (what's currently on the screen) and the `workInProgress` tree (where changes are made).
    ◦ Changes are made to the `workInProgress` tree, and upon completion, pointers are swapped, making the `workInProgress` tree the new current tree. This is analogous to double buffering in video games, preventing inconsistent UI.
    ◦ The alternate property on a Fiber points to its counterpart in the other tree, facilitating the reuse of Fibers.
• Effects:
    ◦ Effects are activities like mutating the DOM or calling specific lifecycle methods (side effects).
    ◦ They cannot be executed during the asynchronous render phase because discarding work in that phase would be problematic if DOM changes or lifecycle methods were already applied.
    ◦ The result of the render phase is not just a new Fiber tree but also a list of Effects.
    ◦ Effects are applied synchronously during the commit phase in a single pass, making changes visible to the user. Examples include adding/updating/removing DOM elements for host components or calling `componentDidMount()`/`componentDidUpdate()` for class components.
• Fiber Tree Processing (Work Loop):
    ◦ React processes a Fiber tree by first going downwards (calling `beginWork()`) until it reaches a Fiber without children, then it goes upwards (calling `completeWork()`), completing parent Fibers.
    ◦ This entire process is not recursive but is done using a while loop, known as the Work loop, enabled by the child, sibling, and return tree architecture.
    ◦ Once the root Fiber is completed, `commitWork()` is called, Effects are applied, and changes are flushed to the DOM.
• Features Enabled by Fiber:
    ◦ Fiber made Error boundaries possible, which act like catch blocks for components to handle errors gracefully.
    ◦ It also allows for future features like Suspense and Concurrent Mode, aiming to make interactions like typing in inputs consistently smooth.
# References
