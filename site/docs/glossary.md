---
layout: documentation
title: Bazel Glossary
---

# Bazel Glossary

### Action

A command to run during the build, for example, a call to a compiler that takes
[artifacts](#artifact) as inputs and produces other artifacts as outputs.
Includes metadata like the command line arguments, action key, environment
variables, and declared input/output artifacts.

**See also:** [Rules documentation](skylark/rules.html#actions)

### Action cache

An on-disk cache that stores a mapping of executed [actions](#action) to the
outputs they created. The cache key is known as the [action key](#action-key). A
core component for Bazel's incrementality model. The cache is stored in the
output base directory and thus survives Bazel server restarts.

### Action graph

An in-memory graph of [actions](#action) and the [artifacts](#artifact) that
these actions read and generate. The graph might include artifacts that exist as
source files (for example, in the file system) as well as generated
intermediate/final artifacts that are not mentioned in BUILD files. Produced
during the [analysis phase](#analysis-phase) and used during the [execution
phase](#execution-phase).

### Action graph query (aquery)

A [query](#query-concept) tool that can query over build [actions](#action).
This provides the ability to analyze how [build rules](#rule) translate into the
actual work builds do.

### Action key

The cache key of an [action](#action). Computed based on action metadata, which
might include the command to be execution in the action, compiler flags, library
locations, or system headers, depending on the action. Enables Bazel to cache or
invalidate individual actions deterministically.

### Analysis phase

The second phase of a build. Processes the [target graph](#target-graph)
specified in [BUILD files](#build-file) to produce an in-memory [action
graph](#action-graph) that determines the order of actions to run during the
[execution phase](#execution-phase). This is the phase in which rule
implementations are evaluated.

### Artifact

A source file or a generated file. Can also be a directory of files, known as
"tree artifacts". Tree artifacts are black boxes to Bazel: Bazel does not treat
the files in tree artifacts as individual artifacts and thus cannot reference
them directly as action inputs / outputs. An artifact can be an input to
multiple actions, but must only be generated by at most one action.

### Aspect

A mechanism for rules to create additional [actions](#action) in their
dependencies. For example, if target A depends on B, one can apply an aspect on
A that traverses *up* a dependency edge to B, and runs additional actions in B
to generate and collect additional output files. These additional actions are
cached and reused between targets requiring the same aspect. Created with the
`aspect()` Starlark Build API function. Can be used, for example, to generate
metadata for IDEs, and create actions for linting.

**See also:** [Aspects documentation](skylark/aspects.html)

### Aspect-on-aspect

An aspect composition mechanism, where aspects can be applied on other aspects.
For an aspect A to inspect aspect B, aspect A must declare the *providers* it
needs from aspect B (with the `required_aspect_providers` attribute), and aspect
B must declare the providers it returns (with the `provides` attribute). For
example, this can be used by IDE aspects to generate files using information
also generated by aspects, like the `java_proto_library` aspect.

### .bazelrc

Bazel’s configuration file used to change the default values for [startup
flags](#startup-flags) and [command flags](#command-flags), and to define common
groups of options that can then be set together on the Bazel command line using
a `--config` flag. Bazel can combine settings from multiple bazelrc files
(systemwide, per-workspace, per-user, or from a custom location), and a
bazelrc file may also import settings from other bazelrcs.

### Bazel

The Google-internal version of Bazel. Google’s main build system for its
mono-repository.

### BUILD File

A BUILD file is the main configuration file that tells Bazel what software
outputs to build, what their dependencies are, and how to build them. Bazel
takes a BUILD file as input and uses the file to create a graph of dependencies
and to derive the actions that must be completed to build intermediate and final
software outputs. A BUILD file marks a directory and any sub-directories not
containing a BUILD file as a [package](#package), and can contain
[targets](#target) created by [rules](#rule). The file can also be named
BUILD.bazel.

### BUILD.bazel File

See [BUILD File](#build-file). Takes precedence over a `BUILD` file in the same
directory.

### .bzl File

A file that defines rules, [macros](#macro), and constants written in
[Starlark](#starlark). These can then be imported into [BUILD
files](#build-file) using the load() function.

<!-- TODO: ### Build event protocol -->

<!-- TODO: ### Build flag -->

### Build graph

The dependency graph that Bazel constructs and traverses to perform a build.
Includes nodes like [targets](#target), [configured
targets](#configured-target), [actions](#action), and [artifacts](#artifact). A
build is considered complete when all [artifacts](#artifact) on which a set of
requested targets depend are verified as up-to-date.

### Build setting

A Starlark-defined piece of [configuration](#configuration).
[Transitions](#transition) can set build settings to change a subgraph's
configuration. If exposed to the user as a [command-line flag](#command-flags),
also known as a build flag.

### Clean build

A build that doesn't use the results of earlier builds. This is generally slower
than an [incremental build](#incremental-build) but commonly considered to be
more [correct](#correctness). Bazel guarantees both clean and incremental builds
are always correct.

### Client-server model

The `bazel` command-line client automatically starts a background server on the
local machine to execute Bazel [commands](#command). The server persists across
commands but automatically stops after a period of inactivity (or explicitly via
bazel shutdown). Splitting Bazel into a server and client helps amortize JVM
startup time and supports faster [incremental builds](#incremental-build)
because the [action graph](#action-graph) remains in memory across commands.

### Command

Used on the command line to invoke different Bazel functions, like `bazel
build`, `bazel test`, `bazel run`, and `bazel query`.

### Command flags

A set of flags specific to a [command](#command). Command flags are specified
*after* the command (`bazel build <command flags>`). Flags can be applicable to
one or more commands. For example, `--configure` is a flag exclusively for the
`bazel sync` command, but `--keep_going` is applicable to `sync`, `build`,
`test` and more. Flags are often used for [configuration](#configuration)
purposes, so changes in flag values can cause Bazel to invalidate in-memory
graphs and restart the [analysis phase](#analysis-phase).

### Configuration

Information outside of [rule](#rule) definitions that impacts how rules generate
[actions](#action). Every build has at least one configuration specifying the
target platform, action environment variables, and command-line [build
flags](#command-flags). [Transitions](#transition) may create additional
configurations, e.g. for host tools or cross-compilation.

<!-- TODO: ### Configuration fragment -->

### Configuration trimming

The process of only including the pieces of [configuration](#configuration) a
target actually needs. For example, if you build Java binary `//:j` with C++
dependency `//:c`, it's wasteful to include the value of `--javacopt` in the
configuration of `//:c` because changing `--javacopt` unnecessarily breaks C++
build cacheability.

### Configured query (cquery)

A [query](#query-concept) tool that queries over [configured
targets](#configured-target) (after the [analysis phase](#analysis-phase)
completes). This means `select()` and [build flags](#command-flags) (e.g.
`--platforms`) are accurately reflected in the results.

**See also:** [cquery documentation](cquery.html)

### Configured target

The result of evaluating a [target](#target) with a
[configuration](#configuration). The [analysis phase](#analysis-phase) produces
this by combining the build's options with the targets that need to be built.
For example, if `//:foo` builds for two different architectures in the same
build, it has two configured targets: `<//:foo, x86>` and `<//:foo, arm>`.

### Correctness

A build is correct when its output faithfully reflects the state of its
transitive inputs. To achieve correct builds, Bazel strives to be
[hermetic](#hermeticity), reproducible, and making [build
analysis](#analysis-phase) and [action execution](#execution-phase)
deterministic.

### Dependency

A directed edge between two [targets](#target). A target `//:foo` has a *target
dependency* on target `//:bar` if `//:foo`'s attribute values contain a
reference to `//:bar`. `//:foo` has an *action dependency* on `//:bar` if an
action in `//:foo` depends on an input [artifact](#artifact) created by an
action in `//:bar`.

### Depset

A data structure for collecting data on transitive dependencies. Optimized so
that merging depsets is time and space efficient, because it’s common to have
very large depsets (e.g. hundreds of thousands of files). Implemented to
recursively refer to other depsets for space efficiency reasons. [Rule](#rule)
implementations should not "flatten" depsets by converting them to lists unless
the rule is at the top level of the build graph. Flattening large depsets incurs
huge memory consumption. Also known as *nested sets* in Bazel's internal
implementation.

**See also:** [Depset documentation](skylark/depsets.html)

### Disk cache

A local on-disk blob store for the remote caching feature. Can be used in
conjunction with an actual remote blob store.

### Distdir

A read-only directory containing files that Bazel would otherwise fetch from the
internet using repository rules. Enables builds to run fully offline.

### Dynamic execution

An execution strategy that selects between local and remote execution based on
various heuristics, and uses the execution results of the faster successful
method. Certain [actions](#action) are executed faster locally (for example,
linking) and others are faster remotely (for example, highly parallelizable
compilation). A dynamic execution strategy can provide the best possible
incremental and clean build times.

### Execution phase

The third phase of a build. Executes the [actions](#action) in the [action
graph](#action-graph) created during the [analysis phase](#analysis-phase).
These actions invoke executables (compilers, scripts) to read and write
[artifacts](#artifact). *Spawn strategies* control how these actions are
executed: locally, remotely, dynamically, sandboxed, docker, and so on.

### Execution root

A directory in the [workspace](#workspace)’s [output base](#output-base)
directory where local [actions](#action) are executed in
non-[sandboxed](#sandboxing) builds. The directory contents are mostly symlinks
of input [artifacts](#artifact) from the workspace. The execution root also
contains symlinks to external repositories as other inputs and the `bazel-out`
directory to store outputs. Prepared during the [loading phase](#loading-phase)
by creating a *symlink forest* of the directories that represent the transitive
closure of packages on which a build depends. Accessible with `bazel info
execution_root` on the command line.

### File

See [Artifact](#artifact).

### Hermeticity

A build is hermetic if there are no external influences on its build and test
operations, which helps to make sure that results are deterministic and
[correct](#correctness). For example, hermetic builds typically disallow network
access to actions, restrict access to declared inputs, use fixed timestamps and
timezones, restrict access to environment variables, and use fixed seeds for
random number generators

### Incremental build

An incremental build reuses the results of earlier builds to reduce build time
and resource usage. Dependency checking and caching aim to produce correct
results for this type of build. An incremental build is the opposite of a clean
build.

<!-- TODO: ### Install base -->

### Label

An identifier for a [target](#target). A fully-qualified label such as
`//path/to/package:target` consists of `//` to mark the workspace root
directory, `path/to/package` as the directory that contains the [BUILD
file](#build-file) declaring the target, and `:target` as the name of the target
declared in the aforementioned BUILD file. May also be prefixed with
`@my_repository//<..>` to indicate that the target is declared in an ]external
repository] named `my_repository`.

### Loading phase

The first phase of a build where Bazel parses `WORKSPACE`, `BUILD`, and [.bzl
files](#bzl-file) to create [packages](#package). [Macros](#macro) and certain
functions like `glob()` are evaluated in this phase. Interleaved with the second
phase of the build, the [analysis phase](#analysis-phase), to build up a [target
graph](#target-graph).

### Macro

A mechanism to compose multiple [rule](#rule) target declarations together under
a single [Starlark](#starlark) function. Enables reusing common rule declaration
patterns across BUILD files. Expanded to the underlying rule target declarations
during the [loading phase](#loading-phase).

**See also:** [Macro documentation](skylark/macros.html)

### Mnemonic

A short, human-readable string selected by a rule author to quickly understand
what an [action](#action) in the rule is doing. Mnemonics can be used as
identifiers for *spawn strategy* selections. Some examples of action mnemonics
are `Javac` from Java rules, `CppCompile` from C++ rules, and
`AndroidManifestMerger` from Android rules.

### Native rules

[Rules](#rule) that are built into Bazel and implemented in Java. Such rules
appear in [`.bzl` files](#bzl-file) as functions in the native module (for
example, `native.cc_library` or `native.java_library`). User-defined rules
(non-native) are created using [Starlark](#starlark).

### Output base

A [workspace](#workspace)-specific directory to store Bazel output files. Used
to separate outputs from the *workspace*'s source tree. Located in the [output
user root](#output-user-root).

### Output groups

A group of files that is expected to be built when Bazel finishes building a
target. [Rules](#rule) put their usual outputs in the "default output group"
(e.g the `.jar` file of a `java_library`, `.a` and `.so` for `cc_library`
targets). The default output group is the output group whose
[artifacts](#artifact) are built when a target is requested on the command line.
Rules can define more named output groups that can be explicitly specified in
[BUILD files](#build-file) (`filegroup` rule) or the command line
(`--output_groups` flag).

### Output user root

A user-specific directory to store Bazel's outputs. The directory name is
derived from the user's system username. Prevents output file collisions if
multiple users are building the same project on the system at the same time.
Contains subdirectories corresponding to build outputs of individual workspaces,
also known as [output bases](#output-base).

### Package

The set of [targets](#target) defined by a [BUILD file](#build-file). A
package's name is the BUILD file's path relative to the workspace root. A
package can contain subpackages, or subdirectories containing BUILD files, thus
forming a package hierarchy.

### Package group

A [target](#target) representing a set of packages. Often used in visibility
attribute values.

### Platform

A "machine type" involved in a build. This includes the machine Bazel runs on
(the "host" platform), the machines build tools execute on ("exec" platforms),
and the machines targets are built for ("target platforms").

### Provider

A set of information passed from a rule [target](#target) to other rule targets.
Usually contains information like: compiler options, transitive source files,
transitive output files, and build metadata. Frequently used in conjunction with
[depsets](#depset).

**See also:** [Provider documentation](rules.html#providers)

### Query (concept)

The process of analyzing a [build graph](#build-graph) to understand
[target](#target) properties and dependency structures. Bazel supports three
query variants: [query](#query-command), [cquery](#configured-query-cquery), and
[aquery](#action-graph-query-aquery).

### query (command)

A [query](#query-concept) tool that operates over the build's post-[loading
phase](#loading-phase) [target graph](#target-graph). This is relatively fast,
but can't analyze the effects of `select()`, [build flags](#command-flags),
[artifacts](#artifact), or build [actions](#action).

**See also:** [Query how-to](query-how-to.html), [Query reference](query.html)

### Repository cache

A shared content-addressable cache of files downloaded by Bazel for builds,
shareable across [workspaces](#workspace). Enables offline builds after the
initial download. Commonly used to cache files downloaded through repository
rules like `http_archive` and repository rule APIs like
`repository_ctx.download`. Files are cached only if their SHA-256 checksums are
specified for the download.

<!-- TODO: ### Repository rule -->

### Reproducibility

The property of a build or test that a set of inputs to the build or test will
always produce the same set of outputs every time, regardless of time, method,
or environment. Note that this does not necessarily imply that the outputs are
[correct](#correctness) or the desired outputs.

### Rule

A function implementation that registers a series of [actions](#action) to be
performed on input [artifacts](#artifact) to produce a set of output artifacts.
Rules can read values from *attributes* as inputs (e.g. deps, testonly, name).
Rule targets also produce and pass along information that may be useful to other
rule targets in the form of [providers](#provider) (e.g. `DefaultInfo`
provider).

**See also:** [Rules documentation](skylark/rules.html)

### Runfiles

The runtime dependencies of an executable [target](#target). Most commonly, the
executable is the executable output of a test rule, and the runfiles are runtime
data dependencies of the test. Before the invocation of the executable (during
bazel test), Bazel prepares the tree of runfiles alongside the test executable
according to their source directory structure.

**See also:** [Runfiles documentation](skylark/rules.html#runfiles)

### Sandboxing

A technique to isolate a running [action](#action) inside a restricted and
temporary [execution root](#execution-root), helping to ensure that it doesn’t
read undeclared inputs or write undeclared outputs. Sandboxing greatly improves
[hermeticity](#hermeticity), but usually has a performance cost, and requires
support from the operating system. The performance cost depends on the platform.
On Linux, it's not significant, but on macOS it can make sandboxing unusable.

### Skyframe

The core parallel, functional, and incremental evaluation framework of Bazel.
[https://bazel.build/designs/skyframe.html](https://bazel.build/designs/skyframe.html)

<!-- TODO: ### Spawn strategy -->

### Stamping

A feature to embed additional information into Bazel-built
[artifacts](#artifact). For example, this can be used for source control, build
time and other workspace or environment-related information for release builds.
Enable through the `--workspace_status_command` flag and [rules](rules) that
support the stamp attribute.

### Starlark

The extension language for writing [rules](rules) and [macros](#macro). A
restricted subset of Python (syntactically and grammatically) aimed for the
purpose of configuration, and for better performance. Uses the [`.bzl`
file](#bzl-file) extension. [BUILD files](#build-file) use an even more
restricted version of Starlark (e.g. no `def` function definitions). Formerly
known as Skylark.

**See also:** [Starlark language documentation](skylark/language.html)

<!-- TODO: ### Starlark rules -->

<!-- TODO: ### Starlark rule sandwich -->

### Startup flags

The set of flags specified between `bazel` and the [command](#query-command),
for example, bazel `--host_jvm_debug` build. These flags modify the
[configuration](#configuration) of the Bazel server, so any modification to
startup flags causes a server restart. Startup flags are not specific to any
command.

### Target

A buildable unit. Can be a [rule](#rule) target, file target, or a [package
group](#package-group). Rule targets are instantiated from rule declarations in
[BUILD files](#build-file). Depending on the rule implementation, rule targets
can also be testable or runnable. Every file used in BUILD files is a file
target. Targets can depend on other targets via attributes (most commonly but
not necessarily `deps`). A [configured target](#configured-target) is a pair of
target and [build configuration](#configuration).

### Target graph

An in-memory graph of [targets](#target) and their dependencies. Produced during
the [loading phase](#loading-phase) and used as an input to the [analysis
phase](#analysis-phase).

### Target pattern

A way to specify a group of [targets](#target) on the command line. Commonly
used patterns are `:all` (all rule targets), `:*` (all rule + file targets),
`...` (current [package](#package) and all subpackages recursively). Can be used
in combination, for example, `//...:*` means all rule and file targets in all
packages recursively from the root of the [workspace](#workspace).

### Tests

Rule [targets](#target) instantiated from test rules, and therefore contains a
test executable. A return code of zero from the completion of the executable
indicates test success. The exact contract between Bazel and tests (e.g. test
environment variables, test result collection methods) is specified in the [Test
Encyclopedia](test-encyclopedia.html).

### Toolchain

A set of tools to build outputs for a language. Typically, a toolchain includes
compilers, linkers, interpreters or/and linters. A toolchain can also vary by
platform, that is, a Unix compiler toolchain's components may differ for the
Windows variant, even though the toolchain is for the same language. Selecting
the right toolchain for the platform is known as toolchain resolution.

### Top-level target

A build [target](#target) is top-level if it’s requested on the Bazel command
line. For example, if `//:foo` depends on `//:bar`, and `bazel build //:foo` is
called, then for this build, `//:foo` is top-level, and `//:bar` isn’t
top-level, although both targets will need to be built. An important difference
between top-level and non-top-level targets is that [command
flags](#command-flags) set on the Bazel command line (or via
[.bazelrc](#bazelrc)) will set the [configuration](#configuration) for top-level
targets, but might be modified by a [transition](#transition) for non-top-level
targets.

### Transition

A mapping of [configuration](#configuration) state from one value to another.
Enables [targets](#target) in the [build graph](#build-graph) to have different
configurations, even if they were instantiated from the same [rule](#rule). A
common usage of transitions is with *split* transitions, where certain parts of
the [target graph](#target-graph) is forked with distinct configurations for
each fork. For example, one can build an Android APK with native binaries
compiled for ARM and x86 using split transitions in a single build.

### Tree artifact

See [Artifact](#artifact).

### Visibility

Defines whether a [target](#target) can be depended upon by other targets. By
default, target visibility is private. That is, the target can only be depended
upon by other targets in the same [package](#package). Can be made visible to
specific packages or be completely public.

**See also:** [Visibility documentation](be/common-definitions.html#common-attributes)

### Workspace

A directory containing a `WORKSPACE` file and source code for the software you
want to build. Labels that start with `//` are relative to the workspace
directory.

### WORKSPACE file

Defines a directory to be a [workspace](#workspace). The file can be empty,
although it usually contains external repository declarations to fetch
additional dependencies from the network or local filesystem.

## Contributing

We welcome contributions to this glossary.

The definitions should strive to be crisp, concise, and easy to understand.

Principles:

* Use the simplest words possible. Try to avoid technical jargon.
* Use the simplest grammar possible. Try not to string too many concepts into a
  sentence.
* Make the last word of each sentence count.
* Make each adjective *really* count.
* Avoid adverbs.
* Keep definitions as self-contained as possible.
* Calibrate a word's value by considering what would be lost without it.
* Hit core concepts foremost. Avoid pedantry.
* Enlightening the reader is more important than being thorough.
* Links can provide deeper dives, for those who want it.