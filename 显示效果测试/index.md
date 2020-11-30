# 显示效果测试


# 显示效果测试

```c++
#include <iostream>

int main() {
	std::cout << "Hello world!" << endl;
	return 0;
}
```

==HIGHLIGHT== (not working)

<details>
    <summary>
        标题
    </summary>
    <p>
        折叠内容
    </p>
</details>


<!--![玄武湖公园](p(185).jpg) -->



$$
{\color{red}E}={\color{blue}mc^2}
$$
![](p(185).jpg)

矩阵和其他环境必须写到一行，反斜线必须是双倍的。
$$
A =
\begin{bmatrix} a_{i1}^1 & a_{i1}^2 & \cdots & a_{i1}^l \\\\ a_{i2}^1 & a_{i2}^2 & \cdots & a_{i2}^l \\\\ \vdots & \vdots & & \vdots \\\\ a_{ir}^1 & a_{ir}^2 & \cdots & a_{ir}^l \\\\ \end{bmatrix}
$$
图像大小建议：

{{< figure src="p(185).jpg" height="350" width="350">}}

{{< figure src="p(185).jpg" height="550" width="550">}}

{{< figure src="p(185).jpg" height="750" width="750">}}

斜体加上括号有可能不会被渲染！比如

```
*(123)*
*（123）*
```

遇到过不被渲染的情况。所以尽量把括号写到外面。
