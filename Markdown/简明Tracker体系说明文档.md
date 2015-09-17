# **简明Tracker体系说明文档**
##### 作者：欧阳业伟
##### 编写日期：2015年9月17日

------------
### **1. Tracker埋码方式**
目前1号店的Tracker埋码方式分为两种，一种是***tracker码方式***，另一种是***自动打点方式***，以下对这两种方式作出阐述。

### **2. 埋码方式：Tracker码方式**
#### **2.1 Tracker码的介绍**
对页面上每个链接打个唯一的标记code，这由各个domain自定义，目前已不推荐使用，PC端和H5端正在逐步过渡至自动打点方式。以精准化推荐的栏位作为例子，如下图：

![tracker_001][tracker_001] 

传统的tracker码分两种，***link_position***和***button_position***，如果是***button_position***，则页面不跳转，直接将商品加入购物车；否则进行跳转。如下图“加入购物车”按钮。

![tracker_002][tracker_002] 

#### **2.2 Tracker码的识别**
主要通过***link_position***和***button_position***的值来识别，比如***rpt_algorithm_config***表能匹配到的就是精准化推荐。同时***rpt_algorithm_config***表中记录了栏位和页面的对应关系。

#### **2.3 Tracker码的配置**
以精准化推荐为例，***rpt_algorithm_config***记录具体算法的tracker码规则，及栏位的对应关系。
***rpt_algorithm_config_section***记录栏位和页面的对应关系。
Backend报表中筛选框的连动效果，由此2表数据决定。
这两个表的配置主要由“精准化推荐部门开发人员”在backend中以下界面中完成。

![tracker_003][tracker_003] 
![tracker_004][tracker_004] 
![tracker_005][tracker_005] 

### **3. 埋码方式：自动打点方式**
#### **3.1 自动打点方式介绍**
详细请参见[Tracker开发手册][]

#### **3.2 自动打点方式背景**
原有的track码系统有两个缺点：

+ 每个domain定义的trackcode规则不同，这就导致无法统一解析trackercode
+ trackercode只能统计到基于内容的统计数据

#### **3.3 自动打点方式与Tracker码方式的差异**
自动打点方式与Tracker码方式的差异如下：

+ ***link_position***、***button_position***。无线端打点方式利用tpa标记栏位在页面中的顺序，利用tpi标记商品在栏位中的顺序；
而不是采用PC端中各个domain自定义的***link_position***和***button_position***方式来标记。自动打点方式中***link_position***和***button_position***变得意义不大；
+ 自动打点方式并不记录***当前页面url***和***上一个页面的url***；

### **4. 相关联系人**
**4.1 自动打点相关负责人：**

+ PC、H5: 吕海振、张飞
+ Android: 周剑华（PD）、黄俊（开发）
+ IOS: 周剑华（PD）、江傲（开发）

**4.2 埋码相关负责人：**
张飞、吕海振

[tracker_001]: http://oyyw-bucket.oss-cn-shenzhen.aliyuncs.com/document%2FMarkdown%2F%E7%AE%80%E6%98%8ETracker%E4%BD%93%E7%B3%BB%E8%AF%B4%E6%98%8E%E6%96%87%E6%A1%A3%2Ftracker_001.png
[tracker_002]: http://oyyw-bucket.oss-cn-shenzhen.aliyuncs.com/document%2FMarkdown%2F%E7%AE%80%E6%98%8ETracker%E4%BD%93%E7%B3%BB%E8%AF%B4%E6%98%8E%E6%96%87%E6%A1%A3%2Ftracker_002.png
[tracker_003]: http://oyyw-bucket.oss-cn-shenzhen.aliyuncs.com/document%2FMarkdown%2F%E7%AE%80%E6%98%8ETracker%E4%BD%93%E7%B3%BB%E8%AF%B4%E6%98%8E%E6%96%87%E6%A1%A3%2Ftracker_003.png
[tracker_004]: http://oyyw-bucket.oss-cn-shenzhen.aliyuncs.com/document%2FMarkdown%2F%E7%AE%80%E6%98%8ETracker%E4%BD%93%E7%B3%BB%E8%AF%B4%E6%98%8E%E6%96%87%E6%A1%A3%2Ftracker_004.png
[tracker_005]: http://oyyw-bucket.oss-cn-shenzhen.aliyuncs.com/document%2FMarkdown%2F%E7%AE%80%E6%98%8ETracker%E4%BD%93%E7%B3%BB%E8%AF%B4%E6%98%8E%E6%96%87%E6%A1%A3%2Ftracker_005.png
[Tracker开发手册]: http://wiki.yihaodian.cn/mediawiki/index.php/Tracker%E5%BC%80%E5%8F%91%E6%89%8B%E5%86%8C