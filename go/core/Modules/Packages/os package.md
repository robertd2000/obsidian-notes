https://pkg.go.dev/os

### Overview [¶](https://pkg.go.dev/os#pkg-overview "Go to Overview")

- [Concurrency](https://pkg.go.dev/os#hdr-Concurrency)

Package os provides a platform-independent interface to operating system functionality. The design is Unix-like, although the error handling is Go-like; failing calls return values of type error rather than error numbers. Often, more information is available within the error. For example, if a call that takes a file name fails, such as [Open](https://pkg.go.dev/os#Open) or [Stat](https://pkg.go.dev/os#Stat), the error will include the failing file name when printed and will be of type [*PathError](https://pkg.go.dev/os#PathError), which may be unpacked for more information.

The os interface is intended to be uniform across all operating systems. Features not generally available appear in the system-specific package syscall.

Here is a simple example, opening a file and reading some of it.

file, err := os.Open("file.go") // For read access.
if err != nil {
	log.Fatal(err)
}

If the open fails, the error string will be self-explanatory, like

open file.go: no such file or directory

The file's data can then be read into a slice of bytes. Read and Write take their byte counts from the length of the argument slice.

data := make([]byte, 100)
count, err := file.Read(data)
if err != nil {
	log.Fatal(err)
}
fmt.Printf("read %d bytes: %q\n", count, data[:count])

#### Concurrency [¶](https://pkg.go.dev/os#hdr-Concurrency "Go to Concurrency")

The methods of [File](https://pkg.go.dev/os#File) correspond to file system operations. All are safe for concurrent use. The maximum number of concurrent operations on a File may be limited by the OS or the system. The number should be high, but exceeding it may degrade performance or cause other issues.

### Index [¶](https://pkg.go.dev/os#pkg-index "Go to Index")

- [Constants](https://pkg.go.dev/os#pkg-constants)
- [Variables](https://pkg.go.dev/os#pkg-variables)
- [func Chdir(dir string) error](https://pkg.go.dev/os#Chdir)
- [func Chmod(name string, mode FileMode) error](https://pkg.go.dev/os#Chmod)
- [func Chown(name string, uid, gid int) error](https://pkg.go.dev/os#Chown)
- [func Chtimes(name string, atime time.Time, mtime time.Time) error](https://pkg.go.dev/os#Chtimes)
- [func Clearenv()](https://pkg.go.dev/os#Clearenv)
- [func CopyFS(dir string, fsys fs.FS) error](https://pkg.go.dev/os#CopyFS)
- [func DirFS(dir string) fs.FS](https://pkg.go.dev/os#DirFS)
- [func Environ() []string](https://pkg.go.dev/os#Environ)
- [func Executable() (string, error)](https://pkg.go.dev/os#Executable)
- [func Exit(code int)](https://pkg.go.dev/os#Exit)
- [func Expand(s string, mapping func(string) string) string](https://pkg.go.dev/os#Expand)
- [func ExpandEnv(s string) string](https://pkg.go.dev/os#ExpandEnv)
- [func Getegid() int](https://pkg.go.dev/os#Getegid)
- [func Getenv(key string) string](https://pkg.go.dev/os#Getenv)
- [func Geteuid() int](https://pkg.go.dev/os#Geteuid)
- [func Getgid() int](https://pkg.go.dev/os#Getgid)
- [func Getgroups() ([]int, error)](https://pkg.go.dev/os#Getgroups)
- [func Getpagesize() int](https://pkg.go.dev/os#Getpagesize)
- [func Getpid() int](https://pkg.go.dev/os#Getpid)
- [func Getppid() int](https://pkg.go.dev/os#Getppid)
- [func Getuid() int](https://pkg.go.dev/os#Getuid)
- [func Getwd() (dir string, err error)](https://pkg.go.dev/os#Getwd)
- [func Hostname() (name string, err error)](https://pkg.go.dev/os#Hostname)
- [func IsExist(err error) bool](https://pkg.go.dev/os#IsExist)
- [func IsNotExist(err error) bool](https://pkg.go.dev/os#IsNotExist)
- [func IsPathSeparator(c uint8) bool](https://pkg.go.dev/os#IsPathSeparator)
- [func IsPermission(err error) bool](https://pkg.go.dev/os#IsPermission)
- [func IsTimeout(err error) bool](https://pkg.go.dev/os#IsTimeout)
- [func Lchown(name string, uid, gid int) error](https://pkg.go.dev/os#Lchown)
- [func Link(oldname, newname string) error](https://pkg.go.dev/os#Link)
- [func LookupEnv(key string) (string, bool)](https://pkg.go.dev/os#LookupEnv)
- [func Mkdir(name string, perm FileMode) error](https://pkg.go.dev/os#Mkdir)
- [func MkdirAll(path string, perm FileMode) error](https://pkg.go.dev/os#MkdirAll)
- [func MkdirTemp(dir, pattern string) (string, error)](https://pkg.go.dev/os#MkdirTemp)
- [func NewSyscallError(syscall string, err error) error](https://pkg.go.dev/os#NewSyscallError)
- [func Pipe() (r *File, w *File, err error)](https://pkg.go.dev/os#Pipe)
- [func ReadFile(name string) ([]byte, error)](https://pkg.go.dev/os#ReadFile)
- [func Readlink(name string) (string, error)](https://pkg.go.dev/os#Readlink)
- [func Remove(name string) error](https://pkg.go.dev/os#Remove)
- [func RemoveAll(path string) error](https://pkg.go.dev/os#RemoveAll)
- [func Rename(oldpath, newpath string) error](https://pkg.go.dev/os#Rename)
- [func SameFile(fi1, fi2 FileInfo) bool](https://pkg.go.dev/os#SameFile)
- [func Setenv(key, value string) error](https://pkg.go.dev/os#Setenv)
- [func Symlink(oldname, newname string) error](https://pkg.go.dev/os#Symlink)
- [func TempDir() string](https://pkg.go.dev/os#TempDir)
- [func Truncate(name string, size int64) error](https://pkg.go.dev/os#Truncate)
- [func Unsetenv(key string) error](https://pkg.go.dev/os#Unsetenv)
- [func UserCacheDir() (string, error)](https://pkg.go.dev/os#UserCacheDir)
- [func UserConfigDir() (string, error)](https://pkg.go.dev/os#UserConfigDir)
- [func UserHomeDir() (string, error)](https://pkg.go.dev/os#UserHomeDir)
- [func WriteFile(name string, data []byte, perm FileMode) error](https://pkg.go.dev/os#WriteFile)
- [type DirEntry](https://pkg.go.dev/os#DirEntry)
- - [func ReadDir(name string) ([]DirEntry, error)](https://pkg.go.dev/os#ReadDir)
- [type File](https://pkg.go.dev/os#File)
- - [func Create(name string) (*File, error)](https://pkg.go.dev/os#Create)
    - [func CreateTemp(dir, pattern string) (*File, error)](https://pkg.go.dev/os#CreateTemp)
    - [func NewFile(fd uintptr, name string) *File](https://pkg.go.dev/os#NewFile)
    - [func Open(name string) (*File, error)](https://pkg.go.dev/os#Open)
    - [func OpenFile(name string, flag int, perm FileMode) (*File, error)](https://pkg.go.dev/os#OpenFile)
    - [func OpenInRoot(dir, name string) (*File, error)](https://pkg.go.dev/os#OpenInRoot)
- - [func (f *File) Chdir() error](https://pkg.go.dev/os#File.Chdir)
    - [func (f *File) Chmod(mode FileMode) error](https://pkg.go.dev/os#File.Chmod)
    - [func (f *File) Chown(uid, gid int) error](https://pkg.go.dev/os#File.Chown)
    - [func (f *File) Close() error](https://pkg.go.dev/os#File.Close)
    - [func (f *File) Fd() uintptr](https://pkg.go.dev/os#File.Fd)
    - [func (f *File) Name() string](https://pkg.go.dev/os#File.Name)
    - [func (f *File) Read(b []byte) (n int, err error)](https://pkg.go.dev/os#File.Read)
    - [func (f *File) ReadAt(b []byte, off int64) (n int, err error)](https://pkg.go.dev/os#File.ReadAt)
    - [func (f *File) ReadDir(n int) ([]DirEntry, error)](https://pkg.go.dev/os#File.ReadDir)
    - [func (f *File) ReadFrom(r io.Reader) (n int64, err error)](https://pkg.go.dev/os#File.ReadFrom)
    - [func (f *File) Readdir(n int) ([]FileInfo, error)](https://pkg.go.dev/os#File.Readdir)
    - [func (f *File) Readdirnames(n int) (names []string, err error)](https://pkg.go.dev/os#File.Readdirnames)
    - [func (f *File) Seek(offset int64, whence int) (ret int64, err error)](https://pkg.go.dev/os#File.Seek)
    - [func (f *File) SetDeadline(t time.Time) error](https://pkg.go.dev/os#File.SetDeadline)
    - [func (f *File) SetReadDeadline(t time.Time) error](https://pkg.go.dev/os#File.SetReadDeadline)
    - [func (f *File) SetWriteDeadline(t time.Time) error](https://pkg.go.dev/os#File.SetWriteDeadline)
    - [func (f *File) Stat() (FileInfo, error)](https://pkg.go.dev/os#File.Stat)
    - [func (f *File) Sync() error](https://pkg.go.dev/os#File.Sync)
    - [func (f *File) SyscallConn() (syscall.RawConn, error)](https://pkg.go.dev/os#File.SyscallConn)
    - [func (f *File) Truncate(size int64) error](https://pkg.go.dev/os#File.Truncate)
    - [func (f *File) Write(b []byte) (n int, err error)](https://pkg.go.dev/os#File.Write)
    - [func (f *File) WriteAt(b []byte, off int64) (n int, err error)](https://pkg.go.dev/os#File.WriteAt)
    - [func (f *File) WriteString(s string) (n int, err error)](https://pkg.go.dev/os#File.WriteString)
    - [func (f *File) WriteTo(w io.Writer) (n int64, err error)](https://pkg.go.dev/os#File.WriteTo)
- [type FileInfo](https://pkg.go.dev/os#FileInfo)
- - [func Lstat(name string) (FileInfo, error)](https://pkg.go.dev/os#Lstat)
    - [func Stat(name string) (FileInfo, error)](https://pkg.go.dev/os#Stat)
- [type FileMode](https://pkg.go.dev/os#FileMode)
- [type LinkError](https://pkg.go.dev/os#LinkError)
- - [func (e *LinkError) Error() string](https://pkg.go.dev/os#LinkError.Error)
    - [func (e *LinkError) Unwrap() error](https://pkg.go.dev/os#LinkError.Unwrap)
- [type PathError](https://pkg.go.dev/os#PathError)
- [type ProcAttr](https://pkg.go.dev/os#ProcAttr)
- [type Process](https://pkg.go.dev/os#Process)
- - [func FindProcess(pid int) (*Process, error)](https://pkg.go.dev/os#FindProcess)
    - [func StartProcess(name string, argv []string, attr *ProcAttr) (*Process, error)](https://pkg.go.dev/os#StartProcess)
- - [func (p *Process) Kill() error](https://pkg.go.dev/os#Process.Kill)
    - [func (p *Process) Release() error](https://pkg.go.dev/os#Process.Release)
    - [func (p *Process) Signal(sig Signal) error](https://pkg.go.dev/os#Process.Signal)
    - [func (p *Process) Wait() (*ProcessState, error)](https://pkg.go.dev/os#Process.Wait)
- [type ProcessState](https://pkg.go.dev/os#ProcessState)
- - [func (p *ProcessState) ExitCode() int](https://pkg.go.dev/os#ProcessState.ExitCode)
    - [func (p *ProcessState) Exited() bool](https://pkg.go.dev/os#ProcessState.Exited)
    - [func (p *ProcessState) Pid() int](https://pkg.go.dev/os#ProcessState.Pid)
    - [func (p *ProcessState) String() string](https://pkg.go.dev/os#ProcessState.String)
    - [func (p *ProcessState) Success() bool](https://pkg.go.dev/os#ProcessState.Success)
    - [func (p *ProcessState) Sys() any](https://pkg.go.dev/os#ProcessState.Sys)
    - [func (p *ProcessState) SysUsage() any](https://pkg.go.dev/os#ProcessState.SysUsage)
    - [func (p *ProcessState) SystemTime() time.Duration](https://pkg.go.dev/os#ProcessState.SystemTime)
    - [func (p *ProcessState) UserTime() time.Duration](https://pkg.go.dev/os#ProcessState.UserTime)
- [type Root](https://pkg.go.dev/os#Root)
- - [func OpenRoot(name string) (*Root, error)](https://pkg.go.dev/os#OpenRoot)
- - [func (r *Root) Chmod(name string, mode FileMode) error](https://pkg.go.dev/os#Root.Chmod)
    - [func (r *Root) Chown(name string, uid, gid int) error](https://pkg.go.dev/os#Root.Chown)
    - [func (r *Root) Chtimes(name string, atime time.Time, mtime time.Time) error](https://pkg.go.dev/os#Root.Chtimes)
    - [func (r *Root) Close() error](https://pkg.go.dev/os#Root.Close)
    - [func (r *Root) Create(name string) (*File, error)](https://pkg.go.dev/os#Root.Create)
    - [func (r *Root) FS() fs.FS](https://pkg.go.dev/os#Root.FS)
    - [func (r *Root) Lchown(name string, uid, gid int) error](https://pkg.go.dev/os#Root.Lchown)
    - [func (r *Root) Link(oldname, newname string) error](https://pkg.go.dev/os#Root.Link)
    - [func (r *Root) Lstat(name string) (FileInfo, error)](https://pkg.go.dev/os#Root.Lstat)
    - [func (r *Root) Mkdir(name string, perm FileMode) error](https://pkg.go.dev/os#Root.Mkdir)
    - [func (r *Root) MkdirAll(name string, perm FileMode) error](https://pkg.go.dev/os#Root.MkdirAll)
    - [func (r *Root) Name() string](https://pkg.go.dev/os#Root.Name)
    - [func (r *Root) Open(name string) (*File, error)](https://pkg.go.dev/os#Root.Open)
    - [func (r *Root) OpenFile(name string, flag int, perm FileMode) (*File, error)](https://pkg.go.dev/os#Root.OpenFile)
    - [func (r *Root) OpenRoot(name string) (*Root, error)](https://pkg.go.dev/os#Root.OpenRoot)
    - [func (r *Root) ReadFile(name string) ([]byte, error)](https://pkg.go.dev/os#Root.ReadFile)
    - [func (r *Root) Readlink(name string) (string, error)](https://pkg.go.dev/os#Root.Readlink)
    - [func (r *Root) Remove(name string) error](https://pkg.go.dev/os#Root.Remove)
    - [func (r *Root) RemoveAll(name string) error](https://pkg.go.dev/os#Root.RemoveAll)
    - [func (r *Root) Rename(oldname, newname string) error](https://pkg.go.dev/os#Root.Rename)
    - [func (r *Root) Stat(name string) (FileInfo, error)](https://pkg.go.dev/os#Root.Stat)
    - [func (r *Root) Symlink(oldname, newname string) error](https://pkg.go.dev/os#Root.Symlink)
    - [func (r *Root) WriteFile(name string, data []byte, perm FileMode) error](https://pkg.go.dev/os#Root.WriteFile)
- [type Signal](https://pkg.go.dev/os#Signal)
- [type SyscallError](https://pkg.go.dev/os#SyscallError)
- - [func (e *SyscallError) Error() string](https://pkg.go.dev/os#SyscallError.Error)
    - [func (e *SyscallError) Timeout() bool](https://pkg.go.dev/os#SyscallError.Timeout)
    - [func (e *SyscallError) Unwrap() error](https://pkg.go.dev/os#SyscallError.Unwrap)

### Examples [¶](https://pkg.go.dev/os#pkg-examples "Go to Examples")

- [Chmod](https://pkg.go.dev/os#example-Chmod)
- [Chtimes](https://pkg.go.dev/os#example-Chtimes)
- [CreateTemp](https://pkg.go.dev/os#example-CreateTemp)
- [CreateTemp (Suffix)](https://pkg.go.dev/os#example-CreateTemp-Suffix)
- [Expand](https://pkg.go.dev/os#example-Expand)
- [ExpandEnv](https://pkg.go.dev/os#example-ExpandEnv)
- [FileMode](https://pkg.go.dev/os#example-FileMode)
- [Getenv](https://pkg.go.dev/os#example-Getenv)
- [LookupEnv](https://pkg.go.dev/os#example-LookupEnv)
- [Mkdir](https://pkg.go.dev/os#example-Mkdir)
- [MkdirAll](https://pkg.go.dev/os#example-MkdirAll)
- [MkdirTemp](https://pkg.go.dev/os#example-MkdirTemp)
- [MkdirTemp (Suffix)](https://pkg.go.dev/os#example-MkdirTemp-Suffix)
- [OpenFile](https://pkg.go.dev/os#example-OpenFile)
- [OpenFile (Append)](https://pkg.go.dev/os#example-OpenFile-Append)
- [ReadDir](https://pkg.go.dev/os#example-ReadDir)
- [ReadFile](https://pkg.go.dev/os#example-ReadFile)
- [Readlink](https://pkg.go.dev/os#example-Readlink)
- [Unsetenv](https://pkg.go.dev/os#example-Unsetenv)
- [UserCacheDir](https://pkg.go.dev/os#example-UserCacheDir)
- [UserConfigDir](https://pkg.go.dev/os#example-UserConfigDir)
- [WriteFile](https://pkg.go.dev/os#example-WriteFile)

### Constants [¶](https://pkg.go.dev/os#pkg-constants "Go to Constants")

[View Source](https://cs.opensource.google/go/go/+/go1.25.1:src/os/file.go;l=79)

const (
	// Exactly one of O_RDONLY, O_WRONLY, or O_RDWR must be specified.
	O_RDONLY [int](https://pkg.go.dev/builtin#int) = [syscall](https://pkg.go.dev/syscall).[O_RDONLY](https://pkg.go.dev/syscall#O_RDONLY) // open the file read-only.
	O_WRONLY [int](https://pkg.go.dev/builtin#int) = [syscall](https://pkg.go.dev/syscall).[O_WRONLY](https://pkg.go.dev/syscall#O_WRONLY) // open the file write-only.
	O_RDWR   [int](https://pkg.go.dev/builtin#int) = [syscall](https://pkg.go.dev/syscall).[O_RDWR](https://pkg.go.dev/syscall#O_RDWR)   // open the file read-write.
	// The remaining values may be or'ed in to control behavior.
	O_APPEND [int](https://pkg.go.dev/builtin#int) = [syscall](https://pkg.go.dev/syscall).[O_APPEND](https://pkg.go.dev/syscall#O_APPEND) // append data to the file when writing.
	O_CREATE [int](https://pkg.go.dev/builtin#int) = [syscall](https://pkg.go.dev/syscall).[O_CREAT](https://pkg.go.dev/syscall#O_CREAT)  // create a new file if none exists.
	O_EXCL   [int](https://pkg.go.dev/builtin#int) = [syscall](https://pkg.go.dev/syscall).[O_EXCL](https://pkg.go.dev/syscall#O_EXCL)   // used with O_CREATE, file must not exist.
	O_SYNC   [int](https://pkg.go.dev/builtin#int) = [syscall](https://pkg.go.dev/syscall).[O_SYNC](https://pkg.go.dev/syscall#O_SYNC)   // open for synchronous I/O.
	O_TRUNC  [int](https://pkg.go.dev/builtin#int) = [syscall](https://pkg.go.dev/syscall).[O_TRUNC](https://pkg.go.dev/syscall#O_TRUNC)  // truncate regular writable file when opened.
)

Flags to OpenFile wrapping those of the underlying system. Not all flags may be implemented on a given system.

[View Source](https://cs.opensource.google/go/go/+/go1.25.1:src/os/file.go;l=95)

const (
	SEEK_SET [int](https://pkg.go.dev/builtin#int) = 0 // seek relative to the origin of the file
	SEEK_CUR [int](https://pkg.go.dev/builtin#int) = 1 // seek relative to the current offset
	SEEK_END [int](https://pkg.go.dev/builtin#int) = 2 // seek relative to the end
)

Seek whence values.

Deprecated: Use io.SeekStart, io.SeekCurrent, and io.SeekEnd.

[View Source](https://cs.opensource.google/go/go/+/go1.25.1:src/os/path_unix.go;l=9)

const (
	PathSeparator     = '/' // OS-specific path separator
	PathListSeparator = ':' // OS-specific path list separator
)

[View Source](https://cs.opensource.google/go/go/+/go1.25.1:src/os/types.go;l=37)

const (
	// The single letters are the abbreviations
	// used by the String method's formatting.
	ModeDir        = [fs](https://pkg.go.dev/io/fs).[ModeDir](https://pkg.go.dev/io/fs#ModeDir)        // d: is a directory
	ModeAppend     = [fs](https://pkg.go.dev/io/fs).[ModeAppend](https://pkg.go.dev/io/fs#ModeAppend)     // a: append-only
	ModeExclusive  = [fs](https://pkg.go.dev/io/fs).[ModeExclusive](https://pkg.go.dev/io/fs#ModeExclusive)  // l: exclusive use
	ModeTemporary  = [fs](https://pkg.go.dev/io/fs).[ModeTemporary](https://pkg.go.dev/io/fs#ModeTemporary)  // T: temporary file; Plan 9 only
	ModeSymlink    = [fs](https://pkg.go.dev/io/fs).[ModeSymlink](https://pkg.go.dev/io/fs#ModeSymlink)    // L: symbolic link
	ModeDevice     = [fs](https://pkg.go.dev/io/fs).[ModeDevice](https://pkg.go.dev/io/fs#ModeDevice)     // D: device file
	ModeNamedPipe  = [fs](https://pkg.go.dev/io/fs).[ModeNamedPipe](https://pkg.go.dev/io/fs#ModeNamedPipe)  // p: named pipe (FIFO)
	ModeSocket     = [fs](https://pkg.go.dev/io/fs).[ModeSocket](https://pkg.go.dev/io/fs#ModeSocket)     // S: Unix domain socket
	ModeSetuid     = [fs](https://pkg.go.dev/io/fs).[ModeSetuid](https://pkg.go.dev/io/fs#ModeSetuid)     // u: setuid
	ModeSetgid     = [fs](https://pkg.go.dev/io/fs).[ModeSetgid](https://pkg.go.dev/io/fs#ModeSetgid)     // g: setgid
	ModeCharDevice = [fs](https://pkg.go.dev/io/fs).[ModeCharDevice](https://pkg.go.dev/io/fs#ModeCharDevice) // c: Unix character device, when ModeDevice is set
	ModeSticky     = [fs](https://pkg.go.dev/io/fs).[ModeSticky](https://pkg.go.dev/io/fs#ModeSticky)     // t: sticky
	ModeIrregular  = [fs](https://pkg.go.dev/io/fs).[ModeIrregular](https://pkg.go.dev/io/fs#ModeIrregular)  // ?: non-regular file; nothing else is known about this file
	// Mask for the type bits. For regular files, none will be set.
	ModeType = [fs](https://pkg.go.dev/io/fs).[ModeType](https://pkg.go.dev/io/fs#ModeType)

	ModePerm = [fs](https://pkg.go.dev/io/fs).[ModePerm](https://pkg.go.dev/io/fs#ModePerm) // Unix permission bits, 0o777
)

The defined file mode bits are the most significant bits of the [FileMode](https://pkg.go.dev/os#FileMode). The nine least-significant bits are the standard Unix rwxrwxrwx permissions. The values of these bits should be considered part of the public API and may be used in wire protocols or disk representations: they must not be changed, although new bits might be added.

[View Source](https://cs.opensource.google/go/go/+/go1.25.1:src/os/file_unix.go;l=241)

const DevNull = "/dev/null"

DevNull is the name of the operating system's “null device.” On Unix-like systems, it is "/dev/null"; on Windows, "NUL".

### Variables [¶](https://pkg.go.dev/os#pkg-variables "Go to Variables")

[View Source](https://cs.opensource.google/go/go/+/go1.25.1:src/os/error.go;l=16)

var (
	// ErrInvalid indicates an invalid argument.
	// Methods on File will return this error when the receiver is nil.
	ErrInvalid = [fs](https://pkg.go.dev/io/fs).[ErrInvalid](https://pkg.go.dev/io/fs#ErrInvalid) // "invalid argument"

	ErrPermission = [fs](https://pkg.go.dev/io/fs).[ErrPermission](https://pkg.go.dev/io/fs#ErrPermission) // "permission denied"
	ErrExist      = [fs](https://pkg.go.dev/io/fs).[ErrExist](https://pkg.go.dev/io/fs#ErrExist)      // "file already exists"
	ErrNotExist   = [fs](https://pkg.go.dev/io/fs).[ErrNotExist](https://pkg.go.dev/io/fs#ErrNotExist)   // "file does not exist"
	ErrClosed     = [fs](https://pkg.go.dev/io/fs).[ErrClosed](https://pkg.go.dev/io/fs#ErrClosed)     // "file already closed"
	ErrNoDeadline       = errNoDeadline()       // "file type does not support deadline"
	ErrDeadlineExceeded = errDeadlineExceeded() // "i/o timeout"
)

Portable analogs of some common system call errors.

Errors returned from this package may be tested against these errors with [errors.Is](https://pkg.go.dev/errors#Is).

[View Source](https://cs.opensource.google/go/go/+/go1.25.1:src/os/file.go;l=71)

var (
	Stdin  = [NewFile](https://pkg.go.dev/os#NewFile)([uintptr](https://pkg.go.dev/builtin#uintptr)([syscall](https://pkg.go.dev/syscall).[Stdin](https://pkg.go.dev/syscall#Stdin)), "/dev/stdin")
	Stdout = [NewFile](https://pkg.go.dev/os#NewFile)([uintptr](https://pkg.go.dev/builtin#uintptr)([syscall](https://pkg.go.dev/syscall).[Stdout](https://pkg.go.dev/syscall#Stdout)), "/dev/stdout")
	Stderr = [NewFile](https://pkg.go.dev/os#NewFile)([uintptr](https://pkg.go.dev/builtin#uintptr)([syscall](https://pkg.go.dev/syscall).[Stderr](https://pkg.go.dev/syscall#Stderr)), "/dev/stderr")
)

Stdin, Stdout, and Stderr are open Files pointing to the standard input, standard output, and standard error file descriptors.

Note that the Go runtime writes to standard error for panics and crashes; closing Stderr may cause those messages to go elsewhere, perhaps to a file opened later.

[View Source](https://cs.opensource.google/go/go/+/go1.25.1:src/os/proc.go;l=16)

var Args [][string](https://pkg.go.dev/builtin#string)

Args hold the command-line arguments, starting with the program name.

[View Source](https://cs.opensource.google/go/go/+/go1.25.1:src/os/exec.go;l=18)

var ErrProcessDone = [errors](https://pkg.go.dev/errors).[New](https://pkg.go.dev/errors#New)("os: process already finished")

ErrProcessDone indicates a [Process](https://pkg.go.dev/os#Process) has finished.

### Functions [¶](https://pkg.go.dev/os#pkg-functions "Go to Functions")

#### func [Chdir](https://cs.opensource.google/go/go/+/go1.25.1:src/os/file.go;l=361) [¶](https://pkg.go.dev/os#Chdir "Go to Chdir")

func Chdir(dir [string](https://pkg.go.dev/builtin#string)) [error](https://pkg.go.dev/builtin#error)

Chdir changes the current working directory to the named directory. If there is an error, it will be of type [*PathError](https://pkg.go.dev/os#PathError).

#### func [Chmod](https://cs.opensource.google/go/go/+/go1.25.1:src/os/file.go;l=647) [¶](https://pkg.go.dev/os#Chmod "Go to Chmod")

func Chmod(name [string](https://pkg.go.dev/builtin#string), mode [FileMode](https://pkg.go.dev/os#FileMode)) [error](https://pkg.go.dev/builtin#error)

Chmod changes the mode of the named file to mode. If the file is a symbolic link, it changes the mode of the link's target. If there is an error, it will be of type [*PathError](https://pkg.go.dev/os#PathError).

A different subset of the mode bits are used, depending on the operating system.

On Unix, the mode's permission bits, [ModeSetuid](https://pkg.go.dev/os#ModeSetuid), [ModeSetgid](https://pkg.go.dev/os#ModeSetgid), and [ModeSticky](https://pkg.go.dev/os#ModeSticky) are used.

On Windows, only the 0o200 bit (owner writable) of mode is used; it controls whether the file's read-only attribute is set or cleared. The other bits are currently unused. For compatibility with Go 1.12 and earlier, use a non-zero mode. Use mode 0o400 for a read-only file and 0o600 for a readable+writable file.

On Plan 9, the mode's permission bits, [ModeAppend](https://pkg.go.dev/os#ModeAppend), [ModeExclusive](https://pkg.go.dev/os#ModeExclusive), and [ModeTemporary](https://pkg.go.dev/os#ModeTemporary) are used.

Example [¶](https://pkg.go.dev/os#example-Chmod "Go to Example")

ShareFormatRun

#### func [Chown](https://cs.opensource.google/go/go/+/go1.25.1:src/os/file_posix.go;l=105) [¶](https://pkg.go.dev/os#Chown "Go to Chown")

func Chown(name [string](https://pkg.go.dev/builtin#string), uid, gid [int](https://pkg.go.dev/builtin#int)) [error](https://pkg.go.dev/builtin#error)

Chown changes the numeric uid and gid of the named file. If the file is a symbolic link, it changes the uid and gid of the link's target. A uid or gid of -1 means to not change that value. If there is an error, it will be of type [*PathError](https://pkg.go.dev/os#PathError).

On Windows or Plan 9, Chown always returns the [syscall.EWINDOWS](https://pkg.go.dev/syscall#EWINDOWS) or [syscall.EPLAN9](https://pkg.go.dev/syscall#EPLAN9) error, wrapped in [*PathError](https://pkg.go.dev/os#PathError).

#### func [Chtimes](https://cs.opensource.google/go/go/+/go1.25.1:src/os/file_posix.go;l=179) [¶](https://pkg.go.dev/os#Chtimes "Go to Chtimes")

func Chtimes(name [string](https://pkg.go.dev/builtin#string), atime [time](https://pkg.go.dev/time).[Time](https://pkg.go.dev/time#Time), mtime [time](https://pkg.go.dev/time).[Time](https://pkg.go.dev/time#Time)) [error](https://pkg.go.dev/builtin#error)

Chtimes changes the access and modification times of the named file, similar to the Unix utime() or utimes() functions. A zero [time.Time](https://pkg.go.dev/time#Time) value will leave the corresponding file time unchanged.

The underlying filesystem may truncate or round the values to a less precise time unit. If there is an error, it will be of type [*PathError](https://pkg.go.dev/os#PathError).

Example [¶](https://pkg.go.dev/os#example-Chtimes "Go to Example")

ShareFormatRun

#### func [Clearenv](https://cs.opensource.google/go/go/+/go1.25.1:src/os/env.go;l=133) [¶](https://pkg.go.dev/os#Clearenv "Go to Clearenv")

func Clearenv()

Clearenv deletes all environment variables.

#### func [CopyFS](https://cs.opensource.google/go/go/+/go1.25.1:src/os/dir.go;l=145) [¶](https://pkg.go.dev/os#CopyFS "Go to CopyFS")added in go1.23.0

func CopyFS(dir [string](https://pkg.go.dev/builtin#string), fsys [fs](https://pkg.go.dev/io/fs).[FS](https://pkg.go.dev/io/fs#FS)) [error](https://pkg.go.dev/builtin#error)

CopyFS copies the file system fsys into the directory dir, creating dir if necessary.

Files are created with mode 0o666 plus any execute permissions from the source, and directories are created with mode 0o777 (before umask).

CopyFS will not overwrite existing files. If a file name in fsys already exists in the destination, CopyFS will return an error such that errors.Is(err, fs.ErrExist) will be true.

Symbolic links in dir are followed.

New files added to fsys (including if dir is a subdirectory of fsys) while CopyFS is running are not guaranteed to be copied.

Copying stops at and returns the first error encountered.

#### func [DirFS](https://cs.opensource.google/go/go/+/go1.25.1:src/os/file.go;l=746) [¶](https://pkg.go.dev/os#DirFS "Go to DirFS")added in go1.16

func DirFS(dir [string](https://pkg.go.dev/builtin#string)) [fs](https://pkg.go.dev/io/fs).[FS](https://pkg.go.dev/io/fs#FS)

DirFS returns a file system (an fs.FS) for the tree of files rooted at the directory dir.

Note that DirFS("/prefix") only guarantees that the Open calls it makes to the operating system will begin with "/prefix": DirFS("/prefix").Open("file") is the same as os.Open("/prefix/file"). So if /prefix/file is a symbolic link pointing outside the /prefix tree, then using DirFS does not stop the access any more than using os.Open does. Additionally, the root of the fs.FS returned for a relative path, DirFS("prefix"), will be affected by later calls to Chdir. DirFS is therefore not a general substitute for a chroot-style security mechanism when the directory tree contains arbitrary content.

Use [Root.FS](https://pkg.go.dev/os#Root.FS) to obtain a fs.FS that prevents escapes from the tree via symbolic links.

The directory dir must not be "".

The result implements [io/fs.StatFS](https://pkg.go.dev/io/fs#StatFS), [io/fs.ReadFileFS](https://pkg.go.dev/io/fs#ReadFileFS), [io/fs.ReadDirFS](https://pkg.go.dev/io/fs#ReadDirFS), and [io/fs.ReadLinkFS](https://pkg.go.dev/io/fs#ReadLinkFS).

#### func [Environ](https://cs.opensource.google/go/go/+/go1.25.1:src/os/env.go;l=139) [¶](https://pkg.go.dev/os#Environ "Go to Environ")

func Environ() [][string](https://pkg.go.dev/builtin#string)

Environ returns a copy of strings representing the environment, in the form "key=value".

#### func [Executable](https://cs.opensource.google/go/go/+/go1.25.1:src/os/executable.go;l=18) [¶](https://pkg.go.dev/os#Executable "Go to Executable")added in go1.8

func Executable() ([string](https://pkg.go.dev/builtin#string), [error](https://pkg.go.dev/builtin#error))

Executable returns the path name for the executable that started the current process. There is no guarantee that the path is still pointing to the correct executable. If a symlink was used to start the process, depending on the operating system, the result might be the symlink or the path it pointed to. If a stable result is needed, [path/filepath.EvalSymlinks](https://pkg.go.dev/path/filepath#EvalSymlinks) might help.

Executable returns an absolute path unless an error occurred.

The main use case is finding resources located relative to an executable.

#### func [Exit](https://cs.opensource.google/go/go/+/go1.25.1:src/os/proc.go;l=62) [¶](https://pkg.go.dev/os#Exit "Go to Exit")

func Exit(code [int](https://pkg.go.dev/builtin#int))

Exit causes the current program to exit with the given status code. Conventionally, code zero indicates success, non-zero an error. The program terminates immediately; deferred functions are not run.

For portability, the status code should be in the range [0, 125].

#### func [Expand](https://cs.opensource.google/go/go/+/go1.25.1:src/os/env.go;l=16) [¶](https://pkg.go.dev/os#Expand "Go to Expand")

func Expand(s [string](https://pkg.go.dev/builtin#string), mapping func([string](https://pkg.go.dev/builtin#string)) [string](https://pkg.go.dev/builtin#string)) [string](https://pkg.go.dev/builtin#string)

Expand replaces ${var} or $var in the string based on the mapping function. For example, [os.ExpandEnv](https://pkg.go.dev/os#ExpandEnv)(s) is equivalent to [os.Expand](https://pkg.go.dev/os#Expand)(s, [os.Getenv](https://pkg.go.dev/os#Getenv)).

Example [¶](https://pkg.go.dev/os#example-Expand "Go to Example")

Output:

Good morning, Gopher!

ShareFormatRun

#### func [ExpandEnv](https://cs.opensource.google/go/go/+/go1.25.1:src/os/env.go;l=50) [¶](https://pkg.go.dev/os#ExpandEnv "Go to ExpandEnv")

func ExpandEnv(s [string](https://pkg.go.dev/builtin#string)) [string](https://pkg.go.dev/builtin#string)

ExpandEnv replaces ${var} or $var in the string according to the values of the current environment variables. References to undefined variables are replaced by the empty string.

Example [¶](https://pkg.go.dev/os#example-ExpandEnv "Go to Example")

Output:

gopher lives in /usr/gopher.

ShareFormatRun

#### func [Getegid](https://cs.opensource.google/go/go/+/go1.25.1:src/os/proc.go;l=46) [¶](https://pkg.go.dev/os#Getegid "Go to Getegid")

func Getegid() [int](https://pkg.go.dev/builtin#int)

Getegid returns the numeric effective group id of the caller.

On Windows, it returns -1.

#### func [Getenv](https://cs.opensource.google/go/go/+/go1.25.1:src/os/env.go;l=101) [¶](https://pkg.go.dev/os#Getenv "Go to Getenv")

func Getenv(key [string](https://pkg.go.dev/builtin#string)) [string](https://pkg.go.dev/builtin#string)

Getenv retrieves the value of the environment variable named by the key. It returns the value, which will be empty if the variable is not present. To distinguish between an empty value and an unset value, use [LookupEnv](https://pkg.go.dev/os#LookupEnv).

Example [¶](https://pkg.go.dev/os#example-Getenv "Go to Example")

Output:

gopher lives in /usr/gopher.

ShareFormatRun

#### func [Geteuid](https://cs.opensource.google/go/go/+/go1.25.1:src/os/proc.go;l=36) [¶](https://pkg.go.dev/os#Geteuid "Go to Geteuid")

func Geteuid() [int](https://pkg.go.dev/builtin#int)

Geteuid returns the numeric effective user id of the caller.

On Windows, it returns -1.

#### func [Getgid](https://cs.opensource.google/go/go/+/go1.25.1:src/os/proc.go;l=41) [¶](https://pkg.go.dev/os#Getgid "Go to Getgid")

func Getgid() [int](https://pkg.go.dev/builtin#int)

Getgid returns the numeric group id of the caller.

On Windows, it returns -1.

#### func [Getgroups](https://cs.opensource.google/go/go/+/go1.25.1:src/os/proc.go;l=52) [¶](https://pkg.go.dev/os#Getgroups "Go to Getgroups")

func Getgroups() ([][int](https://pkg.go.dev/builtin#int), [error](https://pkg.go.dev/builtin#error))

Getgroups returns a list of the numeric ids of groups that the caller belongs to.

On Windows, it returns [syscall.EWINDOWS](https://pkg.go.dev/syscall#EWINDOWS). See the [os/user](https://pkg.go.dev/os/user) package for a possible alternative.

#### func [Getpagesize](https://cs.opensource.google/go/go/+/go1.25.1:src/os/types.go;l=13) [¶](https://pkg.go.dev/os#Getpagesize "Go to Getpagesize")

func Getpagesize() [int](https://pkg.go.dev/builtin#int)

Getpagesize returns the underlying system's memory page size.

#### func [Getpid](https://cs.opensource.google/go/go/+/go1.25.1:src/os/exec.go;l=233) [¶](https://pkg.go.dev/os#Getpid "Go to Getpid")

func Getpid() [int](https://pkg.go.dev/builtin#int)

Getpid returns the process id of the caller.

#### func [Getppid](https://cs.opensource.google/go/go/+/go1.25.1:src/os/exec.go;l=236) [¶](https://pkg.go.dev/os#Getppid "Go to Getppid")

func Getppid() [int](https://pkg.go.dev/builtin#int)

Getppid returns the process id of the caller's parent.

#### func [Getuid](https://cs.opensource.google/go/go/+/go1.25.1:src/os/proc.go;l=31) [¶](https://pkg.go.dev/os#Getuid "Go to Getuid")

func Getuid() [int](https://pkg.go.dev/builtin#int)

Getuid returns the numeric user id of the caller.

On Windows, it returns -1.

#### func [Getwd](https://cs.opensource.google/go/go/+/go1.25.1:src/os/getwd.go;l=26) [¶](https://pkg.go.dev/os#Getwd "Go to Getwd")

func Getwd() (dir [string](https://pkg.go.dev/builtin#string), err [error](https://pkg.go.dev/builtin#error))

Getwd returns an absolute path name corresponding to the current directory. If the current directory can be reached via multiple paths (due to symbolic links), Getwd may return any one of them.

On Unix platforms, if the environment variable PWD provides an absolute name, and it is a name of the current directory, it is returned.

#### func [Hostname](https://cs.opensource.google/go/go/+/go1.25.1:src/os/sys.go;l=8) [¶](https://pkg.go.dev/os#Hostname "Go to Hostname")

func Hostname() (name [string](https://pkg.go.dev/builtin#string), err [error](https://pkg.go.dev/builtin#error))

Hostname returns the host name reported by the kernel.

#### func [IsExist](https://cs.opensource.google/go/go/+/go1.25.1:src/os/error.go;l=80) [¶](https://pkg.go.dev/os#IsExist "Go to IsExist")

func IsExist(err [error](https://pkg.go.dev/builtin#error)) [bool](https://pkg.go.dev/builtin#bool)

IsExist returns a boolean indicating whether its argument is known to report that a file or directory already exists. It is satisfied by [ErrExist](https://pkg.go.dev/os#ErrExist) as well as some syscall errors.

This function predates [errors.Is](https://pkg.go.dev/errors#Is). It only supports errors returned by the os package. New code should use errors.Is(err, fs.ErrExist).

#### func [IsNotExist](https://cs.opensource.google/go/go/+/go1.25.1:src/os/error.go;l=90) [¶](https://pkg.go.dev/os#IsNotExist "Go to IsNotExist")

func IsNotExist(err [error](https://pkg.go.dev/builtin#error)) [bool](https://pkg.go.dev/builtin#bool)

IsNotExist returns a boolean indicating whether its argument is known to report that a file or directory does not exist. It is satisfied by [ErrNotExist](https://pkg.go.dev/os#ErrNotExist) as well as some syscall errors.

This function predates [errors.Is](https://pkg.go.dev/errors#Is). It only supports errors returned by the os package. New code should use errors.Is(err, fs.ErrNotExist).

#### func [IsPathSeparator](https://cs.opensource.google/go/go/+/go1.25.1:src/os/path_unix.go;l=15) [¶](https://pkg.go.dev/os#IsPathSeparator "Go to IsPathSeparator")

func IsPathSeparator(c [uint8](https://pkg.go.dev/builtin#uint8)) [bool](https://pkg.go.dev/builtin#bool)

IsPathSeparator reports whether c is a directory separator character.

#### func [IsPermission](https://cs.opensource.google/go/go/+/go1.25.1:src/os/error.go;l=100) [¶](https://pkg.go.dev/os#IsPermission "Go to IsPermission")

func IsPermission(err [error](https://pkg.go.dev/builtin#error)) [bool](https://pkg.go.dev/builtin#bool)

IsPermission returns a boolean indicating whether its argument is known to report that permission is denied. It is satisfied by [ErrPermission](https://pkg.go.dev/os#ErrPermission) as well as some syscall errors.

This function predates [errors.Is](https://pkg.go.dev/errors#Is). It only supports errors returned by the os package. New code should use errors.Is(err, fs.ErrPermission).

#### func [IsTimeout](https://cs.opensource.google/go/go/+/go1.25.1:src/os/error.go;l=112) [¶](https://pkg.go.dev/os#IsTimeout "Go to IsTimeout")added in go1.10

func IsTimeout(err [error](https://pkg.go.dev/builtin#error)) [bool](https://pkg.go.dev/builtin#bool)

IsTimeout returns a boolean indicating whether its argument is known to report that a timeout occurred.

This function predates [errors.Is](https://pkg.go.dev/errors#Is), and the notion of whether an error indicates a timeout can be ambiguous. For example, the Unix error EWOULDBLOCK sometimes indicates a timeout and sometimes does not. New code should use errors.Is with a value appropriate to the call returning the error, such as [os.ErrDeadlineExceeded](https://pkg.go.dev/os#ErrDeadlineExceeded).

#### func [Lchown](https://cs.opensource.google/go/go/+/go1.25.1:src/os/file_posix.go;l=121) [¶](https://pkg.go.dev/os#Lchown "Go to Lchown")

func Lchown(name [string](https://pkg.go.dev/builtin#string), uid, gid [int](https://pkg.go.dev/builtin#int)) [error](https://pkg.go.dev/builtin#error)

Lchown changes the numeric uid and gid of the named file. If the file is a symbolic link, it changes the uid and gid of the link itself. If there is an error, it will be of type [*PathError](https://pkg.go.dev/os#PathError).

On Windows, it always returns the [syscall.EWINDOWS](https://pkg.go.dev/syscall#EWINDOWS) error, wrapped in [*PathError](https://pkg.go.dev/os#PathError).

#### func [Link](https://cs.opensource.google/go/go/+/go1.25.1:src/os/file_unix.go;l=403) [¶](https://pkg.go.dev/os#Link "Go to Link")

func Link(oldname, newname [string](https://pkg.go.dev/builtin#string)) [error](https://pkg.go.dev/builtin#error)

Link creates newname as a hard link to the oldname file. If there is an error, it will be of type *LinkError.

#### func [LookupEnv](https://cs.opensource.google/go/go/+/go1.25.1:src/os/env.go;l=112) [¶](https://pkg.go.dev/os#LookupEnv "Go to LookupEnv")added in go1.5

func LookupEnv(key [string](https://pkg.go.dev/builtin#string)) ([string](https://pkg.go.dev/builtin#string), [bool](https://pkg.go.dev/builtin#bool))

LookupEnv retrieves the value of the environment variable named by the key. If the variable is present in the environment the value (which may be empty) is returned and the boolean is true. Otherwise the returned value will be empty and the boolean will be false.

Example [¶](https://pkg.go.dev/os#example-LookupEnv "Go to Example")

Output:

SOME_KEY=value
EMPTY_KEY=
MISSING_KEY not set

ShareFormatRun

#### func [Mkdir](https://cs.opensource.google/go/go/+/go1.25.1:src/os/file.go;l=327) [¶](https://pkg.go.dev/os#Mkdir "Go to Mkdir")

func Mkdir(name [string](https://pkg.go.dev/builtin#string), perm [FileMode](https://pkg.go.dev/os#FileMode)) [error](https://pkg.go.dev/builtin#error)

Mkdir creates a new directory with the specified name and permission bits (before umask). If there is an error, it will be of type [*PathError](https://pkg.go.dev/os#PathError).

Example [¶](https://pkg.go.dev/os#example-Mkdir "Go to Example")

ShareFormatRun

#### func [MkdirAll](https://cs.opensource.google/go/go/+/go1.25.1:src/os/path.go;l=19) [¶](https://pkg.go.dev/os#MkdirAll "Go to MkdirAll")

func MkdirAll(path [string](https://pkg.go.dev/builtin#string), perm [FileMode](https://pkg.go.dev/os#FileMode)) [error](https://pkg.go.dev/builtin#error)

MkdirAll creates a directory named path, along with any necessary parents, and returns nil, or else returns an error. The permission bits perm (before umask) are used for all directories that MkdirAll creates. If path is already a directory, MkdirAll does nothing and returns nil.

Example [¶](https://pkg.go.dev/os#example-MkdirAll "Go to Example")

ShareFormatRun

#### func [MkdirTemp](https://cs.opensource.google/go/go/+/go1.25.1:src/os/tempfile.go;l=86) [¶](https://pkg.go.dev/os#MkdirTemp "Go to MkdirTemp")added in go1.16

func MkdirTemp(dir, pattern [string](https://pkg.go.dev/builtin#string)) ([string](https://pkg.go.dev/builtin#string), [error](https://pkg.go.dev/builtin#error))

MkdirTemp creates a new temporary directory in the directory dir and returns the pathname of the new directory. The new directory's name is generated by adding a random string to the end of pattern. If pattern includes a "*", the random string replaces the last "*" instead. The directory is created with mode 0o700 (before umask). If dir is the empty string, MkdirTemp uses the default directory for temporary files, as returned by TempDir. Multiple programs or goroutines calling MkdirTemp simultaneously will not choose the same directory. It is the caller's responsibility to remove the directory when it is no longer needed.

Example [¶](https://pkg.go.dev/os#example-MkdirTemp "Go to Example")

ShareFormatRun

Example (Suffix) [¶](https://pkg.go.dev/os#example-MkdirTemp-Suffix "Go to Example (Suffix)")

ShareFormatRun

#### func [NewSyscallError](https://cs.opensource.google/go/go/+/go1.25.1:src/os/error.go;l=67) [¶](https://pkg.go.dev/os#NewSyscallError "Go to NewSyscallError")

func NewSyscallError(syscall [string](https://pkg.go.dev/builtin#string), err [error](https://pkg.go.dev/builtin#error)) [error](https://pkg.go.dev/builtin#error)

NewSyscallError returns, as an error, a new [SyscallError](https://pkg.go.dev/os#SyscallError) with the given system call name and error details. As a convenience, if err is nil, NewSyscallError returns nil.

#### func [Pipe](https://cs.opensource.google/go/go/+/go1.25.1:src/os/pipe2_unix.go;l=13) [¶](https://pkg.go.dev/os#Pipe "Go to Pipe")

func Pipe() (r *[File](https://pkg.go.dev/os#File), w *[File](https://pkg.go.dev/os#File), err [error](https://pkg.go.dev/builtin#error))

Pipe returns a connected pair of Files; reads from r return bytes written to w. It returns the files and an error, if any.

#### func [ReadFile](https://cs.opensource.google/go/go/+/go1.25.1:src/os/file.go;l=867) [¶](https://pkg.go.dev/os#ReadFile "Go to ReadFile")added in go1.16

func ReadFile(name [string](https://pkg.go.dev/builtin#string)) ([][byte](https://pkg.go.dev/builtin#byte), [error](https://pkg.go.dev/builtin#error))

ReadFile reads the named file and returns the contents. A successful call returns err == nil, not err == EOF. Because ReadFile reads the whole file, it does not treat an EOF from Read as an error to be reported.

Example [¶](https://pkg.go.dev/os#example-ReadFile "Go to Example")

Output:

Hello, Gophers!

ShareFormatRun

#### func [Readlink](https://cs.opensource.google/go/go/+/go1.25.1:src/os/file.go;l=449) [¶](https://pkg.go.dev/os#Readlink "Go to Readlink")

func Readlink(name [string](https://pkg.go.dev/builtin#string)) ([string](https://pkg.go.dev/builtin#string), [error](https://pkg.go.dev/builtin#error))

Readlink returns the destination of the named symbolic link. If there is an error, it will be of type [*PathError](https://pkg.go.dev/os#PathError).

If the link destination is relative, Readlink returns the relative path without resolving it to an absolute one.

Example [¶](https://pkg.go.dev/os#example-Readlink "Go to Example")

Output:

hello.link links to hello.txt

ShareFormatRun

#### func [Remove](https://cs.opensource.google/go/go/+/go1.25.1:src/os/file_unix.go;l=356) [¶](https://pkg.go.dev/os#Remove "Go to Remove")

func Remove(name [string](https://pkg.go.dev/builtin#string)) [error](https://pkg.go.dev/builtin#error)

Remove removes the named file or (empty) directory. If there is an error, it will be of type [*PathError](https://pkg.go.dev/os#PathError).

#### func [RemoveAll](https://cs.opensource.google/go/go/+/go1.25.1:src/os/path.go;l=73) [¶](https://pkg.go.dev/os#RemoveAll "Go to RemoveAll")

func RemoveAll(path [string](https://pkg.go.dev/builtin#string)) [error](https://pkg.go.dev/builtin#error)

RemoveAll removes path and any children it contains. It removes everything it can but returns the first error it encounters. If the path does not exist, RemoveAll returns nil (no error). If there is an error, it will be of type [*PathError](https://pkg.go.dev/os#PathError).

#### func [Rename](https://cs.opensource.google/go/go/+/go1.25.1:src/os/file.go;l=440) [¶](https://pkg.go.dev/os#Rename "Go to Rename")

func Rename(oldpath, newpath [string](https://pkg.go.dev/builtin#string)) [error](https://pkg.go.dev/builtin#error)

Rename renames (moves) oldpath to newpath. If newpath already exists and is not a directory, Rename replaces it. If newpath already exists and is a directory, Rename returns an error. OS-specific restrictions may apply when oldpath and newpath are in different directories. Even within the same directory, on non-Unix platforms Rename is not an atomic operation. If there is an error, it will be of type *LinkError.

#### func [SameFile](https://cs.opensource.google/go/go/+/go1.25.1:src/os/types.go;l=69) [¶](https://pkg.go.dev/os#SameFile "Go to SameFile")

func SameFile(fi1, fi2 [FileInfo](https://pkg.go.dev/os#FileInfo)) [bool](https://pkg.go.dev/builtin#bool)

SameFile reports whether fi1 and fi2 describe the same file. For example, on Unix this means that the device and inode fields of the two underlying structures are identical; on other systems the decision may be based on the path names. SameFile only applies to results returned by this package's [Stat](https://pkg.go.dev/os#Stat). It returns false in other cases.

#### func [Setenv](https://cs.opensource.google/go/go/+/go1.25.1:src/os/env.go;l=119) [¶](https://pkg.go.dev/os#Setenv "Go to Setenv")

func Setenv(key, value [string](https://pkg.go.dev/builtin#string)) [error](https://pkg.go.dev/builtin#error)

Setenv sets the value of the environment variable named by the key. It returns an error, if any.

#### func [Symlink](https://cs.opensource.google/go/go/+/go1.25.1:src/os/file_unix.go;l=417) [¶](https://pkg.go.dev/os#Symlink "Go to Symlink")

func Symlink(oldname, newname [string](https://pkg.go.dev/builtin#string)) [error](https://pkg.go.dev/builtin#error)

Symlink creates newname as a symbolic link to oldname. On Windows, a symlink to a non-existent oldname creates a file symlink; if oldname is later created as a directory the symlink will not work. If there is an error, it will be of type *LinkError.

#### func [TempDir](https://cs.opensource.google/go/go/+/go1.25.1:src/os/file.go;l=490) [¶](https://pkg.go.dev/os#TempDir "Go to TempDir")

func TempDir() [string](https://pkg.go.dev/builtin#string)

TempDir returns the default directory to use for temporary files.

On Unix systems, it returns $TMPDIR if non-empty, else /tmp. On Windows, it uses GetTempPath, returning the first non-empty value from %TMP%, %TEMP%, %USERPROFILE%, or the Windows directory. On Plan 9, it returns /tmp.

The directory is neither guaranteed to exist nor have accessible permissions.

#### func [Truncate](https://cs.opensource.google/go/go/+/go1.25.1:src/os/file_unix.go;l=344) [¶](https://pkg.go.dev/os#Truncate "Go to Truncate")

func Truncate(name [string](https://pkg.go.dev/builtin#string), size [int64](https://pkg.go.dev/builtin#int64)) [error](https://pkg.go.dev/builtin#error)

Truncate changes the size of the named file. If the file is a symbolic link, it changes the size of the link's target. If there is an error, it will be of type [*PathError](https://pkg.go.dev/os#PathError).

#### func [Unsetenv](https://cs.opensource.google/go/go/+/go1.25.1:src/os/env.go;l=128) [¶](https://pkg.go.dev/os#Unsetenv "Go to Unsetenv")added in go1.4

func Unsetenv(key [string](https://pkg.go.dev/builtin#string)) [error](https://pkg.go.dev/builtin#error)

Unsetenv unsets a single environment variable.

Example [¶](https://pkg.go.dev/os#example-Unsetenv "Go to Example")

ShareFormatRun

#### func [UserCacheDir](https://cs.opensource.google/go/go/+/go1.25.1:src/os/file.go;l=507) [¶](https://pkg.go.dev/os#UserCacheDir "Go to UserCacheDir")added in go1.11

func UserCacheDir() ([string](https://pkg.go.dev/builtin#string), [error](https://pkg.go.dev/builtin#error))

UserCacheDir returns the default root directory to use for user-specific cached data. Users should create their own application-specific subdirectory within this one and use that.

On Unix systems, it returns $XDG_CACHE_HOME as specified by [https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html) if non-empty, else $HOME/.cache. On Darwin, it returns $HOME/Library/Caches. On Windows, it returns %LocalAppData%. On Plan 9, it returns $home/lib/cache.

If the location cannot be determined (for example, $HOME is not defined) or the path in $XDG_CACHE_HOME is relative, then it will return an error.

Example [¶](https://pkg.go.dev/os#example-UserCacheDir "Go to Example")

ShareFormatRun

#### func [UserConfigDir](https://cs.opensource.google/go/go/+/go1.25.1:src/os/file.go;l=560) [¶](https://pkg.go.dev/os#UserConfigDir "Go to UserConfigDir")added in go1.13

func UserConfigDir() ([string](https://pkg.go.dev/builtin#string), [error](https://pkg.go.dev/builtin#error))

UserConfigDir returns the default root directory to use for user-specific configuration data. Users should create their own application-specific subdirectory within this one and use that.

On Unix systems, it returns $XDG_CONFIG_HOME as specified by [https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html) if non-empty, else $HOME/.config. On Darwin, it returns $HOME/Library/Application Support. On Windows, it returns %AppData%. On Plan 9, it returns $home/lib.

If the location cannot be determined (for example, $HOME is not defined) or the path in $XDG_CONFIG_HOME is relative, then it will return an error.

Example [¶](https://pkg.go.dev/os#example-UserConfigDir "Go to Example")

ShareFormatRun

#### func [UserHomeDir](https://cs.opensource.google/go/go/+/go1.25.1:src/os/file.go;l=608) [¶](https://pkg.go.dev/os#UserHomeDir "Go to UserHomeDir")added in go1.12

func UserHomeDir() ([string](https://pkg.go.dev/builtin#string), [error](https://pkg.go.dev/builtin#error))

UserHomeDir returns the current user's home directory.

On Unix, including macOS, it returns the $HOME environment variable. On Windows, it returns %USERPROFILE%. On Plan 9, it returns the $home environment variable.

If the expected variable is not set in the environment, UserHomeDir returns either a platform-specific default value or a non-nil error.

#### func [WriteFile](https://cs.opensource.google/go/go/+/go1.25.1:src/os/file.go;l=935) [¶](https://pkg.go.dev/os#WriteFile "Go to WriteFile")added in go1.16

func WriteFile(name [string](https://pkg.go.dev/builtin#string), data [][byte](https://pkg.go.dev/builtin#byte), perm [FileMode](https://pkg.go.dev/os#FileMode)) [error](https://pkg.go.dev/builtin#error)

WriteFile writes data to the named file, creating it if necessary. If the file does not exist, WriteFile creates it with permissions perm (before umask); otherwise WriteFile truncates it before writing, without changing permissions. Since WriteFile requires multiple system calls to complete, a failure mid-operation can leave the file in a partially written state.

Example [¶](https://pkg.go.dev/os#example-WriteFile "Go to Example")

ShareFormatRun

### Types [¶](https://pkg.go.dev/os#pkg-types "Go to Types")

#### type [DirEntry](https://cs.opensource.google/go/go/+/go1.25.1:src/os/dir.go;l=85) [¶](https://pkg.go.dev/os#DirEntry "Go to DirEntry")added in go1.16

type DirEntry = [fs](https://pkg.go.dev/io/fs).[DirEntry](https://pkg.go.dev/io/fs#DirEntry)

A DirEntry is an entry read from a directory (using the [ReadDir](https://pkg.go.dev/os#ReadDir) function or a [File.ReadDir](https://pkg.go.dev/os#File.ReadDir) method).

#### func [ReadDir](https://cs.opensource.google/go/go/+/go1.25.1:src/os/dir.go;l=114) [¶](https://pkg.go.dev/os#ReadDir "Go to ReadDir")added in go1.16

func ReadDir(name [string](https://pkg.go.dev/builtin#string)) ([][DirEntry](https://pkg.go.dev/os#DirEntry), [error](https://pkg.go.dev/builtin#error))

ReadDir reads the named directory, returning all its directory entries sorted by filename. If an error occurs reading the directory, ReadDir returns the entries it was able to read before the error, along with the error.

Example [¶](https://pkg.go.dev/os#example-ReadDir "Go to Example")

ShareFormatRun

#### type [File](https://cs.opensource.google/go/go/+/go1.25.1:src/os/types.go;l=18) [¶](https://pkg.go.dev/os#File "Go to File")

type File struct {
	// contains filtered or unexported fields
}

File represents an open file descriptor.

The methods of File are safe for concurrent use.

#### func [Create](https://cs.opensource.google/go/go/+/go1.25.1:src/os/file.go;l=399) [¶](https://pkg.go.dev/os#Create "Go to Create")

func Create(name [string](https://pkg.go.dev/builtin#string)) (*[File](https://pkg.go.dev/os#File), [error](https://pkg.go.dev/builtin#error))

Create creates or truncates the named file. If the file already exists, it is truncated. If the file does not exist, it is created with mode 0o666 (before umask). If successful, methods on the returned File can be used for I/O; the associated file descriptor has mode [O_RDWR](https://pkg.go.dev/os#O_RDWR). The directory containing the file must already exist. If there is an error, it will be of type [*PathError](https://pkg.go.dev/os#PathError).

#### func [CreateTemp](https://cs.opensource.google/go/go/+/go1.25.1:src/os/tempfile.go;l=35) [¶](https://pkg.go.dev/os#CreateTemp "Go to CreateTemp")added in go1.16

func CreateTemp(dir, pattern [string](https://pkg.go.dev/builtin#string)) (*[File](https://pkg.go.dev/os#File), [error](https://pkg.go.dev/builtin#error))

CreateTemp creates a new temporary file in the directory dir, opens the file for reading and writing, and returns the resulting file. The filename is generated by taking pattern and adding a random string to the end. If pattern includes a "*", the random string replaces the last "*". The file is created with mode 0o600 (before umask). If dir is the empty string, CreateTemp uses the default directory for temporary files, as returned by [TempDir](https://pkg.go.dev/os#TempDir). Multiple programs or goroutines calling CreateTemp simultaneously will not choose the same file. The caller can use the file's Name method to find the pathname of the file. It is the caller's responsibility to remove the file when it is no longer needed.

Example [¶](https://pkg.go.dev/os#example-CreateTemp "Go to Example")

ShareFormatRun

Example (Suffix) [¶](https://pkg.go.dev/os#example-CreateTemp-Suffix "Go to Example (Suffix)")

ShareFormatRun

#### func [NewFile](https://cs.opensource.google/go/go/+/go1.25.1:src/os/file.go;l=133) [¶](https://pkg.go.dev/os#NewFile "Go to NewFile")

func NewFile(fd [uintptr](https://pkg.go.dev/builtin#uintptr), name [string](https://pkg.go.dev/builtin#string)) *[File](https://pkg.go.dev/os#File)

NewFile returns a new [File](https://pkg.go.dev/os#File) with the given file descriptor and name. The returned value will be nil if fd is not a valid file descriptor.

NewFile's behavior differs on some platforms:

- On Unix, if fd is in non-blocking mode, NewFile will attempt to return a pollable file.
- On Windows, if fd is opened for asynchronous I/O (that is, [syscall.FILE_FLAG_OVERLAPPED](https://pkg.go.dev/syscall#FILE_FLAG_OVERLAPPED) has been specified in the [syscall.CreateFile](https://pkg.go.dev/syscall#CreateFile) call), NewFile will attempt to return a pollable file by associating fd with the Go runtime I/O completion port. The I/O operations will be performed synchronously if the association fails.

Only pollable files support [File.SetDeadline](https://pkg.go.dev/os#File.SetDeadline), [File.SetReadDeadline](https://pkg.go.dev/os#File.SetReadDeadline), and [File.SetWriteDeadline](https://pkg.go.dev/os#File.SetWriteDeadline).

After passing it to NewFile, fd may become invalid under the same conditions described in the comments of [File.Fd](https://pkg.go.dev/os#File.Fd), and the same constraints apply.

#### func [Open](https://cs.opensource.google/go/go/+/go1.25.1:src/os/file.go;l=389) [¶](https://pkg.go.dev/os#Open "Go to Open")

func Open(name [string](https://pkg.go.dev/builtin#string)) (*[File](https://pkg.go.dev/os#File), [error](https://pkg.go.dev/builtin#error))

Open opens the named file for reading. If successful, methods on the returned file can be used for reading; the associated file descriptor has mode [O_RDONLY](https://pkg.go.dev/os#O_RDONLY). If there is an error, it will be of type [*PathError](https://pkg.go.dev/os#PathError).

#### func [OpenFile](https://cs.opensource.google/go/go/+/go1.25.1:src/os/file.go;l=410) [¶](https://pkg.go.dev/os#OpenFile "Go to OpenFile")

func OpenFile(name [string](https://pkg.go.dev/builtin#string), flag [int](https://pkg.go.dev/builtin#int), perm [FileMode](https://pkg.go.dev/os#FileMode)) (*[File](https://pkg.go.dev/os#File), [error](https://pkg.go.dev/builtin#error))

OpenFile is the generalized open call; most users will use Open or Create instead. It opens the named file with specified flag ([O_RDONLY](https://pkg.go.dev/os#O_RDONLY) etc.). If the file does not exist, and the [O_CREATE](https://pkg.go.dev/os#O_CREATE) flag is passed, it is created with mode perm (before umask); the containing directory must exist. If successful, methods on the returned File can be used for I/O. If there is an error, it will be of type [*PathError](https://pkg.go.dev/os#PathError).

Example [¶](https://pkg.go.dev/os#example-OpenFile "Go to Example")

ShareFormatRun

Example (Append) [¶](https://pkg.go.dev/os#example-OpenFile-Append "Go to Example (Append)")

ShareFormatRun

#### func [OpenInRoot](https://cs.opensource.google/go/go/+/go1.25.1:src/os/root.go;l=25) [¶](https://pkg.go.dev/os#OpenInRoot "Go to OpenInRoot")added in go1.24.0

func OpenInRoot(dir, name [string](https://pkg.go.dev/builtin#string)) (*[File](https://pkg.go.dev/os#File), [error](https://pkg.go.dev/builtin#error))

OpenInRoot opens the file name in the directory dir. It is equivalent to OpenRoot(dir) followed by opening the file in the root.

OpenInRoot returns an error if any component of the name references a location outside of dir.

See [Root](https://pkg.go.dev/os#Root) for details and limitations.

#### func (*File) [Chdir](https://cs.opensource.google/go/go/+/go1.25.1:src/os/file_posix.go;l=204) [¶](https://pkg.go.dev/os#File.Chdir "Go to File.Chdir")

func (f *[File](https://pkg.go.dev/os#File)) Chdir() [error](https://pkg.go.dev/builtin#error)

Chdir changes the current working directory to the file, which must be a directory. If there is an error, it will be of type [*PathError](https://pkg.go.dev/os#PathError).

#### func (*File) [Chmod](https://cs.opensource.google/go/go/+/go1.25.1:src/os/file.go;l=651) [¶](https://pkg.go.dev/os#File.Chmod "Go to File.Chmod")

func (f *[File](https://pkg.go.dev/os#File)) Chmod(mode [FileMode](https://pkg.go.dev/os#FileMode)) [error](https://pkg.go.dev/builtin#error)

Chmod changes the mode of the file to mode. If there is an error, it will be of type [*PathError](https://pkg.go.dev/os#PathError).

#### func (*File) [Chown](https://cs.opensource.google/go/go/+/go1.25.1:src/os/file_posix.go;l=136) [¶](https://pkg.go.dev/os#File.Chown "Go to File.Chown")

func (f *[File](https://pkg.go.dev/os#File)) Chown(uid, gid [int](https://pkg.go.dev/builtin#int)) [error](https://pkg.go.dev/builtin#error)

Chown changes the numeric uid and gid of the named file. If there is an error, it will be of type [*PathError](https://pkg.go.dev/os#PathError).

On Windows, it always returns the [syscall.EWINDOWS](https://pkg.go.dev/syscall#EWINDOWS) error, wrapped in [*PathError](https://pkg.go.dev/os#PathError).

#### func (*File) [Close](https://cs.opensource.google/go/go/+/go1.25.1:src/os/file_posix.go;l=19) [¶](https://pkg.go.dev/os#File.Close "Go to File.Close")

func (f *[File](https://pkg.go.dev/os#File)) Close() [error](https://pkg.go.dev/builtin#error)

Close closes the [File](https://pkg.go.dev/os#File), rendering it unusable for I/O. On files that support [File.SetDeadline](https://pkg.go.dev/os#File.SetDeadline), any pending I/O operations will be canceled and return immediately with an [ErrClosed](https://pkg.go.dev/os#ErrClosed) error. Close will return an error if it has already been called.

#### func (*File) [Fd](https://cs.opensource.google/go/go/+/go1.25.1:src/os/file.go;l=725) [¶](https://pkg.go.dev/os#File.Fd "Go to File.Fd")

func (f *[File](https://pkg.go.dev/os#File)) Fd() [uintptr](https://pkg.go.dev/builtin#uintptr)

Fd returns the system file descriptor or handle referencing the open file. If f is closed, the descriptor becomes invalid. If f is garbage collected, a finalizer may close the descriptor, making it invalid; see [runtime.SetFinalizer](https://pkg.go.dev/runtime#SetFinalizer) for more information on when a finalizer might be run.

Do not close the returned descriptor; that could cause a later close of f to close an unrelated descriptor.

Fd's behavior differs on some platforms:

- On Unix and Windows, [File.SetDeadline](https://pkg.go.dev/os#File.SetDeadline) methods will stop working.
- On Windows, the file descriptor will be disassociated from the Go runtime I/O completion port if there are no concurrent I/O operations on the file.

For most uses prefer the f.SyscallConn method.

#### func (*File) [Name](https://cs.opensource.google/go/go/+/go1.25.1:src/os/file.go;l=63) [¶](https://pkg.go.dev/os#File.Name "Go to File.Name")

func (f *[File](https://pkg.go.dev/os#File)) Name() [string](https://pkg.go.dev/builtin#string)

Name returns the name of the file as presented to Open.

It is safe to call Name after [Close].

#### func (*File) [Read](https://cs.opensource.google/go/go/+/go1.25.1:src/os/file.go;l=140) [¶](https://pkg.go.dev/os#File.Read "Go to File.Read")

func (f *[File](https://pkg.go.dev/os#File)) Read(b [][byte](https://pkg.go.dev/builtin#byte)) (n [int](https://pkg.go.dev/builtin#int), err [error](https://pkg.go.dev/builtin#error))

Read reads up to len(b) bytes from the File and stores them in b. It returns the number of bytes read and any error encountered. At end of file, Read returns 0, io.EOF.

#### func (*File) [ReadAt](https://cs.opensource.google/go/go/+/go1.25.1:src/os/file.go;l=152) [¶](https://pkg.go.dev/os#File.ReadAt "Go to File.ReadAt")

func (f *[File](https://pkg.go.dev/os#File)) ReadAt(b [][byte](https://pkg.go.dev/builtin#byte), off [int64](https://pkg.go.dev/builtin#int64)) (n [int](https://pkg.go.dev/builtin#int), err [error](https://pkg.go.dev/builtin#error))

ReadAt reads len(b) bytes from the File starting at byte offset off. It returns the number of bytes read and the error, if any. ReadAt always returns a non-nil error when n < len(b). At end of file, that error is io.EOF.

#### func (*File) [ReadDir](https://cs.opensource.google/go/go/+/go1.25.1:src/os/dir.go;l=97) [¶](https://pkg.go.dev/os#File.ReadDir "Go to File.ReadDir")added in go1.16

func (f *[File](https://pkg.go.dev/os#File)) ReadDir(n [int](https://pkg.go.dev/builtin#int)) ([][DirEntry](https://pkg.go.dev/os#DirEntry), [error](https://pkg.go.dev/builtin#error))

ReadDir reads the contents of the directory associated with the file f and returns a slice of [DirEntry](https://pkg.go.dev/os#DirEntry) values in directory order. Subsequent calls on the same file will yield later DirEntry records in the directory.

If n > 0, ReadDir returns at most n DirEntry records. In this case, if ReadDir returns an empty slice, it will return an error explaining why. At the end of a directory, the error is [io.EOF](https://pkg.go.dev/io#EOF).

If n <= 0, ReadDir returns all the DirEntry records remaining in the directory. When it succeeds, it returns a nil error (not io.EOF).

#### func (*File) [ReadFrom](https://cs.opensource.google/go/go/+/go1.25.1:src/os/file.go;l=175) [¶](https://pkg.go.dev/os#File.ReadFrom "Go to File.ReadFrom")added in go1.15

func (f *[File](https://pkg.go.dev/os#File)) ReadFrom(r [io](https://pkg.go.dev/io).[Reader](https://pkg.go.dev/io#Reader)) (n [int64](https://pkg.go.dev/builtin#int64), err [error](https://pkg.go.dev/builtin#error))

ReadFrom implements io.ReaderFrom.

#### func (*File) [Readdir](https://cs.opensource.google/go/go/+/go1.25.1:src/os/dir.go;l=40) [¶](https://pkg.go.dev/os#File.Readdir "Go to File.Readdir")

func (f *[File](https://pkg.go.dev/os#File)) Readdir(n [int](https://pkg.go.dev/builtin#int)) ([][FileInfo](https://pkg.go.dev/os#FileInfo), [error](https://pkg.go.dev/builtin#error))

Readdir reads the contents of the directory associated with file and returns a slice of up to n [FileInfo](https://pkg.go.dev/os#FileInfo) values, as would be returned by [Lstat](https://pkg.go.dev/os#Lstat), in directory order. Subsequent calls on the same file will yield further FileInfos.

If n > 0, Readdir returns at most n FileInfo structures. In this case, if Readdir returns an empty slice, it will return a non-nil error explaining why. At the end of a directory, the error is [io.EOF](https://pkg.go.dev/io#EOF).

If n <= 0, Readdir returns all the FileInfo from the directory in a single slice. In this case, if Readdir succeeds (reads all the way to the end of the directory), it returns the slice and a nil error. If it encounters an error before the end of the directory, Readdir returns the FileInfo read until that point and a non-nil error.

Most clients are better served by the more efficient ReadDir method.

#### func (*File) [Readdirnames](https://cs.opensource.google/go/go/+/go1.25.1:src/os/dir.go;l=69) [¶](https://pkg.go.dev/os#File.Readdirnames "Go to File.Readdirnames")

func (f *[File](https://pkg.go.dev/os#File)) Readdirnames(n [int](https://pkg.go.dev/builtin#int)) (names [][string](https://pkg.go.dev/builtin#string), err [error](https://pkg.go.dev/builtin#error))

Readdirnames reads the contents of the directory associated with file and returns a slice of up to n names of files in the directory, in directory order. Subsequent calls on the same file will yield further names.

If n > 0, Readdirnames returns at most n names. In this case, if Readdirnames returns an empty slice, it will return a non-nil error explaining why. At the end of a directory, the error is [io.EOF](https://pkg.go.dev/io#EOF).

If n <= 0, Readdirnames returns all the names from the directory in a single slice. In this case, if Readdirnames succeeds (reads all the way to the end of the directory), it returns the slice and a nil error. If it encounters an error before the end of the directory, Readdirnames returns the names read until that point and a non-nil error.

#### func (*File) [Seek](https://cs.opensource.google/go/go/+/go1.25.1:src/os/file.go;l=303) [¶](https://pkg.go.dev/os#File.Seek "Go to File.Seek")

func (f *[File](https://pkg.go.dev/os#File)) Seek(offset [int64](https://pkg.go.dev/builtin#int64), whence [int](https://pkg.go.dev/builtin#int)) (ret [int64](https://pkg.go.dev/builtin#int64), err [error](https://pkg.go.dev/builtin#error))

Seek sets the offset for the next Read or Write on file to offset, interpreted according to whence: 0 means relative to the origin of the file, 1 means relative to the current offset, and 2 means relative to the end. It returns the new offset and an error, if any. The behavior of Seek on a file opened with [O_APPEND](https://pkg.go.dev/os#O_APPEND) is not specified.

#### func (*File) [SetDeadline](https://cs.opensource.google/go/go/+/go1.25.1:src/os/file.go;l=677) [¶](https://pkg.go.dev/os#File.SetDeadline "Go to File.SetDeadline")added in go1.10

func (f *[File](https://pkg.go.dev/os#File)) SetDeadline(t [time](https://pkg.go.dev/time).[Time](https://pkg.go.dev/time#Time)) [error](https://pkg.go.dev/builtin#error)

SetDeadline sets the read and write deadlines for a File. It is equivalent to calling both SetReadDeadline and SetWriteDeadline.

Only some kinds of files support setting a deadline. Calls to SetDeadline for files that do not support deadlines will return ErrNoDeadline. On most systems ordinary files do not support deadlines, but pipes do.

A deadline is an absolute time after which I/O operations fail with an error instead of blocking. The deadline applies to all future and pending I/O, not just the immediately following call to Read or Write. After a deadline has been exceeded, the connection can be refreshed by setting a deadline in the future.

If the deadline is exceeded a call to Read or Write or to other I/O methods will return an error that wraps ErrDeadlineExceeded. This can be tested using errors.Is(err, os.ErrDeadlineExceeded). That error implements the Timeout method, and calling the Timeout method will return true, but there are other possible errors for which the Timeout will return true even if the deadline has not been exceeded.

An idle timeout can be implemented by repeatedly extending the deadline after successful Read or Write calls.

A zero value for t means I/O operations will not time out.

#### func (*File) [SetReadDeadline](https://cs.opensource.google/go/go/+/go1.25.1:src/os/file.go;l=685) [¶](https://pkg.go.dev/os#File.SetReadDeadline "Go to File.SetReadDeadline")added in go1.10

func (f *[File](https://pkg.go.dev/os#File)) SetReadDeadline(t [time](https://pkg.go.dev/time).[Time](https://pkg.go.dev/time#Time)) [error](https://pkg.go.dev/builtin#error)

SetReadDeadline sets the deadline for future Read calls and any currently-blocked Read call. A zero value for t means Read will not time out. Not all files support setting deadlines; see SetDeadline.

#### func (*File) [SetWriteDeadline](https://cs.opensource.google/go/go/+/go1.25.1:src/os/file.go;l=695) [¶](https://pkg.go.dev/os#File.SetWriteDeadline "Go to File.SetWriteDeadline")added in go1.10

func (f *[File](https://pkg.go.dev/os#File)) SetWriteDeadline(t [time](https://pkg.go.dev/time).[Time](https://pkg.go.dev/time#Time)) [error](https://pkg.go.dev/builtin#error)

SetWriteDeadline sets the deadline for any future Write calls and any currently-blocked Write call. Even if Write times out, it may return n > 0, indicating that some of the data was successfully written. A zero value for t means Write will not time out. Not all files support setting deadlines; see SetDeadline.

#### func (*File) [Stat](https://cs.opensource.google/go/go/+/go1.25.1:src/os/stat_unix.go;l=15) [¶](https://pkg.go.dev/os#File.Stat "Go to File.Stat")

func (f *[File](https://pkg.go.dev/os#File)) Stat() ([FileInfo](https://pkg.go.dev/os#FileInfo), [error](https://pkg.go.dev/builtin#error))

Stat returns the [FileInfo](https://pkg.go.dev/os#FileInfo) structure describing file. If there is an error, it will be of type [*PathError](https://pkg.go.dev/os#PathError).

#### func (*File) [Sync](https://cs.opensource.google/go/go/+/go1.25.1:src/os/file_posix.go;l=162) [¶](https://pkg.go.dev/os#File.Sync "Go to File.Sync")

func (f *[File](https://pkg.go.dev/os#File)) Sync() [error](https://pkg.go.dev/builtin#error)

Sync commits the current contents of the file to stable storage. Typically, this means flushing the file system's in-memory copy of recently written data to disk.

#### func (*File) [SyscallConn](https://cs.opensource.google/go/go/+/go1.25.1:src/os/file.go;l=701) [¶](https://pkg.go.dev/os#File.SyscallConn "Go to File.SyscallConn")added in go1.12

func (f *[File](https://pkg.go.dev/os#File)) SyscallConn() ([syscall](https://pkg.go.dev/syscall).[RawConn](https://pkg.go.dev/syscall#RawConn), [error](https://pkg.go.dev/builtin#error))

SyscallConn returns a raw file. This implements the syscall.Conn interface.

#### func (*File) [Truncate](https://cs.opensource.google/go/go/+/go1.25.1:src/os/file_posix.go;l=149) [¶](https://pkg.go.dev/os#File.Truncate "Go to File.Truncate")

func (f *[File](https://pkg.go.dev/os#File)) Truncate(size [int64](https://pkg.go.dev/builtin#int64)) [error](https://pkg.go.dev/builtin#error)

Truncate changes the size of the file. It does not change the I/O offset. If there is an error, it will be of type [*PathError](https://pkg.go.dev/os#PathError).

#### func (*File) [Write](https://cs.opensource.google/go/go/+/go1.25.1:src/os/file.go;l=211) [¶](https://pkg.go.dev/os#File.Write "Go to File.Write")

func (f *[File](https://pkg.go.dev/os#File)) Write(b [][byte](https://pkg.go.dev/builtin#byte)) (n [int](https://pkg.go.dev/builtin#int), err [error](https://pkg.go.dev/builtin#error))

Write writes len(b) bytes from b to the File. It returns the number of bytes written and an error, if any. Write returns a non-nil error when n != len(b).

#### func (*File) [WriteAt](https://cs.opensource.google/go/go/+/go1.25.1:src/os/file.go;l=239) [¶](https://pkg.go.dev/os#File.WriteAt "Go to File.WriteAt")

func (f *[File](https://pkg.go.dev/os#File)) WriteAt(b [][byte](https://pkg.go.dev/builtin#byte), off [int64](https://pkg.go.dev/builtin#int64)) (n [int](https://pkg.go.dev/builtin#int), err [error](https://pkg.go.dev/builtin#error))

WriteAt writes len(b) bytes to the File starting at byte offset off. It returns the number of bytes written and an error, if any. WriteAt returns a non-nil error when n != len(b).

If file was opened with the [O_APPEND](https://pkg.go.dev/os#O_APPEND) flag, WriteAt returns an error.

#### func (*File) [WriteString](https://cs.opensource.google/go/go/+/go1.25.1:src/os/file.go;l=319) [¶](https://pkg.go.dev/os#File.WriteString "Go to File.WriteString")

func (f *[File](https://pkg.go.dev/os#File)) WriteString(s [string](https://pkg.go.dev/builtin#string)) (n [int](https://pkg.go.dev/builtin#int), err [error](https://pkg.go.dev/builtin#error))

WriteString is like Write, but writes the contents of string s rather than a slice of bytes.

#### func (*File) [WriteTo](https://cs.opensource.google/go/go/+/go1.25.1:src/os/file.go;l=265) [¶](https://pkg.go.dev/os#File.WriteTo "Go to File.WriteTo")added in go1.22.0

func (f *[File](https://pkg.go.dev/os#File)) WriteTo(w [io](https://pkg.go.dev/io).[Writer](https://pkg.go.dev/io#Writer)) (n [int64](https://pkg.go.dev/builtin#int64), err [error](https://pkg.go.dev/builtin#error))

WriteTo implements io.WriterTo.

#### type [FileInfo](https://cs.opensource.google/go/go/+/go1.25.1:src/os/types.go;l=23) [¶](https://pkg.go.dev/os#FileInfo "Go to FileInfo")

type FileInfo = [fs](https://pkg.go.dev/io/fs).[FileInfo](https://pkg.go.dev/io/fs#FileInfo)

A FileInfo describes a file and is returned by [Stat](https://pkg.go.dev/os#Stat) and [Lstat](https://pkg.go.dev/os#Lstat).

#### func [Lstat](https://cs.opensource.google/go/go/+/go1.25.1:src/os/stat.go;l=24) [¶](https://pkg.go.dev/os#Lstat "Go to Lstat")

func Lstat(name [string](https://pkg.go.dev/builtin#string)) ([FileInfo](https://pkg.go.dev/os#FileInfo), [error](https://pkg.go.dev/builtin#error))

Lstat returns a [FileInfo](https://pkg.go.dev/os#FileInfo) describing the named file. If the file is a symbolic link, the returned FileInfo describes the symbolic link. Lstat makes no attempt to follow the link. If there is an error, it will be of type [*PathError](https://pkg.go.dev/os#PathError).

On Windows, if the file is a reparse point that is a surrogate for another named entity (such as a symbolic link or mounted folder), the returned FileInfo describes the reparse point, and makes no attempt to resolve it.

#### func [Stat](https://cs.opensource.google/go/go/+/go1.25.1:src/os/stat.go;l=11) [¶](https://pkg.go.dev/os#Stat "Go to Stat")

func Stat(name [string](https://pkg.go.dev/builtin#string)) ([FileInfo](https://pkg.go.dev/os#FileInfo), [error](https://pkg.go.dev/builtin#error))

Stat returns a [FileInfo](https://pkg.go.dev/os#FileInfo) describing the named file. If there is an error, it will be of type [*PathError](https://pkg.go.dev/os#PathError).

#### type [FileMode](https://cs.opensource.google/go/go/+/go1.25.1:src/os/types.go;l=30) [¶](https://pkg.go.dev/os#FileMode "Go to FileMode")

type FileMode = [fs](https://pkg.go.dev/io/fs).[FileMode](https://pkg.go.dev/io/fs#FileMode)

A FileMode represents a file's mode and permission bits. The bits have the same definition on all systems, so that information about files can be moved from one system to another portably. Not all bits apply to all systems. The only required bit is [ModeDir](https://pkg.go.dev/os#ModeDir) for directories.

Example [¶](https://pkg.go.dev/os#example-FileMode "Go to Example")

ShareFormatRun

#### type [LinkError](https://cs.opensource.google/go/go/+/go1.25.1:src/os/file.go;l=103) [¶](https://pkg.go.dev/os#LinkError "Go to LinkError")

type LinkError struct {
	Op  [string](https://pkg.go.dev/builtin#string)
	Old [string](https://pkg.go.dev/builtin#string)
	New [string](https://pkg.go.dev/builtin#string)
	Err [error](https://pkg.go.dev/builtin#error)
}

LinkError records an error during a link or symlink or rename system call and the paths that caused it.

#### func (*LinkError) [Error](https://cs.opensource.google/go/go/+/go1.25.1:src/os/file.go;l=110) [¶](https://pkg.go.dev/os#LinkError.Error "Go to LinkError.Error")

func (e *[LinkError](https://pkg.go.dev/os#LinkError)) Error() [string](https://pkg.go.dev/builtin#string)

#### func (*LinkError) [Unwrap](https://cs.opensource.google/go/go/+/go1.25.1:src/os/file.go;l=114) [¶](https://pkg.go.dev/os#LinkError.Unwrap "Go to LinkError.Unwrap")added in go1.13

func (e *[LinkError](https://pkg.go.dev/os#LinkError)) Unwrap() [error](https://pkg.go.dev/builtin#error)

#### type [PathError](https://cs.opensource.google/go/go/+/go1.25.1:src/os/error.go;l=46) [¶](https://pkg.go.dev/os#PathError "Go to PathError")

type PathError = [fs](https://pkg.go.dev/io/fs).[PathError](https://pkg.go.dev/io/fs#PathError)

PathError records an error and the operation and file path that caused it.

#### type [ProcAttr](https://cs.opensource.google/go/go/+/go1.25.1:src/os/exec.go;l=199) [¶](https://pkg.go.dev/os#ProcAttr "Go to ProcAttr")

type ProcAttr struct {
	// If Dir is non-empty, the child changes into the directory before
	// creating the process.
	Dir [string](https://pkg.go.dev/builtin#string)
	// If Env is non-nil, it gives the environment variables for the
	// new process in the form returned by Environ.
	// If it is nil, the result of Environ will be used.
	Env [][string](https://pkg.go.dev/builtin#string)
	// Files specifies the open files inherited by the new process. The
	// first three entries correspond to standard input, standard output, and
	// standard error. An implementation may support additional entries,
	// depending on the underlying operating system. A nil entry corresponds
	// to that file being closed when the process starts.
	// On Unix systems, StartProcess will change these File values
	// to blocking mode, which means that SetDeadline will stop working
	// and calling Close will not interrupt a Read or Write.
	Files []*[File](https://pkg.go.dev/os#File)

	// Operating system-specific process creation attributes.
	// Note that setting this field means that your program
	// may not execute properly or even compile on some
	// operating systems.
	Sys *[syscall](https://pkg.go.dev/syscall).[SysProcAttr](https://pkg.go.dev/syscall#SysProcAttr)
}

ProcAttr holds the attributes that will be applied to a new process started by StartProcess.

#### type [Process](https://cs.opensource.google/go/go/+/go1.25.1:src/os/exec.go;l=37) [¶](https://pkg.go.dev/os#Process "Go to Process")

type Process struct {
	Pid [int](https://pkg.go.dev/builtin#int)
	// contains filtered or unexported fields
}

Process stores the information about a process created by [StartProcess](https://pkg.go.dev/os#StartProcess).

#### func [FindProcess](https://cs.opensource.google/go/go/+/go1.25.1:src/os/exec.go;l=247) [¶](https://pkg.go.dev/os#FindProcess "Go to FindProcess")

func FindProcess(pid [int](https://pkg.go.dev/builtin#int)) (*[Process](https://pkg.go.dev/os#Process), [error](https://pkg.go.dev/builtin#error))

FindProcess looks for a running process by its pid.

The [Process](https://pkg.go.dev/os#Process) it returns can be used to obtain information about the underlying operating system process.

On Unix systems, FindProcess always succeeds and returns a Process for the given pid, regardless of whether the process exists. To test whether the process actually exists, see whether p.Signal(syscall.Signal(0)) reports an error.

#### func [StartProcess](https://cs.opensource.google/go/go/+/go1.25.1:src/os/exec.go;l=264) [¶](https://pkg.go.dev/os#StartProcess "Go to StartProcess")

func StartProcess(name [string](https://pkg.go.dev/builtin#string), argv [][string](https://pkg.go.dev/builtin#string), attr *[ProcAttr](https://pkg.go.dev/os#ProcAttr)) (*[Process](https://pkg.go.dev/os#Process), [error](https://pkg.go.dev/builtin#error))

StartProcess starts a new process with the program, arguments and attributes specified by name, argv and attr. The argv slice will become [os.Args](https://pkg.go.dev/os#Args) in the new process, so it normally starts with the program name.

If the calling goroutine has locked the operating system thread with [runtime.LockOSThread](https://pkg.go.dev/runtime#LockOSThread) and modified any inheritable OS-level thread state (for example, Linux or Plan 9 name spaces), the new process will inherit the caller's thread state.

StartProcess is a low-level interface. The [os/exec](https://pkg.go.dev/os/exec) package provides higher-level interfaces.

If there is an error, it will be of type [*PathError](https://pkg.go.dev/os#PathError).

#### func (*Process) [Kill](https://cs.opensource.google/go/go/+/go1.25.1:src/os/exec.go;l=330) [¶](https://pkg.go.dev/os#Process.Kill "Go to Process.Kill")

func (p *[Process](https://pkg.go.dev/os#Process)) Kill() [error](https://pkg.go.dev/builtin#error)

Kill causes the [Process](https://pkg.go.dev/os#Process) to exit immediately. Kill does not wait until the Process has actually exited. This only kills the Process itself, not any other processes it may have started.

#### func (*Process) [Release](https://cs.opensource.google/go/go/+/go1.25.1:src/os/exec.go;l=272) [¶](https://pkg.go.dev/os#Process.Release "Go to Process.Release")

func (p *[Process](https://pkg.go.dev/os#Process)) Release() [error](https://pkg.go.dev/builtin#error)

Release releases any resources associated with the [Process](https://pkg.go.dev/os#Process) p, rendering it unusable in the future. Release only needs to be called if [Process.Wait](https://pkg.go.dev/os#Process.Wait) is not.

#### func (*Process) [Signal](https://cs.opensource.google/go/go/+/go1.25.1:src/os/exec.go;l=345) [¶](https://pkg.go.dev/os#Process.Signal "Go to Process.Signal")

func (p *[Process](https://pkg.go.dev/os#Process)) Signal(sig [Signal](https://pkg.go.dev/os#Signal)) [error](https://pkg.go.dev/builtin#error)

Signal sends a signal to the [Process](https://pkg.go.dev/os#Process). Sending [Interrupt](https://pkg.go.dev/os#Interrupt) on Windows is not implemented.

#### func (*Process) [Wait](https://cs.opensource.google/go/go/+/go1.25.1:src/os/exec.go;l=339) [¶](https://pkg.go.dev/os#Process.Wait "Go to Process.Wait")

func (p *[Process](https://pkg.go.dev/os#Process)) Wait() (*[ProcessState](https://pkg.go.dev/os#ProcessState), [error](https://pkg.go.dev/builtin#error))

Wait waits for the [Process](https://pkg.go.dev/os#Process) to exit, and then returns a ProcessState describing its status and an error, if any. Wait releases any resources associated with the Process. On most operating systems, the Process must be a child of the current process or an error will be returned.

#### type [ProcessState](https://cs.opensource.google/go/go/+/go1.25.1:src/os/exec_posix.go;l=81) [¶](https://pkg.go.dev/os#ProcessState "Go to ProcessState")

type ProcessState struct {
	// contains filtered or unexported fields
}

ProcessState stores information about a process, as reported by Wait.

#### func (*ProcessState) [ExitCode](https://cs.opensource.google/go/go/+/go1.25.1:src/os/exec_posix.go;l=140) [¶](https://pkg.go.dev/os#ProcessState.ExitCode "Go to ProcessState.ExitCode")added in go1.12

func (p *[ProcessState](https://pkg.go.dev/os#ProcessState)) ExitCode() [int](https://pkg.go.dev/builtin#int)

ExitCode returns the exit code of the exited process, or -1 if the process hasn't exited or was terminated by a signal.

#### func (*ProcessState) [Exited](https://cs.opensource.google/go/go/+/go1.25.1:src/os/exec.go;l=362) [¶](https://pkg.go.dev/os#ProcessState.Exited "Go to ProcessState.Exited")

func (p *[ProcessState](https://pkg.go.dev/os#ProcessState)) Exited() [bool](https://pkg.go.dev/builtin#bool)

Exited reports whether the program has exited. On Unix systems this reports true if the program exited due to calling exit, but false if the program terminated due to a signal.

#### func (*ProcessState) [Pid](https://cs.opensource.google/go/go/+/go1.25.1:src/os/exec_posix.go;l=88) [¶](https://pkg.go.dev/os#ProcessState.Pid "Go to ProcessState.Pid")

func (p *[ProcessState](https://pkg.go.dev/os#ProcessState)) Pid() [int](https://pkg.go.dev/builtin#int)

Pid returns the process id of the exited process.

#### func (*ProcessState) [String](https://cs.opensource.google/go/go/+/go1.25.1:src/os/exec_posix.go;l=108) [¶](https://pkg.go.dev/os#ProcessState.String "Go to ProcessState.String")

func (p *[ProcessState](https://pkg.go.dev/os#ProcessState)) String() [string](https://pkg.go.dev/builtin#string)

#### func (*ProcessState) [Success](https://cs.opensource.google/go/go/+/go1.25.1:src/os/exec.go;l=368) [¶](https://pkg.go.dev/os#ProcessState.Success "Go to ProcessState.Success")

func (p *[ProcessState](https://pkg.go.dev/os#ProcessState)) Success() [bool](https://pkg.go.dev/builtin#bool)

Success reports whether the program exited successfully, such as with exit status 0 on Unix.

#### func (*ProcessState) [Sys](https://cs.opensource.google/go/go/+/go1.25.1:src/os/exec.go;l=375) [¶](https://pkg.go.dev/os#ProcessState.Sys "Go to ProcessState.Sys")

func (p *[ProcessState](https://pkg.go.dev/os#ProcessState)) Sys() [any](https://pkg.go.dev/builtin#any)

Sys returns system-dependent exit information about the process. Convert it to the appropriate underlying type, such as [syscall.WaitStatus](https://pkg.go.dev/syscall#WaitStatus) on Unix, to access its contents.

#### func (*ProcessState) [SysUsage](https://cs.opensource.google/go/go/+/go1.25.1:src/os/exec.go;l=384) [¶](https://pkg.go.dev/os#ProcessState.SysUsage "Go to ProcessState.SysUsage")

func (p *[ProcessState](https://pkg.go.dev/os#ProcessState)) SysUsage() [any](https://pkg.go.dev/builtin#any)

SysUsage returns system-dependent resource usage information about the exited process. Convert it to the appropriate underlying type, such as [*syscall.Rusage](https://pkg.go.dev/syscall#Rusage) on Unix, to access its contents. (On Unix, *syscall.Rusage matches struct rusage as defined in the getrusage(2) manual page.)

#### func (*ProcessState) [SystemTime](https://cs.opensource.google/go/go/+/go1.25.1:src/os/exec.go;l=355) [¶](https://pkg.go.dev/os#ProcessState.SystemTime "Go to ProcessState.SystemTime")

func (p *[ProcessState](https://pkg.go.dev/os#ProcessState)) SystemTime() [time](https://pkg.go.dev/time).[Duration](https://pkg.go.dev/time#Duration)

SystemTime returns the system CPU time of the exited process and its children.

#### func (*ProcessState) [UserTime](https://cs.opensource.google/go/go/+/go1.25.1:src/os/exec.go;l=350) [¶](https://pkg.go.dev/os#ProcessState.UserTime "Go to ProcessState.UserTime")

func (p *[ProcessState](https://pkg.go.dev/os#ProcessState)) UserTime() [time](https://pkg.go.dev/time).[Duration](https://pkg.go.dev/time#Duration)

UserTime returns the user CPU time of the exited process and its children.

#### type [Root](https://cs.opensource.google/go/go/+/go1.25.1:src/os/root.go;l=68) [¶](https://pkg.go.dev/os#Root "Go to Root")added in go1.24.0

type Root struct {
	// contains filtered or unexported fields
}

Root may be used to only access files within a single directory tree.

Methods on Root can only access files and directories beneath a root directory. If any component of a file name passed to a method of Root references a location outside the root, the method returns an error. File names may reference the directory itself (.).

Methods on Root will follow symbolic links, but symbolic links may not reference a location outside the root. Symbolic links must not be absolute.

Methods on Root do not prohibit traversal of filesystem boundaries, Linux bind mounts, /proc special files, or access to Unix device files.

Methods on Root are safe to be used from multiple goroutines simultaneously.

On most platforms, creating a Root opens a file descriptor or handle referencing the directory. If the directory is moved, methods on Root reference the original directory in its new location.

Root's behavior differs on some platforms:

- When GOOS=windows, file names may not reference Windows reserved device names such as NUL and COM1.
- On Unix, [Root.Chmod](https://pkg.go.dev/os#Root.Chmod), [Root.Chown](https://pkg.go.dev/os#Root.Chown), and [Root.Chtimes](https://pkg.go.dev/os#Root.Chtimes) are vulnerable to a race condition. If the target of the operation is changed from a regular file to a symlink while the operation is in progress, the operation may be performed on the link rather than the link target.
- When GOOS=js, Root is vulnerable to TOCTOU (time-of-check-time-of-use) attacks in symlink validation, and cannot ensure that operations will not escape the root.
- When GOOS=plan9 or GOOS=js, Root does not track directories across renames. On these platforms, a Root references a directory name, not a file descriptor.
- WASI preview 1 (GOOS=wasip1) does not support [Root.Chmod](https://pkg.go.dev/os#Root.Chmod).

#### func [OpenRoot](https://cs.opensource.google/go/go/+/go1.25.1:src/os/root.go;l=82) [¶](https://pkg.go.dev/os#OpenRoot "Go to OpenRoot")added in go1.24.0

func OpenRoot(name [string](https://pkg.go.dev/builtin#string)) (*[Root](https://pkg.go.dev/os#Root), [error](https://pkg.go.dev/builtin#error))

OpenRoot opens the named directory. It follows symbolic links in the directory name. If there is an error, it will be of type [*PathError](https://pkg.go.dev/os#PathError).

#### func (*Root) [Chmod](https://cs.opensource.google/go/go/+/go1.25.1:src/os/root.go;l=139) [¶](https://pkg.go.dev/os#Root.Chmod "Go to Root.Chmod")added in go1.25.0

func (r *[Root](https://pkg.go.dev/os#Root)) Chmod(name [string](https://pkg.go.dev/builtin#string), mode [FileMode](https://pkg.go.dev/os#FileMode)) [error](https://pkg.go.dev/builtin#error)

Chmod changes the mode of the named file in the root to mode. See [Chmod](https://pkg.go.dev/os#Chmod) for more details.

#### func (*Root) [Chown](https://cs.opensource.google/go/go/+/go1.25.1:src/os/root.go;l=170) [¶](https://pkg.go.dev/os#Root.Chown "Go to Root.Chown")added in go1.25.0

func (r *[Root](https://pkg.go.dev/os#Root)) Chown(name [string](https://pkg.go.dev/builtin#string), uid, gid [int](https://pkg.go.dev/builtin#int)) [error](https://pkg.go.dev/builtin#error)

Chown changes the numeric uid and gid of the named file in the root. See [Chown](https://pkg.go.dev/os#Chown) for more details.

#### func (*Root) [Chtimes](https://cs.opensource.google/go/go/+/go1.25.1:src/os/root.go;l=182) [¶](https://pkg.go.dev/os#Root.Chtimes "Go to Root.Chtimes")added in go1.25.0

func (r *[Root](https://pkg.go.dev/os#Root)) Chtimes(name [string](https://pkg.go.dev/builtin#string), atime [time](https://pkg.go.dev/time).[Time](https://pkg.go.dev/time#Time), mtime [time](https://pkg.go.dev/time).[Time](https://pkg.go.dev/time#Time)) [error](https://pkg.go.dev/builtin#error)

Chtimes changes the access and modification times of the named file in the root. See [Chtimes](https://pkg.go.dev/os#Chtimes) for more details.

#### func (*Root) [Close](https://cs.opensource.google/go/go/+/go1.25.1:src/os/root.go;l=96) [¶](https://pkg.go.dev/os#Root.Close "Go to Root.Close")added in go1.24.0

func (r *[Root](https://pkg.go.dev/os#Root)) Close() [error](https://pkg.go.dev/builtin#error)

Close closes the Root. After Close is called, methods on Root return errors.

#### func (*Root) [Create](https://cs.opensource.google/go/go/+/go1.25.1:src/os/root.go;l=108) [¶](https://pkg.go.dev/os#Root.Create "Go to Root.Create")added in go1.24.0

func (r *[Root](https://pkg.go.dev/os#Root)) Create(name [string](https://pkg.go.dev/builtin#string)) (*[File](https://pkg.go.dev/os#File), [error](https://pkg.go.dev/builtin#error))

Create creates or truncates the named file in the root. See [Create](https://pkg.go.dev/os#Create) for more details.

#### func (*Root) [FS](https://cs.opensource.google/go/go/+/go1.25.1:src/os/root.go;l=357) [¶](https://pkg.go.dev/os#Root.FS "Go to Root.FS")added in go1.24.0

func (r *[Root](https://pkg.go.dev/os#Root)) FS() [fs](https://pkg.go.dev/io/fs).[FS](https://pkg.go.dev/io/fs#FS)

FS returns a file system (an fs.FS) for the tree of files in the root.

The result implements [io/fs.StatFS](https://pkg.go.dev/io/fs#StatFS), [io/fs.ReadFileFS](https://pkg.go.dev/io/fs#ReadFileFS), [io/fs.ReadDirFS](https://pkg.go.dev/io/fs#ReadDirFS), and [io/fs.ReadLinkFS](https://pkg.go.dev/io/fs#ReadLinkFS).

#### func (*Root) [Lchown](https://cs.opensource.google/go/go/+/go1.25.1:src/os/root.go;l=176) [¶](https://pkg.go.dev/os#Root.Lchown "Go to Root.Lchown")added in go1.25.0

func (r *[Root](https://pkg.go.dev/os#Root)) Lchown(name [string](https://pkg.go.dev/builtin#string), uid, gid [int](https://pkg.go.dev/builtin#int)) [error](https://pkg.go.dev/builtin#error)

Lchown changes the numeric uid and gid of the named file in the root. See [Lchown](https://pkg.go.dev/os#Lchown) for more details.

#### func (*Root) [Link](https://cs.opensource.google/go/go/+/go1.25.1:src/os/root.go;l=235) [¶](https://pkg.go.dev/os#Root.Link "Go to Root.Link")added in go1.25.0

func (r *[Root](https://pkg.go.dev/os#Root)) Link(oldname, newname [string](https://pkg.go.dev/builtin#string)) [error](https://pkg.go.dev/builtin#error)

Link creates newname as a hard link to the oldname file. Both paths are relative to the root. See [Link](https://pkg.go.dev/os#Link) for more details.

If oldname is a symbolic link, Link creates new link to oldname and not its target. This behavior may differ from that of [Link](https://pkg.go.dev/os#Link) on some platforms.

When GOOS=js, Link returns an error if oldname is a symbolic link.

#### func (*Root) [Lstat](https://cs.opensource.google/go/go/+/go1.25.1:src/os/root.go;l=209) [¶](https://pkg.go.dev/os#Root.Lstat "Go to Root.Lstat")added in go1.24.0

func (r *[Root](https://pkg.go.dev/os#Root)) Lstat(name [string](https://pkg.go.dev/builtin#string)) ([FileInfo](https://pkg.go.dev/os#FileInfo), [error](https://pkg.go.dev/builtin#error))

Lstat returns a [FileInfo](https://pkg.go.dev/os#FileInfo) describing the named file in the root. If the file is a symbolic link, the returned FileInfo describes the symbolic link. See [Lstat](https://pkg.go.dev/os#Lstat) for more details.

#### func (*Root) [Mkdir](https://cs.opensource.google/go/go/+/go1.25.1:src/os/root.go;l=149) [¶](https://pkg.go.dev/os#Root.Mkdir "Go to Root.Mkdir")added in go1.24.0

func (r *[Root](https://pkg.go.dev/os#Root)) Mkdir(name [string](https://pkg.go.dev/builtin#string), perm [FileMode](https://pkg.go.dev/os#FileMode)) [error](https://pkg.go.dev/builtin#error)

Mkdir creates a new directory in the root with the specified name and permission bits (before umask). See [Mkdir](https://pkg.go.dev/os#Mkdir) for more details.

If perm contains bits other than the nine least-significant bits (0o777), Mkdir returns an error.

#### func (*Root) [MkdirAll](https://cs.opensource.google/go/go/+/go1.25.1:src/os/root.go;l=161) [¶](https://pkg.go.dev/os#Root.MkdirAll "Go to Root.MkdirAll")added in go1.25.0

func (r *[Root](https://pkg.go.dev/os#Root)) MkdirAll(name [string](https://pkg.go.dev/builtin#string), perm [FileMode](https://pkg.go.dev/os#FileMode)) [error](https://pkg.go.dev/builtin#error)

MkdirAll creates a new directory in the root, along with any necessary parents. See [MkdirAll](https://pkg.go.dev/os#MkdirAll) for more details.

If perm contains bits other than the nine least-significant bits (0o777), MkdirAll returns an error.

#### func (*Root) [Name](https://cs.opensource.google/go/go/+/go1.25.1:src/os/root.go;l=90) [¶](https://pkg.go.dev/os#Root.Name "Go to Root.Name")added in go1.24.0

func (r *[Root](https://pkg.go.dev/os#Root)) Name() [string](https://pkg.go.dev/builtin#string)

Name returns the name of the directory presented to OpenRoot.

It is safe to call Name after [Close].

#### func (*Root) [Open](https://cs.opensource.google/go/go/+/go1.25.1:src/os/root.go;l=102) [¶](https://pkg.go.dev/os#Root.Open "Go to Root.Open")added in go1.24.0

func (r *[Root](https://pkg.go.dev/os#Root)) Open(name [string](https://pkg.go.dev/builtin#string)) (*[File](https://pkg.go.dev/os#File), [error](https://pkg.go.dev/builtin#error))

Open opens the named file in the root for reading. See [Open](https://pkg.go.dev/os#Open) for more details.

#### func (*Root) [OpenFile](https://cs.opensource.google/go/go/+/go1.25.1:src/os/root.go;l=117) [¶](https://pkg.go.dev/os#Root.OpenFile "Go to Root.OpenFile")added in go1.24.0

func (r *[Root](https://pkg.go.dev/os#Root)) OpenFile(name [string](https://pkg.go.dev/builtin#string), flag [int](https://pkg.go.dev/builtin#int), perm [FileMode](https://pkg.go.dev/os#FileMode)) (*[File](https://pkg.go.dev/os#File), [error](https://pkg.go.dev/builtin#error))

OpenFile opens the named file in the root. See [OpenFile](https://pkg.go.dev/os#OpenFile) for more details.

If perm contains bits other than the nine least-significant bits (0o777), OpenFile returns an error.

#### func (*Root) [OpenRoot](https://cs.opensource.google/go/go/+/go1.25.1:src/os/root.go;l=132) [¶](https://pkg.go.dev/os#Root.OpenRoot "Go to Root.OpenRoot")added in go1.24.0

func (r *[Root](https://pkg.go.dev/os#Root)) OpenRoot(name [string](https://pkg.go.dev/builtin#string)) (*[Root](https://pkg.go.dev/os#Root), [error](https://pkg.go.dev/builtin#error))

OpenRoot opens the named directory in the root. If there is an error, it will be of type [*PathError](https://pkg.go.dev/os#PathError).

#### func (*Root) [ReadFile](https://cs.opensource.google/go/go/+/go1.25.1:src/os/root.go;l=253) [¶](https://pkg.go.dev/os#Root.ReadFile "Go to Root.ReadFile")added in go1.25.0

func (r *[Root](https://pkg.go.dev/os#Root)) ReadFile(name [string](https://pkg.go.dev/builtin#string)) ([][byte](https://pkg.go.dev/builtin#byte), [error](https://pkg.go.dev/builtin#error))

ReadFile reads the named file in the root and returns its contents. See [ReadFile](https://pkg.go.dev/os#ReadFile) for more details.

#### func (*Root) [Readlink](https://cs.opensource.google/go/go/+/go1.25.1:src/os/root.go;l=216) [¶](https://pkg.go.dev/os#Root.Readlink "Go to Root.Readlink")added in go1.25.0

func (r *[Root](https://pkg.go.dev/os#Root)) Readlink(name [string](https://pkg.go.dev/builtin#string)) ([string](https://pkg.go.dev/builtin#string), [error](https://pkg.go.dev/builtin#error))

Readlink returns the destination of the named symbolic link in the root. See [Readlink](https://pkg.go.dev/os#Readlink) for more details.

#### func (*Root) [Remove](https://cs.opensource.google/go/go/+/go1.25.1:src/os/root.go;l=188) [¶](https://pkg.go.dev/os#Root.Remove "Go to Root.Remove")added in go1.24.0

func (r *[Root](https://pkg.go.dev/os#Root)) Remove(name [string](https://pkg.go.dev/builtin#string)) [error](https://pkg.go.dev/builtin#error)

Remove removes the named file or (empty) directory in the root. See [Remove](https://pkg.go.dev/os#Remove) for more details.

#### func (*Root) [RemoveAll](https://cs.opensource.google/go/go/+/go1.25.1:src/os/root.go;l=194) [¶](https://pkg.go.dev/os#Root.RemoveAll "Go to Root.RemoveAll")added in go1.25.0

func (r *[Root](https://pkg.go.dev/os#Root)) RemoveAll(name [string](https://pkg.go.dev/builtin#string)) [error](https://pkg.go.dev/builtin#error)

RemoveAll removes the named file or directory and any children that it contains. See [RemoveAll](https://pkg.go.dev/os#RemoveAll) for more details.

#### func (*Root) [Rename](https://cs.opensource.google/go/go/+/go1.25.1:src/os/root.go;l=223) [¶](https://pkg.go.dev/os#Root.Rename "Go to Root.Rename")added in go1.25.0

func (r *[Root](https://pkg.go.dev/os#Root)) Rename(oldname, newname [string](https://pkg.go.dev/builtin#string)) [error](https://pkg.go.dev/builtin#error)

Rename renames (moves) oldname to newname. Both paths are relative to the root. See [Rename](https://pkg.go.dev/os#Rename) for more details.

#### func (*Root) [Stat](https://cs.opensource.google/go/go/+/go1.25.1:src/os/root.go;l=200) [¶](https://pkg.go.dev/os#Root.Stat "Go to Root.Stat")added in go1.24.0

func (r *[Root](https://pkg.go.dev/os#Root)) Stat(name [string](https://pkg.go.dev/builtin#string)) ([FileInfo](https://pkg.go.dev/os#FileInfo), [error](https://pkg.go.dev/builtin#error))

Stat returns a [FileInfo](https://pkg.go.dev/os#FileInfo) describing the named file in the root. See [Stat](https://pkg.go.dev/os#Stat) for more details.

#### func (*Root) [Symlink](https://cs.opensource.google/go/go/+/go1.25.1:src/os/root.go;l=247) [¶](https://pkg.go.dev/os#Root.Symlink "Go to Root.Symlink")added in go1.25.0

func (r *[Root](https://pkg.go.dev/os#Root)) Symlink(oldname, newname [string](https://pkg.go.dev/builtin#string)) [error](https://pkg.go.dev/builtin#error)

Symlink creates newname as a symbolic link to oldname. See [Symlink](https://pkg.go.dev/os#Symlink) for more details.

Symlink does not validate oldname, which may reference a location outside the root.

On Windows, a directory link is created if oldname references a directory within the root. Otherwise a file link is created.

#### func (*Root) [WriteFile](https://cs.opensource.google/go/go/+/go1.25.1:src/os/root.go;l=264) [¶](https://pkg.go.dev/os#Root.WriteFile "Go to Root.WriteFile")added in go1.25.0

func (r *[Root](https://pkg.go.dev/os#Root)) WriteFile(name [string](https://pkg.go.dev/builtin#string), data [][byte](https://pkg.go.dev/builtin#byte), perm [FileMode](https://pkg.go.dev/os#FileMode)) [error](https://pkg.go.dev/builtin#error)

WriteFile writes data to the named file in the root, creating it if necessary. See [WriteFile](https://pkg.go.dev/os#WriteFile) for more details.

#### type [Signal](https://cs.opensource.google/go/go/+/go1.25.1:src/os/exec.go;l=227) [¶](https://pkg.go.dev/os#Signal "Go to Signal")

type Signal interface {
	String() [string](https://pkg.go.dev/builtin#string)
	Signal() // to distinguish from other Stringers
}

A Signal represents an operating system signal. The usual underlying implementation is operating system-dependent: on Unix it is syscall.Signal.

var (
	Interrupt [Signal](https://pkg.go.dev/os#Signal) = [syscall](https://pkg.go.dev/syscall).[SIGINT](https://pkg.go.dev/syscall#SIGINT)
	Kill      [Signal](https://pkg.go.dev/os#Signal) = [syscall](https://pkg.go.dev/syscall).[SIGKILL](https://pkg.go.dev/syscall#SIGKILL)
)

The only signal values guaranteed to be present in the os package on all systems are os.Interrupt (send the process an interrupt) and os.Kill (force the process to exit). On Windows, sending os.Interrupt to a process with os.Process.Signal is not implemented; it will return an error instead of sending a signal.

#### type [SyscallError](https://cs.opensource.google/go/go/+/go1.25.1:src/os/error.go;l=49) [¶](https://pkg.go.dev/os#SyscallError "Go to SyscallError")

type SyscallError struct {
	Syscall [string](https://pkg.go.dev/builtin#string)
	Err     [error](https://pkg.go.dev/builtin#error)
}

SyscallError records an error from a specific system call.

#### func (*SyscallError) [Error](https://cs.opensource.google/go/go/+/go1.25.1:src/os/error.go;l=54) [¶](https://pkg.go.dev/os#SyscallError.Error "Go to SyscallError.Error")

func (e *[SyscallError](https://pkg.go.dev/os#SyscallError)) Error() [string](https://pkg.go.dev/builtin#string)

#### func (*SyscallError) [Timeout](https://cs.opensource.google/go/go/+/go1.25.1:src/os/error.go;l=59) [¶](https://pkg.go.dev/os#SyscallError.Timeout "Go to SyscallError.Timeout")added in go1.10

func (e *[SyscallError](https://pkg.go.dev/os#SyscallError)) Timeout() [bool](https://pkg.go.dev/builtin#bool)

Timeout reports whether this error represents a timeout.

#### func (*SyscallError) [Unwrap](https://cs.opensource.google/go/go/+/go1.25.1:src/os/error.go;l=56) [¶](https://pkg.go.dev/os#SyscallError.Unwrap "Go to SyscallError.Unwrap")added in go1.13

func (e *[SyscallError](https://pkg.go.dev/os#SyscallError)) Unwrap() [error](https://pkg.go.dev/builtin#error)

## ![](https://pkg.go.dev/static/shared/icon/insert_drive_file_gm_grey_24dp.svg) Source Files [¶](https://pkg.go.dev/os#section-sourcefiles "Go to Source Files")

[View all Source files](https://cs.opensource.google/go/go/+/go1.25.1:src/os)

- [dir.go](https://cs.opensource.google/go/go/+/go1.25.1:src/os/dir.go "dir.go")
- [dir_unix.go](https://cs.opensource.google/go/go/+/go1.25.1:src/os/dir_unix.go "dir_unix.go")
- [dirent_linux.go](https://cs.opensource.google/go/go/+/go1.25.1:src/os/dirent_linux.go "dirent_linux.go")
- [eloop_other.go](https://cs.opensource.google/go/go/+/go1.25.1:src/os/eloop_other.go "eloop_other.go")
- [env.go](https://cs.opensource.google/go/go/+/go1.25.1:src/os/env.go "env.go")
- [error.go](https://cs.opensource.google/go/go/+/go1.25.1:src/os/error.go "error.go")
- [error_errno.go](https://cs.opensource.google/go/go/+/go1.25.1:src/os/error_errno.go "error_errno.go")
- [exec.go](https://cs.opensource.google/go/go/+/go1.25.1:src/os/exec.go "exec.go")
- [exec_linux.go](https://cs.opensource.google/go/go/+/go1.25.1:src/os/exec_linux.go "exec_linux.go")
- [exec_posix.go](https://cs.opensource.google/go/go/+/go1.25.1:src/os/exec_posix.go "exec_posix.go")
- [exec_unix.go](https://cs.opensource.google/go/go/+/go1.25.1:src/os/exec_unix.go "exec_unix.go")
- [executable.go](https://cs.opensource.google/go/go/+/go1.25.1:src/os/executable.go "executable.go")
- [executable_procfs.go](https://cs.opensource.google/go/go/+/go1.25.1:src/os/executable_procfs.go "executable_procfs.go")
- [file.go](https://cs.opensource.google/go/go/+/go1.25.1:src/os/file.go "file.go")
- [file_open_unix.go](https://cs.opensource.google/go/go/+/go1.25.1:src/os/file_open_unix.go "file_open_unix.go")
- [file_posix.go](https://cs.opensource.google/go/go/+/go1.25.1:src/os/file_posix.go "file_posix.go")
- [file_unix.go](https://cs.opensource.google/go/go/+/go1.25.1:src/os/file_unix.go "file_unix.go")
- [getwd.go](https://cs.opensource.google/go/go/+/go1.25.1:src/os/getwd.go "getwd.go")
- [path.go](https://cs.opensource.google/go/go/+/go1.25.1:src/os/path.go "path.go")
- [path_unix.go](https://cs.opensource.google/go/go/+/go1.25.1:src/os/path_unix.go "path_unix.go")
- [pidfd_linux.go](https://cs.opensource.google/go/go/+/go1.25.1:src/os/pidfd_linux.go "pidfd_linux.go")
- [pipe2_unix.go](https://cs.opensource.google/go/go/+/go1.25.1:src/os/pipe2_unix.go "pipe2_unix.go")
- [proc.go](https://cs.opensource.google/go/go/+/go1.25.1:src/os/proc.go "proc.go")
- [rawconn.go](https://cs.opensource.google/go/go/+/go1.25.1:src/os/rawconn.go "rawconn.go")
- [removeall_at.go](https://cs.opensource.google/go/go/+/go1.25.1:src/os/removeall_at.go "removeall_at.go")
- [removeall_unix.go](https://cs.opensource.google/go/go/+/go1.25.1:src/os/removeall_unix.go "removeall_unix.go")
- [root.go](https://cs.opensource.google/go/go/+/go1.25.1:src/os/root.go "root.go")
- [root_nonwindows.go](https://cs.opensource.google/go/go/+/go1.25.1:src/os/root_nonwindows.go "root_nonwindows.go")
- [root_openat.go](https://cs.opensource.google/go/go/+/go1.25.1:src/os/root_openat.go "root_openat.go")
- [root_unix.go](https://cs.opensource.google/go/go/+/go1.25.1:src/os/root_unix.go "root_unix.go")
- [stat.go](https://cs.opensource.google/go/go/+/go1.25.1:src/os/stat.go "stat.go")
- [stat_linux.go](https://cs.opensource.google/go/go/+/go1.25.1:src/os/stat_linux.go "stat_linux.go")
- [stat_unix.go](https://cs.opensource.google/go/go/+/go1.25.1:src/os/stat_unix.go "stat_unix.go")
- [sticky_notbsd.go](https://cs.opensource.google/go/go/+/go1.25.1:src/os/sticky_notbsd.go "sticky_notbsd.go")
- [sys.go](https://cs.opensource.google/go/go/+/go1.25.1:src/os/sys.go "sys.go")
- [sys_linux.go](https://cs.opensource.google/go/go/+/go1.25.1:src/os/sys_linux.go "sys_linux.go")
- [sys_unix.go](https://cs.opensource.google/go/go/+/go1.25.1:src/os/sys_unix.go "sys_unix.go")
- [tempfile.go](https://cs.opensource.google/go/go/+/go1.25.1:src/os/tempfile.go "tempfile.go")
- [types.go](https://cs.opensource.google/go/go/+/go1.25.1:src/os/types.go "types.go")
- [types_unix.go](https://cs.opensource.google/go/go/+/go1.25.1:src/os/types_unix.go "types_unix.go")
- [wait_waitid.go](https://cs.opensource.google/go/go/+/go1.25.1:src/os/wait_waitid.go "wait_waitid.go")
- [zero_copy_linux.go](https://cs.opensource.google/go/go/+/go1.25.1:src/os/zero_copy_linux.go "zero_copy_linux.go")
- [zero_copy_posix.go](https://cs.opensource.google/go/go/+/go1.25.1:src/os/zero_copy_posix.go "zero_copy_posix.go")

## ![](https://pkg.go.dev/static/shared/icon/folder_gm_grey_24dp.svg) Directories [¶](https://pkg.go.dev/os#section-directories "Go to Directories")

Show internalExpand all

|||
|---|---|
|[exec](https://pkg.go.dev/os/exec@go1.25.1)|Package exec runs external commands.|
|[signal](https://pkg.go.dev/os/signal@go1.25.1)|Package signal implements access to incoming signals.|
|[user](https://pkg.go.dev/os/user@go1.25.1)|Package user allows user account lookups by name or id.|