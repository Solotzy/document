# **用户轨迹系统说明文档**
**作者：欧阳业伟**  
**修订日期：2015年10月8日**  

Table Of Content
================

  * [用户轨迹系统介绍](#用户轨迹系统介绍)
    * [背景](#背景) 
    * [用途](#用途)
  * [各平台埋码方式调研](#各平台埋码方式调研)
  * [用户轨迹图的建立](#用户轨迹图的建立)

# 用户轨迹系统介绍
## 背景
+ BI提供的KPI数据中并不包含引导性的推荐数据，引导型的推荐数据定义如下：  
    - 猜对了用户想购买的类目，却没有猜中用户想购买的商品；
    - 猜对了用户想购买的品牌，却没有猜中用户想购买的商品；

+ 用户轨迹系统是用于统计用户从某指定类型的节点开始，至某指定类型的节点结束时间段内发生的所有网站行为，并根据这样的行为链条用于分析栏位推荐质量的优劣、不同人群的用户喜好等。为了简化系统复杂度，用户轨迹系统从推荐节点出发(推荐节点指推荐数据在前端页面的直接展示或者推荐数据提供给前端页面的接口)，到用户下订单页面结束。

## 用途
统计用户在1号店网站上，从推荐节点出发，至订单节点结束时间段内发生的网站行为，根据这样的行为链条，用于分析栏位推荐质量优劣、不同人群的用户喜好等。

# 各平台埋码方式调研
+ PC端、H5端目前采用***tracker码方式***进行埋码，通过记录***link_position***、***button_position***来标记推荐栏位，现在正在逐步改用***自动打点方式***；
+ 无线端采用***自动打点方式***，不采用***tracker码方式***；  
（详细请参见“[简明Tracker体系说明文档][]”）

# 用户轨迹图的建立
## 3.1. 建立轨迹图的目标
建立各用户在1号店网站的访问轨迹（包括PC端、H5端、APP端），以计算KPI

## **3.2. 数据预处理环节**
### **3.2.1. 关于Tracker**
+ Tracker的介绍请参考[Tracker手册][]  
+ Tracker记录字段的说明文档请参考[Tracker记录字段][]，值得注意的是，该Excel文档中列出的仅仅是Tracker记录的字段，而后面提到的***trackinfo表，则是BI部门在这个基础上经过清洗、添加字段而来的，因此，trackinfo字段比该Excel文档中提到的Tracker记录字段多！***

### **3.2.2. 预处理环节概要**
用户轨迹工程的预处理环节重要的几步如下：

+ `extract_track_info表`：源数据取自hive中的`default.trackinfo`表，经历抽取、转换过程；
+ `extract_product_id表`：对`extract_track_info`的***product_id***字段进行缺失补偿；
+ `extract_preprocess表`：将`extract_product_id`通过***product_id***字段关联获取类目id、品牌id、系列品、促销活动列表等信息；

预处理过程中各表的表结构请参见[用户轨迹工程-预处理数据][]  

## **3.3. 建立各用户的轨迹图**
### **3.3.1. 基于Tracker码方式**
Tracker码的埋码中记录了如下重要字段：
> 
> ***url***：记录当前页面的url  
> ***referer_url***：记录来源页面的url  
> ***link_position***  
> ***button_position***  

对于tracker埋码方式，有如下规则：
> 
> 要么***link_position***有值，要么***button_position***有值，异常情况下都没有值；  
> 若***button_position***有值，意味着页面并未跳转，因此***url***没有改变；  
> 若***link_position***有值，意味着页面发生了跳转，因此***url***发生改变；  

以session为单位，建立轨迹图的规则如下：

+ 若***button_position***为空，意味着页面发生跳转，则向上寻找到***refer_url***等于当前***url***的跳转节点;
+ 若***button_position***非空，意味着页面未发生跳转，则向上寻找到***url***等于当前***url***，且***refer_url***等于当前***refer_url***的节点;

### **3.3.2. 基于自动打点方式**
自动打点方式的埋码中记录了如下重要字段：
> 
> ***unid***：track_time映射成的数字，同一个session下可以理解为唯一
> ***refunid***：上一个页面的unid

对于自动打点的埋码方式，有如下规则：
> 正常情况下，每条tracker中unid和refunid是必发的；
> unid是将当前时间映射成字符串，因此同一个session，unid几乎不同；

以session为单位，建立轨迹图的规则如下：

+ 由于同一个***session***中***unid***几乎不同，所以通过***unid***直接判断，向上寻找到***unid***等于当前***refunid***的节点（无论是否发生跳转）

# **4. 推荐有效路径的提取**
## **4.1. 直接推荐路径**
## **4.2. 引导型推荐路径**
## **4.3. 交叉销售推荐路径**

# **5. 基于用户轨迹的KPI统计**
## **5.1. 预处理环节**
### **5.1.1. 关联购买节点**
### **5.1.2. 推荐路径补全**
## **5.2. 统计指标**
### **5.2.1. PC端、H5端**
| 字段      | 含义  |
| :-------- | :-------- |
| algorithm_id | 算法id |
| algorithm_name | 算法名称 |
| section_id | 栏位id |
| section_name | 栏位名称 |
| page_id | 页面id |
| page_name | 页面名称 |
| rcmd_type | 推荐类型id |
| rcmd_type_name | 推荐类型名称（-1 无，0 未知物品，1 商品，2 分类，3 品牌，4 搜索，5 促销，6 团购，7 抵用券，8 浏览） |
| related_type | 关联类型id |
| related_type_name | 关联类型名称（-1 无，0 未知，1 直接加车，2 同商品，3 同分类，4 同品牌，5 同促销，6 同系列，7 相关分类，8 相关品牌，9 机器搭配，10 人工搭配） |
| pv | 页面PV |
| uv | 栏位UV = 去重后点击的gu_id个数 |
| click_counts | 点击数 = 点击为link\_position的记录数 |
| click_user_counts | 点击人数 = 点击为link\_position的去重gu\_id的个数 |
| click_rate | 点击率 = （点击数 + 加车数）/ 页面PV |
| add_cart_counts | 加车数 = 点击为button\_position的记录数 |
| add_cart_user_counts | 加车人数 = 点击为button\_position的去重gu\_id的个数 |
| order_sku_counts | 下单的sku数 = 去重的父订单product\_id数 |
| order_user_counts | 下单人数 = 去重的父订单end\_user\_id数 |
| order_counts | 订单数 = 去重的父订单数 |
| order_amount | 订单金额 = 订单的order\_amount求和 |


- [x] @mentions, #refs, [links](), **formatting**, and <del>tags</del> are supported
- [x] list syntax is required (any unordered or ordered list supported)
- [x] this is a complete item
- [ ] this is an incomplete item


[简明Tracker体系说明文档]: https://github.com/ouyangyewei/document/tree/master/YiHaoDian/simple_tracker_introduction
[Tracker手册]: http://wiki.yihaodian.cn/mediawiki/index.php/Tracker%E9%A1%B9%E7%9B%AE
[Tracker记录字段]: http://wiki.yihaodian.cn/mediawiki/index.php/Tracker%E8%AE%B0%E5%BD%95%E5%AD%97%E6%AE%B5%E7%9A%84%E8%AF%B4%E6%98%8E%E6%96%87%E6%A1%A3
[用户轨迹工程-预处理数据]: https://github.com/ouyangyewei/document/blob/master/YiHaoDian/user_trace_introduction/ref/%5B%E7%94%A8%E6%88%B7%E8%BD%A8%E8%BF%B9%E5%B7%A5%E7%A8%8B%5D%E9%A2%84%E5%A4%84%E7%90%86%E6%95%B0%E6%8D%AE.xls?raw=true