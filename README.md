# c00l-g-mes

A minimal, configurable Vercel front door for validating alternate hosting and network-access paths without copying the upstream application's source into this repository.

## How it works

Every request is reverse-proxied by Vercel to a single deployment-time upstream origin while the Vercel hostname remains in the browser:

```text
browser
  -> <vercel-project>.vercel.app/<path>
  -> Vercel external route
  -> UPSTREAM_ORIGIN/<path>
```

The upstream is controlled only by the Vercel project environment variable `UPSTREAM_ORIGIN`. It is not accepted from request data, so this repository must not be extended into an arbitrary/open proxy.

The alternate surface emits:

```text
X-Robots-Tag: noindex, nofollow
```

so it is not intended to become an indexable or canonical product host.

## Vercel setup

1. Connect this repository to the intended Vercel project.
2. In **Project Settings -> Environment Variables**, add `UPSTREAM_ORIGIN` for the environments that should proxy traffic.
3. Set it to an absolute HTTPS origin with no trailing slash, for example:

   ```text
   https://example.com
   ```

4. Redeploy after changing the environment variable.

No application source, credentials, or upstream-specific configuration should be committed here.

## Validation

After deployment, verify:

- `/` and representative nested routes render through the Vercel hostname;
- browser refreshes on nested routes still work;
- query strings are preserved;
- static and lazy-loaded assets remain on the Vercel hostname where expected;
- responses include `X-Robots-Tag: noindex, nofollow`;
- browser Network inspection identifies any third-party or absolute-origin requests that intentionally bypass this proxy.

This repository is an alternate access-path/diagnostic surface. The configured upstream remains the source of application behavior and content.
