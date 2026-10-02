# proton-drive-git

Use [Proton Drive](https://proton.me/drive) as a git remote.

```sh
git remote add proton proton::/my-files/git/myrepo.bundle
git push proton main
git clone proton::/my-files/git/myrepo.bundle
```

An unofficial [git remote helper](https://git-scm.com/docs/gitremote-helpers)
built on the official [Proton Drive CLI](https://proton.me/support/drive-cli)
(`proton-drive`). Not affiliated with Proton AG.

## How it works

The remote is a single [git bundle](https://git-scm.com/docs/git-bundle) file
on Proton Drive.

- **fetch / clone:** download the bundle (cached locally by SHA-1) and unbundle.
- **push:** rebuild the bundle from all refs and upload it as a new file
  revision. Proton's revision history doubles as a push log.

A push is rejected with `fetch first` if the bundle on Proton changed between
the ref listing and the upload. Non-fast-forward pushes are rejected unless
forced, as with any remote.

## Install

Requirements: Python 3.9+, git, and the `proton-drive` CLI, logged in once
with `proton-drive auth login`.

```sh
git clone https://github.com/marosmars/proton-drive-git
ln -s "$PWD/proton-drive-git/git-remote-proton" ~/.local/bin/git-remote-proton
```

Any directory on `PATH` works. Git finds the helper by its name.

The helper runs `proton-drive` from `PATH` and reuses that CLI's login. It
never sees your Proton credentials. Set `PROTON_DRIVE_CLI=/path/to/proton-drive`
to use a binary that is not on `PATH`.

### Platforms

Pure Python standard library plus `git` and `proton-drive`, so any Linux
distribution or macOS with Python 3.9+ works. Windows is untested.

## URLs

`proton::<absolute Proton path ending in .bundle>`

| Location | URL |
|---|---|
| Your drive | `proton::/my-files/git/myrepo.bundle` |
| Shared with you | `proton::/shared-with-me/<NODE-UID>/myrepo.bundle` |

Missing parent folders under `/my-files` are created on first push. Find a
shared folder's node UID with `proton-drive fs list /shared-with-me`.

## Sharing

Share the folder holding the bundle with a Proton invite:

```sh
proton-drive sharing invite -u someone@proton.me -r editor /my-files/git
```

Viewers can clone and fetch. Editors can also push. Public share links do not
work: they are decrypted in the browser, and the helper needs an
authenticated CLI.

## Limitations

- Every push uploads the whole repository. Fine for small repos, slow for big
  ones.
- The `fetch first` guard narrows but does not close the race between two
  concurrent pushers. The overwritten bundle stays recoverable from Proton's
  revision history.
- No per-branch permissions: any editor can force-push or delete branches.
- `/shared-with-me` URLs are untested so far.
- A missing remote is detected by the CLI's `Node not found` message. If a
  future CLI version rewords it, the helper fails loudly rather than treating
  the remote as empty.

## License

MIT
