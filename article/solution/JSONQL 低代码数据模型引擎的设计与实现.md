# JSONQL 低代码数据模型引擎的设计与实现
专业开发中进行数据查询常见有三种方案：
- 裸写 SQL 方案，包括 MyBatis 这种映射方案，这个方案灵活性强，可以使用所有 SQL 功能，缺点是无法适配多种数据库，对于复杂的关联查询需要自己 JOIN 表，由于上手门槛低，是目前国内开发者主要使用的方案  
- Query Builder 方案，它将 SQL 做了一层封装，开发者可以通过代码来构造 SQL，比裸写 SQL 多了类型检查，开发时不容易写错，同时这个方案很适合既想支持多种数据库，又想接近 SQL 开发体验的开发者  
- ORM 方案，比如 JPA/Hibernate/ActiveRecord，主要特点是几乎屏蔽了 SQL，这也导致了上手成本高，初次接触需要学习大量概念，优点是上手后开发效率很高，关联查询写起来也简单，还能适配多种数据库，但这个方案的灵活性比 SQL 弱，有些特殊的数据库方言无法实现，而且屏蔽了 SQL 导致开发者不会怎么关注最终生成 SQL，很容易生成低效查询，比如查询所有列及 N+1 次查询问题  

有没有一种方案，可以同时有这些方案的优点，既能做到和 SQL 一样的灵活性，又能跨数据库，又能支持简易关联查询，又能解决查询无用字段及 N+1 问题呢？  

## 爱速搭中的数据模型
爱速搭中数据模型的使用流程，主要分 3 步：  
- 连接数据源
- 导入表结构，建立关联关系
- 设计前端界面

第 1 步是连接数据源，这个比较简单就是填入数据库地址和账号密码  
第 2 步是导入表结构  
第 3 步是设计前端界面，爱速搭提供了一个脚手架来快速生成 CRUD 界面,勾选需要的功能和展示字段后，一个带增删改查的列表页面就生成出来了  
这个生成的界面是 amis CRUD 组件的配置，因此还可以进一步调整，比如修改列的展示类型，新增列等。这样一个基于数据库的展示和编辑界面就做好了，全程无需写代码。  

实现这个功能除了前端 UI 组件、后端 SQL 查询外，还有个重要部分是前后端如何交互？  
爱速搭之前的做法是后端提供 RESTful 协议的接口，比如默认包含这些 HTTP api  
``` 
// 创建数据
 POST /datamodel

 // 查询数据列表
 GET /datamodel

 // 查询加上过滤条件，下面的示例相当于 name = 'aisuda' AND age < 10
 GET /datamodel?name=aisuda&age[lt]=10

 // 查询某个数据详情，其中 1 是主键值
 GET /datamodel/1

 // 修改某个数据，修改内容在提交 payload 里
 PUT /datamodel/1

 // 删除某个数据
 DELETE /datamodel/1
```
这些接口可以用来实现简单应用，许多低代码平台也就是做到这个程度，但它有很多缺点：  
- 无法控制返回字段，比如文章列表可能只想查询标题而无需返回内容，需要定制特殊参数
- 无法进行复杂组合查询，前面的过滤条件都是且（AND），如果要做负责的嵌套条件，比如 name = aisuda OR age < 10) AND deleted IS NOT NULL 就无能为力了，需要定制特殊参数
- 无法支持聚合查询，要实现图表类的查询需要再定义新接口

数据模型决定了低代码平台能制作应用的上限，如果只有简单的 REST 接口，就只能做简单应用，比起专业开发在灵活性上有很大差距，因此为了让爱速搭能构建更为复杂的应用，开发了全新的前后端交互语言 JSONQL。  

## JSONQL 介绍
JSONQL 其实就是 JSON 格式的自定义 DSL，先来看一个 JSONQL 最简单实例  
``` 
{
   "statement": "select",
   "select": ["name"]
   "from": "user"
 }
```
它的作用相当于下面这段 SQL  
``` 
SELECT name FROM user
```
JSONQL 有什么特点呢：
- 灵活性强
- 易用性好
- 支持多种数据库
- 性能优
- 安全性有保障

从整个系统层面看，JSONQL 是爱速搭前后端交互的中心节点，前端可以通过 REST 接口、SQL、GraphQL 等方式连接后端，这些接口都会先转成 JSONQL，然后再通过 JSONQL 引擎连接各种后端数据。  

**灵活性强**  
JSONQL 的第一个特点是灵活性强，它的语法主要参考 SQL，相当于一种以 JSON 格式表达的 SQL，JSONQL 能表达绝大部分 SQL 语句，所以还实现了另一种模式，可以支持直接写 SQL，类似下面的写法  
``` 
{
   "sql": "select name from user"
 }
```
它的原理是先进行 SQL 解析，再遍历解析后的 AST 树转成等价的 JSONQL，除了前面的简单例子，它还支持非常复杂的嵌套表达式及子查询，比如下面这段 SQL（来自 SQL for Data Analytics）也可以转换为等价 JSONQ  
``` 
SELECT sales_year,
   womens_sales - mens_sales as womens_minus_mens,
   mens_sales - womens_sales as mens_minus_womens
 FROM (
     SELECT date_part('year', sales_month) as sales_year,
       sum(
         case
           when kind_of_business = 'Women''s clothing stores' then sales
         end
       ) as womens_sales,
       sum(
         case
           when kind_of_business = 'Men''s clothing stores' then sales
         end
       ) as mens_sales
     FROM retail_sales
     WHERE kind_of_business in (
         'Men''s clothing stores',
         'Women''s clothing stores'
       )
       and sales_month <= '2019-12-01'
     GROUP BY 1
   ) a
 ORDER BY 1;
```
除了查询还有更新和删除，比如类似下面的 SQL 写法  
``` 
UPDATE table1 SET column1 = column1 + 1
```
JSONQL 中可以表示为  
``` 
{
   "statement": "update",
   "table": "table1",
   "setExp": {
     "column1": {
       "binary": "+",
       "left": "column1",
       "right": 1
     }
   }
 }
```
为什么不直接用 SQL 呢？因为 SQL 是字符串，程序化生成时容易拼接出错，比如要拼接一个 WHERE 条件，要注意字符串转义（字符串里本身就有引号的情况），生成复杂 WHERE 条件也挺麻烦，比如 WHERE (a=1 or (b=2 and c = 3)) and d=4 ，要注意没有条件时这个 WHERE 字符串也不能输出，而用 JSON 可以方便前端编辑器生成。  

JSONQL可以理解为一种用 JSON 结构化表示的 SQL，绝大部分 SQL 都能表示为 JSONQL 形式

