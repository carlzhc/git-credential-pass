# git-credential-pass

A [git] credential helper implementation that allows using [pass] as the
credential backend for your https-based git repositories.  When [git] tries
to interact with an https-based upstream and needs credentials, this helper
will be called to look up the credentials from the user's password store. 
Instead of enforcing a specific layout of the password store, a
configuration file with explicitly defining mappings between hosts and
entries in the password store is used, giving full flexibility to the user
on how to structure or reuse existing password databases for [git]
authentication.

## Preconditions

It is recommended to configure GPG to use a graphical pinentry program. 
That way, you can also use this helper when [git] is invoked via GUI
programs such as your IDE.  For a configuration example, refer to the
[ArchWiki](https://wiki.archlinux.org/index.php/GnuPG#pinentry).  In case
you really want to use the terminal for pinentry (via `pinentry-curses`), be
sure to [appropriately configure the environment variable
`GPG_TTY`](https://www.gnupg.org/documentation/manuals/gnupg/Invoking-GPG_002dAGENT.html),
most likely by adding the following lines to your shell initialization:

```sh
GPG_TTY=$(tty)
export GPG_TTY
```

If you use this setup for remote work via SSH, also consider the alternative of [GPG agent forwarding](https://wiki.gnupg.org/AgentForwarding).

## Installation

Place the script [git-credential-pass] to anywhere in your PATH.

```sh
sudo install -m 755 git-credential-pass /usr/local/bin/
```

## Usage

### Configure git to use git-credential-pass

To instruct git to use the helper, set the `credential.helper` configuration
option of git to `pass`.  
```sh
git config credential.helper pass
git config credential.useHttpPath true
```

This will result in the following contents in `~/.gitconfig`:

```ini
[credential]
        helper = pass
        useHttpPath = true
```

In case you share the `~/.gitconfig` across multiple machines and
`git-credential-pass` is not available on all of them, the following version
does not bail out if pass git helper is missing:

```ini
[credential]
    helper = !type git-credential-pass >/dev/null && git-credential-pass $@
```

`git-credential-pass` can be combined with other helpers.
For instance, the following configuration first tries the git built-in `cache` helper for in-memory password access before falling back to `git-credential-pass` if a cache miss occurs:

```ini
[credential]
    helper = cache
    helper = !type git-credential-pass >/dev/null && git-credential-pass $@
```

### Structure of the [pass] storage

The layout of credentials stored in the [pass] storage looks like this:
```
$ pass ls
Password Store
└── login@git-server.com
    └── user-name
        └── repo-name
```

For example, to store a password for a repository in github.com, use this
command:

```
$ pass insert git@github.com/my-name/my-repo
```

Notice: remove the [.git] suffix from the repository's url.
