# go-ruby-widgets

Three pure-Go, Ruby-runtime-independent adapters over the
[go-widgets](https://github.com/go-widgets) stack, shaped so that
[go-embedded-ruby](https://github.com/go-embedded-ruby/ruby) (rbgo) can bind
each as `require "widgets"`, `require "tui"` and `require "mvvm"`:

| Repo | Adapts | Role |
| --- | --- | --- |
| [`widgets`](https://github.com/go-ruby-widgets/widgets) | [`go-widgets/toolkit`](https://github.com/go-widgets/toolkit) + [`go-widgets/painter`](https://github.com/go-widgets/painter) | A live pixel widget UI toolkit — buttons, labels, lists, menus and the container/layout system, rendered to an RGBA buffer |
| [`tui`](https://github.com/go-ruby-widgets/tui) | [`go-widgets/tui`](https://github.com/go-widgets/tui) | A terminal-cell UI toolkit — widgets, border/split/card layouts, rendered as an ANSI stream or a decoded cell grid |
| [`mvvm`](https://github.com/go-ruby-widgets/mvvm) | [`go-widgets/mvvm`](https://github.com/go-widgets/mvvm) | The data-binding layer — an `Observable` property, a `Command` action, an `ObservableList` collection |

Each is a thin adapter: it exposes its underlying toolkit through Ruby-facing
handles (`Module`, plus `Widget`/`Font`-style handles per repo) whose methods
return **Ruby-shaped values** — a **Hash** (`map[string]any`), an **Array**
(`[]any`) or a scalar. A single dynamic entry point, `Call`, dispatches a
Ruby-style snake_case method name to the matching handle method and coerces
the arguments, which is exactly what an rbgo binding drives from
`method_missing`. Nothing in any of the three depends on the Ruby runtime, so
each is equally usable as a standalone Go library — siblings of
[go-ruby-opentype](https://github.com/go-ruby-opentype/opentype),
[go-ruby-regexp](https://github.com/go-ruby-regexp/regexp) and
[go-ruby-erb](https://github.com/go-ruby-erb/erb) in the go-ruby-* family.

## At a glance

- **CGO-free** — builds and tests identically on `amd64`, `arm64`, `riscv64`,
  `loong64`, `ppc64le`, `s390x` (`widgets` also targets `js/wasm`, the
  [wasmdesk](https://github.com/wasmdesk) target).
- **100 % test coverage**, race-clean, enforced in CI.
- **Handle-addressed object graphs.** `widgets` and `tui` each own a live
  widget/container tree addressed by opaque handles; `mvvm` has no UI tree —
  just bindable Observable/Command/ObservableList primitives.
- **Two render seams, one data-binding seam.** `widgets.Render` paints to an
  RGBA pixel buffer; `tui.Render`/`RenderCells` emit an ANSI stream or a
  decoded cell grid; `mvvm.DrainEvents` pulls a queue of callback-id-tagged
  event Hashes.

## Install (Go)

```sh
go get github.com/go-ruby-widgets/widgets
go get github.com/go-ruby-widgets/tui
go get github.com/go-ruby-widgets/mvvm
```

See [widgets](widgets.md), [tui](tui.md) and [mvvm](mvvm.md) for each
adapter's full Ruby-facing surface, and [Getting started in
rbgo](rbgo.md) for the `require` workflow.