**易用性好**  
JSONQL 基本等于 SQL，所以灵活性强，但除了这点，JSONQL 还提供了许多功能来提升易用性。  
_支持关联查询及 N+1 问题_  
第一个是支持关联查询，这使得 JSONQL 既能做到 SQL 的灵活性又能做到 ORM 的易用性。  
在 user 表里有个一对多关系字段 blogs，也就是一个用户写了多篇博客，查询用户及这个用户写的文章列表可以使用以下 JSONQL  
``` 
{
   "statement": "SELECT",
   "select": ["name", "blogs"]
   "from": "user"
 }
```
查询输出给前端的结果类似如下示例，注意这里的 blogs 是深层结构，嵌入到每个用户对象里  
``` 
[{
     "id": 1,
     "name": "user1",
     "blogs": [{
         "id": 1,
         "title": "title1",
         "user_id": 1,
         "content": "content1"
     }, {
         "id": 2,
         "title": "title2",
         "user_id": 1,
         "content": "content2"
     }]
 }, {
     "id": 2,
     "name": "user2",
     "blogs": [{
         "id": 3,
         "title": "title3",
         "user_id": 2,
         "content": "content3"
     }]
 }]
```
如何实现呢？有两种做法，一种是 ORM 里常见的 N+1 查询，也就是首先查询 user，然后再根据 user id 去查询 blog，类似如下伪代码  
``` 
const users = find(User);
 for (const user of users) {
   const blogs = find(Blog).byUserId(user.id);
 }
```
这种做法下，查到了 10 个用户，就要发起很 10 次 blog 查询，因此也叫 N+1 次查询，如果用户数量很多就会影响数据库性能。  
如果要优化 N+1 查询，可以使用就是基于 JOIN 来一次查出，比如下面 SQL 语句  
``` 
SELECT
   `user`.`id`,
   `user`.`name`,
   `blogs`.`id`,
   `blogs`.`title`,
   `blogs`.`user_id`,
   `blogs`.`content`
 FROM
   `user`
 LEFT JOIN `blog` `blogs` ON `user`.`id` = `blogs`.`user_id`
```
如果要获取所有数据，这个语句是能满足需求的，但大多数时候还有分页需求，比如只想一次性查询 10 个 user 的数据，这时就没法简单地加上 limit，比如  
``` 
SELECT
   `user`.`id`,
   `user`.`name`,
   `blogs`.`id`,
   `blogs`.`title`,
   `blogs`.`user_id`,
   `blogs`.`content`
 FROM
   `user`
 LEFT JOIN `blog` `blogs` ON `user`.`id` = `blogs`.`user_id`
 LIMIT 10 # 这是错的
```
为什么这是错的？因为一个用户对应多个博客，合并到一张表里用户就会重复,因此假设第一个用户 1 有 10 篇文章，加上 LIMIT 后就只能查出有这一个用户的文章列表了，而我们希望的是查 10 个用户的文章列表。  
有个高级办法是使用 LATERAL 语法，上面的语句可以改成  
``` 
 SELECT "u"."id", "u"."name"
 FROM (
     SELECT *
     FROM "user"
     LIMIT 10
   ) u
   LEFT JOIN LATERAL (
     SELECT "blog"."id", "blog"."title", "blog"."content"
     FROM "blog"
     WHERE "u"."id" = "blog"."user_id"
   ) blogs on true
```
实际执行类似下面的伪代码：  
``` 
for user in users:
    for blog in blogs:
      blog.user_id = user.id
```
比普通 SQL 查询多了一层嵌套循环，它会在查询完第一层结果后再查询，使得可以支持 LIMIT。  
但它的缺点是只有少数几种数据库支持，Postgres 9.3、Oracle 12c 版本以上支持，SQL Server 需要改查 APPLY 语法，支持最差的是 MySQL，要等到 8.0.14 版本才支持。  
由于最流行的 MySQL 要高版本才支持，导致在国内实用性不高，因此 JSONQL 默认没有使用这个语法，而是使用了两次查询来解决，方法是：  
第一次查询用户 id，使用如下语句  
``` 
SELECT
   DISTINCT `id`
 FROM
   (SELECT
       `user`.`id`,
       `user`.`name`,
       `blogs`.`id`,
       `blogs`.`title`,
       `blogs`.`user_id`,
       `blogs`.`content`
     FROM
       `user`
     LEFT JOIN `blog` `blogs` ON `user`.`id` = `blogs`.`user_id` ) AS `distinctAlias`
 LIMIT 10
```
接着再基于拿到的用户 id 去查询 blogs，生成类似如下的 SQL 语句  
``` 
 SELECT
   `user`.`id`,
   `user`.`name`,
   `blogs`.`id`,
   `blogs`.`title`,
   `blogs`.`user_id`,
   `blogs`.`content`
 FROM
   `user`
 LEFT JOIN `blog` `blogs` ON `user`.`id` = `blogs`.`user_id`
 WHERE
   `user`.`id` IN (1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
```
这样就只需要两个查询语句，也被称为批量查询方案。  
然而这个方案并不完美，这里的第二次查询还是没法 LIMIT，如果 blogs 表的数据量很大，比如有的用户有几千篇文章，而只想显示最新一篇，也会导致大量 IO 及带宽浪费，这时使用 N+1 查询反而是个更好的方案。  
因此在 JSONQL 中我们增加了一种名为二次查询的语法，使用下面的写法  
``` 
{
   "statement": "SELECT",
   "select": [
     {
       "column": "name"
     }
   ],
   "from": "user",
   "limit": 10,
   "secondQuery": [
     {
       "select": [
         {
           "column": "title"
         }
       ],
       "limit": 2,
       "where": {
         "tag": "tag1"
       }
       "from": "blogs"
     }
   ]
 }
```
在实现的时候，会先查询 user 表，然后再基于返回的 id 一个去查询 blog 表，并且加上 LIMIT 条件。  
除了要发起 N 次查询，这个方案还有个缺陷是子表的过滤条件无法放到主表中，也就是无法查询「写过文章标题为 amis 的作者及文章列表」，因为主查询的过滤条件只能是用户字段里的字段。  
由于 LATERAL 的巨大优势，JSONQL 后续版本打算进行优化，对于支持 LATERAL 的数据库优先使用它，这样就能完美支持两种场景，不需要区分两种写法了。  
除了前面说到的多对一关系场景，JSONQL 还支持多对多、一对多、一对一关系、反向一对一关系，并且支持无限层级嵌套，比如下面的写法：  
``` 
{
   "statement": "SELECT",
   "select": ["books.author.address"]
   "from": "bookshop"
 }
```
这其实是四层关系，bookshop 和 books 是多对多，books 和 author 是多对一，author 和 adress 是一对一，生成 SQL 有四个 JOIN，输出结果也是个深层对象，类似  
``` 
[{
     "id": 1,
     "name": "书店 1",
     "books": [{
         "title": "书籍 1",
         "id": 1,
         "author": {
             "name": "作者 1",
             "id": 1,
             "address": {
                 "id": 1,
                 "location": "地址 1"
             }
         }
     }]
 }]
```
同时除了 select，在 where 里也能使用关系字段，比如根据书籍标题查找作者列表，查找同名的书都有谁写过  
``` 
{
   "statement": "select",
   "where": {
     "books.title": "书籍 1"
   }
   "from": "author"
 }
```
如果用 SQL 需要自己 join，类似如下写法  
``` 
SELECT
   "author"."id",
   "author"."name"
 FROM
   "author"
 LEFT JOIN "book" "books" ON "author"."id" = "books"."author_id"
 WHERE
   "books"."title" = '书籍 1'
```
关联除了查询，还有增删改操作，比如新增一个用户的时候，还可以同时提供他的文章信息，类似下面的请求  
``` 
{
   "statement": "INSERT",
   "table": "user",
   "values": {
     "name": "user1",
     "books": [
       {
         "title": "book1"
       }
     ]
   }
 }
```
在实现的时候会首先写入 user 表，拿到主键的自增 id，然后再写入 book 表，自动将自增 id 填入 author_id 字段中，是不是比写 SQL 要简单很多？  

