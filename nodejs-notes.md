# Node.js — Notes

## 1. What is Node.js?

Node.js is a **runtime environment** that lets you run JavaScript **outside the browser** — most commonly on a server.

Before Node.js, JavaScript could only run inside browsers (to make webpages interactive). Node.js took Chrome's V8 JavaScript engine and embedded it in a standalone program, adding extra capabilities (file system access, networking, etc.) that browsers don't allow for security reasons.

This means you can now use JavaScript to:
- Build backend servers and APIs
- Read/write files
- Interact with databases
- Build CLI tools

---

## 2. Node.js Runtime

The "runtime" is the environment that executes your JS code and provides extra built-in functionality beyond the language itself.

Node.js runtime = **V8 engine** (executes JS) + **libuv** (handles async I/O, the event loop, file system, networking) + **built-in Node APIs** (`fs`, `http`, `path`, etc.)

Key point: Node.js is **single-threaded** for executing your code, but it can handle many operations concurrently thanks to its non-blocking, event-driven design (more on this in the Event Loop section).

---

## 3. npm (Node Package Manager)

npm is the default **package manager** for Node.js — used to install, share, and manage reusable code (packages/libraries).

Common commands:
```bash
npm init            # create a new package.json
npm install express  # install a package (and add to package.json)
npm install          # install all dependencies listed in package.json
npm uninstall express
npm run <script>    # run a custom script defined in package.json
```

Installed packages go into a `node_modules` folder, and exact versions are locked in `package-lock.json`.

---

## 4. `package.json`

The **manifest file** for a Node.js project — describes the project and its dependencies.

```json
{
  "name": "my-app",
  "version": "1.0.0",
  "main": "index.js",
  "type": "commonjs",
  "scripts": {
    "start": "node index.js",
    "dev": "nodemon index.js"
  },
  "dependencies": {
    "express": "^4.19.2"
  },
  "devDependencies": {
    "nodemon": "^3.1.0"
  }
}
```

Key fields:
- `dependencies` — packages needed to run the app
- `devDependencies` — packages only needed during development (testing, linting, etc.)
- `scripts` — shortcuts you run via `npm run <name>`
- `main` — entry point of the app
- `type` — `"commonjs"` (default) or `"module"` (enables ES Modules)

---

## 5. CommonJS vs ES Modules

Two different systems for organizing code into reusable files ("modules") in Node.js.

| | CommonJS (CJS) | ES Modules (ESM) |
|---|---|---|
| Import | `require('module')` | `import module from 'module'` |
| Export | `module.exports = ...` | `export default ...` / `export const ...` |
| Loading | Synchronous | Asynchronous (supports top-level `await`) |
| Default in Node | Yes (unless configured otherwise) | Needs `"type": "module"` in `package.json` or `.mjs` extension |
| File extension | `.js` (default) or `.cjs` | `.mjs`, or `.js` with `"type": "module"` |

CommonJS is Node's original module system. ES Modules is the standard JavaScript module system (same one browsers use), and Node has added support for it over time.

---

## 6. `require()` vs `import`

**CommonJS (`require`)**
```js
// exporting (math.js)
function add(a, b) { return a + b; }
module.exports = { add };

// importing (index.js)
const { add } = require('./math');
```

**ES Modules (`import`)**
```js
// exporting (math.js)
export function add(a, b) { return a + b; }

// importing (index.js)
import { add } from './math.js';
```

You generally shouldn't mix the two systems in the same file. Modern projects increasingly favor ES Modules since it's the JavaScript standard.

---

## 7. Built-in Modules

Node.js ships with built-in modules — no installation needed, just `require`/`import` them.

### `fs` (File System)
Read/write files and directories.
```js
const fs = require('fs');

fs.readFile('data.txt', 'utf8', (err, data) => {
  console.log(data);
});

fs.writeFileSync('output.txt', 'Hello World');
```

### `path`
Work with file/directory paths in a cross-platform way (handles `/` vs `\` differences between OSes).
```js
const path = require('path');

path.join(__dirname, 'files', 'data.txt');
path.extname('index.html'); // '.html'
path.basename('/user/data.txt'); // 'data.txt'
```

### `http`
Create raw web servers without any framework.
```js
const http = require('http');

const server = http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'text/plain' });
  res.end('Hello World');
});

server.listen(3000, () => console.log('Server running on port 3000'));
```

### `os`
Get information about the operating system.
```js
const os = require('os');

