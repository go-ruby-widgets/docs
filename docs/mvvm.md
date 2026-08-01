# mvvm

The pure-Go, Ruby-runtime-independent core of the Ruby **`mvvm`** gem: the
data-binding layer of the go-widgets ecosystem — an **Observable** property,
a **Command** action and an **ObservableList** collection. It is a thin
adapter over the dependency-free primitives of
[`go-widgets/mvvm`](https://github.com/go-widgets/mvvm).

**CGO-free**, builds and tests identically on `amd64`, `arm64`, `riscv64`,
`loong64`, `ppc64le`, `s390x`. **100 % statement coverage**, race-clean.

## The callback seam (id + drain)

A hosted Go library cannot synchronously call a Ruby block, so this package
does not try to. Instead every notification — an observable change, a
command execution, a collection change — is a **registered callback id**
plus a **queued, Ruby-shaped event Hash**. The rbgo binding polls the queue
with `drain_events` once per UI tick and dispatches each Hash to the Ruby
callback named by its `"callback_id"`:

```
Ruby block  ──registers──▶  callback id  ──stored on subscribe/observe/command
   ▲                                              │
   │                                     Go mutation queues an event Hash
   └────────dispatch──── drain_events ◀───────────┘   (once per UI tick)
```

`subscribe` / `observe` take a callback id and return an integer subscription
id (pass it to `unsubscribe` / `unobserve`); `drain_events` yields the
pending events and empties the queue. Ruby owns the actual blocks.

## `Module`

The package-level receiver (the `Mvvm` module under rbgo):

| Method | Returns |
| --- | --- |
| `observable(initial)` | an `Observable` handle |
| `command(can_execute_id, execute_id)` | a `Command` handle |
| `observable_list(items)` | an `ObservableList` handle |
| `drain_events` | Array of event Hashes (drains the queue) |

## `Observable` — a bindable property

| Method | Returns |
| --- | --- |
| `get` | the current value (scalar, Hash or Array) |
| `set(value)` | — (queues a `changed` event when the value differs by content) |
| `subscribe(callback_id)` | an Int subscription id |
| `unsubscribe(sub_id)` | — |

## `Command` — a bindable action

| Method | Returns |
| --- | --- |
| `can_execute?` | Bool |
| `execute(args)` | — (queues an `execute` event carrying `args` when executable) |
| `set_can_execute(bool)` | — (records executability, fires a change) |
| `raise_can_execute_changed` | — |

## `ObservableList` — a bindable collection

| Method | Returns |
| --- | --- |
| `add(v)` / `insert(i, v)` / `set(i, v)` / `remove_at(i)` / `move(from, to)` / `clear` | — (each queues a collection-changed event) |
| `get(i)` | the item, or raises `IndexError` out of range |
| `size` | Int |
| `slice` | a defensive-copy Array |
| `observe(callback_id)` | an Int observer id |
| `unobserve(sub_id)` | — |

## Event shapes

Each drained event is a Hash with a `"callback_id"`, a `"kind"` and its
payload:

```
# observable change
{ "callback_id"=>, "kind"=>"changed", "value"=> }
# command
{ "callback_id"=>, "kind"=>"execute", "args"=>[...] }
{ "callback_id"=>, "kind"=>"can_execute_changed" }
# collection change (kind => "collection_changed")
{ "callback_id"=>, "action"=>"insert"|"remove"|"replace"|"move"|"reset",
  "index"=>, "to"=>, "count"=>, "items"=>[...] }
```

## Usage from Go

```go
m := mvvm.NewModule()

name := m.Observable("")
name.Subscribe("on_name")
name.Set("Ada")

names := m.ObservableList(nil)
names.Observe("on_names")
names.Add("Ada")

save := m.Command("can_save", "do_save")
save.SetCanExecute(true)
save.Execute([]any{"now"})

for _, ev := range m.DrainEvents() {
    h := ev.(map[string]any)
    fmt.Println(h["callback_id"], h["kind"])
}
// on_name changed
// on_names collection_changed
// can_save can_execute_changed
// do_save execute
```

## Usage from Ruby

```ruby
require "mvvm"

name  = Mvvm.observable("")
name.subscribe(:on_name)

names = Mvvm.observable_list([])
names.observe(:on_names)

save  = Mvvm.command(:can_save, :do_save)
save.set_can_execute(true)

name.set("Ada")
names.add(name.get)
save.execute([])                       # queues {callback_id: :do_save, ...}

Mvvm.drain_events.each do |ev|         # => Array<Hash>, once per UI tick
  dispatch(ev[:callback_id], ev)       # Ruby runs the actual block
end
```

`Methods(recv)` lists every snake_case name `Call` accepts for a handle, and
`Call(recv, name, args...)` is the uniform dynamic entry point rbgo binds.
The `require "mvvm"` binding lives in `rbgo`; it is pending in that repo —
see [Getting started in rbgo](rbgo.md).
