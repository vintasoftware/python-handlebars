# python-handlebars - Handlebars.js for Python 3

Python-handlebars provides a template system for Python which is compatible with
Handlebars.js. It is a fork of the handlebars project that adds Python 3
compatibility and numerous features from Handlebars.js 2.0.

This project is strongly inspired in [Pybars3](https://github.com/wbond/pybars3).

## Installation

```bash
pip install python-handlebars
```

## Handlebars.js Compatibility

This is somewhat of a side-project for the current developers, and is
maintained for almost purely pragmatic reasons. Being able to share templates
between the server and client-side is very useful, and we like having something
more powerful than Mustache.

So, with that information, you should realize that the code is probably messy,
that there are certainly bugs and not all of Handlebars 2.0, or even 1.1 is
currently implemented.

Here is a partial list of features that are supported:

- `@root` root data accesor (Handlebars 2.0)
- `@_parent` parent scope accesor (Handlebars 2.0)
- `../` parent scope accessor
- `@index`, `@key` (Handlebars 1.0, 1.2)
- `@first` and `@last` data element in the `#each` helper (Handlebars 1.1)
- kwargs passed to partials (Handlebars 2.0)
- `@../index` syntax for accessing parent scope data items (Handlebars 2.0)
- `{{[segment literal notation]}}` for paths that contain non-word chars (Handlebars 1.1)
- `{{> "quoted partial name"}}` for partials that contain non-word chars (Handlebars 1.1)
- `lookup` helper for dynamic name access (Handlebars 2.0)
- Subexpresions (Handlebars 1.3)
- Lines containing only block statements and whitespace are removed (Handlebars 2.0)
- `handlebars.Compiler().precompile()` that is equivalent to `Handlebars.precompile()`
- `{{> (whichPartial) }}` dynamic partials (Handlebars 3.0)
- `{{{{raw}}}}{{escaped}}{{{{/raw}}}}` raw blocks (Handlebars 2.0)
- Whitespace control, `{{var~}}` (Handlebars 1.1)

Feel free to jump in with issues or pull requests.

## Usage

For details on the template language see the http://handlebarsjs.com
documentation.

Typical usage:

```python
# Get a compiler
from handlebars import Compiler
compiler = Compiler()

# Compile the template
source = u"{{>header}}{{#list people}}{{firstName}} {{lastName}}{{/list}}"
template = compiler.compile(source)

# Add any special helpers
def _list(this, options, items):
    result = [u'<ul>']
    for thing in items:
        result.append(u'<li>')
        result.extend(options['fn'](thing))
        result.append(u'</li>')
    result.append(u'</ul>')
    return result
helpers = {'list': _list}

# Add partials
header = compiler.compile(u'<h1>People</h1>')
partials = {'header': header}

# Render the template
output = template({
    'people': [
        {'firstName': "Yehuda", 'lastName': "Katz"},
        {'firstName': "Carl", 'lastName': "Lerche"},
        {'firstName': "Alan", 'lastName': "Johnson"}
    ]}, helpers=helpers, partials=partials)

print(output)
```

The generated output will be:

```html
<h1>People</h1><ul><li>Yehuda Katz</li><li>Carl Lerche</li><li>Alan Johnson</li></ul>
```

### Handlers

Translating the engine to python required slightly different calling
conventions to the JS version:

* block helpers should accept `this, options, *args, **kwargs`
* other helpers should accept `this, *args, **kwargs`
* closures in the context should accept `this, *args, **kwargs`

A template like `{{foo bar quux=1}}` will pass `bar` as a positional argument and
`quux` as a keyword argument. Keyword arguments have to be non-reserved words in
Python. For instance, `print` as a keyword argument will fail.

## Implementation Notes

Templates with literal boolean arguments like `{{foo true}}` will have the
argument mapped to Python's `True` or `False` as appropriate.

For efficiency, rather that passing strings round, handlebars passes a subclass of
list (`strlist`) which has a `__unicode__` implementation that returns
`u"".join(self)`. Template helpers can return any of `list`, `tuple`, `unicode` or
`strlist` instances. `strlist` exists to avoid quadratic overheads in string
processing during template rendering. Helpers that are in inner loops *should*
return `list` or `strlist` for the same reason.

**NOTE** The `strlist` takes the position of SafeString in the js implementation:
when returning a strlist it will not be escaped, even in a regular `{{}}`
expansion.

```python
import handlebars

source = u"{{bold name}}"

compiler = handlebars.Compiler()
template = compiler.compile(source)

def _bold(this, name):
    return handlebars.strlist(['<strong>', name, '</strong>'])
helpers = {'bold': _bold}

output = template({'name': 'Will'}, helpers=helpers)
print(output)
```

The `data` facility from the JS implementation has not been ported at this
point, if there is demand for it it would be quite easy to add. Similarly
the `stringParams` feature has not been ported - quote anything you wish to force
to a string in a helper call.

## Dependencies

* Python 3.8+
* Python-OMeta

## Development

Running tests:

```bash
python tests.py
```

To display the AST and generated Python code, execute:

```bash
python tests.py --debug
```

To run a specific test:

```bash
python tests.py TestAcceptance.test_subexpression
```

Or to debug a specific test:

```bash
python tests.py --debug TestAcceptance.test_subexpression
```
