# Github搜索技巧
一般搜索项目时，都是直接搜索技术栈相关的项目。高级一点的搜索，会根据 最匹配、最多 Star 来进行排序、选择相应的语言、选择仓库或者代码来进行筛选。
## 搜索语法
**搜索 GitHub 时，你可以构建匹配特定数字和单词的查询。可以使用 >、>=、< 和 <= 搜索大于、大于等于、小于以及小于等于另一个值的值。**  

|  查询  |  示例  |
| ---- | ---- |
| >*n* | cats vue:>1000 匹配含有 "vue" 字样、星标超过 1000 个的仓库。 |
| >=*n* | vue topics:>=5 匹配含有 "vue" 字样、有 5 个或更多主题的仓库 |
| <*n* | vue size:<10000 匹配小于 10 KB 的文件中含有 "vue" 字样的代码。|
| <=*n* | vue stars:<=50 匹配含有 "vue" 字样、星标不超过 50 个的仓库。 |

**可以使用 范围查询 搜索大于等于或小于等于另一个值的值**  

|  查询  |  示例  |
| ---- | ---- |
| *n*..* | vue stars:10..* 等同于 stars:>=10 并匹配含有 "vue" 字样、有 10 个或更多星号的仓库 |
| *..*n* | vue stars:*..10 等同于 stars:<=10 并匹配含有 "vue" 字样、有不超过 10 个星号的仓库。 |

**使用范围语法 *n*..*n* 搜索范围内的值，其中第一个数字 n 是最低值，而第二个是最高值**

|  查询  |  示例  |
| ---- | ---- |
| *n*..*n* | vue stars:10..50 匹配含有 "vue" 字样、有 10 到 50 个星号的仓库 |

**使用 >、>=、<、<= 和 范围查询 搜索早于或晚于另一个日期，或者位于日期范围内的日期。日期格式必须遵循 ISO8601 标准，即 YYYY-MM-DD（年-月-日）**

|  查询  |  示例  |
| ---- | ---- |
| >*YYYY*-*MM*-*DD* | vue created:>2016-04-29 匹配含有 "vue" 字样、在 2016 年 4 月 29 日之后创建的议题。 |
| >=*YYYY*-*MM*-*DD* | vue created:>=2017-04-01 匹配含有 "vue" 字样、在 2017 年 4 月 1 日或之后创建的议题。 |
| <*YYYY*-*MM*-*DD* | vue pushed:<2012-07-05 匹配在 2012 年 7 月 5 日之前推送的仓库中含有 "vue" 字样的代码。 |
| <=*YYYY*-*MM*-*DD* | vue created:<=2012-07-04 匹配含有 "vue" 字样、在 2012 年 7 月 4 日或之前创建的议题 |
| *YYYY*-*MM*-*DD*..*YYYY*-*MM*-*DD* | vue pushed:2016-04-30..2016-07-04 匹配含有 "vue" 字样、在 2016 年 4 月末到 7 月之间推送的仓库 |
| *YYYY*-*MM*-*DD*..* | vue created:2012-04-30..* 匹配在 2012 年 4 月 30 日之后创建、含有 "vue" 字样的议题 |
| *..*YYYY*-*MM*-*DD* | vue created:*..2012-04-30 匹配在 2012 年 7 月 4 日之前创建、含有 "vue" 字样的议题 |

**可以在日期后添加可选的时间信息 THH:MM:SS+00:00，以便按小时、分钟和秒进行搜索**  

|  查询  |  示例  |
| ---- | ---- |
| *YYYY*-*MM*-*DD*T*HH*:*MM*:*SS*+*00*:*00* | vue created:2017-01-01T01:00:00+07:00..2017-03-01T15:30:15+07:00 匹配在 2017 年 1 月 1 日凌晨 1 点（UTC 偏移为 07:00）与 2017 年 3 月 1 日下午 3 点（UTC 偏移为 07:00）之间创建的议题。 UTC 偏移量 07:00，2017 年 3 月 1 日下午 3 点。 UTC 偏移量 07:00 |
| >=*YYYY*-*MM*-*DD* | vue created:2016-03-21T14:11:00Z..2016-04-07T20:45:00Z 匹配在 2016 年 3 月 21 日下午 2:11 与 2016 年 4 月 7 日晚上 8:45 之间创建的议题 |
 
**使用 NOT 语法排除包含特定字词的结果。 NOT 运算符只能用于字符串关键词， 不适用于数字或日期。**  

|  查询  |  示例  |
| ---- | ---- |
| NOT | hello NOT world 匹配含有 "hello" 字样但不含有 "world" 字样的仓库 |
 
