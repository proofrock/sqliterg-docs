# 🏗 Building & Testing

`sqliterg` is distributed with binaries for the major combinations of OSs and architectures. Portable binaries are provided, as much as possible. For these reasons, building `sqliterg` is generally not required, but it could be useful for particular architectures or to really ensure that the binary matches the sources.

## Types of binaries

`sqliterg` binaries are made available in two forms:

* **bundled** builds include sqlite and don't have any external dependency. They are a [bit slower](features/performances.md) but more convenient; builds for Linux are fully static so they should run fine across distributions;
* **dynamic** builds are dynamically linked against the sqlite that is installed in the system.

{% hint style="info" %}
Using dynamic builds requires to install `sqlite` on the system. On MacOS, use `brew`; on Linux, use the distribution's own package system.
{% endhint %}

## Supported platforms

These are platforms for which we'll provide binaries at the time of the release.

|                                                                              | bundled             | dynamic             |
| ---------------------------------------------------------------------------- | ------------------- | ------------------- |
| Linux                                                                        | `x86_64`, `aarch64` | `x86_64`, `aarch64` |
| MacOS                                                                        | `x86_64`, `aarch64` | `x86_64`, `aarch64` |
| Windows                                                                      | `x86_64`            |                     |
| [Docker](https://germ.gitbook.io/sqliterg/documentation/installation/docker) |                     | `x86_64`, `aarch64` |

## Building

`sqliterg` is a Rust program that uses Rust 2021. The Rust toolset, plus Docker, makes compilation and cross-compiling very convenient, but there are some prerequisites.

* Rust
* Make
* The rust toolchain

I included a Make file to script the building under Linux/macos, so if you have all the prerequisites it should be a matter of:

{% code lineNumbers="true" %}
```bash
git clone https://github.com/proofrock/sqliterg
cd sqliterg
make build
# You will find the binary in the bin/ directory.
```
{% endcode %}

For Windows, instead of the step at Line 3 do the following. I don't support using `make` under windows, though it could be feasible.

{% code lineNumbers="true" %}
```bash
cargo build --release
```
{% endcode %}

And get the binary at `target/release/sqliterg.exe`

## Cross-compiling in Linux

The command:

{% code lineNumbers="true" %}
```bash
make docker-zbuild-linux
```
{% endcode %}

allows to cross-build binaries for Linux, both static and dynamic, both for `x86_64` and `aarch64`. It requires Docker and produces compressed binaries in the `bin/` directory.

## Testing

The code includes unit tests (written in Go). Use the appropriate target for make:

{% code lineNumbers="true" %}
```bash
make test
```
{% endcode %}

This command takes a while, because tests of periodic macros or backups take a while. To skip those tests you can use the command:

{% code lineNumbers="true" %}
```bash
make test-short
```
{% endcode %}

If you don't use `make`, beware that a complete test that takes at least six minutes to complete (there are sleep's that ensure this); be sure to provide a suitable timeout value to `go test`.
