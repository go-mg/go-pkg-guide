# Tutorial: How to Publish a Go Package on pkg.go.dev

This tutorial explains how to create, structure, and publish a Go package on [pkg.go.dev](https://pkg.go.dev/).

## 1. Prerequisites

- Go installed (1.21+)
- GitHub account (or another public Git host)
- Git configured locally

## 2. Create the Repository

Create a public repository on GitHub. The repository name will be part of the import path.

Example: `github.com/your-user/my-package`

## 3. Project Structure

### Module with a single package

If the repository contains only one package, the `.go` files go in the root:

```bash
my-package/
├── go.mod
├── go.sum
├── LICENSE
├── README.md
├── CHANGELOG.md
├── my_package.go        # package mypackage
├── my_package_test.go
└── example_test.go      # Example* functions (rendered on pkg.go.dev)
```

Import: `github.com/your-user/my-package`

### Module with multiple packages (subpackages)

If the repository contains more than one package, each goes in its own folder:

```bash
my-module/
├── go.mod               # module github.com/your-user/my-module
├── go.sum
├── LICENSE
├── README.md
├── CHANGELOG.md
├── json/
│   ├── doc.go           # package json — package documentation
│   ├── unmarshal.go
│   ├── decoder.go
│   ├── json_test.go
│   └── example_test.go
└── xml/
    ├── doc.go           # package xml
    ├── unmarshal.go
    ├── decoder.go
    ├── xml_test.go
    └── example_test.go
```

Imports:

- `github.com/your-user/my-module/json`
- `github.com/your-user/my-module/xml`

### Important rules about structure

- `go.mod` goes in the root and defines the module path
- Each folder with `.go` files is a separate package
- The package name (declared in `package`) doesn't have to match the folder name, but by convention it should
- `_test.go` files go in the same folder as the code they test
- The `internal/` folder contains code that cannot be imported by other modules
- The `cmd/` folder is used for executables (CLIs), not for libraries

## 4. Initialize the Module

```bash
mkdir my-package
cd my-package
go mod init github.com/your-user/my-package
```

The generated `go.mod`:

```bash
module github.com/your-user/my-package

go 1.24
```

The `go` directive defines the minimum Go version that consumers need.

## 5. Write the Code

### Main file

```go
// Package mypackage does something useful.
//
// More detailed description here. This comment appears on pkg.go.dev.
package mypackage

// MyFunction does something.
func MyFunction() string {
    return "result"
}
```

### doc.go (for subpackages)

When the package is in a subfolder, create a `doc.go` with the documentation:

```go
// Package json provides case-sensitive JSON unmarshaling.
//
// Detailed description, usage examples, etc.
// All of this appears on pkg.go.dev.
package json
```

### Documentation rules for pkg.go.dev

- The comment above `package` is the package documentation
- Comments above exported functions/types are their documentation
- Use examples in `example_test.go` with `// Output:` so they are rendered
- The root `README.md` appears on the module page on pkg.go.dev

## 6. Write Tests

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

### Example functions (appear on pkg.go.dev)

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

The `// Output:` comment is required for the example to be executed as a test
and rendered on pkg.go.dev.

## 7. License

Include a `LICENSE` file in the root. pkg.go.dev displays the license on the module page.
Common licenses: MIT, Apache 2.0, BSD-3-Clause.

## 8. Versioning

Go uses [Semantic Versioning](https://semver.org/) via git tags.

### Version format

```bash
vMAJOR.MINOR.PATCH
```

- MAJOR: incompatible API changes (breaking changes)
- MINOR: new features backward compatible
- PATCH: backward compatible bug fixes

### Create the first version

```bash
git add .
git commit -m "feat: initial release"
git tag v0.1.0
git push origin main --tags
```

### Subsequent versions

```bash
# Bug fix
git tag v0.1.1

# New feature
git tag v0.2.0

# Breaking change
git tag v1.0.0
```

### Go versioning rules

- Versions `v0.x.x` are considered unstable (the API may change)
- From `v1.0.0` onwards, the API is considered stable
- For `v2.0.0+`, the module path must include the major version:
  - `go.mod`: `module github.com/your-user/my-package/v2`
  - Import: `github.com/your-user/my-package/v2`
- Never delete or move already published tags

### Pre-releases

```bash
git tag v0.1.0-alpha.1
git tag v0.1.0-beta.1
git tag v0.1.0-rc.1
```

Pre-releases are not installed by default with `go get`.

## 9. Publish on pkg.go.dev

pkg.go.dev indexes automatically from the Go Module Proxy. To trigger indexing:

### Option 1: Access the URL directly

Open in your browser:

```bash
https://pkg.go.dev/github.com/your-user/my-package
```

pkg.go.dev will fetch and index the module on the first visit.

### Option 2: Force via Go Proxy

```bash
GOPROXY=https://proxy.golang.org go get github.com/your-user/my-package@v0.1.0
```

### Option 3: Request via API

```bash
curl "https://proxy.golang.org/github.com/your-user/my-package/@v/v0.1.0.info"
```

After any of these options, the package appears on pkg.go.dev within a few minutes.

## 10. Consume the Package

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

## 11. Publication Checklist

- [ ] `go.mod` with correct module path (`github.com/user/repo`)
- [ ] `LICENSE` in the root
- [ ] `README.md` in the root (appears on pkg.go.dev)
- [ ] Documentation comments on exported functions and types
- [ ] `doc.go` in subpackages
- [ ] `Example*` functions in `example_test.go`
- [ ] Tests passing (`go test ./...`)
- [ ] No unnecessary dependencies in `go.sum`
- [ ] Git tag in `vX.Y.Z` format
- [ ] Tag pushed to remote (`git push --tags`)
- [ ] Public repository

## 12. Best Practices

- Maintain a `CHANGELOG.md` following the [Keep a Changelog](https://keepachangelog.com/) format
- Use CI (GitHub Actions) to run tests and lint automatically
- Start with `v0.x.x` until the API stabilizes
- Document breaking changes clearly in the CHANGELOG
- Use `go vet` and `golangci-lint` before each release
- Run `go mod tidy` to clean up unused dependencies
