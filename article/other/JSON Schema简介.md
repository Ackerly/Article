# JSON Schema 简介
假设有一个web api，接受一个json请求，返回某个用户在某个城市关系最近的若干个好友。一个请求的例子如下：  
``` 
{
    "city" : "chicago", 
    "number": 20, 
    "user" : {
        "name":"Alex", 
        "age":20
    }
```
在上面的例子中，web api要求提供city，number，user三个成员，其中city是字符串，number是数值，user是一个对象，又包含了name和age两个成员。  
对于api来说，需要定义什么样的请求合法，即什么样的Json对于api来说是合法的输入。这个规范可以通过Json Schema来描述，对应的Json Schema如下。  
``` 
{ 
    "type": "object",
    "properties": {
        "city": { "type": "string" },
        "number": { "type": "number" },
        "user": { 
            "type": "object",
            "properties": {
                "name" : {"type": "string"},
                "age" : {"type": "number"}
            }                       
        }
    }
}
```
> Json Schema定义了一套词汇和规则，这套词汇和规则用来定义Json元数据，且元数据也是通过Json数据形式表达的.  
Json元数据定义了Json数据需要满足的规范，规范包括成员、结构、类型、约束等。

## 类型关键字
首先需要了解的是"type"关键字，这个关键字定义了Json数据需要满足的类型要求。"type"关键字的用法如下面几个例子：  
1. {"type":"string"}。规定了Json数据必须是一个字符串，符合要求的数据可以是
2. {"type" : "object"}。规定了Json数据必须是一个对象，符合要求的数据可以是
3. {"type" : "number"}。规定了Json数据必须是一个数值，符合要求的数据可以是。JavaScript不区分整数、浮点数，但是Json Schema可以区分。 
4. {"type": "integer"}。要求数据必须是整数。
5. {"type" : "array"}。规定了Json数据必须是一个数组，符合要求的数据可以是
6. {"type" : "boolean"}。这个Json Schema规定了Json数据必须是一个布尔，只有两个合法值
7. {"type" : "null"}。null类型只有一个合法值

## 简单类型
**字符串**  
可以进一步对字符串做规范要求。字符串长度、匹配正则表达式、字符串格式。
``` 
{
    "type" : "string",
    "minLength" : 2,    // 最小长度
    "maxLength" : 3,    // 最大长度
    "pattern" : "^(\\([0-9]{3}\\))?[0-9]{3}-[0-9]{4}$",     // 正则
    "format" : "date",  // 可以是电子邮件、日期、域名
}
```
**数值**
"number"和"integer"的类型特定参数相同，可以限制倍数、范围、最大值、最小值、开区间最大值、开区间最小值
``` 
{
    "type" : "number",
    "multipleOf" : 10,      // 特定值的整数倍，例如要求数值必须是10的整数倍
    "minimum": 0,
    "exclusiveMaximum": 100
}
```
## 复合类型
复合类型可以通过Nest的方式构建复杂的数据结构。包括数组、对象。  
**数组**
Json数组合法数据的例子  
``` 
[1, 2, 3]
[1, "abc", {"name" : "alex"}]
[]
```
数组的类型特定参数，可以用来限制成员类型、是否允许额外成员、最小元素个数、最大元素个数、是否允许元素重复。  
1.数组成员类型  
可以要求数组内每个成员都是某种类型，通过关键字items实现。  
``` 
{
    "type": "array",
    "items": {
        "type": "number"
    }
}
```
关键字items还可以对应一个数组，这时Json数组内的元素必须与Json Schema内items数组内的每个Schema按位置一一匹配。  
``` 
{
    "type": "array",
    "items": [
    {
        "type": "number"
    },
    {
        "type": "string"
    }]
}
```
2.数组是否允许额外成员  
关键字additionalItems规定Json数组内的元素，除了一一匹配items数组内的Schema外，是否还允许多余的元组。当additionalItems为true时，允许额外元素。  
``` 
{
    "type": "array",
    "items": [
    {
        "type": "number"
    },
    {
        "type": "string"
    }],
    "additionalItems" : true
}
```
3.限制数组元素个数
``` 
{
    "type": "array",
    "items": {
        "type": "number"
    },
    "minItems" : 5,
    "maxItems" : 10
}
```
4.数组内元素是否必须唯一  
``` 
{
    "type": "array",
    "items": {
        "type": "number"
    },
    "uniqueItems" : true
}
```

**对象**  
1.  成员的Schema  
``` 
{ 
    "type": "object",     
    "properties": {      
        "name": {"type" : "string"},
        "age" : {"type" : "integer"},
        "address" : {
            "type" : "object",
            "properties" : {
                "city" : {"type" : "string"},
                "country" : {"type" : "string"}
            }
        }
    }
}
```
properties关键字的内容是一个key/value结构的字典，其key对应Json数据中的key，其value是一个嵌套的Json Schema。表示Json数据中key对应的值所应遵守的Json Schema。  
2. 批量定义成员Schema(关键字patternProperties)
``` 
{
    "type": "object",
    "patternProperties": {
        "^S_": { "type": "string" },
        "^I_": { "type": "integer" }
    }
}
```
3. 必须出现的成员（关键字required）  
``` 
{ 
    "type": "object",     
    "properties": {      
        "name": {"type" : "string"},
        "age" : {"type" : "integer"},
    },
    "required" : ["name"]
}
```
4. 成员依赖关系(关键字dependencies)  
``` 
{
    "type": "object",
    "dependencies": {
        "credit_card": ["billing_address"]
    }
}
```
dependencies也是一个字典结构，key是Json数据的属性名，value是一个数组，数组内也是Json数据的属性名，表示key必须依赖的其他属性。

