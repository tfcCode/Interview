# 	Python

## 一、内置函数（BIF）

* 总共有 68 个 BIF，用命令可以查看 

    ```idl
    dir(__builtins__)
    ```

1、**input()**

* 输入函数，输入的内容一律是字符串，后期需要强转

2、print()

* 输出函数，输出内容
* print('I love fishc.com ' * 5)  ：输出 5 次相同的内容



## 二、数据类型

### 1、字符串

* string = "hello Python"

1、**title** 

* 将首字母变为大写

2、**upper / lower** 

* 将字符串变为大写 / 小写

3、**rstrip** 

* 去掉字符串尾部的空格

4、**lstrip** 

* 去掉字符串首部的空格

5、**strip** 

* 去掉首尾空格

6、**len(String)** 

* 测量字符串长度



### 2、数字

1、3 ** 2

* 表示 3 的平方

2、**str(数字)** 

* 将数字转化为字符串



### 3、列表(数组)

* number = [ 1,2,3,4]

1、**append** 

* 在尾部追加数据
    * number.append(5)

2、**insert** 

* 插入元素
    * number.insert( 0，7 )

3、**del 语句** 

* 删除元素
    * del  number[2]

4、**pop** 

* 弹出末尾的一个数据
    * num = number.pop()
    * num = number.pop(1)

5、**remove** 

* 根据值来删除元素
    * number.remove（4）

6、排序

* 永久排序
    * **sort / sort（reverse = True）**：number.sort()

* 临时排序
    * **sorted / sorted（reverse = True）**：number.sort()

7、**reverse** 

* 反转列表
    * number.reverse()

8、**len** 

* 获得列表长度
    * length = len（number）

9、**number[ -1 ]** 

* 用 -1 作为索引总是返回最后一个元素

10、**max、min、sum** 

* 最大、最小、求和

11、列表解析

```python
squares = [values**2 for value in range(1,8)]
```

* 得到 1~7 的平方的一个数组

12、**values[0:3]** 

* 切片，获得数组的一部分数据

13、利用切片复制数据

```python
number = [1,2,3,4,5]
num = number[:]
```

* 要点：单纯的 num = number 这样操作只是将两个变量关联到同一个数据，一变都变



### 4、元组

* 类似于数组，但元组里的数据是不能改变的

```python
number = (3,5)
number[0] = 1  // 错误的
number =(4,6)  // 这样重定义是可以的
其他操作和数组类似
```



### 5、布尔类型

**Ture / False** 

* 首字母大写



### 6、字典（类似 Map）

* 类似于 Map，元素是键值对

```python
alien = {"a":1,"b":2}
```

**1、添加键值对**

```python
alien = {"a":1,"b":2}
alien["c"]=3  # 这样就可以添加一对键值对了，不需要使用函数
alien["d"]=4
```

* 添加是无序的



2、**修改字典中的值**

```python
alien = {"a":1,"b":2}
alien["1"]=1321  # 修改值
```



3、**删除键值对**

```python
alien = {"a":1,"b":2}
del alien["a"] # 删除
```



4、**遍历键值对**

```python
alien = {"a":1,"b":2}
for key,value in alien.items():  # 遍历所有键值对
    print(key)
    print(value)

for key in alien.keys():         # 遍历所有键
    print(key)
    
for value in alien.values():       # 遍历所有值
    print(value)
```





## 三、基本语法

### 1、for 循环

1、**遍历数组**

```python
number = [1,2,3,4,5]
for num in number:
    print(num)
```

* 要点：不要漏了冒号（**：**）



2、遍历区间

* 使用 **range()** 函数

```python
for num in range(1,6):
    print(num)
```

* 要点：区间为**左闭右开区间** 



### 2、if 语句

* 单条件

```python
str=“tfc”
if str = "TFC": 
    print(str)
else:
    print("Error")
    
number = [1,2,3,4,5]
n=10
if n not in number:
    print(n)
```

* 要点
    * 冒号不能掉（**：*）
    * not in / in



* 多条件

```python
if a>1 and a<10:
    print("Yes")
else:
    print("Error")

if a>1 or a<-10:
    print("Yes")
else:
    print("Error")
```



### 3、while 循环

```python
while a < 10:
    print(a)
    a=a+1
```

```python
pets = ["dog","dog","dog","cat"]
while "dog" in pets:
    pets.remove("dog")
```



### 4、函数

1、基本语法

```python
def great_user():
    print("Hello Friend")
    
great_user()
```

* 要点：以 **def** 打头



2、传递参数

```python
def great_user(name,age):
    print("Hello Friend")
    print(name)
    print(age)
    
great_user(name,age)

def make_list(*list)： # 传递任意数量的参数，参数就是一个可变列表
	print(list)
    
make_list(1,2,3)
make_list(1,2,3,4,5,6)
```

