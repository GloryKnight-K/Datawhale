# 06：函数与Lambda表达式

#### 1.怎么给函数编写⽂档？


```python
def():
```

#### 2.怎么给函数参数和返回值注解？


```python
def func(x: int, y: int) -> int:
'''return type int, return x add y '''
    return x+y
```

#### 3.闭包中，怎么对数字、字符串、元组等不可变元素更新。

当内部作用域想修改外部作用域的变量时，就要用到global和nonlocal关键字了。

#### 4.分别根据每一行的首元素和尾元素大小对二维列表 a = [[6, 5], [3, 7], [2, 8]] 排序。(利用lambda表达式)


```python
a=[[6, 5], [3, 7], [2, 8]]
x = sorted(a, key=lambda a: a[0], reverse=False)
print("按照首元素大小正序排列："+str(x))
x = sorted(a, key=lambda a: a[0], reverse=True)
print("按照首元素大小逆序排列："+str(x))
x = sorted(a, key=lambda a: a[1], reverse=False)
print("按照尾元素大小正序排列："+str(x))
x = sorted(a, key=lambda a: a[1], reverse=True)
print("按照尾元素大小逆序排列："+str(x))
```

    按照首元素大小正序排列：[[2, 8], [3, 7], [6, 5]]
    按照首元素大小逆序排列：[[6, 5], [3, 7], [2, 8]]
    按照尾元素大小正序排列：[[6, 5], [3, 7], [2, 8]]
    按照尾元素大小逆序排列：[[2, 8], [3, 7], [6, 5]]
    

#### 5.利用python解决汉诺塔问题？


```python
def hanot(n,a,b,c):
    global i
    if n>0:
        hanot(n-1,a,c,b)
        print('从{}移动到{}'.format(a,c))
        hanot(n-1,b,a,c)
    
hanot(4,'a','b','c')
```

    从a移动到b
    从a移动到c
    从b移动到c
    从a移动到b
    从c移动到a
    从c移动到b
    从a移动到b
    从a移动到c
    从b移动到c
    从b移动到a
    从c移动到a
    从b移动到c
    从a移动到b
    从a移动到c
    从b移动到c
    


```python
i = 0
def move(n, a, b, c):
    global i
    if (n == 1):
        i += 1
        print('移动第 {0} 次 {1} --> {2}'.format(i, a, c))
        return
    move(n - 1, a, c, b)
    move(1, a, b, c)
    move(n - 1, b, a, c)

move(4,'a','b','c')
```

    移动第 1 次 a --> b
    移动第 2 次 a --> c
    移动第 3 次 b --> c
    移动第 4 次 a --> b
    移动第 5 次 c --> a
    移动第 6 次 c --> b
    移动第 7 次 a --> b
    移动第 8 次 a --> c
    移动第 9 次 b --> c
    移动第 10 次 b --> a
    移动第 11 次 c --> a
    移动第 12 次 b --> c
    移动第 13 次 a --> b
    移动第 14 次 a --> c
    移动第 15 次 b --> c
    


```python

```
