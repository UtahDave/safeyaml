SafeYAML
========

SafeYAML is an aggressively small subset of YAML. It's everything you need for
human-readable-and-writable configuration files, and nothing more.

You don't need to integrate a new parser library: keep using your language's
best-maintained YAML parser, and drop the ``safeyaml`` linter into your CI
pipeline, pre-commit hook and/or text editor. It's a standalone script, so you
don't have any new dependencies to worry about.


What's allowed?
---------------

It's best described as JSON plus the following:

- You can use indentation for structure (braces are optional)
- Keys can be unquoted (``foo: 1``, rather than ``"foo": 1``), or quoted with ``''`` instead
- Single-line comments with ``#``
- Trailing commas allowed within ``[]`` or ``{}``

More details are in the specification ``safeyaml.md``

Here's an example::

  title: "SafeYAML Example"

  database:
    server: "192.168.1.1"

    ports:
      - 8000
      - 8001
      - 8002

    enabled: true

  servers:
    # JSON-style objects
    alpha: {
      "ip": "10.0.0.1",
      "names": [
        "alpha",
        "alpha.server",
      ],
    }
    beta: {
      "ip": "10.0.0.2",
      "names": ["beta"],
    }

As for what's disallowed: a lot. String values must always be quoted. Boolean
values must be written as ``true`` or ``false`` (``yes``, ``Y``, ``y``, ``on``
etc are not allowed). *Indented blocks must start on their own line*.

No anchors, no references. No multi-line strings. No multi-document streams. No
custom tagged values. No Octal, or Hexadecimal. *No sexagesimal numbers.*


Why?
----

The prevalence of YAML as a configuration format is testament to the
unfriendliness of JSON, but the YAML language is terrifyingly huge and full of
pitfalls for people just trying to write configuration (the number of ways to
write a Boolean value is a prime example).

There have been plenty of attempts to define other configuration languages
(TOML, Hugo, HJSON, etc). Subjectively, most of them are less friendly than YAML
(``.ini``-style formats quickly become cumbersome for structures with two or
more levels of nesting). Objectively, *all* of them face the uphill struggle of
needing a parser to be written and maintained in every popular programming
language.

A language which is a subset of YAML, however, needs no new parser - just a
linter to ensure that files conform. The ``safeyaml`` linter is an independent
executable, so whatever language and tooling you're currently using, you can
continue to use it - it's just one more step in your code quality process.


How do I use it?
----------------

The ``safeyaml`` executable will validate your YAML code, or fail with an error
if it can't. Here's an example of a passing validation::

  $ cat input.yaml
  title: "My YAML file"

  $ safeyaml input.yaml
  title: "My YAML file"

Here's an example of an error::

  $ cat input.yaml
  command: yes

  $ safeyaml input.yaml
  input.yaml:1:11:Can't use 'yes' as a value. Please either surround it in quotes
  if it's a string, or replace it with `true` if it's a boolean.

With the ``--fix`` option, ``safeyaml`` can automatically repair some problems
within YAML files.

Here's an example file that has some problems::

  $ cat input.yaml 
  name: sonic the hedgehog      # Unquoted string
  settings:{a:1,b:2}            # Missing ' ' after ':'
  list:
  - "item"                      # Unindented list item
  
  $ safeyaml --fix input.yaml
  name: "sonic the hedgehog"
  settings: {a: 1,b: 2}
  list:
    - "item"

To rewrite the fixed YAML back to the input files, pass the ``--in-place`` flag::

  $ safeyaml --fix --in-place input.yaml

You can turn individual "fix" rules off and on:

``--fix-unquoted`` will put quotes around unquoted strings inside an indented map. This does not affect map keys (which must still be in identifier format, i.e ``a1.b2.c2``).

``--fix-nospace`` ensures that at least one space follows the ``:`` after every key.

``--fix-nodent`` ensures list items inside maps are further indented than their parent key.

There are also some more forceful options which aren't included in ``--fix``:

``--force-string-keys`` turns every key into a string. This will replace any key that has a boolean or null ('true' etc) with the string version (i.e ``"true"``).  

``--force-commas`` ensures every non-empty list or map has a trailing comma.


Other Arguments
---------------

``--json`` output JSON instead of YAML.

``--quiet`` don't output YAML on success.


How do I generate it?
---------------------

Don't. Generating YAML is almost always a bad idea. Generate JSON if you need to
serialize data.
