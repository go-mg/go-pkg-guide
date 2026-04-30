# How to Publish a Go Package on pkg.go.dev

If you've written something in Go that could be useful to others (or to yourself across projects), it's worth publishing as a package on [pkg.go.dev](https://pkg.go.dev/). The process is simpler than it looks. This tutorial covers everything from project structure to the moment your package shows up indexed.

## 1. Prerequisites

- Go installed (1.21+)
- GitHub account (or another public Git host)
- Git configured locally

## 2. Create the Repository

Start by creating a public repository on GitHub. The repository name becomes part of your module's import path, so pick something descriptive.

Example: `github.com/your-user/my-package`

## 3. Project Structure

How you organize your files depends on the complexity of what you're building.

### Module with a single package

When the repository has a single package, everything lives in the root. Simple as that:

```
my-package/
├── go.mod
├── go.sum
├── LICENSE
├── README.md
├── CHANGELOG.md
├── my_package.go          # package mypackage
├── my_package_test.go
├── example_test.go         # Example* functions (rendered on pkg.go.dev)
└── internal/
    └── helpers.go          # internal code, not exported to other modules
```

Import: `github.com/your-user/my-package`

### Module with multiple packages

When the module provides more than one package, each one lives in its own folder. The root still holds the `go.mod` and documentation files:

```
my-module/
├── go.mod                  # module github.com/your-user/my-module
├── go.sum
├── LICENSE
├── README.md
├── CHANGELOG.md
├── internal/
│   └── shared.go           # internally shared code
├── json/
│   ├── doc.go              # package json — package documentation
│   ├── unmarshal.go
│   ├── decoder.go
│   ├── json_test.go
│   └── example_test.go
└── xml/
    ├── doc.go              # package xml
    ├── unmarshal.go
    ├── decoder.go
    ├── xml_test.go
    └── example_test.go
```

Imports:

- `github.com/your-user/my-module/json`
- `github.com/your-user/my-module/xml`

### Structure conventions

A few rules worth internalizing:

- `go.mod` lives in the root and defines the module path.
- Each folder with `.go` files is a separate package.
- The package name (declared in `package`) doesn't have to match the folder name, but by convention it should.
- `_test.go` files go in the same folder as the code they test.
- The `internal/` folder is special: Go prevents code inside it from being imported by external modules. Use it for helper logic that isn't part of the public API.
- The `cmd/` folder is used for executables (CLIs), not for libraries.

For details on supporting files (`LICENSE`, `README.md`, `CHANGELOG.md`), see the [License](#7-license) and [Best Practices](#12-best-practices) sections.

## 4. Initialize the Module

With the repository created, initialize the module:

```bash
mkdir my-package && cd my-package
go mod init github.com/your-user/my-package
```

This generates the `go.mod`:

```
module github.com/your-user/my-package

go 1.24
```

The `go` directive indicates the minimum version that consumers need to have installed.

## 5. Write the Code

This is where things get interesting. Let's look at how to structure code with both functionality and pkg.go.dev documentation in mind.

### Package documentation (doc.go)

In Go, the comment right above the `package` declaration is the package documentation. pkg.go.dev renders this text on the main page. You can place this comment in any `.go` file, but the convention is to create a dedicated file called `doc.go`. This keeps the documentation separate from the logic and easy to find:

```go
// doc.go

// Package mypackage does something useful.
//
// More detailed description here. This comment appears
// directly on the package page on pkg.go.dev.
package mypackage
```

This applies to both the root package and subpackages. In a module with multiple packages, each subfolder would have its own `doc.go`:

```go
// doc.go inside json/

// Package json provides case-sensitive JSON unmarshaling.
//
// Detailed description, usage examples, etc.
// All of this appears on pkg.go.dev.
package json
```

### Code

With the documentation in `doc.go`, the other files stay focused on the implementation:

```go
// my_package.go

package mypackage

// MyFunction returns an example string.
func MyFunction() string {
    return "result"
}
```

### How documentation works on pkg.go.dev

pkg.go.dev extracts documentation directly from the source code. There's no separate configuration file. The rules are:

- The comment above `package` becomes the package documentation.
- Comments above exported functions, types, and constants become their documentation.
- Examples in `example_test.go` with the `// Output:` comment are rendered as interactive examples.
- The root `README.md` appears on the module page.

## 6. Write Tests

Tests in Go live in the same directory as the code, in files with the `_test.go` suffix. For published packages, it's worth using the external test pattern (with `_test` in the package name), which simulates how a real consumer would use your code:

```go
package mypackage_test

import (
    "testing"
    "github.com/your-user/my-package"
)

func TestMyFunction(t *testing.T) {
    result := mypackage.MyFunction()
    if result != "result" {
        t.Errorf("expected 'result', got %q", result)
    }
}
```

### Example functions

This is a feature many people overlook, but it makes a real difference. Functions starting with `Example` are executed as tests and show up as usage examples on pkg.go.dev:

```go
package mypackage_test

import (
    "fmt"
    "github.com/your-user/my-package"
)

func ExampleMyFunction() {
    result := mypackage.MyFunction()
    fmt.Println(result)
    // Output:
    // result
}
```

The `// Output:` comment is required. Without it, the example won't be executed as a test and won't appear in the documentation.

## 7. License

pkg.go.dev displays the license on the module page, so include a `LICENSE` file in the repository root. The most common ones in the Go ecosystem are MIT, Apache 2.0, and BSD-3-Clause.

The easiest way to generate the file is through GitHub's interface when creating the repository, or by checking [choosealicense.com](https://choosealicense.com/).

## 8. Versioning

Go uses [Semantic Versioning](https://semver.org/) via git tags. Each release of your package is a tag in the format:

```
vMAJOR.MINOR.PATCH
```

- MAJOR: incompatible API changes (breaking changes)
- MINOR: new backward-compatible features
- PATCH: bug fixes

### Create the first version

```bash
git add .
git commit -m "feat: initial release"
git tag v0.1.0
git push origin main --tags
```

### Subsequent versions

```bash
git tag v0.1.1   # bug fix
git tag v0.2.0   # new feature
git tag v1.0.0   # breaking change (stable API from here on)
```

### Go versioning rules

A few particularities that Go enforces:

- Versions `v0.x.x` are considered unstable. The API can change without ceremony.
- From `v1.0.0` onwards, the API is considered stable. Break compatibility and consumers will notice.
- For `v2.0.0` and above, the module path must include the major version:
  - `go.mod`: `module github.com/your-user/my-package/v2`
  - Import: `github.com/your-user/my-package/v2`
- Never delete or move already published tags. The Go Module Proxy caches versions, and inconsistencies cause real problems.

### Pre-releases

```bash
git tag v0.1.0-alpha.1
git tag v0.1.0-beta.1
git tag v0.1.0-rc.1
```

Pre-releases are not installed by default with `go get`, which is useful for testing before making things official.

## 9. Preview Documentation Locally

Before publishing, it's worth checking how the documentation will look on pkg.go.dev. The [pkgsite](https://pkg.go.dev/golang.org/x/pkgsite) project is the same engine that runs behind pkg.go.dev, and you can run it locally.

Install it with:

```bash
go install golang.org/x/pkgsite/cmd/pkgsite@latest
```

Then, from the module root:

```bash
pkgsite -open .
```

This opens the browser with the documentation rendered exactly as it would appear on pkg.go.dev: README, package comments from `doc.go`, exported functions, examples. It's the best way to validate that the documentation is complete and well-formatted before creating the tag and publishing.

## 10. Publish on pkg.go.dev

There's no "publish" button. pkg.go.dev indexes automatically from the Go Module Proxy. You just need to trigger the indexing in one of the following ways:

### Access the URL directly

Open in your browser:

```
https://pkg.go.dev/github.com/your-user/my-package
```

On the first visit, pkg.go.dev fetches and indexes the module.

### Force via Go Proxy

```bash
GOPROXY=https://proxy.golang.org go get github.com/your-user/my-package@v0.1.0
```

### Request via API

```bash
curl "https://proxy.golang.org/github.com/your-user/my-package/@v/v0.1.0.info"
```

Any of these options works. The package shows up on pkg.go.dev within a few minutes.

## 11. Consume the Package

To use the package:

```bash
go get github.com/your-user/my-package@v0.1.0
```

```go
import "github.com/your-user/my-package"
```

For subpackages:

```bash
go get github.com/your-user/my-module/json@v0.1.0
```

```go
import "github.com/your-user/my-module/json"
```

## 12. Best Practices

- Maintain a `CHANGELOG.md` following the [Keep a Changelog](https://keepachangelog.com/) format. It helps consumers understand what changed between versions without reading commits.
- The `README.md` appears on the module page on pkg.go.dev. Use it to explain what the package does, how to install it, and a quick usage example. For reference, check the [Make a README guide](https://www.makeareadme.com/).
- Use CI (GitHub Actions) to run tests and lint automatically.
- Start with `v0.x.x` until the API stabilizes.
- Use `go vet` and `golangci-lint` before each release.
- Run `go mod tidy` to clean up unused dependencies.

## 13. Publication Checklist

- [ ] `go.mod` with correct module path (`github.com/user/repo`)
- [ ] `LICENSE` in the root
- [ ] `README.md` in the root
- [ ] Documentation comments on exported functions and types
- [ ] `doc.go` in subpackages
- [ ] `Example*` functions in `example_test.go`
- [ ] Tests passing (`go test ./...`)
- [ ] No unnecessary dependencies (`go mod tidy`)
- [ ] Git tag in `vX.Y.Z` format
- [ ] Tag pushed to remote (`git push --tags`)
- [ ] Public repository
