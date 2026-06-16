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

Pushes to `main` and manual workflow runs build the site and deploy `public/` over SFTP. The deploy step times out after 15 minutes.
The workflow runs Zola through the official `ghcr.io/getzola/zola` container image, pinned by digest.
After upload, the workflow checks `https://www.losa.fr/deploy-check.txt` to confirm that OVH is serving the commit that was just deployed.
Configure these repository secrets in GitHub:

- `FTP_SERVER`: SFTP hostname, with or without an `sftp://` prefix
- `FTP_USERNAME`
- `FTP_PASSWORD`
- `FTP_SERVER_DIR`: remote directory to publish into. For OVH shared hosting this is commonly `www/`; use the directory that contains the site's web root for your account. This value must be a bare path without quotes or spaces.
- `SFTP_KNOWN_HOSTS` (optional): known_hosts entry for the SFTP server. If omitted, the workflow accepts the server key on first connection.

## Third-party assets

The inline link icons in `templates/macros/icons.html` use SVG path data from [Lucide Icons](https://lucide.dev/). See `LICENSES.md`.

## License and citation

Unless otherwise noted, content is licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) and code is licensed under the [Apache License 2.0](LICENSE-APACHE-2.0). See [LICENSE.md](LICENSE.md) for the file-type split.

If you use or cite material from this repository, please use the metadata in [CITATION.cff](CITATION.cff).
