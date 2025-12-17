# Ruby Language History and Cross-Language Influences

## Overview

Ruby was created in **1995** by Yukihiro Matsumoto (Matz), making it contemporary with several other important languages. This document explores the history of syntax and features, particularly around templating and common patterns, from a Ruby-centric perspective.

---

## The `<% %>` Template Syntax: Who Did It First?

### The Timeline

1. **1996 - ASP (Active Server Pages)** - Microsoft
   - Introduced `<% %>` and `<%= %>` syntax
   - `<% %>` for code execution
   - `<%= %>` for output (equivalent to `Response.Write()`)

2. **1999 - JSP (JavaServer Pages)** - Sun Microsystems
   - Adopted similar syntax: `<% %>` and `<%= %>`
   - Clearly inspired by ASP

3. **1996-1997 - ERB (Embedded Ruby)** - Ruby community
   - Adopted `<% %>` and `<%= %>` syntax
   - Likely inspired by ASP (which came first)

4. **2000 - ASP.NET** - Microsoft
   - Kept the `<% %>` syntax from classic ASP
   - Added `<%# %>` for comments (Ruby ERB also has this)

### The Answer

**ASP (1996) was first** with the `<% %>` syntax. Ruby's ERB (1996-1997) likely copied it from ASP, though the timeline is very close. JSP (1999) and ASP.NET (2000) also adopted this syntax, making it a widely copied pattern.

**Ruby's perspective:** ERB was created very early in Ruby's history, and the ASP syntax was the most popular templating syntax at the time. It was a pragmatic choice to adopt a familiar syntax.

---

## Language Timeline and Ruby's Position

### 1995 - The Big Year

**1995** was a pivotal year for programming languages:
- **Ruby** - Created by Matz (released 1995)
- **Java** - Released by Sun (1995)
- **JavaScript** - Created by Brendan Eich (1995)
- **PHP** - Version 2.0 released (1995)

**Ruby's influences (pre-1995):**
- **Perl** (1987) - Text processing, regex
- **Smalltalk** (1972) - Object-oriented, message passing
- **Lisp** (1958) - Functional programming, metaprogramming
- **Python** (1991) - Clean syntax, "batteries included"

### Post-1995 Languages

- **C#** (2000) - Microsoft's answer to Java
- **C#/.NET** - Heavily influenced by Java, but also borrowed from Ruby
- **Node.js** (2009) - JavaScript on the server
- **TypeScript** (2012) - Typed JavaScript

---

## Features: Who Copied Whom?

### 1. Blocks and Closures

**Ruby (1995):**
```ruby
[1, 2, 3].each do |item|
  puts item
end
```

**JavaScript (1995):**
- Had functions, but not blocks/closures in the Ruby sense
- Closures became popular later (2000s)

**C# (2000):**
```csharp
new List<int> { 1, 2, 3 }.ForEach(item => Console.WriteLine(item));
```
- Lambda expressions added in C# 3.0 (2007)
- **Ruby had this first** (1995)

**Java (1995):**
- No closures until Java 8 (2014)
- **Ruby had this first** (1995)

**PHP:**
- Closures added in PHP 5.3 (2009)
- **Ruby had this first** (1995)

**Verdict:** Ruby was early with blocks/closures. Many languages copied this pattern later.

---

### 2. Package Management (Gems)

**Ruby Gems (2003):**
- First package manager for Ruby
- `gem install`, `Gemfile`, etc.

**npm (2010):**
- Node.js package manager
- Clearly inspired by Ruby Gems
- `package.json` similar to `Gemfile`

**Composer (2012):**
- PHP package manager
- `composer.json` similar to `Gemfile`
- **Copied from Ruby Gems**

**NuGet (2010):**
- .NET package manager
- Similar concepts to Ruby Gems
- **Inspired by Ruby Gems**

**Verdict:** Ruby Gems was one of the first modern package managers. npm, Composer, and NuGet all borrowed concepts from it.

---

### 3. MVC Web Frameworks

**Ruby on Rails (2004):**
- Popularized MVC for web applications
- Convention over configuration
- Scaffolding, migrations, etc.

**ASP.NET MVC (2009):**
- Microsoft's MVC framework
- Clearly inspired by Rails
- Similar routing, controllers, views pattern
- **Copied from Rails**

**Django (2005):**
- Python web framework
- Similar timeline to Rails
- Both influenced each other

**Express.js (2010):**
- Node.js framework
- Simpler than Rails, but similar concepts
- **Inspired by Rails**

**Laravel (2011):**
- PHP framework
- Very Rails-like
- Artisan CLI (like Rails generators)
- Migrations, Eloquent ORM (like ActiveRecord)
- **Heavily copied from Rails**

**Spring MVC (2002):**
- Java framework
- Existed before Rails, but Rails influenced its evolution

**Verdict:** Rails popularized modern MVC web frameworks. Many frameworks copied Rails' patterns, especially Laravel and ASP.NET MVC.

---

### 4. Metaprogramming

**Ruby (1995):**
```ruby
class User
  attr_accessor :name, :email  # Metaprogramming!
end
```

