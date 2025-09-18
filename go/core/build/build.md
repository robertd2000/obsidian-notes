https://pkg.go.dev/go/build

### Overview [¶](https://pkg.go.dev/go/build#pkg-overview "Go to Overview")

- [Build Constraints](https://pkg.go.dev/go/build#hdr-Build_Constraints)
- [Go Path](https://pkg.go.dev/go/build#hdr-Go_Path)
- [Binary-Only Packages](https://pkg.go.dev/go/build#hdr-Binary_Only_Packages)

Package build gathers information about Go packages.

#### Build Constraints [¶](https://pkg.go.dev/go/build#hdr-Build_Constraints "Go to Build Constraints")

A build constraint, also known as a build tag, is a condition under which a file should be included in the package. Build constraints are given by a line comment that begins

//go:build

Build constraints may also be part of a file's name (for example, source_windows.go will only be included if the target operating system is windows).

See 'go help buildconstraint' ([https://pkg.go.dev/cmd/go#hdr-Build_constraints](https://pkg.go.dev/cmd/go#hdr-Build_constraints)) for details.

#### Go Path [¶](https://pkg.go.dev/go/build#hdr-Go_Path "Go to Go Path")

The Go path is a list of directory trees containing Go source code. It is consulted to resolve imports that cannot be found in the standard Go tree. The default path is the value of the GOPATH environment variable, interpreted as a path list appropriate to the operating system (on Unix, the variable is a colon-separated string; on Windows, a semicolon-separated string; on Plan 9, a list).

Each directory listed in the Go path must have a prescribed structure:

The src/ directory holds source code. The path below 'src' determines the import path or executable name.

The pkg/ directory holds installed package objects. As in the Go tree, each target operating system and architecture pair has its own subdirectory of pkg (pkg/GOOS_GOARCH).

If DIR is a directory listed in the Go path, a package with source in DIR/src/foo/bar can be imported as "foo/bar" and has its compiled form installed to "DIR/pkg/GOOS_GOARCH/foo/bar.a" (or, for gccgo, "DIR/pkg/gccgo/foo/libbar.a").

The bin/ directory holds compiled commands. Each command is named for its source directory, but only using the final element, not the entire path. That is, the command with source in DIR/src/foo/quux is installed into DIR/bin/quux, not DIR/bin/foo/quux. The foo/ is stripped so that you can add DIR/bin to your PATH to get at the installed commands.

Here's an example directory layout:

GOPATH=/home/user/gocode

/home/user/gocode/
    src/
        foo/
            bar/               (go code in package bar)
                x.go
            quux/              (go code in package main)
                y.go
    bin/
        quux                   (installed command)
    pkg/
        linux_amd64/
            foo/
                bar.a          (installed package object)

#### Binary-Only Packages [¶](https://pkg.go.dev/go/build#hdr-Binary_Only_Packages "Go to Binary-Only Packages")

In Go 1.12 and earlier, it was possible to distribute packages in binary form without including the source code used for compiling the package. The package was distributed with a source file not excluded by build constraints and containing a "//go:binary-only-package" comment. Like a build constraint, this comment appeared at the top of a file, preceded only by blank lines and other line comments and with a blank line following the comment, to separate it from the package documentation. Unlike build constraints, this comment is only recognized in non-test Go source files.

The minimal source code for a binary-only package was therefore:

//go:binary-only-package

package mypkg

The source code could include additional Go code. That code was never compiled but would be processed by tools like godoc and might be useful as end-user documentation.

"go build" and other commands no longer support binary-only-packages. [Import](https://pkg.go.dev/go/build#Import) and [ImportDir](https://pkg.go.dev/go/build#ImportDir) will still set the BinaryOnly flag in packages containing these comments for use in tools and error messages.

### Index [¶](https://pkg.go.dev/go/build#pkg-index "Go to Index")

- [Variables](https://pkg.go.dev/go/build#pkg-variables)
- [func ArchChar(goarch string) (string, error)](https://pkg.go.dev/go/build#ArchChar)
- [func IsLocalImport(path string) bool](https://pkg.go.dev/go/build#IsLocalImport)
- [type Context](https://pkg.go.dev/go/build#Context)
- - [func (ctxt *Context) Import(path string, srcDir string, mode ImportMode) (*Package, error)](https://pkg.go.dev/go/build#Context.Import)
    - [func (ctxt *Context) ImportDir(dir string, mode ImportMode) (*Package, error)](https://pkg.go.dev/go/build#Context.ImportDir)
    - [func (ctxt *Context) MatchFile(dir, name string) (match bool, err error)](https://pkg.go.dev/go/build#Context.MatchFile)
    - [func (ctxt *Context) SrcDirs() []string](https://pkg.go.dev/go/build#Context.SrcDirs)
- [type Directive](https://pkg.go.dev/go/build#Directive)
- [type ImportMode](https://pkg.go.dev/go/build#ImportMode)
- [type MultiplePackageError](https://pkg.go.dev/go/build#MultiplePackageError)
- - [func (e *MultiplePackageError) Error() string](https://pkg.go.dev/go/build#MultiplePackageError.Error)
- [type NoGoError](https://pkg.go.dev/go/build#NoGoError)
- - [func (e *NoGoError) Error() string](https://pkg.go.dev/go/build#NoGoError.Error)
- [type Package](https://pkg.go.dev/go/build#Package)
- - [func Import(path, srcDir string, mode ImportMode) (*Package, error)](https://pkg.go.dev/go/build#Import)
    - [func ImportDir(dir string, mode ImportMode) (*Package, error)](https://pkg.go.dev/go/build#ImportDir)
- - [func (p *Package) IsCommand() bool](https://pkg.go.dev/go/build#Package.IsCommand)

### Constants [¶](https://pkg.go.dev/go/build#pkg-constants "Go to Constants")

This section is empty.

### Variables [¶](https://pkg.go.dev/go/build#pkg-variables "Go to Variables")

[View Source](https://cs.opensource.google/go/go/+/go1.25.1:src/go/build/build.go;l=2042)

var ToolDir = getToolDir()

ToolDir is the directory containing build tools.

### Functions [¶](https://pkg.go.dev/go/build#pkg-functions "Go to Functions")

#### func [ArchChar](https://cs.opensource.google/go/go/+/go1.25.1:src/go/build/build.go;l=2056) [¶](https://pkg.go.dev/go/build#ArchChar "Go to ArchChar")

func ArchChar(goarch [string](https://pkg.go.dev/builtin#string)) ([string](https://pkg.go.dev/builtin#string), [error](https://pkg.go.dev/builtin#error))

ArchChar returns "?" and an error. In earlier versions of Go, the returned string was used to derive the compiler and linker tool names, the default object file suffix, and the default linker output name. As of Go 1.5, those strings no longer vary by architecture; they are compile, link, .o, and a.out, respectively.

#### func [IsLocalImport](https://cs.opensource.google/go/go/+/go1.25.1:src/go/build/build.go;l=2046) [¶](https://pkg.go.dev/go/build#IsLocalImport "Go to IsLocalImport")

func IsLocalImport(path [string](https://pkg.go.dev/builtin#string)) [bool](https://pkg.go.dev/builtin#bool)

IsLocalImport reports whether the import path is a local import path, like ".", "..", "./foo", or "../foo".

### Types [¶](https://pkg.go.dev/go/build#pkg-types "Go to Types")

#### type [Context](https://cs.opensource.google/go/go/+/go1.25.1:src/go/build/build.go;l=37) [¶](https://pkg.go.dev/go/build#Context "Go to Context")

type Context struct {
	GOARCH [string](https://pkg.go.dev/builtin#string) // target architecture
	GOOS   [string](https://pkg.go.dev/builtin#string) // target operating system
	GOROOT [string](https://pkg.go.dev/builtin#string) // Go root
	GOPATH [string](https://pkg.go.dev/builtin#string) // Go paths
	// Dir is the caller's working directory, or the empty string to use
	// the current directory of the running process. In module mode, this is used
	// to locate the main module.
	//
	// If Dir is non-empty, directories passed to Import and ImportDir must
	// be absolute.
	Dir [string](https://pkg.go.dev/builtin#string)

	CgoEnabled  [bool](https://pkg.go.dev/builtin#bool)   // whether cgo files are included
	UseAllFiles [bool](https://pkg.go.dev/builtin#bool)   // use files regardless of go:build lines, file names
	Compiler    [string](https://pkg.go.dev/builtin#string) // compiler to assume when computing target paths
	// The build, tool, and release tags specify build constraints
	// that should be considered satisfied when processing go:build lines.
	// Clients creating a new context may customize BuildTags, which
	// defaults to empty, but it is usually an error to customize ToolTags or ReleaseTags.
	// ToolTags defaults to build tags appropriate to the current Go toolchain configuration.
	// ReleaseTags defaults to the list of Go releases the current release is compatible with.
	// BuildTags is not set for the Default build Context.
	// In addition to the BuildTags, ToolTags, and ReleaseTags, build constraints
	// consider the values of GOARCH and GOOS as satisfied tags.
	// The last element in ReleaseTags is assumed to be the current release.
	BuildTags   [][string](https://pkg.go.dev/builtin#string)
	ToolTags    [][string](https://pkg.go.dev/builtin#string)
	ReleaseTags [][string](https://pkg.go.dev/builtin#string)
	// The install suffix specifies a suffix to use in the name of the installation
	// directory. By default it is empty, but custom builds that need to keep
	// their outputs separate can set InstallSuffix to do so. For example, when
	// using the race detector, the go command uses InstallSuffix = "race", so
	// that on a Linux/386 system, packages are written to a directory named
	// "linux_386_race" instead of the usual "linux_386".
	InstallSuffix [string](https://pkg.go.dev/builtin#string)

	// JoinPath joins the sequence of path fragments into a single path.
	// If JoinPath is nil, Import uses filepath.Join.
	JoinPath func(elem ...[string](https://pkg.go.dev/builtin#string)) [string](https://pkg.go.dev/builtin#string)

	// SplitPathList splits the path list into a slice of individual paths.
	// If SplitPathList is nil, Import uses filepath.SplitList.
	SplitPathList func(list [string](https://pkg.go.dev/builtin#string)) [][string](https://pkg.go.dev/builtin#string)

	// IsAbsPath reports whether path is an absolute path.
	// If IsAbsPath is nil, Import uses filepath.IsAbs.
	IsAbsPath func(path [string](https://pkg.go.dev/builtin#string)) [bool](https://pkg.go.dev/builtin#bool)

	// IsDir reports whether the path names a directory.
	// If IsDir is nil, Import calls os.Stat and uses the result's IsDir method.
	IsDir func(path [string](https://pkg.go.dev/builtin#string)) [bool](https://pkg.go.dev/builtin#bool)

	// HasSubdir reports whether dir is lexically a subdirectory of
	// root, perhaps multiple levels below. It does not try to check
	// whether dir exists.
	// If so, HasSubdir sets rel to a slash-separated path that
	// can be joined to root to produce a path equivalent to dir.
	// If HasSubdir is nil, Import uses an implementation built on
	// filepath.EvalSymlinks.
	HasSubdir func(root, dir [string](https://pkg.go.dev/builtin#string)) (rel [string](https://pkg.go.dev/builtin#string), ok [bool](https://pkg.go.dev/builtin#bool))

	// ReadDir returns a slice of fs.FileInfo, sorted by Name,
	// describing the content of the named directory.
	// If ReadDir is nil, Import uses os.ReadDir.
	ReadDir func(dir [string](https://pkg.go.dev/builtin#string)) ([][fs](https://pkg.go.dev/io/fs).[FileInfo](https://pkg.go.dev/io/fs#FileInfo), [error](https://pkg.go.dev/builtin#error))

	// OpenFile opens a file (not a directory) for reading.
	// If OpenFile is nil, Import uses os.Open.
	OpenFile func(path [string](https://pkg.go.dev/builtin#string)) ([io](https://pkg.go.dev/io).[ReadCloser](https://pkg.go.dev/io#ReadCloser), [error](https://pkg.go.dev/builtin#error))
}

A Context specifies the supporting context for a build.

var Default [Context](https://pkg.go.dev/go/build#Context) = defaultContext()

Default is the default Context for builds. It uses the GOARCH, GOOS, GOROOT, and GOPATH environment variables if set, or else the compiled code's GOARCH, GOOS, and GOROOT.

#### func (*Context) [Import](https://cs.opensource.google/go/go/+/go1.25.1:src/go/build/build.go;l=576) [¶](https://pkg.go.dev/go/build#Context.Import "Go to Context.Import")

func (ctxt *[Context](https://pkg.go.dev/go/build#Context)) Import(path [string](https://pkg.go.dev/builtin#string), srcDir [string](https://pkg.go.dev/builtin#string), mode [ImportMode](https://pkg.go.dev/go/build#ImportMode)) (*[Package](https://pkg.go.dev/go/build#Package), [error](https://pkg.go.dev/builtin#error))

Import returns details about the Go package named by the import path, interpreting local import paths relative to the srcDir directory. If the path is a local import path naming a package that can be imported using a standard import path, the returned package will set p.ImportPath to that path.

In the directory containing the package, .go, .c, .h, and .s files are considered part of the package except for:

- .go files in package documentation
- files starting with _ or . (likely editor temporary files)
- files with build constraints not satisfied by the context

If an error occurs, Import returns a non-nil error and a non-nil *[Package](https://pkg.go.dev/go/build#Package) containing partial information.

#### func (*Context) [ImportDir](https://cs.opensource.google/go/go/+/go1.25.1:src/go/build/build.go;l=523) [¶](https://pkg.go.dev/go/build#Context.ImportDir "Go to Context.ImportDir")

func (ctxt *[Context](https://pkg.go.dev/go/build#Context)) ImportDir(dir [string](https://pkg.go.dev/builtin#string), mode [ImportMode](https://pkg.go.dev/go/build#ImportMode)) (*[Package](https://pkg.go.dev/go/build#Package), [error](https://pkg.go.dev/builtin#error))

ImportDir is like [Import](https://pkg.go.dev/go/build#Import) but processes the Go package found in the named directory.

#### func (*Context) [MatchFile](https://cs.opensource.google/go/go/+/go1.25.1:src/go/build/build.go;l=1420) [¶](https://pkg.go.dev/go/build#Context.MatchFile "Go to Context.MatchFile")added in go1.2

func (ctxt *[Context](https://pkg.go.dev/go/build#Context)) MatchFile(dir, name [string](https://pkg.go.dev/builtin#string)) (match [bool](https://pkg.go.dev/builtin#bool), err [error](https://pkg.go.dev/builtin#error))

MatchFile reports whether the file with the given name in the given directory matches the context and would be included in a [Package](https://pkg.go.dev/go/build#Package) created by [ImportDir](https://pkg.go.dev/go/build#ImportDir) of that directory.

MatchFile considers the name of the file and may use ctxt.OpenFile to read some or all of the file's content.

#### func (*Context) [SrcDirs](https://cs.opensource.google/go/go/+/go1.25.1:src/go/build/build.go;l=269) [¶](https://pkg.go.dev/go/build#Context.SrcDirs "Go to Context.SrcDirs")

func (ctxt *[Context](https://pkg.go.dev/go/build#Context)) SrcDirs() [][string](https://pkg.go.dev/builtin#string)

SrcDirs returns a list of package source root directories. It draws from the current Go root and Go path but omits directories that do not exist.

#### type [Directive](https://cs.opensource.google/go/go/+/go1.25.1:src/go/build/build.go;l=509) [¶](https://pkg.go.dev/go/build#Directive "Go to Directive")added in go1.21.0

type Directive struct {
	Text [string](https://pkg.go.dev/builtin#string)         // full line comment including leading slashes
	Pos  [token](https://pkg.go.dev/go/token).[Position](https://pkg.go.dev/go/token#Position) // position of comment
}

A Directive is a Go directive comment (//go:zzz...) found in a source file.

#### type [ImportMode](https://cs.opensource.google/go/go/+/go1.25.1:src/go/build/build.go;l=390) [¶](https://pkg.go.dev/go/build#ImportMode "Go to ImportMode")

type ImportMode [uint](https://pkg.go.dev/builtin#uint)

An ImportMode controls the behavior of the Import method.

const (
	// If FindOnly is set, Import stops after locating the directory
	// that should contain the sources for a package. It does not
	// read any files in the directory.
	FindOnly [ImportMode](https://pkg.go.dev/go/build#ImportMode) = 1 << [iota](https://pkg.go.dev/builtin#iota)

	// If AllowBinary is set, Import can be satisfied by a compiled
	// package object without corresponding sources.
	//
	// Deprecated:
	// The supported way to create a compiled-only package is to
	// write source code containing a //go:binary-only-package comment at
	// the top of the file. Such a package will be recognized
	// regardless of this flag setting (because it has source code)
	// and will have BinaryOnly set to true in the returned Package.
	AllowBinary

	// If ImportComment is set, parse import comments on package statements.
	// Import returns an error if it finds a comment it cannot understand
	// or finds conflicting comments in multiple source files.
	// See golang.org/s/go14customimport for more information.
	ImportComment

	// By default, Import searches vendor directories
	// that apply in the given source directory before searching
	// the GOROOT and GOPATH roots.
	// If an Import finds and returns a package using a vendor
	// directory, the resulting ImportPath is the complete path
	// to the package, including the path elements leading up
	// to and including "vendor".
	// For example, if Import("y", "x/subdir", 0) finds
	// "x/vendor/y", the returned package's ImportPath is "x/vendor/y",
	// not plain "y".
	// See golang.org/s/go15vendor for more information.
	//
	// Setting IgnoreVendor ignores vendor directories.
	//
	// In contrast to the package's ImportPath,
	// the returned package's Imports, TestImports, and XTestImports
	// are always the exact import paths from the source files:
	// Import makes no attempt to resolve or check those paths.
	IgnoreVendor
)

#### type [MultiplePackageError](https://cs.opensource.google/go/go/+/go1.25.1:src/go/build/build.go;l=540) [¶](https://pkg.go.dev/go/build#MultiplePackageError "Go to MultiplePackageError")added in go1.4

type MultiplePackageError struct {
	Dir      [string](https://pkg.go.dev/builtin#string)   // directory containing files
	Packages [][string](https://pkg.go.dev/builtin#string) // package names found
	Files    [][string](https://pkg.go.dev/builtin#string) // corresponding files: Files[i] declares package Packages[i]
}

MultiplePackageError describes a directory containing multiple buildable Go source files for multiple packages.

#### func (*MultiplePackageError) [Error](https://cs.opensource.google/go/go/+/go1.25.1:src/go/build/build.go;l=546) [¶](https://pkg.go.dev/go/build#MultiplePackageError.Error "Go to MultiplePackageError.Error")added in go1.4

func (e *[MultiplePackageError](https://pkg.go.dev/go/build#MultiplePackageError)) Error() [string](https://pkg.go.dev/builtin#string)

#### type [NoGoError](https://cs.opensource.google/go/go/+/go1.25.1:src/go/build/build.go;l=530) [¶](https://pkg.go.dev/go/build#NoGoError "Go to NoGoError")

type NoGoError struct {
	Dir [string](https://pkg.go.dev/builtin#string)
}

NoGoError is the error used by [Import](https://pkg.go.dev/go/build#Import) to describe a directory containing no buildable Go source files. (It may still contain test files, files hidden by build tags, and so on.)

#### func (*NoGoError) [Error](https://cs.opensource.google/go/go/+/go1.25.1:src/go/build/build.go;l=534) [¶](https://pkg.go.dev/go/build#NoGoError.Error "Go to NoGoError.Error")

func (e *[NoGoError](https://pkg.go.dev/go/build#NoGoError)) Error() [string](https://pkg.go.dev/builtin#string)

#### type [Package](https://cs.opensource.google/go/go/+/go1.25.1:src/go/build/build.go;l=437) [¶](https://pkg.go.dev/go/build#Package "Go to Package")

type Package struct {
	Dir           [string](https://pkg.go.dev/builtin#string)   // directory containing package sources
	Name          [string](https://pkg.go.dev/builtin#string)   // package name
	ImportComment [string](https://pkg.go.dev/builtin#string)   // path in import comment on package statement
	Doc           [string](https://pkg.go.dev/builtin#string)   // documentation synopsis
	ImportPath    [string](https://pkg.go.dev/builtin#string)   // import path of package ("" if unknown)
	Root          [string](https://pkg.go.dev/builtin#string)   // root of Go tree where this package lives
	SrcRoot       [string](https://pkg.go.dev/builtin#string)   // package source root directory ("" if unknown)
	PkgRoot       [string](https://pkg.go.dev/builtin#string)   // package install root directory ("" if unknown)
	PkgTargetRoot [string](https://pkg.go.dev/builtin#string)   // architecture dependent install root directory ("" if unknown)
	BinDir        [string](https://pkg.go.dev/builtin#string)   // command install directory ("" if unknown)
	Goroot        [bool](https://pkg.go.dev/builtin#bool)     // package found in Go root
	PkgObj        [string](https://pkg.go.dev/builtin#string)   // installed .a file
	AllTags       [][string](https://pkg.go.dev/builtin#string) // tags that can influence file selection in this directory
	ConflictDir   [string](https://pkg.go.dev/builtin#string)   // this directory shadows Dir in $GOPATH
	BinaryOnly    [bool](https://pkg.go.dev/builtin#bool)     // cannot be rebuilt from source (has //go:binary-only-package comment)
	// Source files
	GoFiles           [][string](https://pkg.go.dev/builtin#string) // .go source files (excluding CgoFiles, TestGoFiles, XTestGoFiles)
	CgoFiles          [][string](https://pkg.go.dev/builtin#string) // .go source files that import "C"
	IgnoredGoFiles    [][string](https://pkg.go.dev/builtin#string) // .go source files ignored for this build (including ignored _test.go files)
	InvalidGoFiles    [][string](https://pkg.go.dev/builtin#string) // .go source files with detected problems (parse error, wrong package name, and so on)
	IgnoredOtherFiles [][string](https://pkg.go.dev/builtin#string) // non-.go source files ignored for this build
	CFiles            [][string](https://pkg.go.dev/builtin#string) // .c source files
	CXXFiles          [][string](https://pkg.go.dev/builtin#string) // .cc, .cpp and .cxx source files
	MFiles            [][string](https://pkg.go.dev/builtin#string) // .m (Objective-C) source files
	HFiles            [][string](https://pkg.go.dev/builtin#string) // .h, .hh, .hpp and .hxx source files
	FFiles            [][string](https://pkg.go.dev/builtin#string) // .f, .F, .for and .f90 Fortran source files
	SFiles            [][string](https://pkg.go.dev/builtin#string) // .s source files
	SwigFiles         [][string](https://pkg.go.dev/builtin#string) // .swig files
	SwigCXXFiles      [][string](https://pkg.go.dev/builtin#string) // .swigcxx files
	SysoFiles         [][string](https://pkg.go.dev/builtin#string) // .syso system object files to add to archive
	// Cgo directives
	CgoCFLAGS    [][string](https://pkg.go.dev/builtin#string) // Cgo CFLAGS directives
	CgoCPPFLAGS  [][string](https://pkg.go.dev/builtin#string) // Cgo CPPFLAGS directives
	CgoCXXFLAGS  [][string](https://pkg.go.dev/builtin#string) // Cgo CXXFLAGS directives
	CgoFFLAGS    [][string](https://pkg.go.dev/builtin#string) // Cgo FFLAGS directives
	CgoLDFLAGS   [][string](https://pkg.go.dev/builtin#string) // Cgo LDFLAGS directives
	CgoPkgConfig [][string](https://pkg.go.dev/builtin#string) // Cgo pkg-config directives
	// Test information
	TestGoFiles  [][string](https://pkg.go.dev/builtin#string) // _test.go files in package
	XTestGoFiles [][string](https://pkg.go.dev/builtin#string) // _test.go files outside package
	// Go directive comments (//go:zzz...) found in source files.
	Directives      [][Directive](https://pkg.go.dev/go/build#Directive)
	TestDirectives  [][Directive](https://pkg.go.dev/go/build#Directive)
	XTestDirectives [][Directive](https://pkg.go.dev/go/build#Directive)
	// Dependency information
	Imports        [][string](https://pkg.go.dev/builtin#string)                    // import paths from GoFiles, CgoFiles
	ImportPos      map[[string](https://pkg.go.dev/builtin#string)][][token](https://pkg.go.dev/go/token).[Position](https://pkg.go.dev/go/token#Position) // line information for Imports
	TestImports    [][string](https://pkg.go.dev/builtin#string)                    // import paths from TestGoFiles
	TestImportPos  map[[string](https://pkg.go.dev/builtin#string)][][token](https://pkg.go.dev/go/token).[Position](https://pkg.go.dev/go/token#Position) // line information for TestImports
	XTestImports   [][string](https://pkg.go.dev/builtin#string)                    // import paths from XTestGoFiles
	XTestImportPos map[[string](https://pkg.go.dev/builtin#string)][][token](https://pkg.go.dev/go/token).[Position](https://pkg.go.dev/go/token#Position) // line information for XTestImports
	// //go:embed patterns found in Go source files
	// For example, if a source file says
	//	//go:embed a* b.c
	// then the list will contain those two strings as separate entries.
	// (See package embed for more details about //go:embed.)
	EmbedPatterns        [][string](https://pkg.go.dev/builtin#string)                    // patterns from GoFiles, CgoFiles
	EmbedPatternPos      map[[string](https://pkg.go.dev/builtin#string)][][token](https://pkg.go.dev/go/token).[Position](https://pkg.go.dev/go/token#Position) // line information for EmbedPatterns
	TestEmbedPatterns    [][string](https://pkg.go.dev/builtin#string)                    // patterns from TestGoFiles
	TestEmbedPatternPos  map[[string](https://pkg.go.dev/builtin#string)][][token](https://pkg.go.dev/go/token).[Position](https://pkg.go.dev/go/token#Position) // line information for TestEmbedPatterns
	XTestEmbedPatterns   [][string](https://pkg.go.dev/builtin#string)                    // patterns from XTestGoFiles
	XTestEmbedPatternPos map[[string](https://pkg.go.dev/builtin#string)][][token](https://pkg.go.dev/go/token).[Position](https://pkg.go.dev/go/token#Position) // line information for XTestEmbedPatternPos
}

A Package describes the Go package found in a directory.

#### func [Import](https://cs.opensource.google/go/go/+/go1.25.1:src/go/build/build.go;l=1534) [¶](https://pkg.go.dev/go/build#Import "Go to Import")

func Import(path, srcDir [string](https://pkg.go.dev/builtin#string), mode [ImportMode](https://pkg.go.dev/go/build#ImportMode)) (*[Package](https://pkg.go.dev/go/build#Package), [error](https://pkg.go.dev/builtin#error))

Import is shorthand for Default.Import.

#### func [ImportDir](https://cs.opensource.google/go/go/+/go1.25.1:src/go/build/build.go;l=1539) [¶](https://pkg.go.dev/go/build#ImportDir "Go to ImportDir")

func ImportDir(dir [string](https://pkg.go.dev/builtin#string), mode [ImportMode](https://pkg.go.dev/go/build#ImportMode)) (*[Package](https://pkg.go.dev/go/build#Package), [error](https://pkg.go.dev/builtin#error))

ImportDir is shorthand for Default.ImportDir.

#### func (*Package) [IsCommand](https://cs.opensource.google/go/go/+/go1.25.1:src/go/build/build.go;l=517) [¶](https://pkg.go.dev/go/build#Package.IsCommand "Go to Package.IsCommand")

func (p *[Package](https://pkg.go.dev/go/build#Package)) IsCommand() [bool](https://pkg.go.dev/builtin#bool)

IsCommand reports whether the package is considered a command to be installed (not just a library). Packages named "main" are treated as commands.

## ![](https://pkg.go.dev/static/shared/icon/insert_drive_file_gm_grey_24dp.svg) Source Files [¶](https://pkg.go.dev/go/build#section-sourcefiles "Go to Source Files")

[View all Source files](https://cs.opensource.google/go/go/+/go1.25.1:src/go/build)

- [build.go](https://cs.opensource.google/go/go/+/go1.25.1:src/go/build/build.go "build.go")
- [doc.go](https://cs.opensource.google/go/go/+/go1.25.1:src/go/build/doc.go "doc.go")
- [gc.go](https://cs.opensource.google/go/go/+/go1.25.1:src/go/build/gc.go "gc.go")
- [read.go](https://cs.opensource.google/go/go/+/go1.25.1:src/go/build/read.go "read.go")

## ![](https://pkg.go.dev/static/shared/icon/folder_gm_grey_24dp.svg) Directories [¶](https://pkg.go.dev/go/build#section-directories "Go to Directories")

|||
|---|---|
|[constraint](https://pkg.go.dev/go/build/constraint@go1.25.1)|Package constraint implements parsing and evaluation of build constraint lines.|