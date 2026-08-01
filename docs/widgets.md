# widgets

The pure-Go, Ruby-runtime-independent core of the Ruby **`widgets`** gem: a
live widget UI toolkit — buttons, labels, text fields, lists, menus and the
container/layout system that arranges them. It is a thin adapter over the
pure-Go widget set [`go-widgets/toolkit`](https://github.com/go-widgets/toolkit)
and the pixel rasteriser [`go-widgets/painter`](https://github.com/go-widgets/painter).

Unlike the stateless data adapters of the go-ruby-* family (opentype, regexp,
erb, …), a `Module` here owns a **live object graph**: every widget and
container is stored under an integer **handle** the Ruby side keeps, and
every operation — mutate, compose, lay out, render, dispatch an event — is
addressed by that handle and returns a Ruby-shaped value.

`CGO_ENABLED=0`, no display, no network — cross-compiles to all six 64-bit Go
architectures **and `js/wasm`** (the target [wasmdesk](https://github.com/wasmdesk) runs).

## Constructors (return an opaque integer handle)

| Kind | Methods |
| --- | --- |
| Leaves | `button(label, cb)`, `label(text)`, `entry(initial, cb)`, `text_view(initial)`, `check_button(label, checked, cb)`, `drop_down(options, selected, cb)`, `list_box(items, cb)`, `menu(items)`, `menu_bar` |
| Containers | `container(layout)` (`fit`/`box`/`hbox`/`vbox`/`border`/`card`), `h_box`, `v_box`, `grid(cols, rows)`, `frame(child)`, `dock(body)`, `border` |

## Mutation

`set_text` / `text`, `set_checked` / `checked`, `select(id, idx)`,
`set_style`, `set_spacing`, and the module-wide `set_theme("light"|"dark")`.

## Composition

`add_widget(parent, child)`, `add(parent, child, {flex:, size:, region:})`,
`add_fixed`, `add_flex`, `attach(grid, child, col, row)`,
`dock_at(dock, child, side, size)`, `set_region(border, child, region, size)`,
`add_menu(bar, name, menu)`, `set_active(card, idx)`,
`set_layout(container, layout)`.

## Layout / query

`set_bounds(id, x, y, w, h)`, `layout(id, w, h)` (at the origin), `bounds(id)`.

## The render seam

```ruby
img = Widgets.render(root, w, h)
# => { "pixels" => <RGBA bytes>, "stride" => w*4, "w" => w, "h" => h }
```

`pixels` is 4 bytes per pixel, row-major, top-left origin — exactly what a
host (wasmbox) blits into a `<canvas>` / `SharedArrayBuffer`.

## The event seam

```ruby
out = Widgets.dispatch(root, { "kind" => "click", "x" => 10, "y" => 4 })
# => { "fired" => ["on_ok"], "repaint" => true }
```

`kind` is one of `click` / `keydown` / `keyup` / `char` / `mousedrag` /
`mouseup`. A widget is wired to a callback by passing an identifier to its
constructor; when it fires, that identifier appears in `fired` so the Ruby
side can invoke the matching block.

## Dynamic dispatch

`Call(recv, method, args...) (any, error)` dispatches a snake_case method
name to the matching `Module` method, coercing Ruby scalars / Arrays / Hashes
to the Go parameter types (a trailing `error` return is unwrapped) — the
single entry point an rbgo `method_missing` shim drives. `Methods(recv)`
lists the accepted names.

## Usage from Go

```go
m := widgets.NewModule()
root := m.VBox()
title := m.Label("Hello")
ok := m.Button("OK", "on_ok")
_ = m.AddWidget(root, title)
_ = m.AddWidget(root, ok)
_ = m.Layout(root, 200, 80)

img, _ := m.Render(root, 200, 80)                         // {"pixels":…, "stride":800, …}
out, _ := m.Dispatch(ok, map[string]any{"kind": "click"}) // out["fired"] == []any{"on_ok"}
```

## Usage from Ruby

```ruby
require "widgets"

root  = Widgets.v_box
ok    = Widgets.button("OK", "on_ok")
Widgets.add_widget(root, ok)
Widgets.layout(root, 200, 80)
img   = Widgets.render(root, 200, 80)
fired = Widgets.dispatch(ok, { "kind" => "click" })   # => { "fired" => ["on_ok"], … }
```

The `require "widgets"` binding lives in `rbgo` (a thin `method_missing` shim
over `Call`); it is pending in that repo — see [Getting started in
rbgo](rbgo.md).
