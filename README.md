# PERA — Skills as Plug-ins.

Public research website for PERA: incremental skill learning for humanoid robots.

**Website:** https://shangchaohu718.github.io/pera-project/

This repository contains the editable website, selected simulation recordings,
posters, research figures and media provenance. Training code, model checkpoints,
private experiment directories and the unreleased manuscript are not included.
**Paper:** https://arxiv.org/html/2611.16188v1 . The research-code link currently
points to the separate private project repository, pending the public release.

## Edit locally

Requires Node.js 22 and npm.

```sh
cd website
npm ci
npm run dev
```

Edit `website/src/` for page content and styles. Web assets are under
`website/public/`. Rendering scripts require the separate robotics checkout;
editing and building the website do not.

## Publish updates

Commit and push changes to `main`. The **Deploy PERA research website** workflow
builds the static website and publishes it to GitHub Pages automatically. It can
also be started manually from the Actions tab. In repository Settings → Pages,
the deployment source is **GitHub Actions**.

To check a production build locally:

```sh
cd website
PAGES_BASE_PATH=/pera-project/ npm run build
npm run preview
```

Keep web videos as ordinary Git files, as configured by
`website/.gitattributes`. Do not commit `node_modules`, build output, local QA,
render masters or export ZIPs. Third-party notices are in `website/licenses/`.
