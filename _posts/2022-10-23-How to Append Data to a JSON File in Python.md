---
title: "How to Append Data to a JSON File in Python"
date: 2022-10-23 08:26:28 -0400
categories: "coding"
tags:
    - Python
    - JSON
---

This article aims to explain how to append data to a JSON file with nested elements. The JSON file is structured in the following way:

If you want to append data to a JSON file in Python, the `json` module is the way to go. Here's a step-by-step guide on how to do it:

1. Import the `json` module at the beginning of your script.

```
import json
```

1. Open the JSON file using the `open()` function.

```
with open('data.json', 'r') as f:
```

The first argument is the name of the file and the mode is set to `'r'` for reading.

1. Load the data from the JSON file into a Python object using `json.load()`.

```
data = json.load(f)
```

This function reads the file and returns a Python object representing the data in the JSON file.

1. Append new data to the Python object.

```
new_data = {'name': 'John', 'age': 30}
data.append(new_data)
```

In this example, we're appending a dictionary with two key-value pairs to the `data` object.

1. Write the updated object back to the JSON file using `json.dump()`.

```
with open('data.json', 'w') as f:
    json.dump(data, f)
```

The first argument is the object you want to write to the file, and the second argument is the file object.

That's it! You've successfully appended data to a JSON file using Python. Just remember that this method assumes that the file already contains a JSON array. If the file contains a single JSON object, you will need to modify the code accordingly.