_避免接口数量不断增长_  
由于 JSONQL 的灵活性，它还可以避免前后端接口数量不断增长。  
在专业开发中前后端通常使用 REST 协议进行交互，这时经常会遇到一个问题，那就是有些字段其实前端不需要，比如查询文章列表的接口，有时候我们只需要返回标题列表，有时候又想返回内容摘要，为了节省数据传输量，通常要用两个接口来实现，或者在接口上加个字段选择的功能，虽然项目发展，前后端接口会越来越多，甚至还出现了专门管理这些接口的 API 平台。  
而在 JSONQL 方案下，数据查询的后端开发被彻底节省了，要查询什么信息前端自己构建 JSONQL 就行，不需要前后端接口约定，也不需要 API 接口文档及管理，前后端只需要一个接口。  
这点和 GraphQL 要解决的问题是类似的，后面我们会单独介绍 GraphQL 和 JSONQL 的区别。  

_简化分页查询_  
列表页除了查询数据，还需要返回页面总数，需要使用两个查询来实现  
``` 
SELECT * FROM table1 WHERE A = 1 ORDER BY name LIMIT 10 OFFSET 10
SELECT COUNT(*) FROM table1 WHERE A = 1
```
JSONQL 简化了这个功能，只需要加上一个 count 配置就能实现类似功能  
``` 
 {
   "statement": "SELECT",
   "select": ["name"],
   "from": "user",
   "count": true,
   "limit": 10
 }
```
系统会自动去掉 LIMIT/OFFSET 及 ORDER BY 进行 COUNT 查询。  
另外除了 LIMIT/OFFSET 及 ORDER BY，如果有 LEFT JOIN 语句且对应的表是对一关系，这个 LEFT JOIN 对 COUNT 结果不会有影响，也可以直接去掉。  

_避免常见问题及更友好的错误提示_   
不同数据库对 SQL 的支持不同，还会有奇怪问题，比如下面这个看起来毫无问题的 SQL  
``` 
SELECT id, date FROM table1;
```
在 Oracle 下会报两个错误：
- ORA-00936: missing expression
- ORA-00933: SQL command not properly ended

这两个问题是：
- date 是保留关键字必须加引号，不然就当成日期字面常量语法了 date '2022-1-1'，这也是为什么报缺少表达式
- 结尾不能加分号，这是 Oracle 特有问题，不靠搜索引擎恐怕完全想不到

在 JSONQL 中会自动解决，因为标识符会自动加引号，且在执行时自动去掉最后的分号。  
另一方面是针对一些错误的写法，比如 GROUP BY 的字段不在 SELECT 中  
``` 
SELECT id FROM table1 GROUP BY type
```
JSONQL 内部解析的时候就会自动加上，变成  
``` 
SELECT id, type FROM table1 GROUP BY type
```
对于无法自动修复的场景会提供友好的报错信息，这样做的好处是报错信息是统一的，比起每个数据库千奇百怪的报错信息，JSONQL 给出的报错更易懂。  

**同时支持 CRUD 及 BI 场景**  
在绝大部分低代码平台中，CRUD 和 BI 往往是分开处理的，因为它们的前后端使用不同接口，毕竟 BI 查询的 SQL 往往特别复杂，很难使用 REST 形式表示。  
但 JSONQL 可以完整支持大部分 SQL，因此在爱速搭里，CRUD 和 BI 的支持全都是用 JSONQL，避免了两次后端开发，简化了开发成本。  


**支持多种数据库**  
_关系数据库数据库方言_  
虽然 SQL 有标准，但每个数据库的实现都不太一样，而且反而是国内最流行的 MySQL 其实最不符合标准，JSONQL 要支持大量数据库就得抹平这些方言差异，主要是这两个问题：  
- LIMIT/OFFSET 的写法不一致
- 函数功能不一致

首先是 LIMIT/OFFSET 写法，这是最为突出的兼容性问题，一方面它非常重要，需要依赖它来做分页查询，另一方面它也是方言问题重灾区，许多数据库都发明了自己的写法，我知道的就有 15 种写法，有些数据库的写法特别繁琐，比如 SQL SERVER 2008。  
这个问题的根本原因是标准化太晚了，SQL 在 1974 年就有了，而 LIMIT/OFFSET 写法要等到 2011 年通过的 SQL:2008 规范才正式确定，这中间隔了 34 年，以至于标准的写法没几个数据库支持，反倒是 MySQL 简洁的 LIMIT OFFSET 写法最为流行。  
另一个差异点是函数支持，这里最为突出的是日期函数的支持，日期函数是 BI 查询的核心功能，比如想要按年聚合数据，查询每年的销售额，需要用到如下写法  
``` 
SELECT SUM(price), YEAR(date) from sales GROUP BY YEAR(date)
```
然而并不是所有数据库都支持 YEAR 函数，对应的替代函数可能有以下这些：  
- toYear(date)
- DATE_FORMAT_STR(date, 'YYYY')
- TIME_EXTRACT(date, 'year')
- to_number(to_char(date, 'yyyy'))
- CAST(strftime('%Y', date) AS INTEGER)
- DATEPART(year, date)

如果要输出更一致的效果，比如有些数据库输出月份前面不会补零，如果想保持输出结果一致，比如类似下面的提取年月的函数  
``` 
DATE_FORMAT(date, '%Y-%m')
```
需要转成  
``` 
CAST(DATEPART(year, date) as varchar) + '-' + CAST(RIGHT('0' + CAST(DATEPART(month, date) as varchar), 2) as varchar)
```
要将所有不兼容函数都替换成方言版本是个非常费时的工作，需要查阅数据库官方文档里的 SQL 语法，而且同一个数据库的不同版本还不一样，为了保证结果正确，还得搭建这些数据库的测试环境来实际测试。  
除了前面提到这两个，还有很多细节方言问题，比如字段类型、如何获取写入后的自增 id 等。  
目前 JSONQL 底层引擎基于 JDBC 实现，可以连接市面上绝大部分数据库，目前已经支持 40+ 数据库，覆盖几乎所有国内开发者可能会使用的数据库，还包括大数据库相关的 Hive、Impala、Doris 等。  

