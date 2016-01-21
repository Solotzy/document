# **Tracker行为解析说明文档（离线处理，实时处理类似）**
**作者：黄传飞**  
**修订日期：2015年9月28日**  

> 概要：本文档是针对离线track进行解析，所使用的是离线hive表是default库下的trackinfo表，与在线表解析使用的trackreal表有一些不同（体现为字段的顺序不同，但包含的主要字段是相似的）。（2015-09-28更新）

# 1. 无线端（app和h5）行为解析(解析用户的浏览和加车行为)
## 1.1. 所使用的字段
+ ***container***字段  
    目前track中container字段有很多种值，在无线行为解析中我们只关注值“yhdapp”，其他的值与具体的部门埋点相关。
+ ***ext_field7***字段  
    该字段实际代表的是pmid，根据相关人员咨询（买家线张飞2）。目前全台（pc、app、h5）都已经启用该字段来记录pmid，对具体的商品有行为的话（不管是浏览还是加车），该字段一定会有值。
+ ***platform***字段  
    该字段记录的是平台，一般而言，我们会通过这个字段来区分访问我们一号店所使用的设备（比如android、ios等）。
+ ***url***字段  
    顾名思义，记录的是track的url
+ ***pagetypeid***字段  
    记录的页面类型，主要使用这个字段来确定是否是浏览商品详情页
+ ***url_page_id***字段  
    pc的页面记录字段，与pagetypeid字段类似都可以区分一种页面类型
+ ***link_position***字段  
+ ***button_position***字段  

## 1.2. 规则详解（注意：一下所有比较都是要忽略大小写的）
### 1.2.1.  app浏览和加车规则

+ A. 总共有两种逻辑来判断是否是app的行为（这两种逻辑是互相补充的关系）:
    - `container=’yhdapp’ and ((platform LIKE '%androidsystem%' or platform LIKE '%android') or (platform LIKE '%iossystem%' or platform LIKE '%iphone')  or (platform LIKE '%ipadsystem%' or platform LIKE '%ipad')) `则说明该行为是由app产生的
    - 仅仅`platform LIKE '%ipadsystem%'`来判断，若true则说明来自ipad app

+ B. 由A中提取出的行为再进行下面的判断
    - 若pagetypeid对应的页面为详情页（如闪购详情页：94000、94100、94200），并且pmid与productid至少有一个不空，则说明本track记录为app浏览
    - 若pmid和productid至少有一个不空，并且`positiontypeid in（3,4,5,6）`，则说明本次行为是加车行为；另外有些加车行为通过前面的部分会丢掉，因此又补充了tpi来判断，如果tpi=999也说明是app加车行为

### 1.2.2. h5浏览和加车规则
+ A. 总共有三种逻辑来判断是否是h5的行为（这三种逻辑互相补充的关系）
    - container不空且不等于’yhd-im-pc’，此外`((platform LIKE '%iossystem%' OR platform LIKE '%androidsystem%))`则说明访问平台为h5
    - `(url NOT LIKE 'http://m.%' OR url NOT LIKE 'http://m.yhd.com%') AND (platform LIKE '%iossystem%' OR platform LIKE '%ipadsystem%' OR platform LIKE '%androidsystem%' or platform LIKE '%iphone' or platform LIKE '%ipad' or platform LIKE '%android')`则说明这个页面是ipad通过非ipad连接方式访问一号店网站的行为
    - `url LIKE 'http://m.yhd.com%' OR url LIKE 'http://m.yihaodian.com%/mw/%' OR url LIKE '%.m.yhd.com%'`是h5访问方式的一般url传送方式

+ B. 由（一）中提取出的行为再进行下面的判断
    - 若***button_position***为空，且productid或pmid至少有一个非空，则说明本次行为是浏览行为
    - 若***button_position***非空，且pmid或productid至少有一个非空，且link_position和button_position通过（附录规则1）规则返回的值是1，则说明本次行为是h5加车行为

# 2. PC行为解析(解析用户的浏览和加车行为)
## 2.1. 所使用的字段
与1.1类似

## 2.2. 规则详解（注意：一下所有比较都是要忽略大小写的）
Trackinfo中的所有记录除了无线端的所有行为（即1中所陈述的行为）外，其他的都是记录pc端的行为，因此下文只需判断是否是浏览或加车行为即可

pc浏览行为
+ 通过url_page_id (与pagetypeid类似，两者可以通过规则2来转换) 来判断是否是详情页，这里使用的字段是，并且productid和pmid（位于ext_field7）至少有一个非空。则说明本次行为是pc浏览行为
+ 若pmid和productid至少有一个不空，并且`positiontypeid in（3,4,5,6）`，此外，link_position和button_position通过（附录规则1）返回的值是1，则说明本次行为是pc加车行为

# 附录
## 规则1：
这里的***link_position***和***button_position***判断是通过bi部门（李磊2）提供的表`dw.dim_position_track`来判断本条track记录是否是加车行为，判断逻辑如下：

+ 将***link_position***与`dim_position_track`表的***position_track_code***字段关联，并向`dim_position_track`表的track_type字段传入1，然后返回add_cart_flag字段值
+ 将***button_position***与`dim_position_track`表的position_track_code字段关联，并向`dim_position_track`表的track_type字段传入1，然后返回add_cart_flag字段值
+ 若1和2的规则同时满足，但是返回的add_cart_flag字段值不同，则以1的返回结果优先（注意cur_flg=1才是有效的记录）
+ 若add_cart_flag字段返回值为1，则说明本条记录是加车记录

## 规则2：
trackinfo表中通`url_page_id`和`pagetypeid`，这两个都可以区分页面id都可以区分出页面类型，两者之间的对应关系可以通过bi的`dw.dim_page_type`，`dw.dim_page_categ`两张表关联得出（关联两张表使用`page_categ_id`字段），表`dim_page_type`中的`page_type_id`对应`trackinfo`表的`url_page_id`，`orig_page_type_id`对应`trackinfo`表的`pagetypeid`。


**离线行为解析代码的svn地址和工程目录：**  

SVN地址：http://svn.yihaodian.com/svn/source/yihaodian/SM/SM-2/bigdata-datamining/branches/userTrace

工程目录：/home/pms/bigDataEngine/recsys/script/useraction/trackAction

