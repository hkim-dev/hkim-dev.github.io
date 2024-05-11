---
title: "Migration Guide to Vite from CRA"
date: 2024-05-10 15:00:00 -0400
categories: "frontend"
header:
  teaser: "/assets/images/vite.png"
tags:
  - React
  - Vite
toc: true
---

![vite](/assets/images/vite.png)

# Introduction

Create React App (CRA) has long served as a reliable starting point for React projects. While initiating a React project from scratch is possible, the React team recommends leveraging a framework for efficiency. However, as CRA is no longer actively maintained, developers are seeking modern alternatives. Enter Vite - not a conventional framework like Next.js or Remix, but a powerful build tool offering significant advantages over CRA. Unlike CRA, which relies on Webpack, Vite utilizes esbuild, a lightning-fast JavaScript bundler. This fundamental difference translates into remarkable speed gains during development. In this article, I'll guide you through the process of migrating your project from Create React App to Vite.

# Migration Steps

1. First, remove CRA from your project.
  ```bash
  npm uninstall react-scripts
  ```

2. Install Vite dependencies.
  ```bash
  npm install vite @vitejs/plugin-react --save-dev
  ```
  For more information on Vite plugins, visit: [Vite Plugins Documentation](https://main.vitejs.dev/plugins/).

3. Modify your `package.json` and add the following scripts.
  ```json
  {
    "scripts": {
      "dev": "vite", // start dev server, aliases: `vite dev`, `vite serve`
      "build": "vite build", // build for production
      "preview": "vite preview" // locally preview production build
    }
  }
  ```

4. Move `index.html` to the root directory.
  CRA uses `public/index.html` for the default entry point, while Vite looks for the file in the root directory. 
  ```bash
  mv public/index.html .
  ```

5. Update the script tag in your `index.html` to link the `index.tsx` file.
  ```html
  <!DOCTYPE html>
  <html lang="en">
    <head>
      <meta charset="utf-8" />
      <meta name="viewport" content="width=device-width, initial-scale=1" />
    </head>
    <body>
      <div id="root"></div>
      <script type="module" src="/src/index.tsx"></script>
    </body>
  </html>
  ```

6. Create `vite.config.ts` at the root of your project.
  ```tsx
  import { defineConfig } from 'vite'
  import react from '@vitejs/plugin-react'

  export default defineConfig({
    plugins: [react()],
    server: {    
      open: true,
      port: 3000,
    },
    build: {
      target: 'esnext',
      outDir: './dist',
      rollupOptions: {
        input: {
          main: './index.html'
        }
      }
    }
  });
  ```
  For more details about configuration options for Vite, visit: [Vite Configuration Docs](https://vitejs.dev/config/).

That’s it. With these simple steps, you have moved away from CRA to Vite. You can now start a dev server for your React app with `npm start`.



# Extras

## Install Tailwind CSS in a Vite Project

If you have been using Tailwind CSS with CRA, you can easily set up Tailwind CSS in a Vite project following these steps:

1. Install required dependencies.
  ```bash
  npm install -D tailwindcss postcss autoprefixer
  ```

2. Create `tailwind.config.js` and `postcss.config.js` with the following command.
  ```bash
  npx tailwindcss init -p
  ```

3. Add your template files to `tailwind.config.js`.
  ```bash
  /** @type {import('tailwindcss').Config} */
  export default {
    content: [
      "./index.html",
      "./src/**/*.{js,ts,jsx,tsx}",
    ],
    theme: {
      extend: {},
    },
    plugins: [],
  }
  ```

## Path Mapping

Path mapping in TypeScript allows developers to define aliases for directory paths, making it easier to reference files and directories. You can improve code readability with this feature. 

You can set up path mapping in tsconfig.json like this:

```json
{
  "compilerOptions": {
    "baseUrl": "./src",
    "paths": {
      "@components/*": ["components/*"],
      "@pages/*": ["pages/*"],
      "@utils/*": ["utils/*"]
    }
  }
} 
```

Then you can import modules using the path aliases in code files:

```json
import { Button } from '@components/Button';
import { Header } from '@components/Header';
import Home from '@pages/Home';
import About from '@pages/About';
import { fetchData } from '@utils/api';
```

More details can be found here: <https://www.typescriptlang.org/tsconfig/#paths>

To use alias imports in a Vite project, you either add the aliases to the `resolve.alias` option or simply use a plugin.

1. Add aliases to vite.config.ts.
  ```tsx
  import { defineConfig } from 'vite'
  import react from '@vitejs/plugin-react'

  export default defineConfig({
    plugins: [react()],
    resolve: {
      alias: {
        "@components": path.resolve(__dirname, "src/components"),
        "@pages": path.resolve(__dirname, "src/pages"),
        "@utils": path.resolve(__dirname, "src/utils")
      },
    },
  });
  ```

2. Or, simply use this plugin: [vite-tsconfig-paths](https://www.npmjs.com/package/vite-tsconfig-paths)
  ```tsx
  import { defineConfig } from 'vite'
  import react from '@vitejs/plugin-react'
  import tsconfigPaths from 'vite-tsconfig-paths';

  export default defineConfig({
    plugins: [react(), tsconfigPaths()],
    // other config options...
  });
  ```


##### References:
- <https://react.dev/learn/start-a-new-react-project>
- <https://www.robinwieruch.de/vite-create-react-app/>
- <https://tailwindcss.com/docs/guides/vite>
- <https://www.wolff.fun/path-mapping-typescript-vite/>
- <https://stackoverflow.com/questions/77249074/how-do-i-use-typescript-path-aliases-in-vite>