---
title: Configuring grid's attributes
date: 2016-01-07 05:13 UTC
tags:
excerpt: "Different ways to configure attributes shown in grids and their forms"
---

Netzke 1.0 streamlines configuration of grid columns and grid's forms layouts. In this post I'll be showing how to
address some typical use cases.

### Listing columns explicitly

One way to configure grid columns is to explicitly list them by either providing the `columns` config option:

```ruby
def configure(c)
  super
  c.model = Book
  c.columns = [
    { name: "title", width: 150 },
    { name: "author__last_name", width: 150 },
    :exemplars
  ]
end
```

...or by overriding the `columns` method (same effect):

```ruby
def columns
  [
    { name: "title", width: 150 },
    { name: "author__last_name", width: 150 },
    :exemplars
  ]
end
```

The grid's forms will use the specified columns, too, so, the result will look like this:

<img src="/images/2016-01-07-01.png" alt="grid one" width=750 height=400/>

Note, that starting from Netzke 1.0, configuring a column with `read_only` won't make the corresponding form field
read-only. For that you'll need to use the `attribute` DSL method (see below).

### Providing form layout explicitly

To override the fields, you may either provide the `form_items` config option, or by overriding the
equally named method:

```ruby
def form_items
  [
    :title,
    {
      layout: :hbox,
      border: false,
      defaults: { flex: 1, layout: :anchor, border: false },
      items: [
        { items: [:author__last_name], defaults: { anchor: "-25"} },
        { items: [:exemplars], defaults: { anchor: "100%"} },
      ]
    }
  ]
end
```

<img src="/images/2016-01-07-02.png" alt="grid two" width=750 height=400/>

### Overriding individual attributes

However sometimes it's not possible (or desirable) to be explicit about all the columns that must be present. What if
all we want is to hide a specific attribute? This is possible by using the `attribute` DSL method. For example, to
exclude the `created_at` attribute from *both* grid and form:

```ruby
attribute :created_at do |c|
  c.excluded = true
end
```

To make an attribute read-only:

```ruby
attribute :exemplars do |c|
  c.read_only = true
end
```

Besides using the `attribute` DSL method, you may also use the `attribute_overrides` config option. This may be
handy when you nest a pre-defined grid component in which you want to override specific attributes. For example, imagine
a composite component that nests `Authors` and `Books`, and you want to hide the `author__name` column from the books
grid, as well as make the `exemplars` column read-only. The code for that component may look something like this:

```ruby
class AuthorsAndBooks < Netzke::Base
  def configure(c)
    super
    c.layout = { type: :hbox, align: :stretch }
    c.defaults = { flex: 1 }
    c.items = [:authors, :books]
  end

  component :authors

  component :books do |c|
    c.attribute_overrides = {
      author__name: { excluded: true },
      exemplars: { read_only: true }
    }
  end
end
```

### Overriding column and field config

When overriding an attribute (either via the `attribute` DSL method or by means of `attribute_overrides` config option),
you may use the special `column_config` and `field_config` keys to specify configuration options that are specific to a
column (such as its width) or to a form field (such as `maxValue` for Number field or `maxLength` for Text field;
refer to the Ext JS documentation for details). Here's an example:

```ruby
class Books < Netzke::Grid::Base
  def model
    Book
  end

  attribute :exemplars do |c|
    c.column_config = { width: 40, text: 'Ex.' }
    c.field_config = { max_value: 99 }
  end
end
```

The result will look like this:

<img src="/images/2016-01-07-03.png" alt="grid two" width=750 height=400/>

For more details on attribute and column configuration, please refer to the
[docs](http://www.rubydoc.info/github/netzke/netzke-basepack/Netzke/Basepack/Attributes).
