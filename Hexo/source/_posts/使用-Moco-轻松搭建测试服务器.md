---
title: 使用 Moco 轻松搭建测试服务器
date: 2016-09-03 17:13:17
categories: 
    - "测试"
tags: [服务器, 接口, 测试]
description: "使用 Moco 轻松搭建简单测试服务器，方便移动开发接口调试。"
---

新公司的项目使用了 [RAP](http://rap.taobao.org/) 这一阿里推出的可视化接口管理工具，通过在上面定义接口，RAP 可以通过分析接口结构，以一系列自动化工具帮助开发人员提升效率。重要的是 RAP 通过 Mock 服务可以生成模拟数据。

以上的这些都不是讨论的重点，本篇文章主要是记录如何通过使用 Moco 搭建本地服务器，以达到在接口定义初期，就可以通过 Moco 调试接口，推进移动开发的进程。

## Moco 介绍

Moco 是一个基于 Java 开发的开源项目，在 GitHub 上有不少的关注：https://github.com/dreamhead/moco
> Moco is an easy setup stub framework.

## 快速开始
1. 下载编译好的 Jar 文件，目前版本是 0.11.0：
https://repo1.maven.org/maven2/com/github/dreamhead/moco-runner/0.11.0/moco-runner-0.11.0-standalone.jar

    > 需要注意的是，之后在运行此 Jar 文件时，需要预先安装好 Java 的 JDK。如果不知道有没有安装过，可在下载完上面的 Jar 文件后，双击打开，如果没有任何提示的话，说明已经安装过 JDK，否则会有一个提示去下载的弹出框。JDK 下载地址为：http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html ，选择适合自己电脑系统的版本进行下载安装。

2. 编写配置文件，像下面这个样子描述你的 Moco 服务的配置：

    ``` Json
    [
      {
        "response" : {
            "text" : "Hello World"
          }
      }
    ]
    ```
    并命名为 foo.json 。
    
3. 通过配置文件 foo.json 启动 Moco 服务
    在命令行输入：
    
    ```
    $ java -jar moco-runner-<version>-standalone.jar http -p 12306 -c foo.json
    ```
    其中 `<version>` 替换为下载的 Jar 包的版本，此处为 `0.11.0` ，所以执行：`java -jar moco-runner-0.11.0-standalone.jar http -p 12306 -c foo.json` 。
    -p 指定 Moco 服务端口。
    > 需要注意的是，执行以上的命令，需要先 cd 到 Jar 文件所在的目录，然后 foo.json 文件也需要放在同一个目录才行，否则会报错。或者可以通过使用 Jar 文件和 .json 文件绝对路径的方式也可以。
    > json 文件路径也可以使用相对路径，如：./json/foo.json
    
4. 访问 Web 服务
    浏览器访问：`http://localhost:12306` ，就可以看到令人惊喜的 “Hello World” 了。: )
    
    ---

## 进阶使用

