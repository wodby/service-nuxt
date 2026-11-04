# Nuxt on Wodby

What this service adds to the Node.js service it is based on.

## Build and start

The service's Dockerfile copies the build context to `/usr/src/app`, runs `npm run build` and starts the container with `npm run start`. `package.json` must define both scripts.

- The supported production output is Nitro's default `node-server` preset: build with `nuxt build` and start with `node .output/server/index.mjs`. Do not set another Nitro preset, and do not use `nuxt generate` as the build script.
- The Dockerfile does not install dependencies. The pipeline does it before the image is built (`wodby ci run -- npm ci`), and `node_modules` is copied with the code. Nuxt may stay in `dependencies` and type-checking tools in `devDependencies`. Keep the starter's `postinstall: nuxt prepare` script; it generates `.nuxt` and needs no runtime variables.
- `npm run build` runs while the image is built. Runtime variables of the service, including the link variables and secrets, are not available to it. Do not read them in `nuxt.config.ts`: values computed there are fixed at build time. Read them at request time, in server routes, server middleware or server plugins.
- The image build leaves out `.git`, `.gitignore`, `.nuxt`, `.output` and `.data`; `nuxt build` regenerates `.nuxt`.
- The application must listen on port 3000. The Nitro server listens on `PORT` (or `NITRO_PORT`) and defaults to 3000; leave both unset.
- The server stops on `SIGTERM` within its 30-second shutdown timeout, which matches the deployment's grace period.

## Runtime configuration

Nuxt's `runtimeConfig` is the supported way to configure a built application. Declare each key in `nuxt.config.ts` with an empty or default value, and set it per environment with a variable named after it: `NUXT_` plus the key path in upper snake case, for example `NUXT_API_SECRET` for `runtimeConfig.apiSecret`. Keys under `runtimeConfig.public` use `NUXT_PUBLIC_` and are sent to the browser; never put secrets there. Only declared keys are overridden, values are parsed (`true`, numbers and JSON become non-strings), and `.env` files are not read by the production server.

The link variables of the Node.js service (`DB_*`, `REDIS_*`, `SMTP_*`) have no `NUXT_` prefix, so they do not override runtime config. Read them with `process.env` inside server code.

## Variables the service sets

| Variable | Meaning |
| --- | --- |
| `NODE_ENV` | `production` in every environment type, including `dev`. The Node.js service's `development` value for `dev` is replaced, because the container runs a production build. |
| `NUXT_APP_SECRET` | A generated 64-character hex secret, stable across deployments and different for each environment. It sets Nuxt's `appSecret`, which `useSession` and `deriveSecret` from `nuxt/server`, and modules built on them, require; without it they fail at request time. Changing it invalidates existing sessions and anything signed with derived secrets. |
| `WORKSPACE_NODE_COMMAND` | The start command of a development workspace, described below. |

## Linked services

The service adds no link variables. The database, mail and Redis or Valkey variables of the Node.js service apply unchanged. Server routes read them from `process.env`; they do not reach the browser.

## In a development workspace

- The application is started with `node node_modules/nuxt/bin/nuxt.mjs dev --host "$HOST" --port "$PORT" --strictPort`, not through the `dev` script of `package.json`. Options in that script do not apply. `nuxt` must be a dependency of the project.
- The development server listens on `0.0.0.0:3000`. Do not change the port in the project configuration.
- The development server rejects requests for host names it does not know, with 403. Add the environment's hosts to `vite.server.allowedHosts` in `nuxt.config.ts`: `WODBY_HOSTS` holds them as a JSON array, and `WODBY_APP_SERVICE_NAME` is the internal name used by health checks. The boilerplate's `nuxt.config.ts` does this. Nuxt applies the list to its own endpoints, to Vite and to the hot-reload connection.
- Hot module replacement uses the same port: the browser connects to `/_nuxt/` on the page's own host, over `wss` behind HTTPS.
- The workspace sets `CHOKIDAR_USEPOLLING`, and all of Nuxt's, Nitro's and Vite's file watchers poll, so saved files, new pages and new server routes are picked up without a restart. The one exception is the watcher that restarts the development server when `nuxt.config.ts` changes; the boilerplate adds `watch: ['../nuxt.config.ts']` when polling is on, so Nuxt's polling watcher does it instead.
- `NODE_ENV` is `development` here.
- `/.nuxt/`, `/.output/` and `/.data/` are kept out of Git status, in addition to the Node.js service's paths.
- The image build's `npm run build` does not run in a workspace.
