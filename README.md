# c00l-g-mes

A minimal, configurable Vercel front door for validating alternate hosting and network-access paths without copying the upstream application's source into this repository.

## How it works

Every request is reverse-proxied by the configured Vercel project while the Vercel hostname remains in the browser:

```text
browser
  -> <vercel-project>.vercel.app/<path>
  -> Vercel external route
  -> configured upstream origin/<path>
```

The upstream is controlled by the GitHub repository variable `UPSTREAM_ORIGIN`. GitHub Actions renders `vercel.json` immediately before deployment, so the upstream URL does not need to be duplicated in Vercel project environment variables.

The destination is fixed at deployment time and cannot be supplied by a request, so this repository must not be extended into an arbitrary/open proxy.

The alternate surface emits:

```text
X-Robots-Tag: noindex, nofollow
```

so it is not intended to become an indexable or canonical product host.

## GitHub configuration

Configure these **repository variables** under **Settings -> Secrets and variables -> Actions -> Variables**:

| Name | Purpose | Example |
| --- | --- | --- |
| `UPSTREAM_ORIGIN` | HTTPS origin to proxy | `https://example.com` |
| `VERCEL_ORG_ID` | Owner/team ID for the existing Vercel project | `team_...` or Vercel account ID |
| `VERCEL_PROJECT_ID` | Existing Vercel project ID | `prj_...` |

Configure this **repository secret** under **Settings -> Secrets and variables -> Actions -> Secrets**:

| Name | Purpose |
| --- | --- |
| `VERCEL_TOKEN` | Vercel access token used only by GitHub Actions to deploy |

`UPSTREAM_ORIGIN` is configuration, not sensitive data, so it should normally be a repository variable rather than a secret.

## Deployment

A push to `main` (or a manual `workflow_dispatch`) deploys to the existing Vercel project's production target.

The workflow:

1. validates the required GitHub variables;
2. requires `UPSTREAM_ORIGIN` to be an HTTPS origin without credentials, path, query, or fragment;
3. renders `vercel.json` from `vercel.template.json`;
4. deploys the rendered configuration to the existing Vercel project using a pinned Vercel CLI.

The legacy Vercel project can therefore be reused without making Vercel the source of upstream configuration. Automatic Vercel Git deployment is not required for this repository.

## Validation

After deployment, verify:

- `/` and representative nested routes render through the Vercel hostname;
- browser refreshes on nested routes still work;
- query strings are preserved;
- static and lazy-loaded assets remain on the Vercel hostname where expected;
- responses include `X-Robots-Tag: noindex, nofollow`;
- browser Network inspection identifies any third-party or absolute-origin requests that intentionally bypass this proxy.

This repository is an alternate access-path/diagnostic surface. The configured upstream remains the source of application behavior and content.
