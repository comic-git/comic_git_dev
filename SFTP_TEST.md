# Live SFTP test branch

`codex/sftp-test` is based on current `master` (`7e9f465`). It deploys only to
the disposable GCP VM, not GitHub Pages. Keep this configuration on the test branch.

- Project / VM: `comic-git` / `comicgit-sftp-test`
- Zone: `us-west1-b` (Oregon)
- Site: http://34.182.0.21/
- SFTP destination: `/home/comicgit/sftp-test.cvilbrandt.com`
- Engine and reusable workflow are pinned to the same SFTP feature commit.
- `Comic subdirectory` is empty because the site is served at the web root.
- The VM IP is ephemeral; update the workflow host, comic domain and verified
  host-key secret if the server address changes.

Pushes to this branch test password authentication. To test the optional SSH key:

```sh
gh workflow run main.yaml --repo comic-git/comic_git_dev --ref codex/sftp-test -f authentication=key
```

Use `authentication=password` to repeat the password test. Dedicated repository
secrets are `SFTP_TEST_USERNAME`, `SFTP_TEST_PASSWORD`, `SFTP_TEST_PRIVATE_KEY`,
and `SFTP_TEST_KNOWN_HOSTS`. The workflow supplies only the selected credential.
No credentials are stored in this repository.

## Checks

- The normal comic builds and is served from the website root.
- `site_root/sftp-test.txt` appears at `/sftp-test.txt`.
- Explicitly allowed `.htaccess` adds `X-Comic-Git-Deploy: sftp` to responses.
- `.nojekyll` is omitted because it is not in the hidden-path allowlist.
- Default cleanup removes obsolete ordinary and hidden files.
- Explicitly excluded `.well-known` files survive deployment untouched.

Verified on 2026-10-18 with real GitHub Actions deployments:

- [Password run](https://github.com/comic-git/comic_git_dev/actions/runs/39163922770): passed.
- [SSH-key run, attempt 2](https://github.com/comic-git/comic_git_dev/actions/runs/39163962005/attempts/2): passed.
- HTTP checks passed for `/`, `/archive/`, `/comic/001/`, and `/sftp-test.txt`,
  including the custom header and all remote-file checks above.

The first key attempt exposed Windows newline conversion when piping the key
through Python's text-mode subprocess input to `gh secret set`. Sending UTF-8
bytes preserved LF endings and fixed it. Preserve LF line endings when setting
the private-key secret; never print the key in workflow logs.

This is an SFTP-only deployment account: the job cannot rely on remote shell
commands. The site currently serves HTTP only. The engine normalizes absolute
URLs to HTTPS, so RSS/canonical/social URLs need DNS and a trusted certificate
before testing them as public links. Planned hostname: `sftp-test.cvilbrandt.com`.

Local VM setup, credentials and teardown commands live outside this repository
at `C:/Users/marco/Documents/ChatGPT/comic_git/gcp-sftp-test/README.md`.
