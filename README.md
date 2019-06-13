# oppoGeek
OGeek算法挑战赛赛题来自OPPO手机搜索排序优化的一个子场景，并做了相应的简化，意在解决query-title语义匹配的问题。简化后，本次题目内容主要为一个实时搜索场景下query-title的ctr预估问题。<br>
记录一下，天池OGEEK算法大赛的比赛代码。 我的代码部分 the_final_code_part(1)/liuyun1128 for B

结果
---
F1:0.7355 排名：36/2888

思路
----
这次比赛主办方限定了只能使用两个模型进行融合，我和我另外一名队友分别按照自己的思路做了两个模型，最后用加权平均的方式进行融合。

思路:<br>  
1，第一阶段提取基础特征<br>
  相似度特征：<br>
    prefix-title 相似度特征：编辑距离，余弦距离，汉明距离，句向量相似度<br>
    prefix-query 加权相似度特征，编辑距离，余弦距离，汉明距离，句向量相似度<br>
    prefix-max_score_query 相似度特征，编辑距离，余弦距离，汉明距离，句向量相似度<br>
    prefix-min_score_query 相似度特征，编辑距离，余弦距离，汉明距离，句向量相似度<br>
  统计特征：<br>
    prefix,title,tag,出现的次数<br>
    prefix,title,tag,被点击的次数<br>
    query中，score的方差，最大，最小，平局，中位数<br>
  文本特征：<br>
    prefix-query 相差加权长度<br>
    prefix-title 相差长度<br>
    prefix,titile,query 文本长度<br>
    prefix是否在title中<br>
    prefix是否在query中<br>
  k-means聚类特征： (强特)
    prefix 句向量表示：分词->词向量->平均。 prefix k-means聚类, K取的35.<br>
2，第二阶段提取点击率特征<br>
  基础点击率特征：<br>
    prefix,title,tag 点击率，以及他们的二阶，三阶交叉点击率。<br>
  基于基础特征的点击率特征：<br>
    将第一部分连续特征按照固定bins分箱转换成离散特征<br>
    所有分箱离散特征的出现次数，被点击次数，点击率，二阶交叉点击率<br>
    
特征选择
----
因为在特征工程，涉及到了各种交叉点击率，这过程中可能产生很多冗余的特征，所以有必要做特征选择减少特征的数量进而优化模型训练时间。<br>
计算特征之间的皮尔森相关系数，将两两之间的皮尔森相关系数构成一个上三角矩阵，如果两个之间大于一个阈值（0.99），则删除这个特征对应的另外一个。<br>

模型
---
我的部分：LightGBM<br>
队友尝试了：libFM,RNN,DNN,LightGBM<br>

我的其他尝试:CATBOOST,XGBOOST,RT 不过最后实践证明，树集成的，只有LightGBM这种在基于单梯度采样的树模型跑大规模数据才靠谱。。。<br>

更多细节
---
点击率贝叶斯平滑
