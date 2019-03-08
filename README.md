# go-libp2p workspace

go-libp2p is a p2p networking stack based on the principle of modularity. Its
modules are scattered across a number of Git repos. We have recently migrated
from gx to gomod as the primary dependency and build management tool.

To make developing with gomod easy, this meta-repository aggregates all
go-libp2p repos under a single roof via git submodules, and provides a
management script (`./workspace.sh`) for automating common workflows.

## ⚠️ Work in progress

This setup is currently at pilot stage and heavily WIP. Feedback and
contributions are welcome. Some wishlist items include:

- [ ] go mod <> IPFS integration, to consume and publish content-addressed
  dependencies easily. [Stebalien/ipgo](https://github.com/Stebalien/ipgo) is
  one possible direction.
- [ ] Automating the "bubbling" and update workflow, via scripts or tooling
  such as [renovate](https://renovatebot.com).
- [ ] Shared libp2p/IPFS workspace.
- [ ] Nightly master builds.

## 👉 Prerequisite: Git pre-commit hook

This [pre-commit
hook](https://gist.github.com/Kubuxu/3fc5639db27f4b072b33a84b51048ff8) will
alert you when you are trying to commit go.mod file with local replace
directives. This is useful pattern for developing, but has no place on remote.

It is best to install it as global githook. Instructions
[here](https://stackoverflow.com/questions/1977610/change-default-git-hooks/37293001#37293001).
 
## Usage

**Getting started**

This will initialise the submodules by cloning repos into their relevant
subdirectories.

```
$ git clone <this repo> $ git submodule init $ git submodule update
```

**Switch to local module resolution**

Interlinks all repos for local development through `replace` directives.

```
$ ./workspace.sh local
```

**Switch to remote module resolution**

Removes the `replace` directives added by `local`.

```
$ ./workspace.sh remote
```

**Stash local changes and update all repos from origin/master**

```
$ ./workspace.sh remote    # to reset go.mod files to original $ ./workspace.sh
refresh
```

## Background: Why is this necessary?

Gomod is great for mono-repo projects that depend on 3rd party dependencies,
but there are some challenges when working with an intervowen set of
modularised packages:

* Go tools resolve module versions from remotes.
* Developers want to make changesets across a number of local repos and have
  them visible by interdependent modules.

Aggregating modules under one roof, and using go mod `replace` directives to
interlink them locally via `replace` directives, provides a neat `GOPATH`-like
development experience.

**Detour: GOPATH vs go modules**

The `GOPATH` monolith is irrelevant in the Go modules universe. Now you can
check out modules anywhere in the filesystem.

Running `go get` from inside a Go module will download packages into the global
go mod cache (`$GOPATH/pkg/mod`), where they are indexed by module path and
version.

All go tools (go build, go test, etc.) are now module-friendly. They
automatically add `require` directives go `go.mod` for new imports in code,
download packages from remotes, and more. It's like magic.

_NOTE: Running `go get` in a tree without a `go.mod` will still place that
package under your `GOPATH`. Running `go` commands from within your `GOPATH`
will behave like before (i.e. no module-based builds) even if the package has a
`go.mod`, unless you explicitly set `GO111MODULE=on`._

_Fun fact: Go mod makes bold assertions about immutability, and this transpires
even to file permissions under the module cache. A quick `ls -l` therein shows
that go strips away write permissions from downloaded modules, even for the
owner._

## Recommended reading

Familiarise yourself with [Go
modules](https://github.com/golang/go/wiki/Modules). In particular, pay
attention to how minimal version selection works.

Reacquaint yourself with go commands. Most of the commands you know and have
come to love (`go get`, `go build`, `go test`) now deal with modules
transparently. Suggested read: [go command manual](https://golang.org/cmd/go/),
and pay special attention to the sections dedicated to modules behaviour.
