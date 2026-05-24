

## Python version
```python
import sys
print(sys.version)
```

## Python indentation
Indentation refers to the spaces at the beginning of a code line.
Where in other programming languages the indentation in code is for readability only, the indentation in Python is very important.
Python uses indentation to indicate a block of code.
```python
if 5 > 2:  
  print("Five is greater than two!")
```
## constants, variables, functions
```python
"""
Example of contants, variables and functions
"""

name = input("What's your name? ")
weather = input("What's the weather like today? ")

temp_c_high = float(input("High temp in °C? ")) # Updated
temp_c_low = float(input("Low temp in °C? ")) # Updated

temp_f_high = (temp_c_high * 9 / 5) + 32 # New
temp_f_low = (temp_c_low * 9 / 5) + 32 # New

print('------------------------------')
print('Good morning, ' + name)
print('Today is going to be ' + weather + '.')
print('High: ' + str(temp_c_high) + ' °C (' + str(temp_f_high) + ' °F)') # Updated
print('Low: ' + str(temp_c_low) + ' °C (' + str(temp_f_low) + ' °F)') # Updated

print("I am", 35, "years old.")
```

- [Built-in Exceptions](https://docs.python.org/3.8/library/exceptions.html)
## Format String
```python
# % operator
temp = 23
"High: %i °C" % temp

# str.format()
temp = -12
"High: {} °C".format(temp)

temp = 32
unit = "F"
"High: {} °{}".format(temp, unit)

# f-string (string interpolation) => since python 3.6

temp_high = 98
temp_low = 72
unit = "F"

output = f"""High: {temp_high} °{unit}
Low: {temp_low} °{unit}"""

print(output)
```

### String Methods & Transformations

- 方法（method）是必须依附于对象调用的函数，例如 `str.format()`，不能单独调用 `format()`。
- 常见字符串转换方法（参考，常用即可，无需强记）：

```python
a_string = "Hello, world. I'm Python-ing!"
a_string.upper()         # "HELLO, WORLD. I'M PYTHON-ING!"
a_string.lower()         # "hello, world. i'm python-ing!"
a_string.capitalize()    # "Hello, world. i'm python-ing!"
a_string.title()         # "Hello, World. I'M Python-Ing!"
a_string.swapcase()      # "hELLO, WORLD. i'M pYTHON-ING!"
a_string.replace('o','0')# "Hell0, w0rld. I'm Pyth0n-ing!"
```
- 去除空白字符常用 strip() 系列方法（尤其清洗用户输入时）：

```python
string_with_spaces = " Hello, world. "
string_with_spaces.strip()        # "Hello, world."
string_with_spaces.strip("H ")   # "ello, world."
string_with_spaces.strip("H")    # " Hello, world. "
string_with_spaces.lstrip()       # "Hello, world. "
string_with_spaces.rstrip()       # " Hello, world."
```
- 获取字符串信息与基本操作：
```python
a_string = "Hello, world."
len(a_string)         # 13 字符个数
'o' in a_string       # True，包含子串

# 索引和切片
# Python索引从0开始
a_string[0]           # 'H', 第一个字符
a_string[4]           # 'o', 第五个字符
a_string[-1]          # '.', 最后一个字符
a_string[1:5]         # 'ello', 从索引1到4
a_string[:5]          # 'Hello', 从索引0到4
a_string[5:]          # ', world.'，从索引5到结尾
a_string[:]           # 'Hello, world.'，整个字符串
a_string[0:13:2]      # 'Hlo ol.'，每隔一个取字符
a_string[::-1]        # '.dlrow ,olleH'，字符串逆序
```
- 字符串分割与拼接：
```python
a_string = "I'm a string!"
words = a_string.split()         # ["I'm", 'a', 'string!']，默认按空白切分

# 指定分隔符
a_list = ["I'm", 'a', 'string!']
'*'.join(a_list)                # "I'm*a*string!"
```
> 更多字符串方法可用：`help(str)` 查看全部。
