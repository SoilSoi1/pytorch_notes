# Lecture1

## 表示学习&深度学习

![image-20260311180216373](C:\Users\29106\pytorch_notes\original_notes\yukun\img\image-20260311180216373.png)

降维feature，原因：学习器面临维度诅咒

![image-20260311180316065](C:\Users\29106\pytorch_notes\original_notes\yukun\img\image-20260311180316065.png)



表示学习：

* feature提取（无标签）和学习器（有标签）是分开训练的

DL：

* 二者统一
* 流程：input-输入原始特征-提取特征-学习器（如MLP）-输出
* 形成一个整个大模型，E2E



## 反向传播

先正向计算，再反向（从顶至底）求偏导，再链式法则，图上路径的一串偏导数相乘

![image-20260311184501598](C:\Users\29106\pytorch_notes\original_notes\yukun\img\image-20260311184501598.png)

![image-20260311184642693](C:\Users\29106\pytorch_notes\original_notes\yukun\img\image-20260311184642693.png)