**可以为任何搜索限定符添加 - 前缀，以排除该限定符匹配的所有结果**  

|  查询  |  示例  |
| ---- | ---- |
| -*QUALIFIER* | 	vue stars:>10 -language:javascript 匹配含有 "vue" 字样、有超过 10 个星号但并非以 JavaScript 编写的仓库。<br> mentions:biaochenxuying -org:github 匹配提及 @biaochenxuying 且不在 GitHub 组织仓库中的议题 |
 
**对带有空格的查询使用引号**  
搜索含有空格的查询，您需要用引号将其括起来。 例如：
- cats NOT "hello world" 匹配含有 "vue" 字样但不含有 "hello world" 字样的仓库
- build label:"bug fix" 匹配具有标签 "bug fix"、含有 "build" 字样的议题

某些非字母数字符号（例如空格）会从引号内的代码搜索查询中删除，因此结果可能出乎意料  

**搜索查询包含需要用户名的限定符，例如 user、actor 或 assignee，您可以使用任何 GitHub 用户名指定特定人员，或使用 @me 指定当前用户。**  
|  查询  |  示例  |
| ---- | ---- |
| QUALIFIER:USERNAME | author:biaochenxuying 匹配 @biaochenxuying 创作的提交 |
|QUALIFIER:@me | is:issue assignee:@me 匹配已分配给结果查看者的议题 |

@me 只能与限定符一起使用，而不能用作搜索词，例如 @me main.workflow。  

## 高级的搜索
**按仓库名称、说明或自述文件内容搜索**  
通过 in 限定符，您可以将搜索限制为仓库名称、仓库说明、自述文件内容或这些的任意组合。如果省略此限定符，则只搜索仓库名称和说明。  

|  限定符  |  示例  |
| ---- | ---- |
| in:name | vue in:name 匹配其名称中含有 "jquery" 的仓库。 |
| in:description | vue in:name,description 匹配其名称或说明中含有 "vue" 的仓库 |
| in:readme | vue in:readme 匹配其自述文件中提及 "vue" 的仓库 |
| repo:owner/name | repo:biaochenxuying/blog 匹配特定仓库名称，比如：用户为 biaochenxuying 的 blog 项目 |
 
**在用户或组织的仓库内搜索**  
在 特定用户或组织 拥有的所有仓库中搜索，您可以使用 user 或 org 限定符  

|  限定符  |  示例  |
| ---- | ---- |
| user:*USERNAME* | user:biaochenxuying forks:>=100 匹配来自 @biaochenxuying、拥有超过 100 复刻的仓库 |
| org:*ORGNAME* | org:github 匹配来自 GitHub 的仓库 |

**按仓库大小搜索**  
size 限定符使用 大于、小于和范围限定符 查找匹配特定大小（以千字节为单位）的仓库  

|  限定符  |  示例  |
| ---- | ---- |
| size:*n* | size:1000 匹配恰好为 1 MB 的仓库。 <br> 	size:>=30000 匹配至少为 30 MB 的仓库。 <br> size:<50 匹配小于 50 KB 的仓库。 <br> size:50..120 匹配介于 50 KB 与 120 KB 之间的仓库。|

**按关注者数量搜索**  
可以使用 followers 限定符以及大于、小于和范围限定符基于仓库拥有的关注者数量过滤仓库

|  限定符  |  示例  |
| ---- | ---- |
| followers:*n* | node followers:>=10000 匹配有10,000或更多关注者提及文字 "node" 的仓库。 <br> styleguide linter followers:1..10 匹配拥有 1 到 10 个关注者并且提及 "styleguide linter" 一词的的仓库。 | 

**按复刻数量搜索**  
forks 限定符使用大于、小于和范围限定符指定仓库应具有的复刻数量。

|  限定符  |  示例  |
| ---- | ---- |
| size:*n* | 	forks:5 匹配只有 5 个复刻的仓库。 <br> forks:>=205 匹配具有至少 205 个复刻的仓库。 <br> forks:<90 匹配具有少于 90 个复刻的仓库。 <br> forks:10..20 匹配具有 10 到 20 个复刻的仓库。|

**按星号数量搜索**  
使用 大于、小于和范围限定符 基于仓库具有的 星标 数量搜索仓库  

|  限定符  |  示例  |
| ---- | ---- |
| stars:*n* | stars:500 匹配恰好具有 500 个星号的仓库。 <br> stars:10..20 匹配具有 10 到 20 个星号、小于 1000 KB 的仓库。 <br> stars:>=500 fork:true language:vue 匹配具有至少 500 个星号，包括复刻的星号（以 vue 编写）的仓库 |

