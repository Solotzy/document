# 背景

+ BI提供的KPI数据中并不包含引导性的推荐数据，引导型的推荐数据定义如下：  
    - 猜对了用户想购买的类目，却没有猜中用户想购买的商品；
    - 猜对了用户想购买的品牌，却没有猜中用户想购买的商品；

+ 用户轨迹系统是用于统计用户从某指定类型的节点开始，至某指定类型的节点结束时间段内发生的所有网站行为，并根据这样的行为链条用于分析栏位推荐质量的优劣、不同人群的用户喜好等。为了简化系统复杂度，用户轨迹系统从推荐节点出发(推荐节点指推荐数据在前端页面的直接展示或者推荐数据提供给前端页面的接口)，到用户下订单页面结束。