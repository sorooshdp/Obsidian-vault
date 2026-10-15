
# My Obsidian Knowledge Vault

> A connected set of study notes on JavaScript, browser and React internals, web performance, software design, databases, and secure federated learning.

## 📁 Structure Overview

📦 Obsidian Vault
├── 🗒️  1 - rough notes/     # Early ideas and material to develop
├── 🏷️  2 - tags/          # Tag definitions and topic hubs
├── 🧭  3 - indexes/       # Navigation and subject indexes
├── 📋  4 - templates/      # Note templates
├── 📝  5 - main notes/     # Primary content and deep dives
├── 🗃️  6 - Bases/         # Base notes or foundational content
└── ⚙️  .obsidian/         # Obsidian configuration

## 🎯 Focus Areas

### **JavaScript & Web Development**
- **JavaScript fundamentals**: Scope, closures, hoisting, `this`, coercion, modules, objects, and class behavior through the [YDKJS notes](5%20-%20main%20notes/YDKJS%20intro%20to%20js.md).
- **React internals**: [Fiber](5%20-%20main%20notes/React%20fiber.md), hooks, batching, the compiler, and the rendering model.
- **Browser performance**: Navigation, parsing, rendering, the [main thread](5%20-%20main%20notes/Browser's%20main%20thread.md), and [Core Web Vitals](5%20-%20main%20notes/Core%20Web%20Vitals.md).
- **Design patterns**: Creational, structural, and behavioral patterns, including [Factory Method](5%20-%20main%20notes/Factory%20method%20pattern.md), [Builder](5%20-%20main%20notes/Builder%20pattern.md), Singleton, Prototype, and Object Pool.

### **Computer Science, Data & Security**
- **Federated learning research**: Algorithms, client selection, personalization, privacy, poisoning, backdoors, Byzantine robustness, and defense strategies.
- **Database systems**: [SQL](5%20-%20main%20notes/SQL.md), relational data modeling, DBMS concepts, and [SQL injection](5%20-%20main%20notes/SQL%20Injection.md).
- **Security principles**: Trust boundaries, least privilege, and attacks against models and data.
- **Foundations**: Data structures, memory management, caching, I/O latency, encoding, and academic writing.

## 🔗 Key Connections

### Core JavaScript Concepts
- Scope and closures → hoisting → `this` and objects → coercion → modules
- The YDKJS notes provide the main reading path, with supporting notes on [lexical scope](5%20-%20main%20notes/YDKJS%20lexical%20scope.md), [scope and closures](5%20-%20main%20notes/YDKJS%20scope%20%26%20closures.md), and modules.

### React Deep Dives
- React Fiber → render and commit → hooks → batching and compiler behavior
- Connected to browser scheduling, the main thread, and JavaScript fundamentals

### Browser Performance
- Navigation → parsing → style and layout → paint and compositing
- Main-thread work → scheduling, batching, yielding, workers, and eliminating redundant work
- Core Web Vitals: LCP, INP, and CLS, with hydration and monitoring considerations

### Machine Learning Security
- Federated learning → attacks and vulnerabilities → aggregation and detection defenses
- Paper notes track centralized aggregation, robust and personalized frameworks, privacy, and open problems

## 🧭 Suggested Starting Points

- [JavaScript modules](5%20-%20main%20notes/JavaScript%20Modules.md) for module systems and runtime boundaries
- [The heart of React](5%20-%20main%20notes/The%20heart%20of%20React.md) for the React learning path
- [Browser Functionality and Web Performance](5%20-%20main%20notes/Browser%20Functionality%20and%20Web%20Performance.md) for the critical rendering path
- [Federated learning](5%20-%20main%20notes/Federated%20learning.md) and [FL Paper Abstracts Notes](5%20-%20main%20notes/FL%20Paper%20Abstracts%20Notes.md) for the machine learning research area

## 🚀 Usage Tips

- **Tag System**: Use the predefined tags in `2 - tags/` for consistent categorization
- **Indexes**: Check `3 - indexes/` for navigation before searching the full vault
- **Templates**: Leverage templates in `4 - templates/` for new notes
- **Base Notes**: Reference `6 - Bases/` for foundational concepts

## 🔄 Recent Focus Areas
- Main-thread scheduling, rendering performance, and Interaction to Next Paint (INP)
- React internals, compiler behavior, and concurrent work
- JavaScript language mechanics and object-oriented design tradeoffs
- Federated learning attacks, defenses, and research paper analysis

---

*This vault is continuously evolving as new knowledge is acquired and connected.*