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
```   

Matrix类转换为矩阵的表现形式为(由于不太懂矩阵的markdown语法，就用表格表示吧)

| 0             | 1             | 2         |
| ------------- |:-------------:| ---------:|
|  MSCALE_X     |   MSKEW_X     | $MTRANS_X |
|   MSKEW_Y     |   MSCALE_Y    | MTRANS_Y  |
| MPERSP_0      |   MPERSP_1    | MPERSP_2  |


##Matrix常用API






##数学知识





##部分疑问解读













