# <font style="color:rgb(38, 218, 38);">完成事项</font>

### Z3-Solver

- 通过pip install z3-solver直接在cmd中安装

### 创建声明

-     类型    语句               说明
          x = Int('x')         #整数
          y = Real('y')        #实数
          p = Bool('p')        #布尔
          x = BitVec('x',8)    #向量
      a,b,c = Reals('a b c')   #批量声明

### 常用API

#### 1. 创建约束求解器

    solver = Solver()

#### 2.添加约束条件



    solver.add()

#### 3.判断是否有解

    if(solver.cheak()==sat)

#### 4.求解

    print(solver.model())



整型与实数类型变量之间可以互相进行转换：

```python
>>> z3.ToReal(x)
ToReal(x)
>>> z3.ToInt(y)
ToInt(y)
```

可以通过格式化字符串进行批量的定义

    a = [] 
    for i in range(0, 30):     
        var_name = f'a{i}'    
        a.append(z3.Int(var_name))

这样获得了一个 `a` 数组，数组的每个变量都对应 `a{i}` 的符号

### 常量表示

除了 Python 原有的常量数据类型外，我们也可以使用 `z3` 自带的常量类型参与运算：

```python
>>> z3.IntVal(val = 114514) # integer
114514
>>> z3.RealVal(val = 1919810) # real number
1919810
>>> z3.BitVecVal(val = 1145141919810, bv = 32) # bit vector，自动截断
2680619074
>>> z3.BitVecVal(val = 1145141919810, bv = 64) # bit vector
1145141919810
```

### 求解器

在使用 `z3` 进行约束求解之前我们首先需要获得一个求解器类实例，**本质上其实就是一组约束的集合**

```python
>>> s = z3.Solver()
```

### 添加约束

我们可以通过求解器的 `add()` 方法为指定求解器添加约束条件，约束条件可以直接通过 `z3` 变量组成的式子进行表示

```python
>>> s.add(x * 5 == 10)
>>> s.add(y * 1/2 == x)
```

对于布尔类型的式子而言，我们可以使用 `z3` 内置的 `And()`、`Or()`、`Not()`、`Implies()` 等方法进行布尔逻辑运算

```python
>>> s.add(z3.Implies(p, q))
>>> s.add(r == z3.Not(q))
>>> s.add(z3.Or(z3.Not(p), r))
```

### 约束求解

当我们向求解器中添加约束条件之后，我们可以使用 `check()` 方法检查约束是否是可满足的（satisfiable，即 `z3` 是否能够帮我们找到一组解）：

* `z3.sat`：约束可以被满足
* `z3.unsat`：约束无法被满足

```python
>>> s.check()
sat
```

若约束可以被满足，则我们可以通过 `model()` 方法获取到一组解

```python
>>> s.model()
[q = True, p = False, x = 2, y = 4, r = False]
```

对于约束条件比较少的情况，我们也可以无需创建求解器，直接通过 `solve()` 方法进行求解：

```python
>>> z3.solve(z3.Implies(p, q), r == z3.Not(q), z3.Or(z3.Not(p), r))
[q = True, p = False, r = False]
```



### 获得结果

直接使用 `model()` 获得的输出并不利于接下来的数据处理，所以可以用 `model().eval()` 函数获得实际的结果

```python
>>> s.model().eval(q)
True
>>> s.model().eval(x)
2
```

但是这样得到的结果并不是字符串或整数的类型，还需要类型转换

```python
>>> int(str(s.model().eval(x)))
2
>>> str(s.model().eval(y))
4
```



## 计算例子

- #### 先导入z3和定义变量

    from z3 import *
    
    x = Int('x')
    y = Int('y')
    
    

- #### 同时创建约束器

    s = Solver()

- #### 添加限制条件

    s.add(x > 2)
    s.add(y < 10)
    s.add(x + 2 * y == 7)

- #### 检查是否有解

    if s.check() == sat:
        print("找到解了！")
        print(s.model())
    else:
        print("无解")

