# losa.fr

Source for <https://www.losa.fr/>, built with [Zola](https://www.getzola.org/).

## Local development

```sh
zola serve
```

## Build

```sh
zola build
```

The generated site is written to `public/`.

## Deployment

Pushes to `main` and manual workflow runs build the site and deploy `public/` over FTPS.
Configure these repository secrets in GitHub:

- `FTP_SERVER`
- `FTP_USERNAME`
- `FTP_PASSWORD`
- `FTP_SERVER_DIR`

## Third-party assets

The inline link icons in `templates/macros/icons.html` use SVG path data from [Lucide Icons](https://lucide.dev/). See `LICENSES.md`.
