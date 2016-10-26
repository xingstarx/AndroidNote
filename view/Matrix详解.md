#Android Matrix深入理解


##Android中的Matrix类

类的主要结构

```java
public class Matrix {
    public static final int MPERSP_0 = 6;
    public static final int MPERSP_1 = 7;
    public static final int MPERSP_2 = 8;
    public static final int MSCALE_X = 0;
    public static final int MSCALE_Y = 4;
    public static final int MSKEW_X = 1;
    public static final int MSKEW_Y = 3;
    public static final int MTRANS_X = 2;
    public static final int MTRANS_Y = 5;
    ...
}
```   

Matrix类转换为3*3矩阵的表现形式为(由于不太懂矩阵的markdown语法，就用表格表示吧)

|      0        |      1        |     2     |
| :-----------: | :-----------: | :-------: |
|  MSCALE_X     |   MSKEW_X     | MTRANS_X  |
|   MSKEW_Y     |   MSCALE_Y    | MTRANS_Y  |
| MPERSP_0      |   MPERSP_1    | MPERSP_2  |




##Matrix常用API

`Matrix`操作方法主要分为三类，分别是`set`,`post`,`pre`三类，其中`set`不用多讲，`pre`，`post`则不一样，代表着不同的矩阵乘法顺序

我们用`translate`来讲

`setTranslate() `

`postTranslate()` 在方法内部进行的是 `M' = T(dx, dy) * M`，t 左乘m

`preTranslate()`  在方法内部进行的是 `M' = M * T(dx, dy)`，m左乘t

`post`,`pre`的乘法计算顺序是相反的，矩阵乘法不满足交换律，所以计算结果几乎是不同的。

**常用API包括下面的组合形式**

|         |  set  |   pre  |  post |
| :-----: |:-----:| :----: | :---: |
|translate|setTranslate|preTranslate|postTranslate|
|  scale  |setScale|preScale|postScale|
| rotate  |setRotate|preRotate|postRotate|

以及其他常用API
setConcat()


##数学知识
1 矩阵乘法（3*3相乘）

| 0 | 1 | 2 |    *   | 0 | 1 | 2 |
|:-:|:-:|:-:|   :-:  |:-:|:-:|:-:|
| a0 | a1 | a2 |     | b0 | b1 | b2 |
| a3 | a4 | a5 |     | b3 | b4 | b5 |
| a6 | a7 | a8 |     | b6 | b7 | b8 |

========>

| 0 | 1 | 2 | 
|:-:|:-:|:-:|
|a0b0+a1b3+a2b6|a0b1+a1b4+a2b7|a0b2+a1b5+a2b8|
|a3b0+a4b3+a5b6|a3b1+a4b4+a5b7|a3b2+a4b5+a5b8|
|a6b0+a7b3+a8b6|a6b1+a7b4+a8b7|a6b2+a7b5+a8b8|

2 矩阵满足结合律，不满足交换律

3下列矩阵

|     0   |     1    |     2     |
| :-----: | :------: | :-------: |
|    1    |    0     |     0     |
|    0    |    1     |     0     |
|    0    |    0     |     1     |

为单位矩阵E，单位矩阵E跟其他矩阵T做乘法操作，得到的结果都是其他矩阵T，Android中通过new Matrix()，或者是reset方法得到的都是单位矩阵E




##部分疑问解读

###案例1
```java
matrix.preScale(0.5f, 0.5f);
matrix.preTranslate(-pivotX, -pivotY);
matrix.postTranslate(pivotX, pivotY);
```
这段代码执行的结果是什么呢？我们假设当前的屏幕是720*1280px，屏幕密度为2，在我的红米2A上就是这个数据，同时我也以这个数据做实验。假设pivotX为屏幕宽度的一半，pivotY为屏幕高度的一半，pivotX = 360, pivotY = 640

Matrix(m)初始化的值为(对于执行了new Matrix()或者是reset()的都是下面的矩阵)

|     0   |     1    |     2     |
| :-----: | :------: | :-------: |
|    1    |    0     |     0     |
|    0    |    1     |     0     |
|    0    |    0     |     1     |


**第一行的执行过程**

matrix.preScale(0.5f, 0.5f)

**左乘 m * s**

**M' = M * S(sx, sy)**


|  0  |  1  |  2  |      |  0  |  1  |  2  |
| :-: | :-: | :-: | :-:  | :-: | :-: | :-: |
|  1  |  0  |  0  |  *   | 0.5 |  0  |  0  |
|  0  |  1  |  0  |      |  0  | 0.5 |  0  |
|  0  |  0  |  1  |      |  0  |  0  |  1  |


结果m变成了下面的结果

| 0 | 1 | 2 |
|:-:|:-:|:-:|
|0.5| 0 | 0 |
| 0 |0.5| 0 |
| 0 | 0 | 1 |


**第二行执行过程**

接着m在进行preTranslate (-360, -640)运算

**M' = M * T(dx, dy)**


| 0 | 1 | 2 |      | 0 | 1 | 2 |
|:-:|:-:|:-:| :-:  |:-:|:-:|:-:|
|0.5| 0 | 0 |  *   | 1 | 0 |-360|
| 0 |0.5| 0 |      | 0 | 1 |-640|  
| 0 | 0 | 1 |      | 0 | 0 | 1 |



结果是m变成了

| 0 | 1 | 2 |
|:-:|:-:|:-:|
|0.5| 0 |-180|
| 0 |0.5|-320|
| 0 | 0 | 1 |

**第三行执行过程**

这一步乘法的顺序跟前两步是不一样的

postTranslate（360, 640）

**M' = T(dx, dy) * M**



| 0 | 1 | 2 |     | 0 | 1 | 2 |
|:-:|:-:|:-:| :-: |:-:|:-:|:-:|
| 1 | 0 |360|  *  |0.5| 0 |-180|
| 0 | 1 |640|     | 0 |0.5|-320|
| 0 | 0 | 1 |     | 0 | 0 | 1 |


结果变成了

| 0 | 1 | 2 |
|:-:|:-:|:-:|
|0.5| 0 |180|
| 0 |0.5|320|
| 0 | 0 | 1 |


*那么这三行代码执行的结果，解释起来就是沿着view的中心位置缩放一半*















