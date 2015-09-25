# 预处理环节概要

用户轨迹工程的预处理环节重要的几步如下：

+ `extract_track_info表`：源数据取自hive中的`default.trackinfo`表，经历抽取、转换过程；
+ `extract_product_id表`：对`extract_track_info`的***product_id***字段进行缺失补偿；
+ `extract_preprocess表`：将`extract_product_id`通过***product_id***字段关联获取类目id、品牌id、系列品、促销活动列表等信息；

预处理过程中各表的表结构请参见[用户轨迹工程-预处理数据][]  


[用户轨迹工程-预处理数据]: ../data/[用户轨迹工程]预处理数据.xls