_对非关系型数据库的支持_  
JSONQL 引擎还实现了对非关系型数据库的支持，使得 JSONQL 还可以对 MongoDB、Elasticsearch、Redis、HBase 及 OData 协议的数据库进行增删改查操作  

**MongoDB**  
MongoDB 的实现原理是转成了官方 Client 里的操作，比如 select 对应的是 projection，where 对应的是 filter。  
简单增删改查比较容易实现，最复杂的是聚合查询问题，比如 group by，它的原理是转成 Aggregation Pipeline 语句。  
比如下面这个 SQL 语句  
``` 
SELECT SUM(transaction_count) AS sumResult
 FROM transactions
 GROUP BY account_id
 ORDER BY sumResult DESC
 LIMIT 2
```
会转换先转成包含 pipeline 及 aggregate 的 json，然后再去执行 pipeline  
``` 
{
   "pipeline": [{
     "$group": {
       "sumResult": {
         "$sum": "$transaction_count"
       },
       "_id": {
         "account_id": "$account_id"
       }
     }
   }, {
     "$project": {
       "sumResult": "$sumResult",
       "_id": 0
     }
   }, {
     "$sort": {
       "sumResult": -1
     }
   }, {
     "$limit": 2
   }],
   "aggregate": "transactions"
 }
```
ongoDB 官方有个 Connector for BI 工具可以实现类似功能，支持使用 SQL 查询 MongoDB，我们没有使用它，主要原因是：  
- Connector for BI 是属于企业版本 MongoDB 的一部分，开源版本只能试用
- Connector for BI 不支持增删改操作，而我们需要完整支持 CUD 功能

**Elasticsearch**  
接下来是 Elasticsearch，它的原理是转成 REST 接口，比如查询转成 _search 接口，文档增删改转成 _doc 接口。  
比如类似下面的 SQL 查询  
``` 
SELECT a FROM table1 WHERE b = 1 and c > 2 ORDER BY d DESC LIMIT 3
```
会转成以下 JSON，然后调用 /table1/_search 进行查询  
``` 
{
   "query": {
     "bool": {
       "must": [{
         "term": {
           "b": 1
         }
       }, {
         "range": {
           "c": {
             "gt": 2
           }
         }
       }]
     }
   },
   "sort": [{
     "d": {
       "order": "desc"
     }
   }],
   "fields": [{
     "field": "a"
   }],
   "size": 3
 }
```
MongoDB 类似 Elasticsearch 官方也提供了 SQL 协议对接，没有使用它，主要原因是：  
- SQL 接口不支持查询 _id 字段，这对于实现增删改功能是必须的，不然就得要求数据里必须有个字段是唯一标识，导致了使用场景受限，比如在日志类场景下一般不会有唯一标识
- SQL 接口是从 6.3 版本开始支持的，低版本无法使用，而 REST 接口可以支持到最早的 Elasticsearch 0.9 版本

**Redis**  
Redis 主要自由 kv 查询，所以支持它其实就是约定返回结果类型，比如下面的语句  
``` 
SELECT * FROM key1
```
**OData**  
OData 是一种基于 REST 规范的数据查询协议，背后主要是微软在支持，国内还没见过有人用，它最大好处是提供了一种通用的数据对接方式，如果你自研了数据存储，就可以通过这个接口将数据查询功能暴露出来给其它服务使用。  
因为本质是 REST 所以支持其实比较简单，就是转成了对应的 REST 接口，类似 Elasticsearch 的实现。  
比如下面这个简单的查询  
``` 
SELECT FirstName, LastName
 FROM People
 WHERE FirstName = 'Vincent'
 ORDER BY Concurrency DESC
 LIMIT 2 OFFSET 0
```
会转成以下 Query（这里为了方便查看进行了 URL 解码，实际发送是编码后的）  
``` 
/People?$skip=0&$top=2
 &$filter=FirstName eq 'Vincent'
 &$orderby=Concurrency desc
 &$select=FirstName, LastName
```

## 性能优
**大 offset 优化**  
如果 offset 比较大会影响数据库性能，比如下面的 SQL  
``` 
SELECT film_id, description
 FROM film
 ORDER BY title
 LIMIT 10 OFFSET 10000
```
数据库在执行的时候要首先读取一万条数据，然后再往后找 10 个，浪费了大量 IO 操作，因此 offset 越大性能越差。  
针对 offset 较大的查询，JSONQL 引擎会进行 SQL 改写，改成如下写法  
``` 
SELECT film_id, description
 FROM film
   INNER JOIN (
     SELECT film_id
     FROM film
     ORDER BY title
     LIMIT 10 OFFSET 10000
   ) AS OFFSETOPT USING(film_id)
```
这个语句能实现同样功能，但它的性能更好，原因是在 OFFSET 查询的时候只选择了 film_id 字段，这是个主键字段，因此只需要遍历主键索引而无需读取数据，在获取了主键 id 后再去读取这些主键对应的数据，从而大幅减少了数据读取量。  

**尽可能使用参数变量**  
JSONQL 在执行的时候，会将变量都变成参数，比如类似如下语句  
``` 
SELECT * FROM table1 WHERE a = 'b' LIMIT 10 OFFSET 0
```
实际执行的时候会自动变成  
``` 
SELECT * FROM table1 WHERE a = ? LIMIT ? OFFSET ?
```
通过参数绑定的方式来执行，这样做的好处有 2 方面的好处：
- 避免 SQL 注入问题
- 优化性能

为什么可以优化性能？因为上面的 3 个参数不管怎么变化都是同一条 SQL 语句，数据库在实现时就能直接复用这个解析及查询计划，无需再计算一次，在 Oracle、SQL Server 等数据库下会有更好性能。  
不过参数变量的数量在有些数据库下有限制，比如 SQL Server 只支持 2100 个参数，超过之后就必须改成内嵌。  

**高级缓存机制**  
缓存是解决任何性能问题的终极办法，因此 JSONQL 在语法上增加了缓存配置，可以控制过期时间。   
除了过期时间，JSONQL 引擎还支持失效过期机制，在数据有变更的时候自动将相关缓存清空。  
如何实现呢？最简单的是将相关表的所有缓存都删了，这是最简单的做法，也是 MySQL 之前 Query Cache 的做法，但这种做法缓存命中率太低，所以在 MySQL 8 之后直接去掉了这个功能。  
能不能更进一步优化呢？JSONQL 引擎由于有数据模型信息，知道主键是什么，因此对数据有更深入理解，可以实现更精确删除缓存，具体分两种情况：  
- 增删改查场景
- BI 聚合查询场景