[![ZddVOe.png](https://i.imgs.ovh/2026/04/12/ZddVOe.png)](https://imgloc.com/image/ZddVOe)

得到解

[![ZddTKh.png](https://i.imgs.ovh/2026/04/12/ZddTKh.png)](https://imgloc.com/image/ZddTKh)

## 注意的点

1.求解的时候一定要调用 **S.check** 函数，否则会报错

2.当问题有多解时，z3会随机给一个正确的解



## 题目

[HNCTF 2022 WEEK2]来解个方程?

#### 还是先看main函数

[![ZddrWL.png](https://i.imgs.ovh/2026/04/12/ZddrWL.png)](https://imgloc.com/image/ZddrWL)

- 1.通过算法得到v4

- 2.通过check函数检查v4

#### 那接下来就是检查check函数

[![ZdEtQA.png](https://i.imgs.ovh/2026/04/12/ZdEtQA.png)](https://imgloc.com/image/ZdEtQA)

因为是对a1进行的计算，我们可以猜测a1就是输入flag的地方，为了方便可以备注一手

### 然后是分析过程

1.先对flag函数进行判断

```python
  if ( len == 22 )
```

2.将输入数据进行赋值

```python
 if ( len == 22 )
  {
    for ( j = 0; j < len; ++j )
      *(&v2 + j) = (char)input_2[j];
```

将 *(char)input_2* 通过循环语句循环22次将每一个值本别赋给 *(&v2 + j)* 中的值

也就是下面这一段int变量中

```python
  int v2; // [rsp+20h] [rbp-A0h]
  int v3; // [rsp+24h] [rbp-9Ch]
  int v4; // [rsp+28h] [rbp-98h]
  int v5; // [rsp+2Ch] [rbp-94h]
  int v6; // [rsp+30h] [rbp-90h]
  int v7; // [rsp+34h] [rbp-8Ch]
  int v8; // [rsp+38h] [rbp-88h]
  int v9; // [rsp+3Ch] [rbp-84h]
  int v10; // [rsp+40h] [rbp-80h]
  int v11; // [rsp+44h] [rbp-7Ch]
  int v12; // [rsp+48h] [rbp-78h]
  int v13; // [rsp+4Ch] [rbp-74h]
  int v14; // [rsp+50h] [rbp-70h]
  int v15; // [rsp+54h] [rbp-6Ch]
  int v16; // [rsp+58h] [rbp-68h]
  int v17; // [rsp+5Ch] [rbp-64h]
  int v18; // [rsp+60h] [rbp-60h]
  int v19; // [rsp+64h] [rbp-5Ch]
  int v20; // [rsp+68h] [rbp-58h]
  int v21; // [rsp+6Ch] [rbp-54h]
  int v22; // [rsp+70h] [rbp-50h]
  int v23; // [rsp+74h] [rbp-4Ch]
```

3.进行函数校验

```python
    if ( 245 * v6 + 395 * v5 + 3541 * v4 + 2051 * v3 + 3201 * v2 + 1345 * v7 != 855009
      || 3270 * v6 + 3759 * v5 + 3900 * v4 + 3963 * v3 + 1546 * v2 + 3082 * v7 != 1515490
      || 526 * v6 + 2283 * v5 + 3349 * v4 + 2458 * v3 + 2012 * v2 + 268 * v7 != 854822
      || 3208 * v6 + 2021 * v5 + 3146 * v4 + 1571 * v3 + 2569 * v2 + 1395 * v7 != 1094422
      || 3136 * v6 + 3553 * v5 + 2997 * v4 + 1824 * v3 + 1575 * v2 + 1599 * v7 != 1136398
      || 2300 * v6 + 1349 * v5 + 86 * v4 + 3672 * v3 + 2908 * v2 + 1681 * v7 != 939991
      || 212 * v22 + 153 * v21 + 342 * v20 + 490 * v12 + 325 * v11 + 485 * v10 + 56 * v9 + 202 * v8 + 191 * v23 != 245940
      || 348 * v22 + 185 * v21 + 134 * v20 + 153 * v12 + 460 * v9 + 207 * v8 + 22 * v10 + 24 * v11 + 22 * v23 != 146392
      || 177 * v22 + 231 * v21 + 489 * v20 + 339 * v12 + 433 * v11 + 311 * v10 + 164 * v9 + 154 * v8 + 100 * v23 != 239438
      || 68 * v20 + 466 * v12 + 470 * v11 + 22 * v10 + 270 * v9 + 360 * v8 + 337 * v21 + 257 * v22 + 82 * v23 != 233887
      || 246 * v22 + 235 * v21 + 468 * v20 + 91 * v12 + 151 * v11 + 197 * v8 + 92 * v9 + 73 * v10 + 54 * v23 != 152663
      || 241 * v22 + 377 * v21 + 131 * v20 + 243 * v12 + 233 * v11 + 55 * v10 + 376 * v9 + 242 * v8 + 343 * v23 != 228375
      || 356 * v22 + 200 * v21 + 136 * v11 + 301 * v10 + 284 * v9 + 364 * v8 + 458 * v12 + 5 * v20 + 61 * v23 != 211183
      || 154 * v22 + 55 * v21 + 406 * v20 + 107 * v12 + 80 * v10 + 66 * v8 + 71 * v9 + 17 * v11 + 71 * v23 != 96788
      || 335 * v22 + 201 * v21 + 197 * v11 + 280 * v10 + 409 * v9 + 56 * v8 + 494 * v12 + 63 * v20 + 99 * v23 != 204625
      || 428 * v18 + 1266 * v17 + 1326 * v16 + 1967 * v15 + 3001 * v14 + 81 * v13 + 2439 * v19 != 1109296
      || 2585 * v18 + 4027 * v17 + 141 * v16 + 2539 * v15 + 3073 * v14 + 164 * v13 + 1556 * v19 != 1368547
      || 2080 * v18 + 358 * v17 + 1317 * v16 + 1341 * v15 + 3681 * v14 + 2197 * v13 + 1205 * v19 != 1320274
      || 840 * v18 + 1494 * v17 + 2353 * v16 + 235 * v15 + 3843 * v14 + 1496 * v13 + 1302 * v19 != 1206735
      || 101 * v18 + 2025 * v17 + 2842 * v16 + 1559 * v15 + 2143 * v14 + 3008 * v13 + 981 * v19 != 1306983
      || 1290 * v18 + 3822 * v17 + 1733 * v16 + 292 * v15 + 816 * v14 + 1017 * v13 + 3199 * v19 != 1160573
```

上述每一条都是对比函数，若计算结果对不上就会跳error

就是通过以上校验来判断flag是否正确

### 到此就可以还原结果

1.因为上述是判断是否为flag所以我们可以将 *不等于* 改为 *等于* 作为约束求解的约束条件

2.最后将每一个求解后的结果拼接起来就是flag

3.要注意最后是int形

#### 写出求解代码






