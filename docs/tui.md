# tui

The pure-Go, Ruby-runtime-independent core of the Ruby **`tui`** gem: a
terminal-cell user-interface toolkit — widgets, layout containers, frame
rendering and input routing. It is a thin adapter over the terminal-cell
widget toolkit [`go-widgets/tui`](https://github.com/go-widgets/tui), sharing
the `Widget`/`Event`/`Rect`/`Theme` vocabulary of
[`go-widgets/toolkit`](https://github.com/go-widgets/toolkit).

It exposes them through Ruby-facing handles — `Module` and `Widget` — whose
methods return **Ruby-shaped values**. **CGO-free**, builds and tests
identically on `amd64`, `arm64`, `riscv64`, `loong64`, `ppc64le`, `s390x`.
**100 % test coverage**, race-clean; rendering is asserted at **cell
precision** — no real terminal required.

## `Module` — construction

Each returns a `Widget` handle:

| Method | Widget |
| --- | --- |
| `label(text)` | static single-line text |
| `button(label)` | clickable action |
| `entry(initial)` | single-line text input |
| `check_button(label, checked)` | boolean toggle |
| `list_box(items)` | single-selection list |
| `progress_bar` | horizontal fill indicator |
| `container` | **border layout**: fixed header + footer bands, filling body, overlays |
| `h_split` / `v_split` | draggable **split** panes |
| `notebook` | **card layout**: a tab strip selects the visible page |

## `Module` — layout, render and events

| Method | Returns |
| --- | --- |
| `set_size(root, cols, rows)` | lays the tree out to fill the terminal |
| `bounds(w)` | Hash `{x, y, w, h}` |
| `render(root, cols, rows)` | the frame as an **ANSI String** |
| `render_cells(root, cols, rows)` | the decoded **cell grid** (Hash, see below) |
| `decode_cells(ansi, cols, rows)` | a cell grid decoded from any ANSI string |
| `dispatch(root, event)` | Hash `{fired => [ids], repaint => true}` |

## `Widget`

An opaque handle. Its methods mutate props (`set_text`, `set_align`,
`set_label`, `set_checked`, `set_fraction`, `set_items`, `set_selected`, …),
compose a tree (`set_header` / `set_body` / `set_footer` / `add_overlay`,
`set_left` / `set_right`, `set_top` / `set_bottom`, `add_tab`, `set_active`)
and wire callbacks by id (`on_click`, `on_toggle`, `on_select`, `on_change`,
`on_tab_changed`). Every method checks the handle's `kind` and returns a
clear error when it does not apply.

## The render seam

`render` returns a self-contained ANSI stream a Ruby host prints straight to
a terminal. `render_cells` (and `decode_cells` over any ANSI string) return
the decoded grid as a Hash:

```
{ "cols"=>, "rows"=>,
  "cells"=> [ [ {"char"=>String, "fg"=>color|nil, "bg"=>color|nil}, ... ], ... ],
  "text" => [ "row 0 text", "row 1 text", ... ] }
```

where a color is `{"r"=>, "g"=>, "b"=>}` or `nil` for the terminal default.
The decode path is backed by `go-widgets/tui`'s `DecodeANSI`, so a test can
assert the output at cell precision.

## Events

`dispatch` routes one event Hash — `{"kind"=>, "x"=>, "y"=>, "code"=>,
"ctrl"=>, "shift"=>}` — into the tree, whose containers translate
coordinates and deliver it to the right leaf. Recognised kinds: `click`,
`key_down`, `key_up`, `char`, `mouse_drag`, `mouse_up`, `tick`. It returns the
ids of the callbacks that fired and whether a repaint is warranted.

## Usage from Go

```go
m := tui.NewModule()

button := m.Button("OK")
_ = button.OnClick("ok")

root := m.Container()
_ = root.SetHeaderHeight(1)
_ = root.SetHeader(m.Label("Title"))
_ = root.SetBody(button)

_ = m.SetSize(root, 40, 10)

frame, _ := m.Render(root, 40, 10) // an ANSI String
res, _ := m.Dispatch(button, map[string]any{"kind": "click"})
// res["fired"] == []any{"ok"}
```

## Usage from Ruby

```ruby
require "tui"

button = Tui.button("OK")
button.on_click("ok")

root = Tui.container
root.set_header_height(1)
root.set_header(Tui.label("Title"))
root.set_body(button)

Tui.set_size(root, 40, 10)
puts Tui.render(root, 40, 10)                     # => String (ANSI frame)

fired = Tui.dispatch(button, {"kind" => "click"}) # => {"fired"=>["ok"], "repaint"=>true}
```

`Methods(recv)` lists every snake_case name `Call` accepts for a handle, and
`Call(recv, name, args...)` is the uniform dynamic entry point rbgo binds.
The `require "tui"` binding lives in `rbgo`; it is pending in that repo — see
[Getting started in rbgo](rbgo.md).
