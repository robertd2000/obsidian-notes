https://go.dev/ref/mod#go-mod-download

### `go mod download`[¶](https://go.dev/ref/mod#go-mod-download)

Usage:

```
go mod download [-x] [-json] [-reuse=old.json] [modules]
```

Example:

```
$ go mod download
$ go mod download golang.org/x/mod@v0.2.0
```

The `go mod download` command downloads the named modules into the [module cache](https://go.dev/ref/mod#glos-module-cache). Arguments can be module paths or module patterns selecting dependencies of the main module or [version queries](https://go.dev/ref/mod#version-queries) of the form `path@version`. With no arguments, `download` applies to all dependencies of the [main module](https://go.dev/ref/mod#glos-main-module).

The `go` command will automatically download modules as needed during ordinary execution. The `go mod download` command is useful mainly for pre-filling the module cache or for loading data to be served by a [module proxy](https://go.dev/ref/mod#glos-module-proxy).

By default, `download` writes nothing to standard output. It prints progress messages and errors to standard error.

The `-json` flag causes `download` to print a sequence of JSON objects to standard output, describing each downloaded module (or failure), corresponding to this Go struct:

```
type Module struct {
    Path     string // module path
    Query    string // version query corresponding to this version
    Version  string // module version
    Error    string // error loading module
    Info     string // absolute path to cached .info file
    GoMod    string // absolute path to cached .mod file
    Zip      string // absolute path to cached .zip file
    Dir      string // absolute path to cached source root directory
    Sum      string // checksum for path, version (as in go.sum)
    GoModSum string // checksum for go.mod (as in go.sum)
    Origin   any    // provenance of module
    Reuse    bool   // reuse of old module info is safe
}
```

The `-x` flag causes `download` to print the commands `download` executes to standard error.

The -reuse flag accepts the name of file containing the JSON output of a previous ‘go mod download -json’ invocation. The go command may use this file to determine that a module is unchanged since the previous invocation and avoid redownloading it. Modules that are not redownloaded will be marked in the new output by setting the Reuse field to true. Normally the module cache provides this kind of reuse automatically; the -reuse flag can be useful on systems that do not preserve the module cache.

### `go mod edit`[¶](https://go.dev/ref/mod#go-mod-edit)

Usage:

```
go mod edit [editing flags] [-fmt|-print|-json] [go.mod]
```

Example:

```
# Add a replace directive.
$ go mod edit -replace example.com/a@v1.0.0=./a

# Remove a replace directive.
$ go mod edit -dropreplace example.com/a@v1.0.0

# Set the go version, add a requirement, and print the file
# instead of writing it to disk.
$ go mod edit -go=1.14 -require=example.com/m@v1.0.0 -print

# Format the go.mod file.
$ go mod edit -fmt

# Format and print a different .mod file.
$ go mod edit -print tools.mod

# Print a JSON representation of the go.mod file.
$ go mod edit -json
```

The `go mod edit` command provides a command-line interface for editing and formatting `go.mod` files, for use primarily by tools and scripts. `go mod edit` reads only one `go.mod` file; it does not look up information about other modules. By default, `go mod edit` reads and writes the `go.mod` file of the main module, but a different target file can be specified after the editing flags.

The editing flags specify a sequence of editing operations.

- The `-module` flag changes the module’s path (the `go.mod` file’s module line).
- The `-go=version` flag sets the expected Go language version.
- The `-require=path@version` and `-droprequire=path` flags add and drop a requirement on the given module path and version. Note that `-require` overrides any existing requirements on `path`. These flags are mainly for tools that understand the module graph. Users should prefer `go get path@version` or `go get path@none`, which make other `go.mod` adjustments as needed to satisfy constraints imposed by other modules. See [`go get`](https://go.dev/ref/mod#go-get).
- The `-exclude=path@version` and `-dropexclude=path@version` flags add and drop an exclusion for the given module path and version. Note that `-exclude=path@version` is a no-op if that exclusion already exists.
- The `-replace=old[@v]=new[@v]` flag adds a replacement of the given module path and version pair. If the `@v` in `old@v` is omitted, a replacement without a version on the left side is added, which applies to all versions of the old module path. If the `@v` in `new@v` is omitted, the new path should be a local module root directory, not a module path. Note that `-replace` overrides any redundant replacements for `old[@v]`, so omitting `@v` will drop replacements for specific versions.
- The `-dropreplace=old[@v]` flag drops a replacement of the given module path and version pair. If the `@v` is provided, a replacement with the given version is dropped. An existing replacement without a version on the left side may still replace the module. If the `@v` is omitted, a replacement without a version is dropped.
- The `-retract=version` and `-dropretract=version` flags add and drop a retraction for the given version, which may be a single version (like `v1.2.3`) or an interval (like `[v1.1.0,v1.2.0]`). Note that the `-retract` flag cannot add a rationale comment for the `retract` directive. Rationale comments are recommended and may be shown by `go list -m -u` and other commands.
- The `-tool=path` and `-droptool=path` flags add and drop a `tool` directive for the given paths. Note that this will not add necessary dependencies to the build graph. Users should prefer `go get -tool path` to add a tool, or `go get -tool path@none` to remove one.

The editing flags may be repeated. The changes are applied in the order given.

`go mod edit` has additional flags that control its output.

- The `-fmt` flag reformats the `go.mod` file without making other changes. This reformatting is also implied by any other modifications that use or rewrite the `go.mod` file. The only time this flag is needed is if no other flags are specified, as in `go mod edit -fmt`.
- The `-print` flag prints the final `go.mod` in its text format instead of writing it back to disk.
- The `-json` flag prints the final `go.mod` in JSON format instead of writing it back to disk in text format. The JSON output corresponds to these Go types:

```
type Module struct {
    Path    string
    Version string
}

type GoMod struct {
    Module  ModPath
    Go      string
    Require []Require
    Exclude []Module
    Replace []Replace
    Retract []Retract
}

type ModPath struct {
    Path       string
    Deprecated string
}

type Require struct {
    Path     string
    Version  string
    Indirect bool
}

type Replace struct {
    Old Module
    New Module
}

type Retract struct {
    Low       string
    High      string
    Rationale string
}

type Tool struct {
    Path      string
}
```

Note that this only describes the `go.mod` file itself, not other modules referred to indirectly. For the full set of modules available to a build, use `go list -m -json all`. See [`go list -m`](https://go.dev/ref/mod#go-list-m).

For example, a tool can obtain the `go.mod` file as a data structure by parsing the output of `go mod edit -json` and can then make changes by invoking `go mod edit` with `-require`, `-exclude`, and so on.

Tools may also use the package [`golang.org/x/mod/modfile`](https://pkg.go.dev/golang.org/x/mod/modfile?tab=doc) to parse, edit, and format `go.mod` files.

### `go mod graph`[¶](https://go.dev/ref/mod#go-mod-graph)

Usage:

```
go mod graph [-go=version]
```

The `go mod graph` command prints the [module requirement graph](https://go.dev/ref/mod#glos-module-graph) (with replacements applied) in text form. For example:

```
example.com/main example.com/a@v1.1.0
example.com/main example.com/b@v1.2.0
example.com/a@v1.1.0 example.com/b@v1.1.1
example.com/a@v1.1.0 example.com/c@v1.3.0
example.com/b@v1.1.0 example.com/c@v1.1.0
example.com/b@v1.2.0 example.com/c@v1.2.0
```

Each vertex in the module graph represents a specific version of a module. Each edge in the graph represents a requirement on a minimum version of a dependency.

`go mod graph` prints the edges of the graph, one per line. Each line has two space-separated fields: a module version and one of its dependencies. Each module version is identified as a string of the form `path@version`. The main module has no `@version` suffix, since it has no version.

The `-go` flag causes `go mod graph` to report the module graph as loaded by the given Go version, instead of the version indicated by the [`go` directive](https://go.dev/ref/mod#go-mod-file-go) in the `go.mod` file.

See [Minimal version selection (MVS)](https://go.dev/ref/mod#minimal-version-selection) for more information on how versions are chosen. See also [`go list -m`](https://go.dev/ref/mod#go-list-m) for printing selected versions and [`go mod why`](https://go.dev/ref/mod#go-mod-why) for understanding why a module is needed.

### `go mod init`[¶](https://go.dev/ref/mod#go-mod-init)

Usage:

```
go mod init [module-path]
```

Example:

```
go mod init
go mod init example.com/m
```

The `go mod init` command initializes and writes a new `go.mod` file in the current directory, in effect creating a new module rooted at the current directory. The `go.mod` file must not already exist.

`init` accepts one optional argument, the [module path](https://go.dev/ref/mod#glos-module-path) for the new module. See [Module paths](https://go.dev/ref/mod#module-path) for instructions on choosing a module path. If the module path argument is omitted, `init` will attempt to infer the module path using import comments in `.go` files and the current directory (if in `GOPATH`).

### `go mod tidy`[¶](https://go.dev/ref/mod#go-mod-tidy)

Usage:

```
go mod tidy [-e] [-v] [-x] [-diff] [-go=version] [-compat=version]
```

`go mod tidy` ensures that the `go.mod` file matches the source code in the module. It adds any missing module requirements necessary to build the current module’s packages and dependencies, and it removes requirements on modules that don’t provide any relevant packages. It also adds any missing entries to `go.sum` and removes unnecessary entries.

The `-e` flag (added in Go 1.16) causes `go mod tidy` to attempt to proceed despite errors encountered while loading packages.

The `-v` flag causes `go mod tidy` to print information about removed modules to standard error.

The `-x` flag causes `go mod tidy` to print the commands `tidy` executes.

The `-diff` flag causes `go mod tidy` not to modify go.mod or go.sum but instead print the necessary changes as a unified diff. It exits with a non-zero code if the diff is not empty.

`go mod tidy` works by loading all of the packages in the [main module](https://go.dev/ref/mod#glos-main-module), all of its tools, and all of the packages they import, recursively. This includes packages imported by tests (including tests in other modules). `go mod tidy` acts as if all build tags are enabled, so it will consider platform-specific source files and files that require custom build tags, even if those source files wouldn’t normally be built. There is one exception: the `ignore` build tag is not enabled, so a file with the build constraint `// +build ignore` will not be considered. Note that `go mod tidy` will not consider packages in the main module in directories named `testdata` or with names that start with `.` or `_` unless those packages are explicitly imported by other packages.

Once `go mod tidy` has loaded this set of packages, it ensures that each module that provides one or more packages has a `require` directive in the main module’s `go.mod` file or — if the main module is at `go 1.16` or below — is required by another required module. `go mod tidy` will add a requirement on the latest version of each missing module (see [Version queries](https://go.dev/ref/mod#version-queries) for the definition of the `latest` version). `go mod tidy` will remove `require` directives for modules that don’t provide any packages in the set described above.

`go mod tidy` may also add or remove `// indirect` comments on `require` directives. An `// indirect` comment denotes a module that does not provide a package imported by a package in the main module. (See the [`require` directive](https://go.dev/ref/mod#go-mod-file-require) for more detail on when `// indirect` dependencies and comments are added.)

If the `-go` flag is set, `go mod tidy` will update the [`go` directive](https://go.dev/ref/mod#go-mod-file-go) to the indicated version, enabling or disabling [module graph pruning](https://go.dev/ref/mod#graph-pruning) and [lazy module loading](https://go.dev/ref/mod#lazy-loading) (and adding or removing indirect requirements as needed) according to that version.

By default, `go mod tidy` will check that the [selected versions](https://go.dev/ref/mod#glos-selected-version) of modules do not change when the module graph is loaded by the Go version immediately preceding the version indicated in the `go` directive. The versioned checked for compatibility can also be specified explicitly via the `-compat` flag.

### `go mod vendor`[¶](https://go.dev/ref/mod#go-mod-vendor)

Usage:

```
go mod vendor [-e] [-v] [-o]
```

The `go mod vendor` command constructs a directory named `vendor` in the [main module’s](https://go.dev/ref/mod#glos-main-module) root directory that contains copies of all packages needed to support builds and tests of packages in the main module. Packages that are only imported by tests of packages outside the main module are not included. As with [`go mod tidy`](https://go.dev/ref/mod#go-mod-tidy) and other module commands, [build constraints](https://go.dev/ref/mod#glos-build-constraint) except for `ignore` are not considered when constructing the `vendor` directory.

When vendoring is enabled, the `go` command will load packages from the `vendor` directory instead of downloading modules from their sources into the module cache and using packages those downloaded copies. See [Vendoring](https://go.dev/ref/mod#vendoring) for more information.

`go mod vendor` also creates the file `vendor/modules.txt` that contains a list of vendored packages and the module versions they were copied from. When vendoring is enabled, this manifest is used as a source of module version information, as reported by [`go list -m`](https://go.dev/ref/mod#go-list-m) and [`go version -m`](https://go.dev/ref/mod#go-version-m). When the `go` command reads `vendor/modules.txt`, it checks that the module versions are consistent with `go.mod`. If `go.mod` changed since `vendor/modules.txt` was generated, `go mod vendor` should be run again.

Note that `go mod vendor` removes the `vendor` directory if it exists before re-constructing it. Local changes should not be made to vendored packages. The `go` command does not check that packages in the `vendor` directory have not been modified, but one can verify the integrity of the `vendor` directory by running `go mod vendor` and checking that no changes were made.

The `-e` flag (added in Go 1.16) causes `go mod vendor` to attempt to proceed despite errors encountered while loading packages.

The `-v` flag causes `go mod vendor` to print the names of vendored modules and packages to standard error.

The `-o` flag (added in Go 1.18) causes `go mod vendor` to output the vendor tree at the specified directory instead of `vendor`. The argument can be either an absolute path or a path relative to the module root.

### `go mod verify`[¶](https://go.dev/ref/mod#go-mod-verify)

Usage:

```
go mod verify
```

`go mod verify` checks that dependencies of the [main module](https://go.dev/ref/mod#glos-main-module) stored in the [module cache](https://go.dev/ref/mod#glos-module-cache) have not been modified since they were downloaded. To perform this check, `go mod verify` hashes each downloaded module [`.zip` file](https://go.dev/ref/mod#zip-files) and extracted directory, then compares those hashes with a hash recorded when the module was first downloaded. `go mod verify` checks each module in the [build list](https://go.dev/ref/mod#glos-build-list) (which may be printed with [`go list -m all`](https://go.dev/ref/mod#go-list-m)).

If all the modules are unmodified, `go mod verify` prints “all modules verified”. Otherwise, it reports which modules have been changed and exits with a non-zero status.

Note that all module-aware commands verify that hashes in the main module’s `go.sum` file match hashes recorded for modules downloaded into the module cache. If a hash is missing from `go.sum` (for example, because the module is being used for the first time), the `go` command verifies its hash using the [checksum database](https://go.dev/ref/mod#checksum-database) (unless the module path is matched by `GOPRIVATE` or `GONOSUMDB`). See [Authenticating modules](https://go.dev/ref/mod#authenticating) for details.

In contrast, `go mod verify` checks that module `.zip` files and their extracted directories have hashes that match hashes recorded in the module cache when they were first downloaded. This is useful for detecting changes to files in the module cache _after_ a module has been downloaded and verified. `go mod verify` does not download content for modules not in the cache, and it does not use `go.sum` files to verify module content. However, `go mod verify` may download `go.mod` files in order to perform [minimal version selection](https://go.dev/ref/mod#minimal-version-selection). It will use `go.sum` to verify those files, and it may add `go.sum` entries for missing hashes.

### `go mod why`[¶](https://go.dev/ref/mod#go-mod-why)

Usage:

```
go mod why [-m] [-vendor] packages...
```

`go mod why` shows a shortest path in the import graph from the main module to each of the listed packages.

The output is a sequence of stanzas, one for each package or module named on the command line, separated by blank lines. Each stanza begins with a comment line starting with `#` giving the target package or module. Subsequent lines give a path through the import graph, one package per line. If the package or module is not referenced from the main module, the stanza will display a single parenthesized note indicating that fact.

For example:

```
$ go mod why golang.org/x/text/language golang.org/x/text/encoding
# golang.org/x/text/language
rsc.io/quote
rsc.io/sampler
golang.org/x/text/language

# golang.org/x/text/encoding
(main module does not need package golang.org/x/text/encoding)
```

The `-m` flag causes `go mod why` to treat its arguments as a list of modules. `go mod why` will print a path to any package in each of the modules. Note that even when `-m` is used, `go mod why` queries the package graph, not the module graph printed by [`go mod graph`](https://go.dev/ref/mod#go-mod-graph).

The `-vendor` flag causes `go mod why` to ignore imports in tests of packages outside the main module (as [`go mod vendor`](https://go.dev/ref/mod#go-mod-vendor) does). By default, `go mod why` considers the graph of packages matched by the `all` pattern. This flag has no effect after Go 1.16 in modules that declare `go 1.16` or higher (using the [`go` directive](https://go.dev/ref/mod#go-mod-file-go) in `go.mod`), since the meaning of `all` changed to match the set of packages matched by `go mod vendor`.