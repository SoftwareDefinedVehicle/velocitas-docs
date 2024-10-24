# Velocitas-Docs

[![License: Apache](https://img.shields.io/badge/License-Apache-yellow.svg)](http://www.apache.org/licenses/LICENSE-2.0)

This repository contains the source code for the Velocitas Documentation.

:notebook: **Please visit the generated documentation to learn more about Velocitas!** <https://eclipse.dev/velocitas/>

## Directory structure

* Markdown files used for the generated documentation exists in the `content` folder.
* Examples shown in the documentation may exist as source code in the `examples`folder.

## Building documentation locally

### Using Devcontainer

Velocitas-docs provides a  [Devcontainer](https://code.visualstudio.com/docs/devcontainers/containers) that can be used to build and run the documentation server locally.
As the last step when starting the devcontainer the webserver is started, and it is reachable outside Devcontainer.
If you update source pages the local server will be automatically updated.

`Web Server is available at http://localhost:1313/velocitas/ (bind address 127.0.0.1)`

### Using Native Envioronment

It is also possible to build and run the documentation outside the Devcontainer. If using Debian you could install Hugo like this:

`sudo apt install hugo`

For other environments please visit [Hugo documentation](https://gohugo.io/installation/).

Then you can build and start a local hugo server and access documentation at `http://localhost:1313`.

`/usr/bin/hugo server -D -s .`

If you update source pages the local server will be automatically updated.

## Deploying official documentation

The Velocitas Documentation Site is not automatically updated when something is merged to main.
Instead a Velocitas committer needs to manually run the `Publish Documentation` workflow.
That results in that the `docs` branch is updated and triggers the `pages-build-deployment` workflow that publishes the new version to the Velocitas Documentation Site.

## Theme Upgrade

Velocitas-docs use the [Docsy Theme](https://github.com/google/docsy).
To upgrade change version in [postCreateCommand.sh](.devcontainer/scripts/postCreateCommand.sh). Then rebuild the devcontainer.
This will result in that `go.mod`and `go.sum` gets updated.

## Community

- [GitHub Issues](https://github.com/eclipse-velocitas/velocitas-docs/issues)
- [Mailing List](https://accounts.eclipse.org/mailing-list/velocitas-dev)
- [Contribution](content/en/docs/Contributing/contribution.md)
