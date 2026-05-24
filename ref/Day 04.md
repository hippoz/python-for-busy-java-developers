[Moving to Python: Simon Roberts](https://learning.oreilly.com/live-events/moving-to-python/0636920407201/)

```python

names = ["Fred", "Jim", "Sheila"]

# functional
lengths = list(map(lambda x: len(x), names)) # map is lazy
print(lengths)
print(names) # not changed
```


## Functions

Both java and python is pass by value, and the value is usually a reference.

```python

def add(a, b):  
    return a + b  
  
x = 10  
y = 20  

print(add(x, y))  
print(add("Hello", " world"))

## Extensions can parse/verify type correctness if specified  
## Underlying language ignores these "annotations"/type hints  
# def add(a: int, b: int) -> int:

def dayOfWeek(day, month, year):  
    # Right hand side are "tuples" parens are optional...  
    # left hand side is "destructuring assignment" (other destructurings exist...)    
    m, y = (month, year) if month > 2 else (month + 12, year - 1)  
    ##  oops, function isn't working, not going to bother to fix  
    ## check out "Zeller's congruence"    
    return (day + (13 * (m + 1) // 5) + y + y // 4 - y // 100 + y // 400) % 7


print(dayOfWeek(month=2, day=11, year=2000)) # 2000-02-11
print(2, 11, 2000) # 2000-11-02

#  parameter with a default value

def showCount(c = 100):  
    print(f"count is {c}")
    
showCount()
showCount(99)


### variable length param functions

def show_allsorts(a, b, *args, **kwargs):  
    print(f"a is {a}, b is {b}")  
    for arg in args:  
        print(f"arg is {arg}")  
    for k,v in kwargs.items():  
        print(f"{k} has value {v}")  
  
  
show_allsorts(1, 3, "hello", "banana", 99.9, name="Fred", origin="Planet Earth")  
  
names = [1, 2, "Fred", "Jim", "Sheila"]  
show_allsorts(*names)


```


## Exception handling

```python

def day_name(d):  
    if d < 0 or d > 7:  
        raise IndexError(f"{d} is not a valid day number")  
    return day_names[d]  
  
print(f"The name of day zero is {day_name(0)}")  
try:  
    print("calculating day number")  
    from random import random  
    if random() > 0.5:  
        raise RuntimeError("Bad random value")  
    day_num = int(random() * 8)  
    print(f"The name of day number is {day_name(day_num)}")  
    print("wasn't that clever")  
except IndexError as ie:  
    print(f"oops that broke with {ie}")  
except:  
    print("somethign else (potentially really bad!) broke")  
finally:  
    print("all done")
```


## Class

```python

class Date:  
    @staticmethod  
    def isLeap(year):  
        return year % 4 == 0  # plus more logic  
  
    #  self is "this" in many other languages    #  explicit in argument list, not optional!!!    def __init__(self, d, m, y):  
        #  no "class field declaration" just add fields  
        # self prefix is NOT OPTIONAL (c.f. javascript)        self.day = d  
        self.month = m  
        self.year = y  
        # a = 99 # kinda static variable  
  
	# enduser showcase
    def __str__(self):  
        return f"Date with day = {self.day}"  
	
	# programmer showcase
    def __repr__(self):  
        return self.__str__()  
  
    def show_me(self):  
        # write an instance method using self!  
        print(f"hello, from an instance of date with day = {self.day}")  
  
class Holiday(Date):  # subclass  
    def __init__(self, d, m, y, n):  
        super().__init__(d, m, y)  
        self.name = n  
  
    def __str__(self):  
        return f"Holiday {super().__str__()} a holiday named {self.name}"  
  
print(f"1984 is leap? {Date.isLeap(1984)}")  
  
today = Date(3, 2, 2021)  
print(today, type(today))  
print(today.day)  
  
today.bad_value = "hahaha"  
print(today.bad_value)  
# no real "private" and can usually force items into an objec, like any map\  
  
days =[today, today]  
print(days)  
  
today.show_me()  
  
# print(today.a)  
  
today = Holiday(1, 1, 2021, "New year's day")  
print(today)


```





## comprehensions

```python
x = range(1, 10)

# this is a map operation :)  
squares = [y ** 2 for y in x]  
print(squares)  
  
# filter like operation  
squares = [y ** 2 for y in x if y % 2 == 0]  
print(squares)  
  
# flat map ish...  
triangle = [ y for x in range(1, 5) for y in range(1, x) ]  
print(triangle)

# triangle = [ y for x in range(1, 5) for y in range(1, x) ]  

# 这行代码是一个列表推导式，用于生成一个“扁平化的三角形数字序列”。
# 它的含义是：对 x 从 1 到 4，每个 x 下，再对 y 从 1 到 x-1，把所有 y 的值收集到列表中。
# 最终生成的列表是 [1, 1, 2, 1, 2, 3]。这种写法类似于嵌套循环，将所有 y 的取值按顺序依次放入列表。
# 1
# 1 2
# 1 2 3
```