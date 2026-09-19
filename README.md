# Hister on VibeNest

One-click, private-by-default deployment of [Hister](https://github.com/asciimoo/hister), a self-hosted search engine for pages you visit and files you choose to index.

This repository is a thin deployment adapter. It does not modify Hister. The container is pinned to upstream release `v0.17.0` and its multi-architecture OCI index digest so a future upstream tag change cannot silently alter a deployment.

## Deploy

Deploy this repository with Docker Compose. VibeNest fills both variables from `.env.example` automatically:

- `APP_URL`: the final HTTPS origin of the deployment.
- `ADMIN_PASSWORD`: a generated high-entropy access token. Keep it in a password manager.

Open the generated URL and enter `ADMIN_PASSWORD` on Hister's sign-in screen. The instance stays private: public mode and multi-user registration are both disabled.

## Browser extension

Install the official Hister extension from the links in the [upstream browser-extension guide](https://github.com/asciimoo/hister/blob/v0.17.0/webui/website/src/content/docs/browser-extension.md). In the extension settings:

1. Set **Server URL** to the exact HTTPS URL of your VibeNest deployment.
2. Sign in to the Hister web UI with `ADMIN_PASSWORD`.
3. Use **Authenticate Extension** in Hister so the extension receives the scoped session cookie.

Page content is sent to the configured Hister server, not to a Hister cloud service. Review the upstream guide before enabling capture on sensitive sites.

## Free-tier profile

The adapter is intentionally conservative for VibeNest Free (256 MB RAM, 0.5 vCPU, 4 GB storage):

- language detection is disabled because upstream documents large transient memory spikes for multilingual indexes;
- semantic search is disabled because it needs an external embeddings endpoint and adds background work;
- the bundled `yt-dlp` extractor remains disabled by Hister's default;
- the container runs as upstream's non-root user with a read-only root filesystem, all Linux capabilities dropped, and `no-new-privileges`;
- only `/hister/data` is persistent; `/tmp` is a bounded 64 MB tmpfs.

Upstream estimates roughly 80–100 MB of index storage per 1,000 saved web pages, so 4 GB is suitable for a personal history, not an unlimited team archive. Index growth depends on page size and imported files. Back up the `hister_data` volume before major changes.

## Persistence

Hister's SQLite database, full-text indexes, configuration, and session key live in the named `hister_data` volume. Redeploying or recreating the container must keep that volume. Deleting the volume permanently deletes the Hister data.

## Updates and source

The pinned image is built and published by the Hister project. To update, change both the version and digest only after reviewing upstream release notes and repeating the Free-tier resource and persistence tests.

- Upstream source: <https://github.com/asciimoo/hister>
- Pinned source tag: <https://github.com/asciimoo/hister/tree/v0.17.0>
- Upstream license: GNU AGPL v3 or later

The copied `LICENSE` is the upstream AGPL license text. Hister remains copyrighted by its upstream contributors. Adapter configuration changes are published in this repository so users can reproduce exactly what is deployed.
