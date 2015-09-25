# 基于Tracker码方式

Tracker码的埋码记录了如下重要字段：
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


**以session为单位，建立轨迹图的规则如下：**

+ 若***button_position***为空，意味着页面发生跳转，则向上寻找到***refer_url***等于当前***url***的跳转节点;
+ 若***button_position***非空，意味着页面未发生跳转，则向上寻找到***url***等于当前***url***，且***refer_url***等于当前***refer_url***的节点;