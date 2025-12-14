# onedrive-cli

Cross-platform command line interface for OneDrive (Personal)
This fork adds PKCE-based login, refresh tokens, and automatic token rotation for long-running operations.

## Development

The provided `shell.nix` for the [Nix](https://nixos.org) package manager contains all development dependencies.
Use it with `nix-shell` or [Direnv](https://direnv.net)'s `use nix`.

### Update Nix files

```sh
node2nix -l package-lock.json
```

## Installation

From source:

```sh
$ git clone https://github.com/mundanelunacy/onedrive-cli.git
$ cd onedrive-cli
$ npm install
$ bin/onedrive login
```

## Usage

`usage: onedrive COMMAND [arguments]`

This little utility supports the following commands:

-   `cat` - dumps the contents of a file to stdout
-   `chmod` - change sharing permissions
-   `cp` - copies local file(s) to OneDrive or vice-versa
-   `df` - shows OneDrive storage usage stats
-   `find` - find file(s) or folder(s) by name, optionally separated by `NUL`
-   `help` - shows list of supported commands
-   `ln` - create a link to the remote item
-   `login` - request/store an OAuth access token
-   `ls` - list the contents of a folder
-   `mkdir` - create a remote folder
-   `mv` - move a local file to OneDrive or vice-versa
-   `rm` - delete a file from OneDrive (not implemented)
-   `sendmail` - send an invitation email for editing to recipients
-   `stat` - dump all information for particular file(s)
-   `wget` - copy a remote URL to OneDrive (server side)

## Examples

##### List the contents of the Public folder

`onedrive ls Public`

##### Grep one file

`onedrive cat Documents/passwords | grep boa`

##### Let OneDrive upload a file server side

`onedrive wget http://mega.com/somehugepublicfile Documents/somehugepublicfile`

##### Upload files recursively

`find * -type f -print0 | xargs -0 -n1 -I{} onedrive cp "./{}" "Shared Favorites/{}"`

##### Move remote files to a new folder

`onedrive find 'Pictures/Camera Roll' -regex 2015 -type f -print0 | xargs -0 onedrive mv -t :/Pictures/2015/`

## FAQ

##### Access token was not found; 'login' first.

The `onedrive` utility needs an access token in order to read/write to your OneDrive storage.
Use the`onedrive login` command to get the address of the Microsoft login page. After login,
this page will redirect to the file `oauthcallbackhandler.html` (https://github.com/lionello/onedrive-cli/blob/master/docs/oauthcallbackhandler.html)
and extract the authorization `code` from the URL parameters. Copy-paste this code into the command line
using `onedrive login code <AUTHORIZATION_CODE>`.
This will save the access token and refresh token in a file called `~/.onedrive-cli-token`.
Tokens are refreshed automatically about every 30 minutes while the tool is running so long operations stay authenticated.
The `onedrive login` command generates the PKCE code challenge/verifier for you; run it to get the URL, then use the resulting `login code ...` command in the same environment so the verifier can be redeemed.
To host the callback handler yourself, set `ONEDRIVE_REDIRECT_URI` to your hosted callback URL before running `onedrive login`; the authorization URL and token exchange will both use this value.
When using your own Azure app registration, grant delegated permissions `offline_access` and `onedrive.readwrite` and ensure the redirect URI matches `ONEDRIVE_REDIRECT_URI`; then re-run `onedrive login` to obtain fresh tokens.

##### "An item with the same name already exists under the parent"

Currently, a copy will fail if a file with the same it already exists.
Change the name of the target, or use other means to delete/rename the existing file in your OneDrive.

##### Invalid source name

You cannot copy folders. Specify a source file instead, or use wildcards.

##### Invalid target name

The target file name cannot be determined from the source path. Specify a target file name.

##### Use ./ or :/ path prefix for local or remote paths.

The `cp` command supports both local->remote as well as remote->local copy.
To make it clear which path is remote and which is local, either use `./` as a prefix for
the local path, or use `:/` as a prefix for the remote path. Either one will suffice.

##### chmod: Invalid file mode

The `chmod` command currently only supports `-w` or `-rw`. The former tried to downgrade _write_
shares to _read_-only, whereas the latter removes all shares for the given item(s). Octal modes are accepted (for example `644`, `0700`) as well as `og-rw` or `g-w`.

## TODO

-   Implement `rm`
-   Support gzip/deflate encoding for downloads
-   Uploads larger than 100MiB are not yet supported (needs range API)
-   Support OneDrive for Business
-   Ability to get the link for a file
-   Skip upload/download if the SHA1 matches
-   Adding write permissions (+w) to existing share links

## DONE

-   Register with NPM ([@lionello/onedrive-cli](https://www.npmjs.com/package/@lionello/onedrive-cli))
-   Fixed OAuth redirect on Safari (https://bugs.webkit.org/show_bug.cgi?id=24175)
-   Use XDG path spec for token file (https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html)
-   Using `async`/`await`
