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

Matrix操作方法主要分为三类，分别是set,post,pre三类，其中set不用多讲，pre，post则不一样，代表着不同的矩阵乘法顺序

我们用translate来讲

setTranslate() 

postTranslate() 在方法内部进行的是 `M' = T(dx, dy) * M`，t 左乘m

preTranslate()  在方法内部进行的是 `M' = M * T(dx, dy)`，m左乘t

post,pre的乘法计算顺序是相反的，矩阵乘法不满足交换律，所以计算结果几乎是不同的。

**常用API包括下面的组合形式**

|         |  set  |   pre  |  post |
| :-----: |:-----:| :----: | :---: |
|translate|setTranslate|preTranslate|postTranslate|
|  scale  |setScale|preScale|postScale|
| rotate  |setRotate|preRotate|postRotate|

以及其他常用API



##数学知识
矩阵里面包括




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















