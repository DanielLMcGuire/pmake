# NAME

pmake - portable CMake wrapper

# SYNOPSIS

**pmake** \[*preset*\] \[**-b** *dir*\] \[**-i** *dir*\] \[**-s**
*dir*\] \[**-c**\] \[**-f**\] \[**-h**\]

# DESCRIPTION

**pmake** is a portable CMake wrapper that drives the configure, build,
and install stages of a CMake project through a single command. It
caches the choices made on the first invocation so that subsequent runs
require no arguments at all.

On startup **pmake** reads **project.pmake** from the current directory
(if present) to obtain project metadata and path overrides. All
intermediate state is written to a cache directory (default *.pmake/*).
A lock file prevents two **pmake** processes from operating on the same
project simultaneously.

# PROJECT FILE

If **project.pmake** exists in the working directory, **pmake** reads it
line by line:

> -------- ------------------------------------------------------------------
>   Line 1   Project name
>   Line 2   Version string
>   Line 3   Sub-version shell command (optional; output appended to version)
>   Line 4   Install prefix directory
>   Line 5   Source directory
>   Line 6   Cache directory
>   -------- ------------------------------------------------------------------

Missing or empty fields fall back to compiled-in defaults (see
**FILES**). If the file is absent, **pmake** continues with defaults and
prints a warning to standard error.

# OPTIONS

*preset*

:   The CMake preset name to pass to **cmake**(1) during configuration.
    Required on the first invocation; thereafter the value is read from
    the cache and this argument is optional. If supplied together with
    **-b** or **-i** it also triggers a fresh configure step.

**-b** \[***dir***\], **\--build** \[***dir***\]

:   Build the project. *dir* is the CMake binary / build directory
    (default: *.pmake/CMakeBuild*). If *dir* is omitted the cached value
    is used, falling back to the default. The chosen path is written to
    the cache.

**-i** \[***dir***\], **\--install** \[***dir***\]

:   Install the built artefacts. *dir* is the installation prefix
    (default: *out*). If *dir* is omitted the cached value is used,
    falling back to the default. The chosen path is written to the
    cache.

**-s** ***dir***, **\--src** ***dir***

:   Override the source directory. *dir* is required. The resolved
    absolute path is written to the cache.

**-c**, **\--clean**

:   Remove the build directory and cache directory before proceeding.
    When used alone (without **-b** or **-i**) **pmake** exits after
    cleaning. When combined with **-b** or **-i** a full configure--dash
    build--dash install cycle is performed.

**-f**, **\--force**

:   Clear the status lock file and remove the build directory, then
    continue normally. Use this to recover from a previous interrupted
    run.

**-h**, **\--help**

:   Print a usage summary and exit. If the project has already been
    configured the help output omits the CMake preset listing; otherwise
    **cmake** \--list-presets is run to show available presets.

# WORKFLOW

**pmake** executes up to three sequential stages. Each stage is skipped
unless explicitly requested or implicitly required.

## Configure

Invoked when *preset* is given on the command line, or when the build
directory does not yet exist. Runs:

> **cmake** **-S*** srcDir* **-B*** buildDir* **\--preset*** preset*

## Build

Invoked when **-b** is given, or when neither **-b**, **-i**, nor **-c**
is given (default action). Runs:

> **ninja** **-C*** buildDir*

## Install

Invoked only when **-i** is given. Runs:

> **cmake \--install** *buildDir* **\--prefix*** installDir*

# DEFAULT BEHAVIOUR

When **pmake** is invoked with only a *preset* and no other flags, it
configures and then builds the project. When invoked with no arguments
and a cached preset exists, it builds the project. When invoked with no
arguments and no cached preset, it prints help and exits with a non-zero
status.

# LOCK FILE

**pmake** writes a lock file containing the current process ID before
performing any work. If a lock file already exists and belongs to a
different PID, **pmake** aborts immediately with exit code 2 without
removing the lock. Use **-f** to forcibly clear a stale lock left by a
crashed process.

# FILES

*project.pmake*

:   Per-project configuration file, read from the working directory.

*.pmake/*

:   Default cache directory. Contains *\*.cache* files (one per cached
    value) and the lock file *\<project\>.lock*.

*.pmake/CMakeBuild/*

:   Default CMake binary directory.

*out/*

:   Default installation prefix.

# EXIT STATUS

**0**

:   Success.

**1**

:   A required argument was missing, an unknown argument was supplied,
    or a build/configure/install step failed.

**2**

:   Aborted because a lock file held by another process was detected.

**-1**

:   The cache directory could not be created, or no preset was
    available.

# SEE ALSO

**cmake**(1), **ninja**(1)

# BUGS

The lock detection compares PIDs only; if the owning process has exited
and the PID has been reused by an unrelated process, **pmake** will
falsely report a conflict. Use **-f** to recover.