其中 BI 聚合查询场景比较复杂，想要精确知道某个数据修改是否会影响到缓存非常困难，因此无法实现。  
增删改查场景就相对容易了，因为模型中有主键，所以查询到的数据后，引擎可以根据主键确认这个缓存都会设计那些行数据库，如果对应的数据有增删改操作，就能将对应行的缓存删除，避免了表级别粒度的命中率太低问题。  
但这要求所有数据变更都用 JSONQL，如果有应用不使用 JSONQL 就无法感知了，只能通过自动分析 binlog 来感知数据库变更。  
如果你觉得缓存有风险，JSONQL 还可以单独针对 count 加缓存。  
对于一个常见的列表页，我们除了查询前十条记录，还需要查询总数，也就是类似如下的语句  
``` 
SELECT count(*) FROM table1
```
在 MySQL InnoDB 下，这个 count 查询的性能会随着表中数据量的变多越来越慢，几百万行数据后可能就变成秒级别了，因此这个语句对系统负担很重。  
而有时候我们并不需要绝对准确的数据，毕竟一千万零一和一千万零二区别不大，因此可以单独针对 count 开启缓存，这样既能保证性能，又不担心数据缓存过期问题。  

**乐观锁**  
出于性能和安全考虑，JSONQL 下不提供排它锁支持，无法使用 SELECT FOR UPDATE 语句。  
但提供了另一种被称为乐观锁的机制，可以设置模型的某个字段为版本号，更新的时候会将这个版本号加入条件，比如  
``` 
UPDATE user SET name = 'jsonql', version = version + 1 WHERE id = 1 and version = 1
```

**减少行锁冲突**  
有时候我们可能需要频繁更新某个数据，比如  
``` 
UPDATE pageview set views = views + 1 where page_id = 1
```
在高并发场景下，比如某个页面的 QPS 很高会导致严重锁冲突，针对这个问题有个小技巧是增加一个随机字段，将一行锁变成多行锁，比如改成如下语句  
``` 
 INSERT INTO pageview (page_id, random, views)
 VALUES (1, FLOOR(RAND() * 100), 1) ON DUPLICATE KEY
 UPDATE views = views + 1;
```
这样做就将锁冲突概率减小了 100 倍，但要实现这个功能比较麻烦，ON DUPLICATE KEY 语句 和 RAND 都有数据库方言问题，因此 JSONQL 引擎内置了这种特殊情况的支持，只要配置后就自动支持。  

**支持分库分表**  
当数据量增长到亿级别时，单个数据库将很难支持，这时除了切换为分布式数据库以外，还有许多公司使用分库分表来支持，但分库分表会影响查询，导致业务代码复杂。  
JSONQL 底层引擎通过引入 ShardingSphere 项目支持了自动分表分库，在实际使用时可以当成一个表来使用。  

**性能统计和控制**  
JSONQL 引擎内置了性能统计功能，会统计语句执行性能，提供性能报告方便分析。  
同时为了防止慢查询影响性能，还可以临时对某语句进行控制，比如
- 强制开启缓存，减低对数据库影响
- 禁用这个语句
- 金额类型特殊优化

由于浮点数是不精确的，针对金额字段通常需要使用定点数，但定点数没有硬件加速支持，在数据库中都是软件实现的，效率不高。  
JSONQL 的增加了一种数据类型，就叫金额，它实际使用整数存储，读出来后自动转成小数，整数计算的性能远比定点数高，所以能提升性能。  
这个方案的缺点是在除法运算时会丢失精度，因为有可能算出的结果小于分，比如定点数即便小数位是 2，在计算除法时输出结果还会变成 3 位，比如，比如 0.01/2 变成 0.005，但如果是使用分作为单位，就变成 0 了，所以如果数据量不太推荐用定点数。  

**随机值返回的优化**  
有时想随机从数据库里取个值  
``` 
SELECT * FROM table1 ORDER BY rand() LIMIT 1
```
这个语句在执行的时候需要将所有数据都进行一次排序，再取第一条，数据量较大的时候还会使用临时表或文件存储，对系统性能影响较大。  
针对这种情况设计了个特殊语法，实现的时候先 count 总数，然后算个随机数 xxx，再通过下面语句来获取  
``` 
SELECT * FROM table1 LIMIT xxx, 1
```
但这个优化必须没有 WHERE 条件，限制比较大  

## 安全性有保障
前面介绍了 JSONQL 在灵活性、易用性及性能方面的功能，可以看到它拥有和 ORM 一样的易用性，同时又能做到和手写 SQL 一样的灵活性，因此 JSONQL 可以覆盖大部数据增删改查的后端开发工作，彻底降低这部分的研发成本。  
但还有一个重要的问题没解决，那就是如何保证安全？JSONQL 相当于将 SQL 能力赋予了前端，但也意味着 SQL 注入及越权访问的风险，如果这个问题不解决，这个方案就彻底没法用，前面提到的所有功能都是浮云。  
JSONQL 如何解决呢？主要有以下几点：  
- 后台 JSONQL 模式
- 数据模型控制
- 支持数据验证
- 禁止原始 SQL
- 限制部分语句
- 细粒度权限控制
- 避免误更新或误删
- 数据脱敏
- 防止 DDOS

**后台 JSONQL 模式**  
对于安全性要求较高的场景，JSONQL 可以不直接暴露给前端，而是只在后台使用，这就彻底杜绝了用户任意构造 JSONQL 进行查询的能力，因此安全性和专业开发是一样的。  
这种模式叫后台 JSONQL 模式，JSONQL 不是前端动态生成，而是预先编写好的，比如在页面开发的时生成 JSONQL，实际查询的时前端只传递变量，比如在开发时存储了下面这段 JSONQL  
``` 
{
   "statement": "select",
   "select": ["name"],
   "from": "table1",
   "where": {
     "name": "${name}"
   }
 }
```
这段 JSONQL 等价于  
``` 
SELECT name FROM table1 WHERE name = ${name}
```
前端实际发起请求的时候是类似下面的 api 地址  
``` 
GET /api?name=amis
```
这和专业开发是一样的，前端只能传递部分变量，而后端 JSONQL 引擎会根据这个变量做替换，如果前端不传递这个变量，比如请求  
``` 
GET /api
```
引擎会自动去掉过滤条件，将查询变成  
``` 
SELECT name FROM table1
```
如果想支持默认值该怎么办呢？可以这样写  
``` 
{
   "statement": "select",
   "select": ["name"],
   "from": "table1",
   "where": {
     "name": "${name || 'amis'}"
   }
 }
```
其实 ${} 里是个表达式引擎，这里是实现了和 amis 一样的表达式引擎，原理是基于 antlr4 实现的语法树遍历解释器，语法主要参考 JavaScript，可以支持四则运算、函数嵌套调用等功能，为什么不直接用 JavaScript 语法？因为嵌入 JavaScript 引擎性能较差且有安全性问题。  
同时为了提升这个模式下的灵活性，在 JSONQL 语法中还增加了一种动态结构语法，比如下面的语句  
``` 
{
   "statement": "select",
   "select": [
     {
       "if": [
         {
           "test": "a == 1",
           "body": {
             "column": "xx",
             "func": "DATE_FORMAT",
             "arg": "YYYY-MM"
           }
         },
         {
           "test": "a == 2",
           "body": {
             "column": "yy"
           }
         }
       ]
     }
   ],
   "from": "xxx"
 }
```
当前端传递 GET /api?a=1 的时候，上面 JSONQL 会转成如下格式  
``` 
{
   "statement": "SELECT",
   "select": [
     {
       "column": "xx",
       "func": "DATE_FORMAT",
       "arg": "YYYY-MM"
     }
   ],
   "from": "xxx"
 }
```
等价的 SQL 是  
``` 
SELECT DATE_FORMAT(xx, 'YYYY-MM') from xxx
```
果前端传递 GET /api?a=2，上面 JSONQL 就会变成  
``` 
{
   "statement": "select",
   "select": [{
     "column": "yy"
   }],
   "from": "xxx"
 }
```
可以看到我们不仅能改变某个值，还能改变语句本身，从而实现复杂场景的支持。  
除了 if 语句外，还有 choose 语句实现循环功能，熟悉 MyBatis 的读者可能会发现和 MyBatis 非常像，这里确实是参考了 MyBatis 的命名，也使得 JSONQL 拥有不亚于 MyBatis 的灵活性。  
后台 JSONQL 模式在安全性上和专业开发一致，除此之外 JSONQL 引擎还提供了大量机制来保证安全性，使得可以让前端直接发起动态 JSONQL 调用，下面会分别介绍这些机制。  

