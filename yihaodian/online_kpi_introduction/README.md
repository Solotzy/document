# 实时指标统计介绍文档

作者：欧阳业伟
修改时间：2016/2/18

> 实时地统计1号店各个推荐栏位、算法带来的点击次数、加车次数和下订单次数。

---

[TOC]

---

## 1. 项目说明
目前，实时指标统计分两个部分：

* 数据生产层：storm任务
    * 取来自kafka的Tracker，统计点击数和加车数；
    * 取来自jumper的订单，统计订单数；
* GUI展示层：合并在brain-mis系统中

其中：

* storm任务的工程代码的SVN地址：
`http://svn.yihaodian.com/svn/source/yihaodian/SM/SM-2/brain-dataplatform/branches/datatask-online-kpi-tc`

* GUI展示层的工程代码的SVN地址（brain-mis，根据前端迭代来进行，具体的SVN分支地址请咨询黄凡）：
`http://svn.yihaodian.com/svn/source/yihaodian/SM/SM-2/brain-mis/branches/2016/sprint201602/brain-mis-onlinekpi`

## 2. Storm应用的统计流程
![实时指标统计流程图](https://raw.githubusercontent.com/ouyangyewei/document/master/yihaodian/online_kpi_introduction/ref/%E5%AE%9E%E6%97%B6%E6%8C%87%E6%A0%87%E7%BB%9F%E8%AE%A1%E6%B5%81%E7%A8%8B.png)

## 3. Storm应用的统计逻辑
### 3.1. 统计来自kafka tracker的点击数和加车数
#### 3.1.1. 算法解析器
根据TC埋码规则，TC参数构成：`系统来源.算法id.数据类型.数据id.顺位`
其中，`系统来源`，精准化系统全部传入`4`，因此，利用如下规则来区分来自精准化推荐的tracker：
```
系统来源 = 4 && 算法id有效
```

#### 3.1.2. 加车解析器
目前，1号店的埋码方式有两种，一种是`Tracker埋码方式`，另一种是`自动打点埋码方式`，现在正在逐步全面推行`自动打点埋码方式`，这两种埋码方式中，判断tracker是否为加车行为，逻辑分别如下：

##### 3.1.2.1. 加车解析：Tracker埋码方式
BI部门维护了一张Tracker埋码的加车模板表，hive表结构如下：
```sql
hive> desc dw.dim_position_track;
OK
position_track_skid         bigint
position_track_id           bigint
position_track_code         string
position_track_code_name    string
track_type                  int
add_cart_flag             int
start_date                string
end_date                  string
delet_flag                int
etl_batch_id              bigint
updt_time                 string
cur_flag                  int
```
取tracker中的两个字段`link_position`或者`button_position`，只要这两个字段中的任意一个能与`dw.dim_position_track`表的`position_track_code`字段进行正则匹配的话，就认为是tracker是加车行为。

##### 3.1.2.2. 加车解析：自动打点埋码方式
根据`positiontypeid`字段的取值范围来判断是否加车：

* 1 = 跳转 
* 2 = 不跳转 
* 3 = 跳转加入购物车 
* 4 = 不跳转加入购物车(需确认是否新增加一键购) 
* 5 = 一键购跳转加购物车 
* 6 = 一键购不跳转加入购物车

因此，`positiontypeid`取值为3、4、5、6，都认为tracker是加车行为。

#### 3.1.3. 统计点击数
* 取来自kafka的tracker数据，若tracker中的页面id、栏位id和算法id有效，则认为是一次精准化点击；
* 在将精准化点击数据流向下一环节的同时，在Redis中记录如下信息，用来为并行环节中的 **加车数统计、订单数统计** 提供数据支持：
```
key = 用户Id,点击产品Id
value包含：
  pageId:       页面ID
  sectionId:    栏位ID
  algorithmId:  算法ID
  platform:     平台名称
```

#### 3.1.4. 统计加车数
* 取来自kafka的tracker数据，若tracker是加车行为，且用户加车的商品在精准化栏位上最近有过点击行为则认为是一次精准化加车。
* 点击数统计环节，每一次精准化点击都将以下信息记录在Redis中：
```
key = 用户Id,点击产品Id
value包含：
  pageId:       页面ID
  sectionId:    栏位ID
  algorithmId:  算法ID
  platform:     平台名称
```
因此，判断用户加车的商品在24小时内是否有过点击行为，只需要判断Redis中用户以及产品是否存在即可

### 3.2. 统计来自jumper的订单数
* 1号店的实时订单是通过jumper来发送的，并不经过kafka来发送，具体的jumper订单接入方式，请参见系列文档`http://svndoc.yihaodian.com.cn/svndoc/documents/SM/SM-2/SM-209-搜索与精准化-推荐架构/整体架构/架构方案/实时指标统计/订单Jumper`
* 取来自jumper的订单数据，若用户下订单的商品在精准化栏位上最近有过点击行为则认为是一次精准化下单。
* 点击数统计环节，每一次精准化点击都将以下信息记录在Redis中：
```
key = 用户Id,点击产品Id
value包含：
  pageId:       页面ID
  sectionId:    栏位ID
  algorithmId:  算法ID
  platform:     平台名称
```
因此，判断用户下订单的商品在24小时内是否有过下订单行为，只需要判断Redis中用户以及产品是否存在即可

## 4. GUI展示层
可视化网址：http://brain-mis.yihaodian.com/mis/login.do

***单日KPI统计***
![单日KPI统计](https://raw.githubusercontent.com/ouyangyewei/document/master/yihaodian/online_kpi_introduction/ref/%E5%AE%9E%E6%97%B6%E6%8C%87%E6%A0%87%E7%BB%9F%E8%AE%A1GUI%E5%8D%95%E6%97%A5%E7%BB%9F%E8%AE%A1.png)
***月度KPI统计***
![月度KPI统计](https://raw.githubusercontent.com/ouyangyewei/document/master/yihaodian/online_kpi_introduction/ref/%E5%AE%9E%E6%97%B6%E6%8C%87%E6%A0%87%E7%BB%9F%E8%AE%A1GUI%E6%9C%88%E5%BA%A6KPI%E7%BB%9F%E8%AE%A1.png)
***多维TOPN分析***
![多维TOPN分析](https://raw.githubusercontent.com/ouyangyewei/document/master/yihaodian/online_kpi_introduction/ref/%E5%AE%9E%E6%97%B6%E6%8C%87%E6%A0%87%E7%BB%9F%E8%AE%A1GUI%E5%A4%9A%E7%BB%B4TOPN%E5%88%86%E6%9E%90.png)