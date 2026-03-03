# murk-action

Decrypt [murk](https://github.com/iicky/murk) secrets and inject them
into your GitHub Actions workflow.

## Usage

```yaml
- uses: iicky/murk-action@v1
  with:
    murk-key: ${{ secrets.MURK_KEY }}
```

All decrypted values are written to `$GITHUB_ENV` and masked in logs.
Subsequent steps can use them as regular environment variables.

```yaml
steps:
  - uses: actions/checkout@v4

  - uses: iicky/murk-action@v1
    with:
      murk-key: ${{ secrets.MURK_KEY }}

  - run: echo "connected to $DATABASE_URL"
    # DATABASE_URL is available and its value is masked in logs
```

## Inputs

| Input      | Required | Default  | Description                            |
|------------|----------|----------|----------------------------------------|
| `murk-key` | yes      |          | Age secret key (`AGE-SECRET-KEY-1...`) |
| `version`  | no       | `latest` | murk version (e.g. `0.2.1`)           |
| `vault`    | no       | `.murk`  | Path to vault file                     |
| `tags`     | no       |          | Space-separated tags to filter secrets |

## Examples

### Pin a version

```yaml
- uses: iicky/murk-action@v1
  with:
    murk-key: ${{ secrets.MURK_KEY }}
    version: '0.2.1'
```

### Custom vault path

```yaml
- uses: iicky/murk-action@v1
  with:
    murk-key: ${{ secrets.MURK_KEY }}
    vault: 'prod.murk'
```

### Filter by tags

```yaml
- uses: iicky/murk-action@v1
  with:
    murk-key: ${{ secrets.MURK_KEY }}
    tags: 'deploy aws'
```

### Use with murk exec

If you prefer subprocess injection over `$GITHUB_ENV`:

```yaml
- uses: iicky/murk-action@v1
  with:
    murk-key: ${{ secrets.MURK_KEY }}

- run: murk exec --vault .murk -- ./deploy.sh
  env:
    MURK_KEY: ${{ secrets.MURK_KEY }}
```

## How it works

1. Downloads the murk binary from GitHub Releases for the runner platform
2. Runs `murk export` to decrypt the vault
3. Masks each secret value with `::add-mask::`
4. Writes each key-value pair to `$GITHUB_ENV`

## Supported runners

- `ubuntu-latest` (x86_64, arm64)
- `macos-latest` (x86_64, arm64)

## Security

- `murk-key` should always come from a GitHub Actions secret
- All decrypted values are masked before being written anywhere
- The murk binary is downloaded over HTTPS from GitHub Releases
- No secrets are written to disk

## License

MIT OR Apache-2.0
