# Contributing

go-ruby-widgets is BSD-3-Clause. The code lives in three repos:
[widgets](https://github.com/go-ruby-widgets/widgets),
[tui](https://github.com/go-ruby-widgets/tui) and
[mvvm](https://github.com/go-ruby-widgets/mvvm).

## Ground rules

- **CGO-free.** No cgo, ever — each package must build and test on `amd64`,
  `arm64`, `riscv64`, `loong64`, `ppc64le`, `s390x` (`widgets` also targets
  `js/wasm`).
- **100 % coverage.** The CI gate enforces 100 % statement coverage,
  including error branches.
- **Backed by go-widgets.** The widget sets, layout engines and data-binding
  primitives live in [go-widgets](https://github.com/go-widgets); these repos
  are only the Ruby-idiomatic, Hash/Array-returning adapters. Changes to
  widget/layout/rendering behaviour belong upstream.
- **Generic UI vocabulary only.** All public content uses generic terms —
  widgets, containers, layouts, border/card layout, data-binding, MVVM — with
  no third-party proprietary UI-framework naming.

## Build & test

```sh
go build ./...
go vet ./...
go test -race -coverprofile=cover.out ./...
go tool cover -func=cover.out | tail -1   # must read 100.0%
```

## Adding a Ruby-facing method

New surface follows the existing shape:

1. Add the exported Go method to the relevant handle (`Module`, `Widget`,
   `Observable`, `Command`, `ObservableList`, …).
2. Return a Ruby-shaped value — a `map[string]any`, a `[]any`, or a scalar —
   never a bare Go struct.
3. `Call` and `Methods` pick it up automatically via reflection (its
   snake_case name derives from the Go method name); no dispatch table to
   update.
4. Document it on the relevant [widgets](widgets.md) / [tui](tui.md) /
   [mvvm](mvvm.md) page and, if it changes the intended rbgo surface, in
   [Getting started in rbgo](rbgo.md).
