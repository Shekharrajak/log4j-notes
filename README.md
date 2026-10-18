# Log4j Notes

Personal source-backed Log4j learning notes, built with Zensical.

## Preview

Install Python 3.12 and uv, then run:

```sh
uv sync --frozen
uv run zensical serve
```

Open the local address printed by the server. Build with:

```sh
uv run zensical build --strict
```

The generated site is in `site/`. Commit Markdown, configuration, and `uv.lock`; generated HTML stays ignored.

## Publishing

The check workflow validates pull requests and pushes to main. The deployment workflow builds and uploads a Pages artifact on main pushes or manual dispatch, then deploys it through the `github-pages` environment.

To activate publishing, connect this repository to GitHub, enable Pages with **GitHub Actions** as its source, and set `site_url` in `zensical.toml` to the actual Pages URL. No remote or public site is configured yet. Deployments are gated on that setting.

## Authoring

Use relative links between notes and commit-pinned GitHub links for source code. Keep each page focused on one question, include a concrete example, and finish with recall questions. Use ASCII arrows in prose and native Mermaid syntax in diagrams. Record the source revision when updating implementation details.

The initial five pages cover the roadmap, mental model, public API, SLF4J, and a synchronous logging call. Java examples and referenced upstream tests have not been executed as part of this notes project.

Bundled Ubuntu fonts retain their license in `docs/assets/fonts/Ubuntu-LICENSE.txt`.
