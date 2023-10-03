# 🏗 Building & Testing



`sqliterg` is distributed with binaries for the major combinations of OSs and architectures. I'm trying to do portable builds as much as possible. For these reasons, building `sqliterg` is generally not required, but it could be useful to really ensure that the binary matches the sources.

### Types of binaries

`sqliterg` is compiled in two forms:

* **bundled** builds include sqlite and don't have any external dependency. A bit slower but more convenient; linux builds are fully static so they should run fine across distributions;
* **dynamic** builds are dynamically linked against the sqlite that is installed in the system.&#x20;

{% hint style="info" %}
On macos, use `brew`; on linux, use the distribution package system.
{% endhint %}

### Supported platforms

These are platforms for which we'll provide binaries at the time of the release.



|                                                                              | bundled             | dynamic             |
| ---------------------------------------------------------------------------- | ------------------- | ------------------- |
| Linux                                                                        | `x86_64`, `aarch64` | `x86_64`, `aarch64` |
| MacOS                                                                        | `x86_64`, `aarch64` | `x86_64`, `aarch64` |
| Windows                                                                      | `x86_64`            |                     |
| [Docker](https://germ.gitbook.io/sqliterg/documentation/installation/docker) |                     | `x86_64`, `aarch64` |

### Building

`sqliterg` is a Rust program that uses Rust 2021. The Rust toolset, plus Docker, makes compilation and cross-compiling very convenient, but there are some prerequisites.

* Rust
* Make
* The rust toolchain

I included a Make file to script the building under Linux/macos, so if you have all the prerequisites it should be a matter of:

```bash
git clone https://github.com/proofrock/sqliterg
cd sqliterg
make build
# You will find the binary in the bin/ directory.
```

For Windows, instead of the step at Line 3 do the following. I don't support using `make` under windows, though it could be feasible.

```bash
cargo build --release
```

And get the binary from `target/release/sqliterg.exe`

To cross-build, for Linux use `make docker-zbuild-linux` ; it requires Docker and produces compressed binaries in the `bin/` directory for `x86_64` and `aarch64`, both bundled and dynamic. It can take one hour or more to run.

### Testing

The code includes unit tests (written in Rust). Use the appropriate target for make:

```bash
make test
```

Or, to avoid the longer tests (that test the periodic macros/backups, so wait for minutes):

```bash
make test-short
```

If you don't use `make`, beware that a complete test that takes at least six minutes to complete (there are sleep's that ensure this); be sure to provide a suitable timeout value.
