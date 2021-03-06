# 近邻推荐

## 基于用户的协同过滤

1. 用户之间相似度计算（找出兴趣相似的同户）
1. 计算用户和物品的关联权重（预测用户可能感兴趣的物品）

### 方案落地：

1. 用户向量矩阵构造：用户对物品的关系是稀疏的，在 Spark ML 和 NumPy 中都有相关的数据构造。
1. 用户向量相似度计算，如果数据量太大，可以用 MapReduce 分布式并行计算。
1. 用户和物品推荐计算，可以先缩小范围，比如只计算 Top N 相似用户的物品；

## 基于物品的协同过滤

1. 物品之间的相似度计算（找出受众目标用户相似的物品）
1. 根据某个物品可以得到 Top N 相似的物品（看了这个商品的用户又看了哪些商品）

### Slope One 算法改良
改良物品相似度计算的准确性

## 一些相似度计算公式


1. 欧氏距离
1. 余弦相似度
1. 皮尔逊相关度
1. 杰卡德相似度

## 更多阅读

我觉得项目基于近邻推荐算法落地中，项亮老师的推荐系统实战包含更多细节。