**按仓库创建或上次更新时间搜索**  
基于创建时间或上次更新时间过滤仓库:  
- 对于仓库创建，您可以使用 created 限定符
- 要了解仓库上次更新的时间，您要使用 pushed 限定符。 pushed 限定符将返回仓库列表，按仓库中任意分支上最近进行的提交排序

两者均采用日期作为参数。 日期格式必须遵循 ISO8601 标准，即 YYYY-MM-DD（年-月-日）  
可以在日期后添加可选的时间信息 THH:MM:SS+00:00，以便按小时、分钟和秒进行搜索。 这是 T，随后是 HH:MM:SS（时-分-秒）和 UTC 偏移 (+00:00)。  
日期支持 大于、小于和范围限定符。  

|  限定符  |  示例  |
| ---- | ---- |
| language:*LANGUAGE* | vue created:<2020-01-01 匹配具有 "vue" 字样、在 2020 年之前创建的仓库 <br> css pushed:>2020-02-01 匹配具有 "css" 字样、在 2020 年 1 月之后收到推送的仓库。 <br> vue pushed:>=2020-03-06 fork:only 匹配具有 "vue" 字样、在 2020 年 3 月 6 日或之后收到推送并且作为复刻的仓库。 |

**按语言搜索**  

|  限定符  |  示例  |
| ---- | ---- |
| language:*LANGUAGE* | vue language:javascript 匹配具有 "vue" 字样、以 JavaScript 编写的仓库。 |

**按主题搜索**  
|  限定符  |  示例  |
| ---- | ---- |
| topic:*TOPIC* | topic:algorithm 匹配已归类为 "algorithm" 主题的仓库 |

**按主题数量搜索**  
使用 topics 限定符以及 大于、小于和范围限定符 按应用于仓库的 主题 数量搜索仓库  

|  限定符  |  示例  |
| ---- | ---- |
| topics:*n* | topics:5 匹配具有五个主题的仓库。 <br> topics:>3 匹配超过三个主题的仓库 |

**使用可视界面搜索**  
使用 search page 或 advanced search page 搜索 GitHub,可以按各种因素过滤搜索，例如仓库具有的星标数或复刻数。 在填写高级搜索字段时，您的查询将在顶部搜索栏中自动构建。

**按许可搜索**  

|  限定符  |  示例  |
| ---- | ---- |
| license:*LICENSE_KEYWORD* | license:apache-2.0 匹配根据 Apache License 2.0 授权的仓库 |

**按公共或私有仓库搜索**  
可以基于仓库是公共还是私有来过滤搜索  

|  限定符  |  示例  |
| ---- | ---- |
| is:public | is:public org:github 匹配 GitHub 拥有的公共仓库 |
| is:private | is:private pages 匹配您有访问权限且包含 "pages" 字样的私有仓库 |

**按仓库否为镜像以及托管于搜索**  
根据仓库是否为镜像以及托管于其他位置来搜索它们  

|  限定符  |  示例  |
| ---- | ---- |
| mirror:true| mirror:true GNOME 匹配是镜像且包含 "GNOME" 字样的仓库 |
| mirror:false | mirror:false GNOME 匹配并非镜像且包含 "GNOME" 字样的仓库 |

**基于仓库是否已存档搜索**  
 
|  限定符  |  示例  |
| ---- | ---- |
| archived:true| archived:true GNOME 匹配已存档且包含 "GNOME" 字样的仓库 |
| archived:false | archived:false GNOME 匹配未存档且包含 "GNOME" 字样的仓库 |

**基于具有 good first issue 或 help wanted 标签的议题数量搜索**  
使用限定符 help-wanted-issues:>n 和 good-first-issues:>n 搜索具有最少数量标签为 help-wanted 或 good-first-issue 议题的仓库  

|  限定符  |  示例  |
| ---- | ---- |
| good-first-issues:>n| good-first-issues:>2 javascript 匹配具有超过两个标签为 good-first-issue 的议题且包含 "javascript" 字样的仓库 |
| help-wanted-issues:>n | help-wanted-issues:>4 react 匹配具有超过四个标签为 help-wanted 的议题且包含 "React" 字样的仓库 |

 
参考:  
[恕我直言，你可能连 GitHub 搜索都不会用 - 如何精准搜索的神仙技巧](https://juejin.cn/post/6891056415440535565?utm_source=gold_browser_extension)
