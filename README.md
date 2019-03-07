# go-libp2p workspace

The go-libp2p project has migrated to [Go
modules](https://github.com/golang/go/wiki/Modules). As a result, our
development workflow will need to change a bit.

## `go get` and `GOPATH`

Goodbye monolithic `GOPATH`.

Running `go get` from outside a go module (i.e. a Go source file tree that
doesn't contain a go.mod) will still act under your `GOPATH`.

Conversely, running `go get` inside a Go module will update that module's
`go.mod` (if necessary), and will downloaded any packages into the global go
mod cache (`$GOPATH/pkg/mod`), where they are indexed by module name and
version, and used by the rest of the go tools (go build, go test, etc.), which
are all now module-aware.

_Fun fact: Go mod makes bold assertions about immutability, and this transpires
even to file permissions under the module cache. A quick `ls -l` therein shows
that go devoids downloaded modules of write permissions even for the owner._

## Local development

The go tools resolve module dependencies off from the uptstream repos, based on
the `require` directives in `go.mod` files.

While this is great for build safety and reproceability, developers want to
make changesets across a number of local repos and have them picked up by
interdependent modules immediately. go mod `replace` directives enable
resolving modules from local paths.

This repo aggregates all libp2p modules under a single roof via git submodules.
It aims to provide an experience analogous to the `GOPATH` monolith.

The accompanying `workspace.sh` script leverages `replace` directives to map
dependencies to their locally checked out versions:

* `./workspace.sh local` adds maps all libp2p modules to their local checkouts.
* `./workspace.sh remote` resets all libp2p modules to resolve against remotes.

## Recommended reading

Please familiarise yourself with [Go
modules](https://github.com/golang/go/wiki/Modules). In particular, pay
attention to how minimal version selection works.

Reacquaint yourself with go commands. Most of the commands you know and have
come to love (`go get`, `go build`, `go test`) now deal with modules
transparently. We suggest you go through the [go command
manual](https://golang.org/cmd/go/), and pay special attention to the sections
dedicated to modules behaviour.
