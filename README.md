NAME
----
  `clee`

TL;DR;
--------
  `clee` is a tiny, 0 dependency, DSL for building über clean CLIs in Ruby

INSTALL
-------
```sh
  gem install clee
```

URI
---
  http://github.com/ahoward/clee

ABOUT
-----

`clee` is a minimalist version of `main` (https://github.com/ahoward/main), a
command line DSL i wrote almost 15 years ago, that has seen over [4 million
downloads](https://drawohara.io/rubygems/)

> then why should i use `clee` instead of `main` ?

* `clee` has 0 dependencies beyond ruby itself

* `clee` is very very small
  ```sh
    drawohara@drawohara.dev:ahoward #=> loc clee/lib

    clee/lib/clee.rb: 478
    ===
    @loc: 478


    drawohara@drawohara.dev:ahoward #=> loc main/lib

    main/lib/main/cast.rb: 139
    main/lib/main/daemon.rb: 395
    main/lib/main/dsl.rb: 65
    main/lib/main/factories.rb: 24
    main/lib/main/getoptlong.rb: 245
    main/lib/main/logger.rb: 43
    main/lib/main/mode.rb: 41
    main/lib/main/parameter.rb: 589
    main/lib/main/program/class_methods.rb: 362
    main/lib/main/program/instance_methods.rb: 274
    main/lib/main/program.rb: 6
    main/lib/main/softspoken.rb: 12
    main/lib/main/stdext.rb: 34
    main/lib/main/test.rb: 69
    main/lib/main/usage.rb: 159
    main/lib/main/util.rb: 96
    main/lib/main.rb: 70
    ===
    @loc: 2623
  ```

* you can use this to decide which to use:

```ruby
  case
    when wants?(:simple, :tiny, :scripting)
      :clee

    when wants?(:powerful, :testable, :complete, :mature)
      :main

    else
      [:clee, :main].sort_by{ rand }.first
  end
```

API
---

##### `clee`'s api is very simple.  it has every feature you need, and none that you
don't, including:

- auto generated help messages
- support for `my_clee --help` and `my_clee help` to 'just work'
- support for modifying help/usage messages simply
- sane exit codes
- support for --options, env=val pairs, etc
- argv parsing
- fancy color'd logging
- modes, and sub-modes

##### the smallest clee script looks like this

```ruby
require 'clee'

clee do
  run do
    p 42
  end
end
```

##### you can name your scripts

```ruby
require 'clee'

clee do
  run do
    help!  #=> this will print a default usage message that will include 'my_clee'
  end
end
```

##### you can alter the default tldr, and help messages

```ruby
require 'clee'

clee do
  tldr <<~____
    avoid using the default 'tldr'
  ____
end
```

```ruby
require 'clee'

clee do
  help <<~____
    NAME
      my_clee

    USAGE
      fully custom help...
  ____
end
```

##### specifying params is trivial

```ruby
require 'clee'

clee do
# support `my_clee --verbose` and -v
#
  option :verbose, :v

# support `my_clee --path=./lib`
#
  option :path, value: :required

# support `my_clee API_KEY=123` *and* `API_KEY=123 my_clee`
#
  env :API_KEY

# support `my_clee --foo=42` *and* `my_clee foo=42` *and* `foo=42 my_clee` syntax
#
  param :foo, value: 'required'

# the interface and help messages work the same way for all the above
#
  def run
    if @options.has_key?(:verbose)
      @verbose = true
    end

    @path = @options.fetch(:path)

    @api_key = @env.fetch(:API_KEY)

    @foo = @parms.fetch(:foo)
  end
end
```

##### modes, and sub-modes, are supported

```ruby
require 'clee'

clee :my_clee do
  run :foo do
    p 42
  end

  run :foo, :bar do
    p 42.0
  end

  run do
    p 42.42
  end
end
```

##### assuming you saved the above as `my_clee`, you could then do
```sh
  ~> my_clee foo     #=> 42
  ~> my_clee foo bar #=> 42.0
  ~> my_clee         #=> 42.42
```

##### `clee` scripts have a sweet dependency-less colored logger that understands what a #tty really is...
```ruby
require 'clee'

clee do
  def run
    log 'hai!'
    log 'hai!', level: :warning
    log 'blue', color: :blue
  end
end
```

##### `clee` ships with a lil code-gen-thang
```sh
~> clee new my_clee > my_clee
~> chmod 755 my_clee
~> ./my_clee
```

i could write more docs but, they would then outnumber the LOC of the library
so:

1. see [./lib/clee.rb](./lib/clee.rb)
2. if that still doesn't float your boat install `ima`, a universal
command-line filter built on `clee`, that brings AI to your CLI and do
something like this

```sh
  ~> gem install clee ima
  ~> ima explain clee to me --context=$(gem which clee)
```

which might produce something like this ->

---

Clee is a Ruby library that provides a simple way to create command-line interfaces (CLI) for Ruby applications. It allows developers to define commands, options, and parameters for their application, and handles the parsing and execution of these commands.

The core features of Clee include:

* Command definition: Clee allows developers to define commands and their associated options and parameters.
* Option parsing: Clee can parse command-line options and parameters, and provides a simple way to define and handle these options.
* Parameter handling: Clee provides a way to handle command-line parameters, including required and optional parameters.
* Help generation: Clee can generate help text for commands and options, making it easy to provide documentation for users.
* Logging: Clee provides a logging mechanism that allows developers to log messages at different levels (e.g. debug, info, warning, error).

Clee is designed to be flexible and customizable, making it easy to integrate into existing Ruby applications. It also provides a number of features that make it easy to use, including automatic help generation and logging.

Some of the key concepts in Clee include:

* Commands: These are the top-level actions that a user can perform with the application.
* Options: These are the flags or switches that can be used to modify the behavior of a command.
* Parameters: These are the values that are passed to a command or option.
* Modes: These are alternative behaviors that a command can exhibit, depending on the options or parameters passed to it.

Overall, Clee is a powerful and flexible library that makes it easy to create command-line interfaces for Ruby applications. Its simple and intuitive API makes it easy to use, even for developers who are new to CLI development.

---

EXAMPLES
--------

### Options

Options are command-line flags parsed from `--long` or `-s` (short) syntax.

```ruby
clee(:example) do
  # Boolean flag: --verbose or -v
  # Also generates --no-verbose
  option :verbose, :v

  # Required value: --output FILE or -o FILE
  option :output, :o, value: :required

  # Optional value: --format [FORMAT] or -f [FORMAT]
  option :format, :f, value: :optional

  run do
    p options
    # => {:verbose=>true, :output=>"out.txt", :format=>"json"}
  end
end
```

```sh
~> example --verbose --output out.txt --format json
~> example -v -o out.txt -f json
~> example --no-verbose -o out.txt
```

### Environment Variables

Environment variables can be passed inline or from the shell environment.

```ruby
clee(:deploy) do
  env :API_KEY, value: :required
  env :DEBUG

  run do
    p env
    # => {:API_KEY=>"secret123", :DEBUG=>true}
  end
end
```

```sh
# Inline syntax
~> deploy API_KEY=secret123 DEBUG

# Shell environment
~> API_KEY=secret123 DEBUG=1 deploy

# Mixed
~> API_KEY=secret123 deploy DEBUG
```

### Params (Options + Environment Combined)

Use `param` to accept a value as either an option OR environment variable.

```ruby
clee(:backup) do
  # Accepts --database, -d, DATABASE=, or DATABASE env var
  param :database, :d, value: :required

  run do
    puts "Backing up: #{params[:database]}"
  end
end
```

```sh
~> backup --database mydb
~> backup -d mydb
~> backup database=mydb
~> DATABASE=mydb backup
```

### Repeating Options

When an option is passed multiple times, values accumulate into an array.

```ruby
clee(:tag) do
  option :tag, :t, value: :required

  run do
    tags = Array(params[:tag])
    tags.each { |t| puts "Tagged: #{t}" }
  end
end
```

```sh
~> tag -t ruby -t cli -t awesome
Tagged: ruby
Tagged: cli
Tagged: awesome
```

### Accessing Arguments

After options are parsed, remaining arguments are available in `argv`.

```ruby
clee(:copy) do
  option :recursive, :r

  run do
    source, dest = argv
    puts "Copying #{source} -> #{dest}"
    puts "(recursively)" if params[:recursive]
  end
end
```

```sh
~> copy -r /src /dest
Copying /src -> /dest
(recursively)
```

### Modes (Subcommands)

Define subcommands with `run :mode` or nested modes with `run :mode, :submode`.

```ruby
clee(:git) do
  option :verbose, :v

  run(:status) do
    puts "Checking status..."
  end

  run(:remote, :add) do
    name, url = argv
    puts "Adding remote #{name} -> #{url}"
  end

  run(:remote, :remove) do
    name = argv.first
    puts "Removing remote #{name}"
  end

  # Default when no mode matches
  run do
    help!
  end
end
```

```sh
~> git status
Checking status...

~> git remote add origin https://github.com/...
Adding remote origin -> https://github.com/...

~> git remote remove origin
Removing remote origin

~> git --help
# prints auto-generated help
```

### Standard IO

`clee` provides duplicated IO handles for safe manipulation.

```ruby
clee(:process) do
  run do
    # stdin, stdout, stderr are available
    stdin.each_line do |line|
      stdout.puts line.upcase
    end
  end
end
```

```sh
~> echo "hello world" | process
HELLO WORLD

~> cat file.txt | process > output.txt
```

### Logging

Built-in colored logger that respects TTY detection.

```ruby
clee(:deploy) do
  run do
    log.message "Starting deployment..."
    log.warning "This may take a while"
    log.success "Step 1 complete"
    log.failure "Step 2 failed!"
    log.special "Check the dashboard"

    # Shorthand
    log "Just a message"
    log "Custom color", color: :magenta
  end
end
```

Log levels: `message`, `warning`, `success`, `failure`, `special`

### ANSI Colors

Direct access to ANSI escape codes via the `ansi` helper.

```ruby
clee(:colorful) do
  run do
    puts "#{ansi.red}Error:#{ansi.clear} something went wrong"
    puts "#{ansi.bold}#{ansi.green}Success!#{ansi.clear}"
    puts "#{ansi.underline}Important#{ansi.clear}"
  end
end
```

Available colors: `black`, `red`, `green`, `yellow`, `blue`, `magenta`, `cyan`, `white`

Background colors: `on_black`, `on_red`, `on_green`, `on_yellow`, `on_blue`, `on_magenta`, `on_cyan`, `on_white`

Styles: `bold`, `dark`, `underline`, `blink`, `reverse`, `concealed`, `clear`/`reset`

### Help Messages

Auto-generated help is available via `--help`, `-h`, or the `help` subcommand.

```ruby
clee(:mytool) do
  tldr 'a tool that does amazing things'

  option :verbose, :v
  option :output, :o, value: :required
  env :API_KEY, value: :required

  run(:process) do
    # ...
  end

  run do
    help!
  end
end
```

```sh
~> mytool --help
~> mytool -h
~> mytool help
```

Override with custom help:

```ruby
clee(:mytool) do
  help <<~HELP
    NAME
      mytool - does amazing things

    USAGE
      mytool [options] <command> [args]

    COMMANDS
      process    Process input files

    OPTIONS
      -v, --verbose    Enable verbose output
      -o, --output     Specify output file

    EXAMPLES
      mytool process input.txt -o output.txt
      mytool --verbose process *.txt
  HELP

  run do
    help!
  end
end
```

### Exit Codes

`clee` handles exit codes automatically:

* `exit 0` - success (default for completed runs)
* `exit 1` - error (default for help, missing args, invalid options)

Trigger help with a specific exit code:

```ruby
run do
  help!(exit: 0)   # success
  help!(exit: 1)   # error (default)
  help!(exit: 42)  # custom
end
```

### Error Handling

Invalid options and missing arguments are handled automatically:

```sh
~> mytool --invalid
invalid option: --invalid

~> mytool --output
missing argument: --output
```

### Utility Methods

```ruby
clee(:util) do
  run do
    # Get the program name
    puts progname  # => "util"

    # Format exception messages
    begin
      raise "oops"
    rescue => e
      puts emsg(e)  # => "oops (RuntimeError)\n<backtrace>"
    end
  end
end
```

### Complete Example

A realistic example combining multiple features:

```ruby
#!/usr/bin/env ruby
require 'clee'

clee(:dbutil) do
  tldr 'database utility commands'

  param :database, :d, value: :required
  option :verbose, :v
  option :dry_run, :n

  run(:migrate) do
    db = params[:database]
    log.message "Migrating #{db}..." if params[:verbose]

    if params[:dry_run]
      log.warning "DRY RUN - no changes made"
    else
      # perform migration
      log.success "Migration complete"
    end
  end

  run(:seed) do
    files = argv
    db = params[:database]

    if files.empty?
      log.failure "No seed files specified"
      exit 1
    end

    files.each do |file|
      log.message "Seeding #{db} from #{file}"
    end

    log.success "Seeding complete"
  end

  run(:console) do
    db = params[:database]
    exec "psql #{db}"
  end

  run do
    help!(exit: 0)
  end
end
```

```sh
~> dbutil migrate -d myapp_dev -v
~> dbutil seed -d myapp_dev users.sql products.sql
~> DATABASE=myapp_dev dbutil console
~> dbutil --help
```

---

SING IT 🎵
----------
* **DOCS are dead, long live AI!**
* **UIs are dead, long live CLIS!**

FINALLY
-------
> why `clee`? that is honestly such a stupid name...

a good friend used to pronounce 'cli' as 'clee'.  it stuck.  i like it.

> i still need you write more docs for this free code

[c'mon in!](https://github.com/ahoward/clee/pulls)