os.platform();   // 'linux', 'win32', 'darwin'
os.cpus();        // CPU info
os.totalmem();    // total system memory
```

---

## 8. Environment Variables

Environment variables store configuration values **outside your code** — useful for secrets (API keys, DB passwords) and settings that differ between environments (development vs production).

```js
console.log(process.env.PORT);
console.log(process.env.DATABASE_URL);
```

Usually managed via a `.env` file (not committed to version control) and loaded using a package like `dotenv`:
```js
require('dotenv').config();
```

`.env` file example:
```
PORT=3000
DATABASE_URL=postgres://localhost:5432/mydb
```

---

## 9. `process`

A global object in Node.js that gives information about, and control over, the current running program.

Common uses:
```js
process.env       // environment variables
process.argv      // command-line arguments passed to the script
process.exit(1)   // exit the program (1 = error, 0 = success)
process.cwd()     // current working directory
process.on('exit', () => console.log('Bye!')) // listen for events
```

---

## 10. Asynchronous Programming

JavaScript (and Node.js) is **single-threaded**, but many operations (reading files, network requests, database queries) take time. Instead of freezing everything while waiting, Node handles these **asynchronously** — it starts the operation, moves on to other work, and comes back once it's done.

This is essential for backend servers, which need to handle many requests at once without getting blocked by slow operations.

Three main patterns evolved to handle this: **Callbacks → Promises → async/await** (each an improvement on the last).

---

## 11. Callbacks

A callback is simply **a function passed as an argument**, to be called once an operation finishes.

```js
fs.readFile('data.txt', 'utf8', (err, data) => {
  if (err) {
    console.error(err);
    return;
  }
  console.log(data);
});
```

**Downside — "Callback Hell":** nesting many callbacks becomes hard to read/maintain.
```js
doTask1((err, result1) => {
  doTask2(result1, (err, result2) => {
    doTask3(result2, (err, result3) => {
      // deeply nested, hard to follow
    });
  });
});
```

---

## 12. Promises

A **Promise** represents a value that will be available in the future — either resolved (success) or rejected (failure). Promises fix callback hell by allowing chaining.

```js
const fetchData = () => {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      const success = true;
      success ? resolve('Data loaded') : reject('Error loading data');
    }, 1000);
  });
};

fetchData()
  .then(result => console.log(result))
  .catch(error => console.error(error))
  .finally(() => console.log('Done'));
```

A promise has 3 states: **pending**, **fulfilled**, **rejected**.

---

## 13. `async/await`

Syntactic sugar built on top of Promises — lets you write asynchronous code that *looks* synchronous, making it much easier to read.

```js
async function loadData() {
  try {
    const result = await fetchData();
    console.log(result);
  } catch (error) {
    console.error(error);
  }
}
```

- `async` marks a function as asynchronous (it always returns a Promise).
- `await` pauses execution *within that function* until the Promise resolves — without blocking the rest of the program.
- Use `try/catch` to handle errors (replaces `.catch()`).

This is the standard, modern way to write async code in Node.js.

---

## 14. Event Loop

The event loop is the mechanism that lets Node.js perform **non-blocking, asynchronous operations** despite JavaScript being single-threaded.

**Simplified flow:**
1. Your code runs on the **main thread** (call stack).
2. When an async operation is triggered (file read, timer, network call), Node hands it off to the system (via libuv) and continues running other code.
3. Once that operation finishes, its callback is placed in a **queue**.
4. The event loop constantly checks: *"Is the call stack empty? If so, take the next item from the queue and run it."*

**Simple illustration:**
```js
console.log('1: Start');

setTimeout(() => {
  console.log('2: Timeout callback');
}, 0);

Promise.resolve().then(() => {
  console.log('3: Promise callback');
});

console.log('4: End');

// Output order: 1, 4, 3, 2
```

Why this order?
- `1` and `4` run immediately (synchronous code).
- Promises (`3`) are handled in the **microtask queue** — runs before...
- Timers/callbacks (`2`) which are handled in the **macrotask queue**.

This is *why* Node.js can handle thousands of concurrent connections efficiently with a single thread — it's not doing multiple things literally at once, it's just never sitting idle waiting for slow I/O.

---

## Quick Recap

- **Node.js** = JavaScript runtime for running JS outside the browser (servers, tools, scripts)
- **npm** = package manager; **`package.json`** = project manifest
- **CommonJS vs ESM** = two module systems (`require` vs `import`)
- **Built-in modules** (`fs`, `path`, `http`, `os`) = core capabilities with no install needed
- **Environment variables / `process`** = configuration and runtime info
- **Async programming** evolved: **Callbacks → Promises → async/await**
- **Event loop** = the mechanism enabling non-blocking concurrency on a single thread
