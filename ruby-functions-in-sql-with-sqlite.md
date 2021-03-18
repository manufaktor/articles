# Ruby functions in SQL with SQLite

While trying to use UUIDs in SQLite I was diving a bit into the source code of the SQLite3/Ruby interface. You may better know this by it's gem name, `sqlite3`.
I was trying to understand how to load SQLite extensions. Through the ruby interface, it's done using `db.enable_load_extension(true)` and `db.load_extension("extname")`).

But I discovered something more interesting: defining my own ruby functions, and using them in SQLite.

## Here's how it works

Let's say instead of UUIDs, we want to use ULIDs for our primary keys. ULIDs are cool, because they contain a timestamp and are sortable. They're also a bit shorter and more URL friendly than UUIDs. So let's use it, and thankfully there is a ruby library for that!

```ruby
# Gemfile

gem 'sqlite3'
gem 'ulid'
```

On the database object we get from the ruby interface, there's a `create_function` method. You pass it a block and receiving a `function` object, on which you have to store the `result` of whatever you want to do with ruby inside your SQL statement.

```ruby
require 'sqlite3'
require 'ulid'

SQLite3::Database.new("data.db") do |db|
  db.create_function('ulid', 0) do |func|
    func.result = ULID.generate
  end
end
```

Pretty easy, now we have a `ulid` function at our disposal in any SQL statement we send to our SQLite database:

```ruby
puts db.execute("SELECT ulid()") # prints a ULID, like "01F12R4J7HCEPAWJV9EXNQBK14"
```

Nice!

_Mar. 2021_