5. 是否允许额外属性(关键字additionaProperties)  
规定object类型是否允许出现不在properties中规定的属性，只能取true/false。
``` 
{ 
    "type": "object",     
    "properties": {      
        "name": {"type" : "string"},
        "age" : {"type" : "integer"},
    },
    "required" : ["name"],
    "additionalProperties" : false
}
```
6. 属性个数的限制(关键字minProperties, maxProperties)
``` 
{
    "type": "object",
    "minProperties": 2,
    "maxProperties": 3
}
```
## 逻辑组合
1. allOf  
``` 
{
    "allOf" : [
        Schema1,
        Schema2,
        ...
    ]
}
```
需要注意，不论在内嵌的Schema里还是外部的Schema里，都不应该使"additionalProperties"为false。否则可能会生成任何数据都无法满足的矛盾Schema。  
可以用来实现类似“继承”的关系，例如我们定义了一个Schema_base，如果想要对其进行进一步修饰，可以这样来实现。  
``` 
{
    "allOf" : [
        Schema_base
    ]
    "properties" : {
        "other_pro1" : {"type" : "string"},
        "other_pro2" : {"type" : "string"}
    },
    "required" : ["other_pro1", "other_pro2"]
}
```
Json数据既需要满足Schema_base，又要具备属性"other_pro1"、"other_pro2"。
2. anyOf
``` 
{
    "anyOf" : [
        Schema1,
        Schema2,
        ...
    ]
}
```
Json数据需要满足Schema1、Schema2中的一个或多个
3. oneOf(满足且进满足oneOf数组中的一个Schema)  
``` 
{
    "oneOf" : [
        Schema1,
        Schema2,
        ...
    ]
}
```
4. not(它告诉Json不能满足not所对应的Schema)
``` 
{
    "not" : {"type" : "string"}
}
```
只要不是string类型的都Json数据都可以  
## 复杂结构
对复杂结构的支持包括定义和引用。可以将相同的结构定义成一个“类型”，需要使用该“类型”时，可以通过其路径或id来引用。  
**定义**  
定义一个类型，并不需要特殊的关键字。通常的习惯是在root节点的definations下面，定义需要多次引用的schema。definations是一个json对象，key是想要定义的“类型”的名称，value是一个json schema  
``` 
{
    "definitions": {
        "address": {
            "type": "object",
            "properties": {
                "street_address": { "type": "string" },
                "city":           { "type": "string" },
                "state":          { "type": "string" }
            },
            "required": ["street_address", "city", "state"]
        }
    }，
    "type": "object",
    "properties": {
        "billing_address": { "$ref": "#/definitions/address" },
        "shipping_address": { "$ref": "#/definitions/address" }
    }
}
```
上例中定义了一个address的schema，并且在两个地方引用了它，#/definitions/address表示从根节点开始的路径  
**$id**  
可以在上面的定义中加入id属性，这样可以通过id属性的值对该schema进行引用，而不需要完整的路径  
``` 
...
    "address": {
            "type": "object",
            "$id" : "address",
            "properties": {
                "street_address": { "type": "string" },
                "city":           { "type": "string" },
                "state":          { "type": "string" }
            },
            "required": ["street_address", "city", "state"]
        }
...
```
**引用**  
关键字$ref可以用在任何需要使用json schema的地方。如上例中，billing_address的value应该是一个json schema，通过一个$ref替代了。  
$ref的value，是该schema的定义在json中的路径，以#开头代表根节点。  
``` 
{
    "properties": {
        "billing_address": { "$ref": "#/definitions/address" },
        "shipping_address": { "$ref": "#/definitions/address" }
    }
}
```
如果schema定义了$id属性，也可以通过该属性的值进行引用  
``` 
{
    "properties": {
        "billing_address": { "$ref": "#address" },
        "shipping_address": { "$ref": "#address" }
    }
}
```
## 通用关键字
**enum**  
可以在任何json schema中出现，其value是一个list，表示json数据的取值只能是list中的某个  
``` 
{
    "type": "string",
    "enum": ["red", "amber", "green"]
}
```
上例的schema规定数据只能是一个string，且只能是"red"、"amber"、"green"之一  
**metadata**  
关键字：title，description，default，example  
``` 
{
    "title" : "Match anything",
    "description" : "This is a schema that matches anything.",
    "default" : "Default value",
    "examples" : [
        "Anything",
        4035
    ]
}
```

原文:  
[SON Schema 简介](https://www.cnblogs.com/terencezhou/p/10474617.html)
