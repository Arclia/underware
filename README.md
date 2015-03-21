[![Gem Version](https://badge.fury.io/rb/underware.svg)](http://badge.fury.io/rb/underware)
[![Build Status](https://travis-ci.org/briandamaged/underware.svg)](https://travis-ci.org/briandamaged/underware)

# Underware #

Middleware for Ruby

## Installation ##

```bash
gem install underware
```

## 2-Minute Tour ##

If your object defines a ```#call``` method, then it can be used as middleware.  For example:

```ruby
class Add
  attr_reader :delta

  def initialize(delta)
    @delta = delta
  end

  def call(x)
    puts "BEFORE: #{x} + #{delta}"

    # Pass control to the next piece of middleware
    yield x + delta

    puts "AFTER:  #{x} + #{delta}"
  end
end
```

Now that we've defined some middleware, let's chain it together:

```ruby
require 'underware'

f = Underware.fold([Add.new(1), Add.new(-2), Add.new(3)]) do |x|
  puts "THE VALUE IS: #{x}"
end

f.call(3)
```

This produces the following output:

```text
BEFORE: 3 + 1
BEFORE: 4 + -2
BEFORE: 2 + 3
THE VALUE IS: 5
AFTER:  2 + 3
AFTER:  4 + -2
AFTER:  3 + 1
```

Note that the block to the ```Underware.fold``` method is just for convenience.  You could have also written:

```ruby
require 'underware'

g = Proc.new do |x|
  puts "THE VALUE IS: #{x}"
end

f = Underware.fold([Add.new(1), Add.new(-2), Add.new(3), g])

f.call(3)
```

Wouldn't it be nice if you could somehow fold the middleware and call it at the same time?  Well, guess what -- you can!


```ruby
require 'underware'

Underware.exec_underware([Add.new(1), Add.new(-2), Add.new(3)], 3) do |x|
  puts "THE VALUE IS: #{x}"
end
```

In reality, the ```Underware#exec_underware``` is really intended to be used as a mixin (ie: ```include Underware```).  Otherwise, it's a lot to type!  Fortunately, there's another shortcut:

```ruby
require 'underware'

# This is the same as Underware.exec_underware
Underware([Add.new(1), Add.new(-2), Add.new(3)], 3) do |x|
  puts "THE VALUE IS: #{x}"
end
```