* 传递参数可以设置默认值

* 如果参数是列表（数组），则在函数中对列表的修改回影响到源列表（数组）



3、调用方式

```python
def great_user(name,age):
    print("Hello Friend")
    print(name)
    print(age)
    
great_user(name,age)
great_user(name="tfc",age=21)
```



4、带返回值的函数

```python
def result add(a,b):
    c=a+b
    return c

sum=add(11,12)
```



5、引入模块

* 将函数单独写在一个文件中，可以通过 **import** 语句来引入

```python
import Hello    # 引入 Hello 文件中的所有函数
使用：Hello.a()  # 需要带上文件名

from Hello import *  # 引入 Hello 文件中的所有函数，也可以引入特定函数
使用：a()             # 不需要带上文件名

import  Hello as  H # 给引入的 模块 / 函数 起别名，可以使用别名来引用函数
```



### 5、类

1、创建类

```python
class Dog():
    """A simple attempt to model a dog.""" 			# 文档描述
    
    def __init__(self, name, age): 					# 类似于构造器，self 参数必不可少
        """Initialize name and age attributes."""
        self.name = name   							# 定义变量（属性）同时初始化
        self.age = age								# 定义变量（属性）同时初始化
        
    def sit(self):                                  # 操作属性必须带上 self 参数
        """Simulate a dog sitting in response to a command."""
        print(self.name.title() + " is now sitting.")

    def roll_over(self):
        """Simulate rolling over in response to a command."""
        print(self.name.title() + " rolled over!")
```



2、创建对象

```python
my_dog = Dog('willie', 6)
your_dog = Dog('lucy', 3)

print("My dog's name is " + my_dog.name.title() + ".")
print("My dog is " + str(my_dog.age) + " years old.")
my_dog.sit()

print("\nMy dog's name is " + your_dog.name.title() + ".")
print("My dog is " + str(your_dog.age) + " years old.")
your_dog.sit()
```



3、继承

* 子类继承父类的所有方法和属性

```python
class PengDog(Dog):
    def __init__(self, name, age, weight):
        super().__init__(name, age)
        self.weight = weight

    def setWeight(self, weight):
        self.weight = weight
```



## 四、函数库

### 1、collections

1、**OrderedDict**

* 作用于字典（键值对），添加的元素是有序的

```python
from collections import OrderedDict

test = OrderedDict()

test["a"]=1
test["b"]=2
test["c"]=3
test["d"]=4
```



## 五、文件

### 1、从文件中读取数据

1、读取整个文件

```python
with open("C:/Users/tfc/Desktop/pi_digits.txt") as file:
    context = file.read()
    print(context)
```

* **with** 可以在文件不需要访问的时候将其关闭，不需要手动调用函数关闭



2、逐行读取

```python
with open("C:/Users/tfc/Desktop/pi_digits.txt") as file:
    for line in file:
        print(line.strip())

```



3、创建一个包含文件**每一行**的列表

```python
with open("C:/Users/tfc/Desktop/pi_digits.txt") as file:
    lines=file.readlines()

for line in lines:
    print(line.strip())
```



### 2、往文件中写数据

```python
with open("output.txt","w") as write_file:
    write_file.write("I Love You")
```

* 要点
    1. 要指定读写模式，省略表示只读模式
        * **w**：写模式   **r**：读模式   **a**：追加模式  **r+**：读写模式
    2. 指定文件路径，不指定默认在项目路径下
    3. 如果需要换行，则需要写入换行符（**\n**）



## 六、异常

1、基本语法

```python
try:
    a=5/3
except ZeroDivisionError:
    print("You don't divide by zero!")
else:
    print(a)      # try 代码块中的内容正确执行才会执行 else 代码块中的内容
```



## 七、存储数据

### 1、利用 JSON 存储数据

```python
import json

numbers = [2, 3, 5, 7, 11, 13]

filename = 'numbers.json'					# 写
with open(filename, 'w') as file_object:
    json.dump(numbers, file_object)

filename = 'numbers.json'					# 读
with open(filename) as file_object:
    numbers = json.load(file_object)

print(numbers)
```

* 引入 json 模块



## 八、测试

```python
import unittest										# 引入测试类模块
from name_function import get_formatted_name		# 引入需要测试的方法


class NamesTestCase(unittest.TestCase):				# 测试类需要继承 unittest.TestCase 类
    """Tests for 'name_function.py'."""

        def setUp(self):							# 每次运行测试方法之前都会前执行这个方法
        ....										# 可以在该方法中做一些全局性的事情
    
    def test_first_last_name(self):					# 测试方法需要以 test 打头
        formatted_name = get_formatted_name('janis', 'joplin')
        self.assertEqual(formatted_name, 'Janis Joplin')

if __name__ == "__main__":
    unittest.main()									# 运行所有测试方法        
```

























