具体使用可参考 Moco 的 [官方文档](https://github.com/dreamhead/moco/blob/master/moco-doc/apis.md)

### description 字段作为注释

```Json
[
    {
        "description": "any response",
        "response": {
            "text": "foo"
        }
    }
]
```
在所有的 JSON APIs 中，使用 `description` 字段作为描述此配置的作用，在服务启动时此字段会被自动忽略掉。

---

### Request
#### URI

```Json
[
    {
        "request" : {
          "uri" : "/foo"
        },
         "response" : {
          "text" : "bar"
        }
    }
]
```
访问 `http://localhost:12306/foo` ,就可以获取到内容：bar。

#### Query Parameter
带有参数的请求

```Json
[
    {
      "request" : {
          "uri" : "/foo",
          "queries" : {
              "param" : "blah"
            }
        },
      "response" : {
          "text" : "bar"
        }
    }
]
```
访问 `http://localhost:12306/foo?param=blah` 获取到内容：bar。

#### HTTP Method

根据指定的 HTTP 方法返回 response ，也非常容易设置。
设置 GET 请求：

```Json
[
   {
     "request" : {
         "method" : "get",
         "uri" : "/foo"
       },
     "response" : {
         "text" : "bar"
       }
   }
]
```
GET 请求 `http://localhost:12306/foo` 获取到内容：bar。

设置其他请求同理。

#### Header

```Json
[
    {
      "request" : {
          "method" : "post",
          "headers" : {
            "content-type" : "application/json"
          }
        },
      "response" : {
          "text" : "bar"
        }
    }
]
```

#### Cookie

```Json
[
    {
      "request" : {
          "uri" : "/cookie",
          "cookies" : {
              "login" : "true"
            }
        },
      "response" : {
          "text" : "success"
        }
    }
]
```
    
#### Form

```Json
[
    {
      "request" : {
          "method" : "post",
          "forms" : {
              "name" : "foo"
            }
        },
      "response" : {
          "text" : "bar"
        }
    }
]
```

#### XML
XML 格式目前也还在流行着，配置方法如下：

```Json
[
    {
      "request": {
          "uri": "/xml",
          "text": {
              "xml": "<request><parameters><id>1</id></parameters></request>"
            }
        },
      "response": {
          "text": "foo"
        }
    }
]
```
如果 XML 文件很大的话，可以放在一个 xml 文件中，然后通过引用文件路径的方式来发起请求：

```Json
[
    {
       "request": {
            "uri": "/xml",
            "file": {
                "xml": "your_file.xml"
              }
        },
      "response": {
          "text": "foo"
        }
    }
]
```

#### JSON Request
##### JSON Text

```Json
[
   {
        "request": {
            "uri": "/json",
            "json": {
                "foo": "bar"
            }
        },
        "response": {
            "text": "foo"
        }
    }
]
```
访问 `http://localhost:12306/json` ，将 `{"foo": "bar"}` 作为字符串 POST 到服务器。

#### Operator
##### Match
你可能想通过正则表达式来匹配你的 request，**match** 可以帮到你
比如：

```Json
[
    {
      "request": {
          "uri": {
              "match": "/\\w*/foo"
            }
        },
      "response": {
          "text": "bar"
        }
    }
]
```
匹配任意类似 `http://localhost:12306/xxx/foo` 的请求，并返回：bar。其中的 `/\\w*` 表示以 `/` 开始，之后是任意数量的数字或字母。

##### Starts With, Ends With, Contain

```Json
[
    {
      "request": {
          "uri": {
              "startsWith": "/foo"
              //"endsWith": "foo"
              //"contain": "foo"
            }
        },
      "response": {
          "text": "bar"
        }
    }
]
```

---

### Response
#### Content
正如在之前的实例中看到的，直接返回一个文字内容非常简单：

```Json
[
    {
      "request" : {
          "text" : "foo"
        },
      "response" : {
          "text" : "bar"
        }
    }
]
```
访问 `http://localhost:12306/` 将 foo 作为字符串 POST 到服务端，则会直接返回内容：bar。
和 request 一样，如果 response 内容很多的话，可以放在一个文件中返回。例如：

```Json
[
    {
      "request" : {
          "text" : "foo"
        },
      "response" : {
          "file" : "bar.response"
        }
    }
]
```

#### Status Code
Moco 支持 HTTP 状态码的返回：

```Json
[
    {
      "request" : {
          "text" : "foo"
        },
      "response" : {
          "status" : 200
        }
    }
]
```

#### Header
我们同样可以在 response 中指定 HTTP Header。

```Json
[
    {
      "request" : {
          "text" : "foo"
        },
      "response" : {
          "headers" : {
              "content-type" : "application/json"
            }
        }
    }
]
```

#### Proxy
##### Single URL
我们可以像 proxy 一样，返回一个指定的 URL。

```Json
[
    {
      "request" : {
          "text" : "foo"
        },
      "response" : {
          "proxy" : "http://www.github.com"
        }
    }
]
```
实际上，proxy 要更加的强大。它可以将所有的请求都指向指定的 url，包括 HTTP 的 method，version，header，content 等等。

##### Failover
除了基础功能之外，proxy 也支持 failover，这意味着如果远程服务暂时不能起作用的话，moco 服务将知道从本地的配置中修复。
例如：

```Json
[
    {
      "request" : {
          "text" : "foo"
        },
      "response" : {
          "proxy" : {
              "url" : "http://localhost:12306/unknown",
              "failover" : "failover.json"
            }
        }
    }
]
```
Proxy 会将 request/response 成对地保存在你指定的 failover 文件中。如果 proxy 指定的目标不可用，proxy 将从这个文件中恢复。这个功能在开发环境下非常有用，尤其是当集成服务还没有稳定的时候。

##### Playback
Moco 也支持 playback，支持将远程的 request 和 response 保存到本地文件中。failover 和 playback 的不同支持在于，playback 只会当本地的 request 和 response 不可用时才会进入远程服务。

```Json
[
    {
      "request" : {
          "text" : "foo"
        },
      "response" : {
          "proxy" : {
              "url" : "http://localhost:12306/unknown",
              "playback" : "playback.json"
            }
        }
    }
]
```

#### Redirect

```Json
[
    {
      "request" : {
          "uri" : "/redirect"
        },
      "redirectTo" : "http://www.github.com"
    }
]
```

#### Cookie
Cookie 同样可以放在 response 中。比如：

```Json
[
    {
      "request" : {
          "uri" : "/cookie"
        },
      "response" : {
          "cookies" : {
            "login" : "true"
          }
        }
    }
]
```
当访问 `http://localhost:12306/cookie` 时，在返回的 Header 中将会看到：`Set-Cookie : login=true; Path=/`

#### JSON Response
对于 JSON API， 直接返回一个 json object 就可以了。比如:

```Json
[
    {
        "request": {
            "uri": "/json"
          },
        "response": {
            "json": {
                "foo" : "bar"
              }
          }
    }
]
```
访问 `http://localhost:12306/json`，则会返回 json：`{"foo":"bar"}`

## 使用技巧
### 配置文件
简单地启动 moco 服务时是通过命令来启动的：

```
$ java -jar moco-runner-<version>-standalone.jar http -p 12306 -c foo.json
```
注意到这里有一个 `-c foo.json` ，此时只会将 `foo.json` 配置文件中的内容加载到服务中，如果仅仅通过以上的命令，则在调试多个接口时，需要不断的停止旧的服务、通过新的 json 配置文件启动新的服务，非常麻烦。

> Moco 支持动态加载配置文件，无论是修改还是添加配置文件都是不需要重启服务的。

#### 配置文件的设置技巧
配置文件的格式为一个 **数组** 类型的 JSON 格式，数组的每一个元素是一个 `request/response` 的配对。比如：

```Json
[
    {
        // 此处配置了一个 request
        "request" : {
          "uri" : "/foo"
        },
        // 此处配置了对应于上方 request 的 response
        "response" : {
          "text" : "bar"
        }
    }
]
```
> 一个 request/response 对，当中的 request 可以没有，则此时访问 `http://localhost:12306/` 时，Moco 就会直接将配置的 response 中的内容返回。

上面说了配置文件内容是 **数组** 类型的，所以我们还可以这样写：

```Json
[
    {
        "request" : {
          "uri" : "/foo"
        },
        "response" : {
          "text" : "bar"
        }
    },
    {
        "request" : {
          "uri" : "/foo2"
        },
        "response" : {
          "text" : "bar2"
        }
    }
]
```
此时我们加载了这个配置文件后，可以通过 `http://localhost:12306/foo` 和 `http://localhost:12306/foo2` 分别获取到 bar 和 bar2。这种对于需要测试的接口数量较少时，比较快速方便。

#### 全局配置文件
Moco 支持在全局的配置文件中引入其他配置文件，这样就可以分服务定义配置文件，便于管理。
例如你有两个不同路径的 API：`http://xxx.com/path1/login` 和 `http://xxx.com/path2/pay` （登录和支付接口）。
按照上一小节（4.1.1）所讲，我们可以写好 login 和 pay 的两个配置文件（或写在一起），分别设置 request 的 url 为 `/path1/login` 和 `/path2/pay` 。如果需要测试的接口很多，则不利于管理，且 path1、path2 这么混乱的分布于不同的配置文件中，对于以后想要更改也很不方便。

正确的姿势应该是这样的：
同样写好 login.json 和 pay.json 两个配置文件，然后写一个 **全局配置文件** ，配置如下：

```Json
// config.json
[
	{"context":"/path1", "include":"login.json"},
	{"context":"/path2", "include":"pay.json"}
]
```
login 和 pay 两个文件没有特殊要求，和之前的写法一样。比如：

```Json
// register.json
[
    {
        "request": {
            "uri": "/register",
            "method": "POST",
            "json": {
                "phone":"18688886666",
                "password":"123456"
            }
        },
        "response": {
            "json": {
                "state":"0"
            }
        }
    }
]

// login.json
[
    {
        "request": {
            "uri": "/login",
            "method": "POST",
            "json": {
                "amount":"100"
            }
        },
        "response": {
            "json": {
                "state":"0"
            }
        }
    }
]
```

然后启动 Moco 服务的命令是：

```
$ java -jar moco-runner-<version>-standalone.jar http -p 12306 -g config.json
```
要注意的是，最后指定的参数是 **`-g config.json`** !

如果只是想引入多个 json 文件的话，全局配置文件中可以不使用 `context` 字段。比如：

```Json
// 不使用 context 字段的 config.json。
[
    {"include":"login.json"},
    {"include":"pay.json"}
]
```

（持续更新中...）

