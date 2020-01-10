---
layout: post
title: Transforming Code Into Beautiful, Idiomatic Python
category: Programming
tags: ['Python', 'Clean Code']
excerpt: Read about simple transformations for more beautiful, idiomatic python. A companion piece for Pycon workshop.
---

I recently watched a workshop by Raymod Hettinger, a core developer for Python langauge on youtube. He talks about C-style idioms he commonly sees in code, and their pythonic equivalents which are often faster, simpler and more beautiful.

The video was fairly old and aimed at python 2.x, so I thought of going over code examples and updating for the python 3.x.

**Key takeways for the workshop were**:
1. Replace traditional index manipulation with Python's core looping idioms.
2. Learn advanced techniques with for-else clauses and two argument form of iter()
3. Improve your craftmanship and aim for clean, fast, idiomatic Python code.

{% include toc.html %}

### Looping over a collection with index

```python
presidents = ['Washington', 'Adams', 'Jefferson', 'Madision', 'Adams', 'Jackson']

# Bad - Builds an intermedidate range
for i in range(len(presidents)):
  print(i + 1, presidents[i])

# Good - Uses enumerator instead
for num, president in enumerate(presidents, start=1):
  print(num, president)
```

### Looping backwards

```python
colors = ['red', 'green', 'blue', 'yellow']

# Bad - Builds an intermediate range
for i in range(len(colors) - 1, -1, -1):
  print colors[i]

# Good - Uses enumerator instead
for color in reversed(colors):
  print color
```

### Looping over two collections

```python
names = ['raymond', 'rachel', 'matthew']
colors = ['red', 'green', 'blue', 'yellow']

# Bad - Builds both lists in memory

n = min(len(names), len(colors))
for i in range(n):
  print(names[i], colors[i])

# Good - Returns a zip enumerator
for name, color in zip(names, colors):
  print(name, color)
```

### Custom sort order

Avoid using a compare function as long as it is possible.

The compare function will be called `O(N log N)` times (for each comparison). 

The key function will be called just `O(N)` times and the key value will be used for subsequent comparsions.

```python
colors = ['red', 'green', 'blue', 'yellow']

# Bad - Uses avoidable compare function
def compare_length(c1, c2):
  if len(c1) < len(c2): return -1
  if len(c1) > len(c2): return 1
  return 0

print(sorted(colors, cmp=compare_length))

# Good - Uses key function
print(sorted(colors, key=len))
```

### Call a function until a sentinel value is reached

`partial(func, *args, **keywords)` creates a function with partial application of given arguments. Read more about them [here](https://www.learnpython.org/en/Partial_functions)

`iter(callable, sentinel)` calls the callable function until it returns the sentinel. 

```python
# Bad
blocks = []
while True:
  block = f.read(32)
  if block == '':
    break
  blocks.append(block)

# Good
from functools import partial
blocks = [block for block in iter(partial(f.read, 32), '')]
```

### Distinguishing mutliple exit points

It's little known feature of Python - often eliminates the need of flag variables.

```python
# Bad
def find(seq, target):
  found = False
  for i, value in enumerate(seq):
    if value == target:
      found = True
      break
  if not found:
    return -1
  return i

# Good
def find(seq, target):
  for i, value in enumerate(seq):
    if value == target:
      break
  else: 
    # Called if the for loop breaks before finishing
    return -1
  return i
```


### Looping over dictionaries

```python
d = {'matthew': 'blue', 'rachel':'green', 'raymond':'red'}

# Bad - Need to hash key and look up in the hash table in every loop
for k in d:
  print(k, d[k])

# Good - Avoids hashing and lookups.
for k, v in d.items():
  print(k, d[k])
```

### Constructing dictionaries

```python
names = ['raymond', 'rachel', 'matthew']
colors = ['red', 'green', 'blue']

# Bad - Avoids loading lists in memories,
# checking if names[i] is a key and subsequent insertion
d = {}
for i in len(names):
  d[names[i]] = colors[i]

# Good - Uses zip enumerator, dict constructor
d = dict(zip(names, colors)

# Also good
d = dict(enumerate(names))
# {0: 'raymond', 1: 'rachel', 2: 'matthew'}
```

### Counting dictionaries

Collections is a library designed to provide high performance container datatypes. Counter is a dict subclass for counting hashable objects.

```python
colors = ['red', 'green', 'red', 'blue', 'green', 'red']

# Bad - Last minute decision making
d = {}
for color in colors:
  if color not in d:
    d[color] = 0
  d[color] += 1

# Good
from collections import Counter
d = Counter(colors)
```

### Grouping with dictionaries

defaultdict uses an factory function to provide default value if key is missing. The code below uses empty [] as the default value.

```python
names = ['raymond', 'rachel', 'matthew', 'roger', 'betty', 'melissa', 'judith', 'charlie']

# Group by length of name

# Bad
d = {}
for name in names:
  key = len(name)
  if key not in d:
    d[key] = []
  d[key].append(name)

# Good
from collections import defaultdict
d = defaultdict(list)
for name in names:
  key = len(name)
  d[key].append(name)
```

### Keyword Arguments

```python
# Bad - Hard to understand
twitter_search('@obama', False, 20, True)

# Good
twitter_search('@obama', retweets=False, numtweets=20, popular=True)
```

### Named Tuples

Named Tuples add a subclass and access by name, making code easier to understand.
```
# Bad
doctest.testmod()
> (0, 4)

# Good
doctest.testmod()
> TestResults(failed=0, attempted=4)
```

### Simultaneous state updates

Use tuple unpacking to make changes to the entire state at once. Consider the example below, evaluating partial derivatives.

```python
# Bad
tmp_x = x + dx * t
tmp_y = y + dy * t
tmp_dx = influence(m, x, y, dx, dy, partial='x')
tmp_dy = influence(m, x, y, dx, dy, partial='y')
x = tmp_x
y = tmp_y
dx = tmp_dx
dy = tmp_dy

# Good
x, y, dx, dy = (x + dx * t,
                y + dy * t,
                influence(m, x, y, dx, dy, partial='x'),
                influence(m, x, y, dx, dy, partial='y'))
```

### Concatenating strings

```python
names = ['raymond', 'rachel', 'matthew', 'roger', 'betty', 'melissa', 'judith', 'charlie']

# Bad - O(N^2)
s = names[0]
for name in names[1:]:
  s += ', ' + name

# Good - O(N)
s = ', .'.join(names)
```

### Updating sequences

```python
# Bad - Lists pop and insert at the first index in O(n)

names = ['raymond', 'rachel', 'matthew', 'roger', 'betty', 'melissa', 'judith', 'charlie']

del names[0]
names.pop(0)
names.insert(0, 'marks')

# Good - Deque pops and inserts at the first index in O(1)

from collections import deque
names = deque(['raymond', 'rachel', 'matthew', 'roger', 'betty', 'melissa', 'judith', 'charlie'])

del names[0]
names.popleft()
names.insert(0, 'marks')
```

> There are some more useful transformations using decorators and context managers in particular. I would urge you to go through the slides linked below.

## References

1. [Transforming Code Into Beautiful, Idiomatic Python (49 minutes, Youtube)](https://www.youtube.com/watch?v=OSGv2VnC0go)
2. [Slides for the Talk](https://speakerdeck.com/pyconslides/transforming-code-into-beautiful-idiomatic-python-by-raymond-hettinger-1)
