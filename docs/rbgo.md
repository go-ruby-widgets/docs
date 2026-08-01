# Getting started in rbgo

!!! note "Status"
    `require "widgets"`, `require "tui"` and `require "mvvm"` are **pending**
    in [go-embedded-ruby/ruby](https://github.com/go-embedded-ruby/ruby)
    (rbgo). This page documents the intended surface — everything below is
    exactly what the three adapters already provide today via `Call`; only
    the thin `method_missing` shim inside rbgo is outstanding.

## Why three separate adapter repos

None of `widgets`, `tui` or `mvvm` import rbgo or the Ruby runtime. Each is a
standalone Go library whose handles already speak in Ruby-shaped values —
Hashes, Arrays, scalars — and whose `Call` function is the exact uniform
surface a `method_missing` shim needs. rbgo's job, once wired, is only to:

1. register a Ruby module (`Widgets`, `Tui`, `Mvvm`) backed by each adapter's
   `NewModule()`;
2. route `Widgets.foo(*args)` / `Tui.foo(*args)` / `Mvvm.foo(*args)` to
   `Call(mod, "foo", args...)`;
3. wrap any handle returned by a constructor (a widget id in `widgets`, a
   `Widget` in `tui`, an `Observable`/`Command`/`ObservableList` in `mvvm`) as
   a Ruby object whose own `method_missing` also routes through `Call`.

## Intended usage: widgets

```ruby
require "widgets"

root  = Widgets.v_box
title = Widgets.label("Hello")
ok    = Widgets.button("OK", "on_ok")
Widgets.add_widget(root, title)
Widgets.add_widget(root, ok)
Widgets.layout(root, 200, 80)

img = Widgets.render(root, 200, 80)
# => { "pixels" => <RGBA bytes>, "stride" => 800, "w" => 200, "h" => 80 }

fired = Widgets.dispatch(ok, { "kind" => "click" })
# => { "fired" => ["on_ok"], "repaint" => true }
```

## Intended usage: tui

```ruby
require "tui"

button = Tui.button("OK")
button.on_click("ok")

root = Tui.container
root.set_header_height(1)
root.set_header(Tui.label("Title"))
root.set_body(button)

Tui.set_size(root, 40, 10)
puts Tui.render(root, 40, 10)                      # => String (ANSI frame)

fired = Tui.dispatch(button, {"kind" => "click"})  # => {"fired"=>["ok"], "repaint"=>true}
```

## Intended usage: mvvm

```ruby
require "mvvm"

name = Mvvm.observable("")
name.subscribe(:on_name)
name.set("Ada")

Mvvm.drain_events.each do |ev|      # => Array<Hash>, once per UI tick
  dispatch(ev[:callback_id], ev)    # Ruby runs the actual block
end
```

## Argument and result shapes

Common to all three adapters:

| Ruby side | Go side |
| --- | --- |
| Integer / Float | `int`, `int64`, or a numeric coercion |
| String / Symbol | `string` (a callback id may be a Symbol or a String) |
| Hash | `map[string]any` (event Hashes, `add`/`region`/`opts` payloads) |
| `nil` | the zero value — an omitted trailing argument, an unset pointer/slice/map |
| result: single record | a Ruby Hash |
| result: a list | a Ruby Array of Hashes |
| result: a count/flag/string | a plain scalar |
| a live object (widget, `Observable`, `Command`, `ObservableList`) | a Ruby object whose `method_missing` routes back through `Call` |

Any error from the underlying Go call raises inside `Call` and should surface
as a Ruby exception once wired into rbgo's error-mapping convention.

## Standalone Go usage today

Everything above is already exercised, without Ruby, in each repo's README
and `example_test.go`:

- [`widgets/example_test.go`](https://github.com/go-ruby-widgets/widgets/blob/main/example_test.go)
- [`tui/example_test.go`](https://github.com/go-ruby-widgets/tui/blob/main/example_test.go)
- [`mvvm/example_test.go`](https://github.com/go-ruby-widgets/mvvm/blob/main/example_test.go)

Follow [go-embedded-ruby/ruby](https://github.com/go-embedded-ruby/ruby) for
the bindings' landing; once they ship, `require "widgets"` / `"tui"` /
`"mvvm"` will need no code change in these three adapters.