**数据模型控制**  
在前面的所有例子中，虽然看起来是查询某个表，但在 JSONQL 实际使用时，其实查询的是预先定义的数据模型，比如最开始举的例子  
以模型 user 为例，它的模型定义类似如下 JSON 表示（省略了很多，比如关联关系定义）  
``` 
{
   "key": "user",
   "name": "用户",
   "fields": [{
     "type": "int",
     "key": "id",
     "name": "id",
     "isPrimaryKey": true,
     "isGenerated": true
   }, {
     "type": "text",
     "key": "name",
     "name": "名称"
   }, {
     "type": "int",
     "key": "age",
     "name": "年龄"
   }]
 }
```
其中最顶层的 key 名相当于表名，而 fields 里的字段相当于列。  
在进行 JSONQL 查询的时候，比如下面这个语句  
``` 
SELECT * FROM user
```
执行的时候会展开为  
``` 
SELECT id, name, age FROM user
```
即便原始数据库的 user 表有很多其它字段，只要这个字段没在模型定义里就没法查，如果遇到类似下面的 SQL 会直接报错，因为在模型定义里没有这个字段  
``` 
SELECT password FROM user
```
同样也没法查询没定义的模型，比如  
``` 
SELECT * FROM information_schema.tables
```
因为我们的模型定义里没有 information_schema.tables 这个模型，所以上面的查询会报模型不存在。  
有了数据模型这一层限制，就可以有效避免查询任意表及字段，限制了查询范围。  

**禁止原始 SQL**  
JSONQL 可以表达大部分 SQL 语法，因此 JSONQL 无需支持原始 SQL 形式，所有 SQL 都需要先转成 JSONQL 结构才能使用。  
举个例子，SQL 语句在很多地方都支持表达式，比如在字段上使用，类似下面的语句中的第二行表达式  
``` 
SELECT customer_id
     ,max(case when fruit = 'apple' and quantity > 5 then 1 else 0 end) as loves_apples
 FROM order
 GROUP BY 1
```
要用结构化的代码构造第二行表达式写起来比较复杂，很多 ORM 框架及查询 DSL 会在这里支持写原始 SQL，框架不对这个字符串进行二次处理，在生成 SQL 的时候直接原样输出。  
原始 SQL 的问题是对框架来说是个黑盒，完全不知道里面用到了什么函数，查询了哪些字段，而 JSONQL 中的实现是先转成结构化数据，前面的 SQL 可以用如下 JSONQL 表示  
``` 
{
   "select": [{
     "column": "customer_id"
   }, {
     "exp": "max(case when fruit = 'apple' and quantity > 5 then 1 else 0 end)",
     "as": "loves_apples"
   }],
   "statement": "SELECT",
   "from": "order",
   "groupBy": [{
     "val": 1
   }]
 }
```
看上去想使用了原始 SQL，但实际不是，引擎底层实现的时会先转换为如下 JSONQL  
``` 
{
   "select": [{
     "column": "customer_id"
   }, {
     "args": [{
       "cases": [{
         "then": 1,
         "when": {
           "children": [{
             "op": "=",
             "left": {
               "column": "fruit"
             },
             "right": "apple"
           }, {
             "op": ">",
             "left": {
               "column": "quantity"
             },
             "right": 5
           }],
           "operator": "and"
         }
       }],
       "else": 0
     }],
     "func": "max",
     "as": "loves_apples"
   }],
   "statement": "SELECT",
   "from": "order",
   "groupBy": [{
     "val": 1
   }]
 }
```
可以看到所有 SQL 都进行了结构化处理，这使得 JSONQL 引擎可以准确分析出用到了那些语句、函数、查询了哪些字段，配合后面提到的权限控制机制，就能做到即便 JSONQL 完全暴露给前端使用，恶意用户无论怎么构造 JSONQL 也无法越过权限控制。  
由于 SQL 的灵活性，实际实现时这里有很多细节问题需要处理，比如类似下面的写法  
``` 
SELECT u.id FROM table1 u
```
第一个 u.id 在 SQL 解析的时候时没法区分 u 到底是表名还是别名，因此需要在解析后二次处理，通过上下文确认实际表名是什么，类似情况还有在子查询、JOIN、WITH 语句里也能写别名。  
你可能会说别名很少用，直接不支持就好了，但有些场景下必须用别名，比如 FROM 语句使用子查询时必须有别名，还有涉及到多个相同表进行 JOIN 的时候必须有别名来区分，比如有个 staff 员工表，其中有个字段是这个员工的上级 leader_id，它指向另一行 staff 表里数据，如果要同时查询员工及其上级，就需要对这个 staff 表进行 JOIN，这时必须用别名来区分，比如  
``` 
SELECT staff.name, leader.name
 FROM staff
 JOIN staff leader ON staff.leader_id = leader.id
```
禁止 SQL 的主要缺点是导致 JSONQL 对数据库扩展语法支持不好，比如 Postgres 的列支持数组类型，可以使用下面语法查询二维数组里的值  
``` 
SELECT schedule[1:2][1:1] FROM sal_emp;
```
目前 JSONQL 还没有对应的结构表示导致无法使用，需要后续扩展，但扩展起来倒是比较容易。  

**不支持部分语句**  
虽然 JSONQL 支持大部分 SQL 语句转换，但有些功能不方便权限检查以及有严重安全问题，所以就直接不支持了，比如  
- DDL 相关的语句，这个没必要暴露给前端
- 数据库管理语句，比如授权的 GRANT 语句等
- 创建存储过程

另外就是针对表名和字段名都默认做了限制，比如不允许出现分号、引号等特殊字符，也避免了引擎内部实现遗漏时导致 SQL 注入问题。  

