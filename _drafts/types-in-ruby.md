---
layout: post
title: Working with Types in Ruby
description:
categories: Ruby Typing Sorbet SoftwareArchitecture DuckTyping
---

What is to follow is neither an admonishment nor praise of duck-typing in Ruby or any other language, but a polyglot path towards more discoverable and more maintainable code. We use the facade of Ruby here because the syntax is unobtrusive and if we can avoid meta programming it is as close to pseudocode as I could get on my own.

Let's consider that typing isn't an exclusive concept, and we can have both duck typed and pseudo static typed elements in our interpreted code. Sounds pretty good to me, here is the plan we are going to follow:
- Primitives while Objects in Ruby still represent an "un-complex" way to store simple data.
- Structured data is built from primitives and other structured data.
- Commands contain ephemeral state, which could be primitive or structured.

## Primitives
While this is not an exhaustive list, a majority of the primitives we deal with have values that fall into the categories of; Strings, Numbers, and Boolean. I am intentionally excluding Nil because it's a complex and nuanced, prickly thing that's worth its own discussion. For now, all considerations should be made that no value should be nil, and we will never use Nil as a sentinel for a Void return from our methods and lambdas.

## Structures
Once again a rather short list of possibilities and this time we will make the attempt to rank them for discoverability and portability. So let's go over what we mean by discoverable and portable:

__Portable__ -> Is a trait of structured data that can conveniently be passed between call sites (function calls) and IPC (Inter-Process communication) via some sort of serialization (eg: JSON, Binary, ...). Highly-Portable structures are generally free of behaviors(methods) and dependencies, where construction of an instance doesn't require additional classes.

__Discoverable__ -> The cornerstone of discoverability is the presence of a contract describing the primitives and other structured data contained within the abstraction. Largely, discoverability has two components, their contracts are indivisible and immutable. Let's take a look at an example of a discoverable object.

```ruby
class Discoverable

  attr_reader :first_name, :last_name, :email

  def initialize(
    first_name:
    last_name:
    email:
  )
    @first_name = not_nil("first_name", first_name)
    @last_name = not_nil("last_name", last_name)
    @email = not_nil("email", email)
  end

private
  def not_nil(field, value) = raise "#{field} can't be nil" if value.nil?
end

Discoverable.new(
  first_name: "Paul",
  last_name: "Scarrone",
  email: "spam@spam.spam"
)
```

Some commentary…
1. Use of an initializer with named parameters, while verbose using named vs positional parameters, requires the implementer to meet the same contract as the class describing the structure. This creates a condition where the data is indivisible when the object is constructed.
2. No optional parameters, which means any location where a  `Discoverable` instance is used we can guarantee that at least those fields were required at construction, albeit they may still be nil but we can guarantee they are not. This reinforces the indivisibility of our data as present and usable.
3. Restrictive access control, our data is only readable and incapable of being mutated. This immutability is non-trivial but simple in this case because our values are primitives. This changes when our structure has to deal with reference objects like Arrays and Hashes. We will cover this in the next section.
