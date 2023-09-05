## Autocoder

### Overview

`autocoder` automatically generates JSON schemas from Python functions, classes, and data structures to be called with the OpenAI function calling API.

### Installation

```bash
# Clone the repo
git clone git@github.com:ttumiel/autocoder.git
cd autocoder
pip install -e .
```

### Quick Start

#### Annotate your function with `@json_schema`

```python
from autocoder import json_schema

@json_schema
def my_function(x: float, y: float) -> bool:
    """This is a sample function.

    Args:
        x: The first float.
        y: Another float.
    """
    return x > y
```

After this, `my_function` will have an additional `.json` attribute containing its JSON schema.

```python
print(my_function.json)

{'description': 'This is a sample function.',
 'name': 'my_function',
 'parameters': {'properties': {'x': {'description': 'The first float.', 'type': 'number'},
                               'y': {'description': 'Another float.', 'type': 'number'}},
                'required': ['x', 'y'],
                'type': 'object'}}
```

#### Using Custom Classes

`json_schema` works with classes or dataclasses too. Set `descriptions=False` to not generate object descriptions from docstrings.

```python
@json_schema(descriptions=False)
@dataclass
class Data:
    a: int = 0

print(Data.json)
{'name': 'Data',
 'parameters': {'type': 'object', 'properties': {'a': {'type': 'integer'}}}}
```

#### Calling Functions with JSON Arguments

`function_call` provides additional functionality for calling functions with JSON arguments. It automatically converts JSON arguments to Python objects and returns the result as JSON. It validates the JSON, raising `FunctionCallError` if something is unexpected.

```python
import json
from autocoder import function_call, collect_functions

def plusplus(x: float, y: float) -> float:
    "Add two floats."
    return x + y

functions = collect_functions(globals(), collect_imports=False)
arguments = json.dumps({"x": 1.0, "y": 2.0})
result = function_call("plusplus", arguments, functions)
print(result) # 3.0

arguments = json.dumps({"x": "a", "y": 2.0})
result = function_call("plusplus", arguments, functions)
# FunctionCallError: Function call failed.
```