**细粒度权限控制**  
JSONQL 中没有原始 SQL，使得底层引擎可以准确分析出查询了哪些字段，这使得 JSONQL 能够实现细粒度权限控制，目前支持以下 4 种粒度的权限控制  
- 表级别，如果没有某个表权限，那也没法 join，这种情况直接报错返回无权限。
- 行级别权限，自动在对应的表上加行过滤条件 。
- 列级别权限，在查询字段的时候自动删掉无权限的字段，如果这个字段是 join 所需的就只有 join on 上保留。
- 表达式权限，比如对一个字段取 MAX，同时设置 GROUP BY 的值，在这个权限设置下就只能查询这个字段的总数，无法查看明细，比如能查询公司总收入，但无法知道每个部门的收入分别是多少。
权限相关的配置采用类似如下格式

``` 
{
   //表名
   "table1": {
     // 表级别的写入权限
     "canInsert": true,
     // 列权限配置
     "field": {
       // 代表所有字段
       "*": ["select", "update"],
       // 针对某个字段
       "fild1": ["select", "update"]
     },
     // 表达式权限，这里就只有读，因此是个数组
     "fieldSelectExp": ["SUM(fieldKey)"],
     // 行级别权限配置，针对查、改、删分别配置
     "filters": {
       "select": {},
       "update": {},
       "delete": {}
     }
   }
 }
```
列级别的配置使得你无法选择自己没权限的列，如果使用下面的 JSONQL 进行查询时  
``` 
{
   "statement": "SELECT",
   "select": ["*"]
   "from": "user"
 }
```
最终生成 SQL 语句会数据模型配置及权限进行展开，变成  
``` 
SELECT name, age FROM user
```
也就是只选择模型中已经定义的列及自己有权限的列，如果主动查询没权限的字段会报权限不足。  
这里还能做些有意思的扩展，比如下面的写法  
``` 
{
   "statement": "SELECT",
   "select": ["*", "!content"]
   "from": "blog"
 }
```
意思是查询除了 content 之外的所有列，这样后续增加其它字段就自动支持了，这是 SQL 语句无法实现的功能。  
不过也不是所有情况都能展开，如果 FROM 是子查询就不可以  
``` 
SELECT * FROM (SELECT name FROM user) u
```
行级别的权限配置是在 filters 下，这里针对查、改、删是独立的，常见的场景是我可以查询所有用户的信息，但我只能更新自己的信息，因此使用类似如下语句的 JSONQL 进行更新的时候  
``` 
UPDATE user set name = 'amis'
```
实际执行的 SQL 是  
``` 
UPDATE user set name = 'amis' WHERE user_id = xxx
```
行级别过滤条件是后添加的，用户无法控制，比如写了个  
``` 
UPDATE user set name = 'amis' WHERE 1=1 or 2=2
```
最终执行会变成  
``` 
UPDATE user set name = 'amis' WHERE (1=1 or 2=2) and user_id = xx
```
因此 JSONQL 引擎底层的权限控制彻底杜绝了水平越权问题，只要配置好权限，无论前端怎么构造 JSONQL 请求也无法绕过行列级别的控制，保证了 JSONQL 可以直接暴露给前端使用。  
另外行级别权限控制还能用来支持另外两个重要功能：软删除和多租户，对软删除的支持就是在删除时只是更新删除时间字段而不是真删除，而在查询时加上时间不为 NULL，比如  
``` 
SELECT
   "author"."id",
   "author"."name"
 FROM
   "author"
 WHERE
   "author"."deleted_at" IS NULL
```
而多租户就是在查询时自动加上租户 ID。  

**避免误批量更新或删除大量数据**  
前面提到可以通过权限来避免更新和删除自己没权限的内容，如果无法阻止有权限用户的误删问题，比如管理员不小心执行了下面的语句  
``` 
DELETE FROM user
```
就直接将所有数据都删了，默认情况下 JSONQL 不允许不带 WHERE 的语句执行，上面的预计无法支持，但这个限制很容易绕过，比如随便加个肯定是真的条件  
``` 
DELETE FROM user WHERE id > 0
```
为了解决这个问题，还可以配置 UPDATE 和 DELETE 里必须有主键等于的条件（id = xx），这时引擎会就不允许前面的语句执行，当然下面的语句也是不允许的，主键等于必须在最顶层且和其它条件是 AND 连接  
``` 
DELETE FROM user WHERE id = xx OR 1 = 1
```
通过这种模式可以避免误更新和删除大量数据。  
但攻击者还是能通过写个循环来遍历所有 id，因此还需要下面的功能。  

**自动 Hash id**  
在设计表结构时，一般会使用自增 id 来当成主键，但这导致了两个问题：  
- 暴露总数，因为新增一个就知道目前最大值是什么，通过 id 就知道你有多少注册用户数了
- 遍历攻击，如果有接口出现越权漏洞，攻击者就能写个简单的循环遍历所有数据

有个解决办法是不使用自增 id 作为主键，比如使用 UUID，这对基于 LSM 实现的 NewSQL 分布式数据库比较友好，但对于 MySQL 这种单机数据库却会影响写入性能，因为 MySQL InnoDB 是按主键的 B 树来组织数据的，UUID 的随机性导致经常要在已经写入的两个数据间插入新数据，造成随机读写。  
另一个办法是使用 Hash id，但这种做法对业务侵入性较强，因为所有使用到 id 的地方都得注意对 id 的编码和解码，开发起来麻烦还容易遗漏。  
为了简化开发，JSONQL 引擎内置了对 Hash id 的支持，在模型设计的时可以对主键开启 Hash id 功能，这时主键的输出结果会自动进行 Hash id 编码，而进行查询的时候会进行 Hash id 解码。  
但因为 SQL 的复杂性，有时候虽然能判断字段使用到了主键，也不知道输出结果是不是主键，比如  
``` 
SELECT (id * 0) + 1 FROM table1
```
虽然查询字段有主键，但实际上输出结果是 1，如果对这个结果也进行 Hash id 编码会有问题，因为攻击者可以通过遍历来构造，比如  
``` 
SELECT (id * 0) + 1 FROM table1
 SELECT (id * 0) + 2 FROM table1
 ...
 SELECT (id * 0) + 100000 FROM table1
```
这样就能拿到 10 万个 id 值对应的 Hash id，后续就能通过这些 Hash id 来反查真实 id，也被称为彩虹表攻击。  
除了上面的例子还有很多其他可能情况，为了避免这个问题，JSONQL 引擎在开启 Hash id 时主键就只允许查询字段，不允许使用四则运算及函数调用等功能。  

**数据脱敏**  
和前面 Hash id 类似的功能是数据脱敏，可以针对某些字段设置脱敏，这些字段在输出的时候会进行脱敏处理，比如手机号自动隐藏中间内容。  
另外就是有些字段只能写入不能查询，比如密码字段。  

