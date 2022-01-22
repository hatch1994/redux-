---
title: 'Redux基础教程,第3部分 - state、action和reducer'
date: '2022-01-20'
---
> **你会学到什么**
> - 如何定义包含应用程序的状态值
> - 如何定义描述您的应用程序中发生的事情的Action 对象
> - 如何编写基于现有`state`和`action`计算更新状态的reducer 函数

> **先决条件**
> - 熟悉Redux的关键术语和概念, 如"actions"、“reducers”、“store”和“dispatching”.(有关这些术语的解释,请参阅[第2部分: Redux概念和数据流](http://localhost:3000/posts/redux-concepts-dataflow))

# 介绍
在[第2部分: Redux概念和数据流](http://localhost:3000/posts/redux-concepts-dataflow)中,我们研究了Redux如何通过为我们提供一个放置全局应用程序状态的单-中心store仓库来帮助我们构建可维护的应用程序. 我们还讨论了Redux的核心概念,例如 dispatch an action对象和使用返回新状态值的reducer函数.

现在您已经这些部分有所了解, 是时候将这些知识付诸实践了. 我们将构建一个小型示例应用程序,以了解这些部分如何实际协同工作.

> ⚠️警告
> **示例应用程序并不是一个完整的生产就绪项目*. *目标是帮组您学习核心Redux API和使用模式,并使用一些有限的示例为您指明正确的方向.此外,我们构建的一些早期作品将在稍后更新,以展示更好的做事方式.**请通读整个教程以查看所有使用中的概念**.

# Project Setup

对于本教程,我们创建了一个预配置的启动项目,该项目已经设置了React,包括一些默认样式,并且有一个假 REST API, 允许我们可以在我们的应用程序中编写实际的API请求.您将以此作为编写实际应用程序代码的基础.

```javascriptx
import React from 'react'

export default function App() {
    return (
        <div className="App">
            <nav>
                <section>
                    <h1> Redux 基础知识示例 </h1>
                    <div className="navContent">
                        <div className="navLinks"></div>
                    </div>
                </section>
            </nav>
            <section>
                <h2>欢迎使用Redux基础知识示例应用程序!</h2>
            </section>
        </div>
    )
}
```
你也可以[从这个GitHub repo克隆同一个项目](https://github.com/reduxjs/redux-fundamentals-example-app).克隆repo后,您可以使用安装项目的工具`npm install `,并启动项目`npm start`.

如果您想查看我们将要构建的最终奔波,你可以查看分支`[tutorial-steps](https://github.com/reduxjs/redux-fundamentals-example-app/tree/tutorial-steps)`,或查看此[CodeSandbox中的最终版本](https://codesandbox.io/s/github/reduxjs/redux-fundamentals-example-app/tree/tutorial-steps).

### 创建一个新的Redux + React Project

完成本教程后,您可能会想尝试处理自己的项目,**我们建议使用[Create-React-App的Redux模版作为创建](https://github.com/reduxjs/cra-template-redux)新Redux+Reduct项目的最快方式**.它附带Redux Toolkit和React-Redux,并且已经配置,使用[您在第1部分看到的“计数器”应用程序示例的现代化版本](http://localhost:3000/posts/redux-overview). 这使您可以直接编写实际的应用程序代码,而无需添加Redux包和设置store.

如果您想了解如何将Redux添加到项目中的具体细节,请参阅一下说明:

<details>
<summary>详解: 将Redux添加到React项目</summary>

> CRA的Redux模版附带Redux Toolkit和以配置的React-Redux.如果您在没有该模版的情况下从头开始设置新项目,请按照以下步骤操作:
> - 添加`@reduxjs/toolkit`和`react-redux`包
> - 使用RTK的API创建Redux存储`configureStore`,并出入至少一个reducer函数
> - 将Redux存储导入应用程序的入口点文件 (例如`src/index.js`)
> - `<Provider>`用React-Redux中的组件包装你的根React组件,例如:
> ```javascript
> ReactDOM.render(
>   <Provider store={store}>
>       <App/>
>   </Provider>,
>   document.getElementById('root')
> )
> ```

</details>

### exploring the initial Project 探明这个初始化项目

这个初始项目基于[标准的Create-React-App]项目模版,并进行了一些修改.

让我们快速查看一下初始项目包含的内容:
- /src
    - `index.js`: 应用程序的入口点文件.它呈现主要 `<App>`组件.
    - `App.js`: 主要的应用程序组件.
    - `index.css`: 完整的应用程序的样式
    - `/api`
        - `client.js`: 一个小型的AJAX请求客户端,允许我们发出GET和POST请求
        - `server.js`: 为我们的数据提供一个虚假的REST API. 我们的应用稍后会从这些假API获取数据.
    / exampleAddons: 包含一些额外的Redux插件,我们将在教程后面使用它们来展示事情是如何工作的

如果您现在加载应用程序, 您应该会看到一条欢迎消息,但应用程序的其余部分都是空的.

随即, 让我们开始吧!

# Starting the Todo Example App

我们的示例应用程序将示一个小型“待办”应用程序. 您之前可能已经看过todo应用程序示例 - 它们是很好的示例, 因为它们让我们展示了如何执行诸如跟踪项目列表、处理用户输入以及在数据更改时更新UI, 这些都是在正常应用程序中发生的事情.

## Defining Requirements  定义TODO的需求

让我们首先弄清楚这个应用程序的初始业务需求:
- UI应包括三个主要部分:
    - 一个输入框,让用户输入新待办事项的文本
    - 展现所有现有待办事项的列表
    - 页脚部分, 显示未完成的待办事项数量, 并显示过滤过滤选项
- 
