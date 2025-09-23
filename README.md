# vite-vanilla-prerenderer

Vite plugin that prerenders HTML at build time and during development. Ideal for vanilla JS projects or static sites where you need to inject dynamically generated content (e.g., from JSON files) directly into your HTML without a client-side framework.

This allows you to maintain clean data files, generate complex HTML structures, and have them server-rendered into your static files for improved SEO and performance.

## Features

-   **Server-Side Rendering**: Injects HTML into your .html files before they are served by Vite's dev server or written to the build output.
-   **Data-Driven Content**: Use data from local .json or .js modules to dynamically generate content.
-   **Virtual Module Compatibility**: Module imports have access to Vite's virtual module pipeline.
-   **Live Reloading**: Watches your data files and automatically restarts the dev server on change.
-   **Route Specific**: Apply rendering logic to all pages or isolate it to specific HTML files.
-   **Zero Client-Side Overhead**: All operations are done at build time or on the server; no client-side JavaScript is added.

## Installation

```
# Using npm
npm install vite-vanilla-prerenderer --save-dev
```

```
# Using yarn
yarn add vite-vanilla-prerenderer --dev
```

```
# Using pnpm
pnpm add -D vite-vanilla-prerenderer
```

## Basic Usage

1. Add the plugin to your `vite.config.js` or `vite.config.ts`.

2. Configure a `moduleGroups` array with a selector and a render function.

`vite.config.js`

```
import { defineConfig } from 'vite';
import viteVanillaPrerenderer from 'vite-vanilla-prerenderer';

export default defineConfig({
  plugins: [
    viteVanillaPrerenderer({
      moduleGroups: [
        {
          // A CSS selector for the target element.
          selector: '#copyright-year',
          // A function that returns the HTML string to inject.
          renderFunction: () => {
            const year = new Date().getFullYear();
            return `<p>Copyright © ${year} My Company</p>`;
          },
        },
      ],
    }),
  ],
});
```

`index.html`

Your target element in the HTML file will be replaced.

```
<body>
  <!-- This div will be replaced -->
  <div id="copyright-year"></div>
</body>
```

**Output**

The final HTML will have the prerendered content.

```
<body>
  <div id="copyright-year"><p>Copyright © 2025 My Awesome Company</p></div>
</body>
```

## Usage with Data Modules

The real power of this plugin comes from using external data to generate your HTML.

1. Create a data file (e.g., `src/data/projects.json`).

2. Update your `vite.config.js` to include the `dataModules` option. The plugin will load this data and pass it to your `renderFunction`.

`src/data/projects.json`

```
[
  { "id": 1, "title": "Project Alpha", "description": "A revolutionary new tool." },
  { "id": 2, "title": "Project Beta", "description": "The next generation of awesome." }
]
```

`vite.config.js`

```
import { defineConfig } from 'vite';
import viteVanillaPrerenderer from 'vite-vanilla-prerenderer';

// Define the function that will render your data
function renderProjects(data) {
  // The key 'projects' matches the filename 'projects.json'
  const projects = data.projects || [];
  return projects.map(project => `
    <div class="project-card">
      <h3>${project.title}</h3>
      <p>${project.description}</p>
    </div>
  `).join('');
}

export default defineConfig({
  plugins: [
    viteVanillaPrerenderer({
      moduleGroups: [
        {
          selector: '#project-list',
          renderFunction: renderProjects,
          // Path to the data file
          dataModules: ['./src/data/projects.json'],
        },
      ],
    }),
  ],
});
```

`index.html`

```
<body>
  <h1>My Projects</h1>
  <div id="project-list">
    <!-- This content will be replaced by the generated list -->
    <p>Loading projects...</p>
  </div>
</body>
```

## Configuration Options

The plugin is configured with a single options object.

`viteVanillaPrerenderer(options)`

`options.moduleGroups`

**Type**: `Array<Object>`
**Required**

An array of module group objects. Each object defines a distinct prerendering task.

| **Option**       | **Type**                   | **Required** | **Description**                                                                                                                                                   |
| ---------------- | -------------------------- | ------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `renderFunction` | `(data: Object) => string` | **Yes**      | A function that receives loaded data and returns an HTML string. The data object's keys are derived from the filenames in dataModules.                            |
| `selector`       | `string`                   | **Yes**      | The CSS query selector to identify the target DOM element(s) to be replaced.                                                                                      |
| `dataModules`    | `string[]`                 | **No**       | A path or array of paths to data modules (.json, .js). The default export of each module will be passed to renderFunction.                                        |
| `pathIsolate`    | `string[]`                 | **No**       | An array of file paths to exclusively apply this render rule to. The root directory is the same as your Vite project root (e.g., /index.html, /pages/about.html). |
| `pathIgnore`     | `string[]`                 | **No**       | An array of file paths to ignore for this render rule.                                                                                                            |
| `outer`          | `boolean`                  | **No**       | If true, replaces the element's outerHTML. If false (default), it replaces the innerHTML.                                                                         |

## License

This project is licensed under the MIT License.