**性能白名单模式**  
前面提到权限控制解决了安全性问题，但攻击者依然能够通过构造复杂的语句来拖慢系统性能（也被称为 CC 攻击），主要有以下几方面：
- 查询太多数据，比如不加 limit 导致变量全表，这个很好解决，可以配置强制最大 limit 数量，如果超过就不允许，同时 offset 过大也会影响性能，也可以配置最大数量
- JOIN 太多表，这主要是关联查询导致的，关联查询依赖 JOIN，如果使用深层关联查询就需要 JOIN 许多表，比如一个多对多关系就要 JOIN 两张表。因此需要支持配置 JOIN 数量限制
- 使用某些语句导致无法使用索引，比如 SELECT * from table WHERE MONTH (date) = 12 ，这个语句即便 date 字段加了索引也没法使用，因为实际查询是经过某个函数后的值，这和索引中的值并不匹配，因此这样的语句只能遍历全表，除非在创建索引时用的就是 MONTH (date)
- ORDER BY 的字段必须要能用上索引，包括字段顺序、排序方向的一致性

对于数据量较大的表，为了避免性能风险，JSONQL 引擎支持高性能白名单模式，在这个模式下就只能进行最基本的增删改查操作，限制 LIMIT 及 JOIN 数量，所有 WHERE 条件及 ORDER 字段必须使用索引字段且无法使用函数，禁止 GROUB BY 语句，字段函数只支持几种常用的，如果再配合 Nginx 之类的反向代理基于 IP 限制 QPS，就能保证恶意用户无法通过构造复杂 JSONQL 来进行 CC 攻击。  

**支持数据验证**  
后端写入和更新数据有时候需要对数据进行验证，JSONQL 支持数据校验，而且使用和 amis 一样的数据校验配置。  
这样做的好处是同样的配置可以直接用于前后端，无需像专业开发那样前后端都分别进行配置。  
具体配置就不展开了，可以参考 amis 文档，比如对数字的验证是如下配置  
``` 
{
   "validations": {
     "isNumeric": true,
     "minimum": 10
   },
   "validationErrors": {
     "isNumeric": "请输入数字"
   }
 }
```
数据校验机制保证了用户无法通过构造请求的方式来绕过前端验证，保证了数据准确性。  

**和 GraphQL 的区别是什么？**  
主要区别是：
- GraphQL 的抽象程度更高，它设计理念就是从业务出发，不关心具体实现，JSONQL 相对来说更底层，使用者对 SQL 越了解用得越好，需要关注具体实现，比如针对 N+1 问题提供了两种语法来让用户选择
- GraphQL 不自带实现，需要自己实现或使用第三方引擎，这些引擎的实现参差不齐，对数据库的支持不全，JSONQL 还包括了底层引擎实现，支持大量常见数据库
- JSONQL 灵活性更强，很多地方可以使用复杂表达式，覆盖绝大部分 SQL 语句
- GraphQL 依赖自定义的 DSL 语言，前后端都需要使用 SDK 才能方便生成，JSONQL 本身就是 JSON，无需 SDK
- GraphQL 难以表示树形结构，JSONQL 原生支持
- GraphQL 对底层数据存储没有要求，JSONQL 只能对接关系型数据库或 MongoDB 等底层引擎支持的数据存储，如果是自定义存储需要使用 OData 协议对接

因为 JSONQL 的能力可以完全覆盖 GraphQL，所以还能将 GraphQL 转成 JSONQL，比如下面的 GraphQL 查询  
``` 
{
   blog(id: "1000") {
     title
     user {
       name
     }
   }
 }
```
可以转换成等价的 JSONQL 查询  
``` 
{
   "statement": "select",
   "select": ["title", "user.name"]
   "from": "blog",
   "where": {
     "id": "1000"
   }
 }
```
但对于复杂的过滤条件，由于 GraphQL 没有规定标准写法导致难以适配，比如想做字段大于的过滤，在 Hasura 里用的是类似 MongoDB 的语法  
``` 
query {
   movies (where: {rating: {_gt: 4}}) {
     id
     name
   }
 }
```
而 Prisma 用的是字段后面加 _gt 约定  
``` 
 query {
   movies (where: {rating_gt: 4}) {
     id
     name
   }
 }
```
还有 limit、offset、orderBy 及聚合查询等功能都没有标准写法，导致了 GraphQL 很容易被实现绑定，不具备可移植性。  
**和直接用 SQL 的区别是什么？**  
- 支持关联查询，尤其是多层级关联查询用 SQL 写起来很复杂，参考前面关联查询的介绍。
- JSONQL 有更严格的安全检查，参考前面的安全性保障，类似行列级别权限的控制都是自动的，无需自己在 SQL 里加条件判断。
- JSONQL 会改写 SQL 进行性能优化，比如前面提到的大 OFFSET、随机值等场景。
- JSONQL 是 JSON，前端生成比较容易，而拼接 SQL 字符串容易出错。

**和 APIJSON 的区别是什么**  
APIJSON 和 JSONQL 解决了类似问题，同样也是基于 JSON 格式，但在细节思路上有许多不同：
- APIJSON 是自定义的 DSL 语法，有大量自定义语法，比如 key{}@ 代表 EXISTS 子查询，而 JSONQL 尽可能参考 SQL，这时设计上最大的不同，我猜测 APIJSON 最开始只是为了解决简单的 CRUD 问题，语法上没考虑复杂场景，导致后面发现不能满足时就只好不断在语法上打补丁，而 JSONQL 从一开始支持完整 SQL 语法，因此在灵活性方面更强，上手门槛更低。
- APIJSON 只适合 CRUD 查询，JSONQL 还能支持任意复杂的 BI 查询，比如字段和查询条件上使用表达式，APIJSON 许多地方只支持字符串，比如字段、group by 等，无法实现类似按年 group by 的功能。
- APIJSON 的关联查询默认实现是 N+1 查询，而 JSONQL 使用批量查询。
- JSONQL 不仅支持关系数据库，还支持 MongoDB、Redis 等非关系数据库。
- APIJSON 支持原始 SQL，而 JSONQL 由于完整支持 SQL 所以无需这个功能，这个重要设计区别使得 APIJSON 不能直接暴露给前端，后续也难以支持行列级别细粒度权限控制

**如何实现复杂业务逻辑？**  
有时候不仅是数据查询，还需要一些特殊的业务逻辑处理，比如需要对数据进行二次加工后再输出，或者再调用另一个 API 获取其它数据再进行聚合，这时单纯使用 JSONQL 就不够了，可以另一个服务编排的功能实现，它具备查询数据、发起 API 请求、循环、分支、自定义代码等功能，可以用来实现简单的业务逻辑。  
但如果是非常复杂的业务逻辑，比如涉及到复杂的算法，更推荐写代码实现，这方面没有任何可视化工具能简化，所有方案屏幕利用率都极低，展现凌乱还不如代码看起来简洁。  

**缺点**  
基本上是 SQL 子集，所以缺点是开发时没有强类型保护，只提供了 JSON Schema 检查，没法做到 ORM 框架那样的强类型检查。  
但相应的，不需要写代码使得无需编译，可以进行实时查询，特别适合低代码平台。


原文:  
[JSONQL 低代码数据模型引擎的设计与实现](https://mp.weixin.qq.com/s/60LKmsZAuyevlswwSI7_8w)
