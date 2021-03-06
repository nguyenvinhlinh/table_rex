<img src="http://i.imgur.com/ipa4UVa.png" width="500" />

[![Hex.pm](https://img.shields.io/hexpm/v/table_rex.svg)](https://hex.pm/packages/table_rex)
[![Build Status](https://travis-ci.org/djm/table_rex.svg?branch=master)](https://travis-ci.org/djm/table_rex)
[![Inline docs](http://inch-ci.org/github/djm/table_rex.svg)](http://inch-ci.org/github/djm/table_rex)

**An Elixir app which generates text-based tables for display**

Currently supports output:

* in customisable ASCII format.
* with your own renderer.

####Features

* A one-liner for those that just want to render ASCII tables with sane defaults.
* Support for table titles & alignable headers.
* Support for column & cell level alignment (center, left, right).
* Automatic cell padding but also the option to set padding per column<sup>1</sup>.
* Frame the table with various vertical & horizontal styles<sup>1</sup>.
* Style the table how you wish with custom separators<sup>1</sup>.
* Works well with the Agent module to allow for easy sharing of state.
* Clients can supply their own rendering modules and still take advantage of the table manipulation API.

<sup>1</sup> The text renderer supports these features, others may not or might not need to.

####Stability

This software is pre-v1 and therefore the public API *may* change; any breaking changes will be clearly
denoted in the [CHANGELOG](CHANGELOG.md). After v1, the API will be stable.

####Documenation

See the quick start below or check out the [full API docs at HexDocs](https://hexdocs.pm/table_rex/).


## Installation

The package is [available on Hex](https://hex.pm/packages/table_rex), therefore:

**Add** `table_rex` to your list of dependencies in `mix.exs`:

```elixir
def deps do
  [{:table_rex, "~> 0.4.0"}]
end
```

Then **start** `table_rex` by adding it to `application/0` in `mix.exs`:

```elixir
def application do
  [applications: [:logger, :table_rex]]
end
```

##Quick Start

Use the `TableRex.quick_render` and `TableRex.quick_render!` functions; for those that just want a table quickly.

Given this data:

```elixir
title = "Drum & Bass Releases"
header = ["Artist", "Track", "Label", "Year"]
rows = [
  ["Konflict", "Cyanide", "Renegade Hardware", 1999],
  ["Marcus Intalex", "Temperance", "Soul:r", 2004],
  ["Kryptic Minds", "The Forgotten", "Defcom Records", 2007]
]
```

###TableRex.quick_render!/1

```elixir
rows
|> TableRex.quick_render!
|> IO.puts
```

```
+-----------------+---------------+-------------------+------+
|    Konflict     |    Cyanide    | Renegade Hardware | 1999 |
| Marcus Intalex  |  Temperance   |      Soul:r       | 2004 |
|  Kryptic Minds  | The Forgotten |  Defcom Records   | 2007 |
+-----------------+---------------+-------------------+------+
```

###TableRex.quick_render!/2

```elixir
rows
|> TableRex.quick_render!(header)
|> IO.puts
```

```
+----------------+---------------+-------------------+------+
|     Artist     |     Track     |       Label       | Year |
+----------------+---------------+-------------------+------+
|    Konflict    |    Cyanide    | Renegade Hardware | 1999 |
| Marcus Intalex |  Temperance   |      Soul:r       | 2004 |
| Kryptic Minds  | The Forgotten |  Defcom Records   | 2007 |
+----------------+---------------+-------------------+------+
```

###TableRex.quick_render!/3

```elixir
rows
|> TableRex.quick_render!(header, title)
|> IO.puts
```

```
+-----------------------------------------------------------+
|                   Drum & Bass Releases                    |
+----------------+---------------+-------------------+------+
|     Artist     |     Track     |       Label       | Year |
+----------------+---------------+-------------------+------+
|    Konflict    |    Cyanide    | Renegade Hardware | 1999 |
| Marcus Intalex |  Temperance   |      Soul:r       | 2004 |
| Kryptic Minds  | The Forgotten |  Defcom Records   | 2007 |
+----------------+---------------+-------------------+------+
```

###Utilising TableRex.Table for deeper customisation

These examples all use: `alias TableRex.Table` to shorten the namespace.

**Set alignment & padding for specific columns or column ranges:**

```elixir
Table.new
|> Table.add_rows(rows)
|> Table.set_header(header)
|> Table.set_column_meta(0, align: :right, padding: 5)
|> Table.set_column_meta(1..2, align: :left)
|> Table.render!
|> IO.puts
```

```
+------------------------+---------------+-------------------+------+
|             Artist     | Track         | Label             | Year |
+------------------------+---------------+-------------------+------+
|           Konflict     | Cyanide       | Renegade Hardware | 1999 |
|     Marcus Intalex     | Temperance    | Soul:r            | 2004 |
|      Kryptic Minds     | The Forgotten | Defcom Records    | 2007 |
+------------------------+---------------+-------------------+------+
```

**Change the table styling:**

```elixir
Table.new
|> Table.add_rows(rows)
|> Table.set_header(header)
|> Table.render!(header_separator_symbol: "=", horizontal_style: :all)
|> IO.puts
```

```
+----------------+---------------+-------------------+------+
|     Artist     |     Track     |       Label       | Year |
+================+===============+===================+======+
|    Konflict    |    Cyanide    | Renegade Hardware | 1999 |
+----------------+---------------+-------------------+------+
| Marcus Intalex |  Temperance   |      Soul:r       | 2004 |
+----------------+---------------+-------------------+------+
| Kryptic Minds  | The Forgotten |  Defcom Records   | 2007 |
+----------------+---------------+-------------------+------+
```

*Available render options:*

`horizontal_style`: one of `:off`, `:frame` (just the outside frame), `:header` (frame + header seperators), `:all` (frame + header + row seperators).  
`vertical_style`: one of `:off`, `:frame` (just the outside frame) or `:all` (frame + column separators).  
`horizontal_symbol`: used for drawing horizontal row separators. Default: `-`.  
`vertical_symbol`: used for drawing vertical separators. Default: `-`.  
`intersection_symbol`: used to draw the symbol where horizontal and vertical seperators intersect. Default: `+`.  
`top_frame_symbol`: used to draw the frame's top horizontal separator. Default: `-`.  
`title_separator_symbol`:  used to draw the horizontal separator under the (optional) title. Default: `-`.  
`header_separator_symbol`: used to draw the horizontal separator under the (optional) header. Default: `-`.  
`bottom_frame_symbol`: used to draw the frame's bottom horizontal separator. Default: `-`.  

**Set cell level meta (including for the header cells):**

```elixir
Table.new
|> Table.add_rows(rows)
|> Table.set_header(header)
|> Table.set_header_meta(0, align: :left)
|> Table.set_cell_meta(2, 1, align: :right)
|> Table.render!
|> IO.puts
```

```
+----------------+---------------+-------------------+------+
| Artist         |     Track     |       Label       | Year |
+----------------+---------------+-------------------+------+
|    Konflict    |    Cyanide    | Renegade Hardware | 1999 |
| Marcus Intalex |  Temperance   |            Soul:r | 2004 |
| Kryptic Minds  | The Forgotten |  Defcom Records   | 2007 |
+----------------+---------------+-------------------+------+
```

*NB:*

* in `set_header_meta` the `0` referred to the first cell of the header which was aligned left.
* in `set_cell_meta`, the `2` referred to the `Label` column, and the `1` referred to the row index. The cell was aligned right.

**Change/pass in your own renderer module:**

The default renderer is `TableRex.Renderer.Text`.

Custom renderer modules must be behaviours of `TableRex.Renderer`.

```elixir
Table.new
|> Table.add_rows(rows)
|> Table.set_header(header)
|> Table.render!(renderer: YourCustom.Renderer.Module)
```

**Go mad:**

```elixir
Table.new
|> Table.add_rows(rows)
|> Table.set_header(header)
|> Table.render!(horizontal_style: :all, top_frame_symbol: "*", header_separator_symbol: "=", horizontal_symbol: "~", vertical_symbol: "!")
|> IO.puts
```

```
+****************+***************+*******************+******+
!     Artist     !     Track     !       Label       ! Year !
+================+===============+===================+======+
!    Konflict    !    Cyanide    ! Renegade Hardware ! 1999 !
+~~~~~~~~~~~~~~~~+~~~~~~~~~~~~~~~+~~~~~~~~~~~~~~~~~~~+~~~~~~+
! Marcus Intalex !  Temperance   !      Soul:r       ! 2004 !
+~~~~~~~~~~~~~~~~+~~~~~~~~~~~~~~~+~~~~~~~~~~~~~~~~~~~+~~~~~~+
! Kryptic Minds  ! The Forgotten !  Defcom Records   ! 2007 !
+----------------+---------------+-------------------+------+
```

##Run the tests

We have an extensive test suite which helps showcase project usage. For example: the [quick render functions](https://github.com/djm/table_rex/blob/master/test/table_rex_test.exs),
[table manipulation API](https://github.com/djm/table_rex/blob/master/test/table_rex/table_test.exs) or [the text renderer module](https://github.com/djm/table_rex/blob/master/test/table_rex/renderer/text_test.exs).

To run the test suite, from the project directory, do:

```bash
mix test
```

##Roadmap/Contributing

We use the Github Issues tracker.

If you have found something wrong, please raise an issue.

If you'd like to contribute, check the issues to see where you can help.

Contributions are welcome from anyone at any time but if the piece of work is significant in size, please raise an issue first.

##License

MIT. See the [full license](LICENSE).


##Thanks

* Ryanz720, for the [original T-Rex image](https://commons.wikimedia.org/wiki/File:Trex_Roar.jpg).

* Everyone in #elixir-lang on freenode, for answering the endless questions.
