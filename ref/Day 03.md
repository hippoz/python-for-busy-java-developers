[Moving to Python: Simon Roberts](https://learning.oreilly.com/live-events/moving-to-python/0636920407201/)
86:00+

Q: library 都是 C 写的吗？
A: 很多 library 都是 python 写的，但是很多需要性能很好的library 的确是 C 写的。Python 原生没有 array, 只有 List; 

```python

# slicing:  
numbers = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

# 自定义的类也可以复写dunder methods 重载下面这个运算符
print(numbers[1 : 7 : 2]) # 1st inclusive, 2nd is exclusive ("fence"), 3rd "stride" 
print(numbers[::-1])
print(numbers[7:4:-2]) # => [7, 5]
print(numbers[:-2:])   # => [0, 1, 2, 3, 4, 5, 6, 7]

# range
# r = range(1, 10, 2)
r = range(0, 10)
print(r)
print(list(r))


# tuples
# **元组（tuple）** 是 Python 中的一种数据结构，用圆括号 ( ) 表示，如 `r = (1, 2, 3)`。  
# 与列表（list）相比，元组的主要区别是：元组是不可变的（immutable），即创建后不能修改其内容，而列表是可变的（mutable），可以增删元素。  
# 元组常用于保存不需要更改的数据，且在需要数据不可被修改时更安全、占用更少内存。
r = (1, 2, 3)



# pythonic -- good, expected, python style

```

## Dictionaries

```python
names = {"Fred": "Johns", "Jim": "Smith"}
print(names, type(names), sep="++++", end="")
print("X")

print(names.items())  #  key/value pairs in "tuples"
print(names.keys())
print(names.values())
print("Fred" in names)  #  tests for does this key exist...

names = {"Fred", "Jim", "Sheila"} # create a set
print(names, type(names))
```

## Complex Numbers

```python
x = 3 + 1j
print(x, type(x))
print(1j, (1j ** 2))
```

# logical operators

```python
print(yes and no)
print(yes or no)

#  bitwise operations!!! (work on bool, but not really the rigth thing!)  
print(yes & no)  
print(yes | no)
```


## conditions

```python
from random import random

x = random()

if x > 0.7:  
    print("It's big")  
    print("Really Really big")  
elif x > 0.5:  #  controls indentation runaway!  
    print("more than half")  
else:  
    print("small and insignificant")

y = "Hello" if x > 0.5 else "goodbye"

print(y)

#  python does NOT have switch/case...
#  开发者常用“变通方法”（fudge) exist, often using dict

# 使用dict来模拟 switch/case 结构
def handle_case(option):
    def case_a():
        return "你选中了A"
    def case_b():
        return "你选中了B"
    def case_default():
        return "未知选项"

    switch_dict = {
        "A": case_a,
        "B": case_b
    }
    # get()方法允许设置默认分支
    return switch_dict.get(option, case_default)()
```


## Loops

```python

x = 0

while x < 3:
	print("x is", x)
	x += 1 # no x++ in python
	if x == 3:
		break # exit the else as well.
else: # execute here IFF the while test ends the loop by evaluating false
	print("What is this?")
	
# check
x = 15  
while x < 10:  
    print(x)  
else:  
    print("x is greater than 10")


print("finished the loop")


# iterables

numbers = list(range(1, 20, 2))
for n in numbers:   # anything that is "iterable"  
    print(f"n is {n}")
```


178:00











