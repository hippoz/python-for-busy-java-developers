[Moving to Python: Simon Roberts](https://learning.oreilly.com/live-events/moving-to-python/0636920407201/)
## Primitives

```python
print('"Hello World"') # single/double quote are equivalent in String
print("'Hello World'") # single/double quote are equivalent in String
print('''
aaa
''')

x = "Hello"
print(type(x))

x = 3.23

# A Python variable has no type; the object which it refers to has a type.

x = 123  
y = 100
print(x / y)   #  floating point division with /  
print(x // y)  #  integer division with //  
print(x % -y)  #  modular -77 

print(x := 100) # "Walrus operator" assignment but an expression with the assigned value: since python 3.8 (assignments in python has no return value)

# python has a "truthiness concept" -> everything has a boolean value
print(bool("Hello"))
print(bool(""))
print(bool(100))
print(bool(0))


x = "Hello"
y = "He"

y += "llo"

print(x == y) # content comparision (defined by __eq__(self, other) "dunder" methods)
print(x is y) # object identity (cannot override)

y = "Hello"
print(x == y)
print(x is y) # True string literals are pooled, strings are immutable :)

print(3 is 3) # generate a warning

```

## List

```python
names = ["Fred", "Jim", "Sheila"]
print(names)

names[0] = "Frederick" # python lists are mutable
print(names)

names.sort() # natural order, defined by __gt__ etc.

names.sort(reverse=True) # call by param name
names.sort(key = lambda s: len(s)) # sort with lambda function

```


86:00