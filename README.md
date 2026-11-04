# Node.js (Nuxt) service for Kubernetes on Wodby

Build and run Nuxt applications on Kubernetes with Wodby.

This repository defines the Wodby service manifests and operational
configuration for Node.js (Nuxt).

- [Browse Wodby services](https://wodby.com/services)
- [Wodby service documentation](https://wodby.com/docs/2.0/services/)
- [Service manifest reference](https://wodby.com/docs/2.0/services/template/)

## Start with a boilerplate

Use one of the boilerplates exposed by this service to start with compatible
build configuration and Wodby CI:

- [Nuxt Boilerplate](https://github.com/wodby/nuxt-boilerplate)

## Wodby stacks using this service

- [Nuxt application stack](https://github.com/wodby/stack-nuxt)

## Service overview

| Property | Manifest configuration |
| --- | --- |
| Service name | `nuxt` |
| Type | Application service |
| Inherits from | [`node`](https://github.com/wodby/service-node) with version constraint `^2.0.0` |
| Application build | Git source connection enabled; Dockerfile: `Dockerfile`; boilerplates: Nuxt Boilerplate |

## Use this service

Use this service through [Nuxt application stack](https://github.com/wodby/stack-nuxt), or reference
`nuxt` from a custom Wodby stack.

A service is a reusable component and does not deploy by itself. The stack
defines its links, settings, versions, resources, and relationship to the rest
of the application.

The image build runs `npm run build` and the container starts with
`npm run start`. Applications use Nuxt's default Node.js server output:
`nuxt build` and `node .output/server/index.mjs`. Configure a built
application with `runtimeConfig` and `NUXT_*` variables at runtime. The
service generates `NUXT_APP_SECRET`, Nuxt's application secret, for each
environment.

## Maintain a custom version

1. Fork this repository.
2. Edit the service manifest and referenced files.
3. Import the repository as a [Git-backed service](https://wodby.com/docs/2.0/services/create/#create-a-git-backed-service).
4. Reference the service from a stack manifest.

Keep service, workload, container, endpoint, link, volume, config, and
derivative names stable unless dependent stacks and app-level overrides are
updated at the same time.

Validate the manifests with:

```bash
wodby service validate-manifest service.yml --org <org-id> --include GUIDANCE.md
```

See the [service manifest reference](https://wodby.com/docs/2.0/services/template/) and the [managed services index](https://github.com/wodby/services).

## Workspace startup

Workspaces start Nuxt's development server directly:
`node node_modules/nuxt/bin/nuxt.mjs dev --host "$HOST" --port "$PORT" --strictPort`.
This command does not run custom `dev`, `predev` or `postdev` scripts.
Override `WORKSPACE_NODE_COMMAND` when those scripts are required. Nuxt's,
Nitro's and Vite's file watchers poll shared storage through
`CHOKIDAR_USEPOLLING`, which the workspace sets. Add the environment's hosts to
`vite.server.allowedHosts` in `nuxt.config.ts`. Standard build/start behavior
is unchanged. Requires a Node image with workspace contract 1.
