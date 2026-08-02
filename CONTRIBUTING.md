# Contributing

Most good PRs start as a failed attempt to fool the verifier.

## Ways to Contribute

- **Find a bypass** — a file, command, or pattern that slips past a scanner. These are treated as security findings first (see [SECURITY.md](SECURITY.md)), then as fixes.
- **Improve the wrapper** — input handling, output format, SARIF quality, documentation.
- **New engine ideas** — these belong in the [core repo](https://github.com/QWED-AI/qwed-verification), not here.

## Development

This repo is intentionally thin. It declares the action contract (`action.yml`) and wraps the published QWED core image. There is no build step.

Validate the contract:

```bash
python -c "import yaml; yaml.safe_load(open('action.yml'))"
```

Test locally with Docker (the same image the action runs):

```bash
docker run --rm -e INPUT_ACTION=scan-secrets -e INPUT_PATHS="**/*.env" \
  -v "$PWD:/github/workspace" -w /github/workspace \
  qwedai/qwed-verification:latest
```

## Release Process

1. Tag a version: `git tag v1.2.0 && git push origin v1.2.0`
2. The [release workflow](.github/workflows/release.yml) validates `action.yml`, moves the `v1` major tag, and creates a GitHub Release.
3. Marketplace listing updates from the release.

## Code of Conduct

Be respectful. This is a security project — reviewers may be rigorous about edge cases. That rigor is the product.