**JavaScript (1995):**
- Has some metaprogramming (prototypes, `Object.defineProperty`)
- But Ruby's metaprogramming is more powerful and easier

**C# (2000):**
- Reflection (metaprogramming) from the start
- Attributes, reflection API
- But more verbose than Ruby

**Java (1995):**
- Reflection API
- Annotations (Java 5, 2004)
- More verbose than Ruby

**PHP:**
- Magic methods (`__get`, `__set`, etc.)
- Reflection API
- But less elegant than Ruby

**Verdict:** Ruby made metaprogramming accessible and elegant. Other languages had reflection, but Ruby's approach influenced how it's used in frameworks.

---

### 5. Mixins/Modules

**Ruby (1995):**
```ruby
module Loggable
  def log(message)
    puts "[LOG] #{message}"
  end
end

class User
  include Loggable
end
```

**Java (1995):**
- Interfaces (but no implementation)
- Default methods in interfaces (Java 8, 2014)
- **Ruby had mixins first**

**C# (2000):**
- Interfaces (but no implementation)
- Extension methods (C# 3.0, 2007) - similar concept
- **Ruby had mixins first**

**JavaScript:**
- No native mixins
- Prototype-based inheritance
- Mixins added via libraries
- **Ruby had mixins first**

**PHP:**
- Traits (PHP 5.4, 2012)
- Very similar to Ruby modules
- **Copied from Ruby**

**Verdict:** Ruby's modules/mixins were innovative. PHP's traits are a direct copy. C# extension methods and Java default methods are similar concepts added later.

---

### 6. String Interpolation

**Ruby (1995):**
```ruby
name = "World"
puts "Hello, #{name}!"  # "Hello, World!"
```

**PHP (1995):**
```php
$name = "World";
echo "Hello, $name!";  // Similar, but different syntax
```

**JavaScript (1995):**
- No string interpolation until ES6 (2015)
- Template literals: `` `Hello, ${name}!` ``
- **Ruby had this first**

**C# (2000):**
- String interpolation in C# 6.0 (2015)
- `$"Hello, {name}!"`
- **Ruby had this first**

**Java:**
- No string interpolation until Java 15 (2020) - text blocks
- **Ruby had this first**

**Verdict:** Ruby and PHP both had string interpolation early (1995). JavaScript and C# added it much later, likely inspired by Ruby/PHP.

---

### 7. Symbol Type

**Ruby (1995):**
```ruby
:name  # Symbol
"name" # String
```

**JavaScript:**
- No native symbol type until ES6 (2015)
- `Symbol('name')`
- **Ruby had this first**

**C#:**
- No symbol type
- Uses strings or enums

**Java:**
- No symbol type
- Uses strings or enums

**PHP:**
- No symbol type
- Uses strings

**Verdict:** Ruby's symbols were unique. JavaScript added symbols much later, possibly inspired by Ruby.

---

### 8. Method Chaining / Fluent Interface

**Ruby (1995):**
```ruby
[1, 2, 3].map { |x| x * 2 }.select { |x| x > 2 }
```

**JavaScript (1995):**
- Had method chaining, but less elegant
- jQuery (2006) popularized fluent interfaces
- **Ruby had this first**

**C# (2000):**
- LINQ (2007) popularized method chaining
- `list.Select(x => x * 2).Where(x => x > 2)`
- **Ruby had this first**

**Java:**
- Streams API (Java 8, 2014) added method chaining
- `list.stream().map(x -> x * 2).filter(x -> x > 2)`
- **Ruby had this first**

**PHP:**
- Method chaining added in various libraries
- **Ruby had this first**

**Verdict:** Ruby's method chaining was elegant from the start. Many languages added similar patterns later, especially for collections/streams.

---

### 9. Convention over Configuration

**Ruby on Rails (2004):**
- Popularized "convention over configuration"
- File naming, directory structure, etc.

**ASP.NET MVC (2009):**
- Adopted convention over configuration
- Similar folder structure to Rails
- **Copied from Rails**

**Laravel (2011):**
- Heavy use of conventions
- **Copied from Rails**

**Django (2005):**
- Also uses conventions
- Influenced by Rails

**Verdict:** Rails popularized this philosophy. Many frameworks copied it.

---

### 10. ActiveRecord Pattern / ORM

**Ruby on Rails (2004):**
- ActiveRecord ORM
- `User.find(1)`, `user.save`, etc.

**Hibernate (2001):**
- Java ORM
- Existed before Rails
- But Rails made it more accessible

**Entity Framework (2008):**
- .NET ORM
- Similar patterns to ActiveRecord
- **Inspired by Rails**

**Eloquent (2011):**
- Laravel's ORM
- Very similar to ActiveRecord
- **Copied from Rails**

**Sequelize (2010):**
- Node.js ORM
- Similar patterns
- **Inspired by Rails**

**Verdict:** Rails' ActiveRecord pattern was influential. Many ORMs copied its approach, especially Eloquent (PHP) and Entity Framework (.NET).

---

## What Ruby Copied

Ruby didn't exist in a vacuum. Here's what Ruby borrowed:

### From Perl
- Regular expressions as first-class citizens
- Text processing focus
- `$_` (implicit variable)
- `$` for global variables

### From Smalltalk
- Everything is an object
- Message passing
- Blocks/closures
- `self` keyword

### From Lisp
- Metaprogramming
- Code as data
- Functional programming concepts

### From Python
- Clean, readable syntax
- "Batteries included" philosophy
- Indentation matters (somewhat)

### From C
- Basic syntax structure
- Operators
- Control flow

---

## Modern Influences: Ruby → Others

### What Modern Languages Copied from Ruby

1. **Package Management**
   - npm (Node.js) - copied from Ruby Gems
   - Composer (PHP) - copied from Ruby Gems
   - NuGet (.NET) - inspired by Ruby Gems

2. **Web Frameworks**
   - Laravel (PHP) - heavily copied Rails
   - ASP.NET MVC - copied Rails patterns
   - Express.js - inspired by Rails
   - Django - influenced by Rails

3. **ORM Patterns**
   - Eloquent (PHP) - copied ActiveRecord
   - Entity Framework - inspired by ActiveRecord
   - Sequelize (Node.js) - inspired by ActiveRecord

4. **Language Features**
   - PHP Traits - copied Ruby modules
   - JavaScript Symbols - possibly inspired by Ruby
   - String interpolation (C#, JavaScript) - copied from Ruby
   - Method chaining - Ruby did it elegantly first

5. **Development Tools**
   - Rake → Grunt, Gulp (Node.js)
   - Bundler → npm, Composer
   - Rails generators → Artisan (Laravel), Yeoman (Node.js)

---

## The Rails Effect

**Ruby on Rails (2004)** was a game-changer. It didn't just influence Ruby developers - it influenced the entire web development industry:

### Direct Copies
- **Laravel** - "PHP on Rails"
- **ASP.NET MVC** - Microsoft's Rails
- **Grails** (Groovy) - "Rails for Java"

### Concepts Borrowed
- **Convention over Configuration** - copied everywhere
- **Scaffolding** - copied in many frameworks
- **Migrations** - standard in modern frameworks
- **RESTful routing** - popularized by Rails
- **Asset pipeline** - copied in various forms

---

## Summary: Ruby's Influence Timeline

### 1995-2000: Ruby's Early Years
- Ruby created (1995)
- Borrowed from Perl, Smalltalk, Lisp
- ERB templating (copied from ASP)
- Blocks/closures (innovative)

### 2000-2005: Ruby Matures
- Ruby Gems (2003) - first modern package manager
- Ruby on Rails (2004) - game changer
- Rails popularizes MVC, conventions, ORM

### 2005-2010: Ruby Influences Others
- Laravel (2011) - heavily copies Rails
- ASP.NET MVC (2009) - copies Rails patterns
- npm (2010) - copies Ruby Gems
- Many frameworks adopt Rails concepts

### 2010-Present: Ruby's Legacy
- Ruby's features copied into other languages
- String interpolation (C#, JavaScript)
- Symbols (JavaScript)
- Traits (PHP)
- Method chaining everywhere

---

## Key Takeaways

### What Ruby Did First (or Early)
1. ✅ Blocks/closures (1995) - before most languages
2. ✅ Package management (Gems, 2003) - before npm, Composer, NuGet
3. ✅ Modern MVC web frameworks (Rails, 2004) - influenced many
4. ✅ Convention over configuration - popularized by Rails
5. ✅ Elegant metaprogramming - made it accessible
6. ✅ Mixins/modules - PHP copied as traits
7. ✅ String interpolation - C# and JavaScript copied later
8. ✅ Symbols - JavaScript copied later

### What Ruby Copied
1. ❌ Template syntax (`<% %>`) - from ASP (1996)
2. ❌ Regex - from Perl
3. ❌ OOP concepts - from Smalltalk
4. ❌ Metaprogramming ideas - from Lisp
5. ❌ Clean syntax - from Python

### Ruby's Unique Contributions
- **Developer happiness** - "Matz is nice, so I am nice"
- **Elegant syntax** - "Beautiful code"
- **Convention over configuration** - popularized by Rails
- **Rapid web development** - Rails changed the industry

---

## Conclusion

Ruby (1995) was created at a pivotal time, borrowing the best ideas from Perl, Smalltalk, Lisp, and Python. It then innovated with blocks, elegant syntax, and powerful metaprogramming.

**Ruby on Rails (2004)** was the real game-changer. It didn't just influence Ruby developers - it influenced the entire web development industry. Many modern frameworks (Laravel, ASP.NET MVC, Express.js) and tools (npm, Composer) can trace their origins back to Rails and Ruby Gems.

While Ruby copied the `<% %>` template syntax from ASP, it gave back much more: package management, modern MVC frameworks, convention over configuration, and elegant language features that are now standard in many languages.

**From a Ruby-centric perspective:** Ruby was a net contributor to the programming world. It borrowed some syntax (like ERB from ASP), but it gave back far more in terms of frameworks, tools, and language features that are now industry standard.
