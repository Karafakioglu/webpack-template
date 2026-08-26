# webpack-template

A ready-to-use webpack setup for vanilla HTML/CSS/JavaScript projects. Marked as a
GitHub template repository so new projects don't need webpack configured from
scratch every time.

## Starting a new project

1. On GitHub, click "New repository" and pick `webpack-template` from the
   **Repository template** dropdown at the top.
2. Clone the new repo.
3. Run `npm install` — `node_modules` isn't committed, it gets rebuilt from
   `package-lock.json`.
4. Update the `name`, `description`, `repository`, `homepage` and `bugs` fields in
   `package.json` for the new project.
5. Run `npm run dev` to start.

## Commands

| Command | What it does |
|---|---|
| `npm run dev` | Starts the dev server using `webpack.dev.js` (development mode, live reload) |
| `npm run build` | Builds into `dist/` using `webpack.prod.js` (production mode, minified) |
| `npm run deploy` | Pushes the `dist/` folder to the `gh-pages` branch |

Order matters when deploying: run `npm run build` first, commit `dist`, then run
`npm run deploy`. Deploying without building publishes the previous output.

## File structure

```
├── src/
│   ├── index.js        entry point
│   ├── template.html   the template HtmlWebpackPlugin builds from
│   └── style.css       imported from index.js
├── dist/               build output (committed, needed for gh-pages)
├── webpack.common.js   settings shared by both environments
├── webpack.dev.js      development-only settings
├── webpack.prod.js     production-only settings
└── .gitignore          node_modules only
```

`dist` is deliberately **not** in `.gitignore`. The `git subtree push --prefix dist`
command only works if git is tracking the `dist` folder.

## Why three config files?

Development and production want different settings: strong source maps and a dev
server in development, minified output and a lighter source map in production. With
a single config file, switching modes would mean editing that file by hand every
time.

The `merge()` function from `webpack-merge` combines two config objects into the
single config webpack receives. It isn't a plain spread (`{...common, ...dev}`) —
arrays get concatenated rather than overwritten. That's why declaring `plugins` in
`webpack.dev.js` doesn't wipe out the plugins from `webpack.common.js`.

How it's split:

- **common** — `entry`, `output`, `plugins`, `module.rules`
- **dev** — `mode: "development"`, `devtool: "eval-source-map"`, `devServer`
- **prod** — `mode: "production"`, `devtool: "source-map"`

## Installed loaders and what they do

| Package | Job |
|---|---|
| `html-webpack-plugin` | Generates `dist/index.html` from `src/template.html` and injects the bundle's script tag |
| `style-loader` + `css-loader` | Make `import "./style.css"` work from JS and inject the CSS into the page |
| `html-loader` | Lets webpack follow `<img src="...">` paths inside HTML |
| `webpack-dev-server` | Local server with live reload |
| `webpack-merge` | Merges the config files |

Images don't need a separate loader — webpack 5's built-in `asset/resource` type
handles them. To use one from JS:

```javascript
import logo from "./logo.png";
img.src = logo;
```

## How it was set up from scratch

In case the template needs updating or rebuilding:

```bash
npm init -y --init-type=module        # package.json with "type": "module"
npm install --save-dev webpack webpack-cli
npm install --save-dev html-webpack-plugin
npm install --save-dev style-loader css-loader
npm install --save-dev html-loader
npm install --save-dev webpack-dev-server
npm install --save-dev webpack-merge

mkdir src
touch src/index.js src/template.html src/style.css
touch webpack.common.js webpack.dev.js webpack.prod.js
touch .gitignore                       # contents: node_modules
```

Then add the `build`, `dev` and `deploy` scripts to `package.json`.

To make a repo a template: GitHub → repo Settings → tick the **Template repository**
checkbox, just below the rename field.

## Notes

- Uses `path.resolve(import.meta.dirname, "dist")`. The `fileURLToPath(import.meta.url)`
  setup in webpack's official docs does the same thing but takes more lines;
  `import.meta.dirname` landed in Node 20.11.
- The `"type": "module"` field in `package.json` is why the config files use
  `import`/`export`. Removing it means switching back to `require`.