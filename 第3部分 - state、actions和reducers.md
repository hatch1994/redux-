---
title: 'Redux基础教程,第3部分 - state、action和reducer'
date: '2022-01-20'
---
> **你会学到什么**
> - 如何定义包含应用程序的状态值
> - 如何定义描述您的应用程序中发生的事情的Action 对象
> - 如何编写基于现有`state`和`action`计算更新状态的reducer 函数

> **先决条件**
> - 熟悉Redux的关键术语和概念, 如"actions"、“reducers”、“store”和“dispatching”.(有关这些术语的解释,请参阅[第2部分: Redux概念和数据流](https://github.com/hatch1994/redux-tutorial/blob/main/%E7%AC%AC2%E9%83%A8%E5%88%86%20-%20%E6%A6%82%E5%BF%B5%E5%92%8C%E6%95%B0%E6%8D%AE%E6%B5%81.md))

# 介绍
在[第2部分: Redux概念和数据流](https://github.com/hatch1994/redux-tutorial/blob/main/%E7%AC%AC2%E9%83%A8%E5%88%86%20-%20%E6%A6%82%E5%BF%B5%E5%92%8C%E6%95%B0%E6%8D%AE%E6%B5%81.md)中,我们研究了Redux如何通过为我们提供一个放置全局应用程序状态的单-中心store仓库来帮助我们构建可维护的应用程序. 我们还讨论了Redux的核心概念,例如 dispatch an action对象和使用返回新状态值的reducer函数.

现在您已经这些部分有所了解, 是时候将这些知识付诸实践了. 我们将构建一个小型示例应用程序,以了解这些部分如何实际协同工作.

> ⚠️警告
> **示例应用程序并不是一个完整的生产就绪项目**. 目标是帮组您学习核心Redux API和使用模式,并使用一些有限的示例为您指明正确的方向.此外,我们构建的一些早期作品将在稍后更新,以展示更好的做事方式. **请通读整个教程以查看所有使用中的概念**.

# Project Setup

对于本教程,我们创建了一个预配置的启动项目,该项目已经设置了React,包括一些默认样式,并且有一个假 REST API, 允许我们可以在我们的应用程序中编写实际的API请求.您将以此作为编写实际应用程序代码的基础.

```javascript
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

完成本教程后,您可能会想尝试处理自己的项目,**我们建议使用[Create-React-App的Redux模版作为创建](https://github.com/reduxjs/cra-template-redux)新Redux+Reduct项目的最快方式**.它附带Redux Toolkit和React-Redux,并且已经配置,使用[您在第1部分看到的“计数器”应用程序示例的现代化版本](https://github.com/hatch1994/redux-tutorial/blob/main/%E7%AC%AC1%E9%83%A8%E5%88%86%20-%20redux%20%E6%A6%82%E8%BF%B0.md). 这使您可以直接编写实际的应用程序代码,而无需添加Redux包和设置store.

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
- 待办事项列表项应该有一个复选框来切换它们的“完成”状态.我们还应该能够为预定义的颜色列表添加颜色编码的类别标签,并删除待办事项.
- 计数器应该复数活动待办事项的数量: “0项”、“1项”、“3项”等
- 应该有按钮将所有待办事项标记为已完成,并通过删除它们来清楚所有已完成的待办事项
- 应该有两种方法可以过滤列表中显示的待办事项:
    - 基于显示“All”、“Active”和“Completed”待办事项进行过滤
    - 基于选择一种或多种颜色进行过滤,并显示其标签与这些颜色匹配的任何待办事项

稍后我们将添加更多要求,现在这已经足以让我们开始了.

最终的目标应该是一个如下所示的应用程序:

![示例待办事项应用程序屏幕截图](https://redux.js.org/assets/images/todos-app-screenshot-b88cee51d457022943b3697ac0b010a7.png)

# Designing the State Values

React和Redux的核心原则之一是**你的UI应该基于你的state**.因此, 设计应用程序的一种方法是首先考虑描述应用程序如何工作所需的所有状态. 尝试使用尽可能少的状态值来描述您的UI也是一个好的注意,因此您需要跟踪和更新的数据更少.

从概念上讲, 此应用应该有两个主要方面:
- 当前待办事项的实际列表
- 当前的过滤选项

我们还需要跟踪用户在“添加待办事项”输入框中输入的数据,但这并不重要,我们稍后会处理.
对于每个待办事项,我们需要存储一些信息:

- 用户输入的文本
- 表示是否完成的布尔标志
- 唯一的ID值
- 颜色类别(如果已选择)

我们的过滤行为可能可以用一些枚举值来描述:
- Completed status: “All”、“Active”和“Completed”
- Colors: “Red”、“Yellow”、“Green”、“Blue”、“Orange”、“Purple”

查看这些值,我们也可以说todos是“app state”(应用程序使用的核心数据), 而过滤值是“UI状态”(描述应用程序当前正在做什么的状态).考虑这些不同种类的类别有助于理解不同的状态是如何被使用的。

# Designing the State Structure

使用Redux, **我们的应用程序状态始终保存在纯JavaScript对象和数组中**. 这意味着您不能将其他东西放入Redux状态 - 没有类实例 、 内置JS类型(如`Map`、函数或任何其他非纯JS数据). `Set` `Promise` `Date`

** root Redux 状态值几乎总是一个普通的JS对象**, 其中嵌套了其他数据.

基于这些信息, 我们现在应该能够描述我们需要在Redux状态中拥有的值的种类:
- 首先, 我们需要一个待办事项对象数据. 每个项目都应具有以下字段:
    - `id`: 唯一编号
    - `text`: 用户输入的文本
    - `completed`: 用户输入的文本
    - `color`: 可选的颜色类别
- 然后, 我们需要描述我们的过滤选项. 我们需要:
    - 当前"completed"过滤器值
    - 当前选定颜色类别的数组

因此, 以下是我们应用程序状态的示例:

```javascript
const todoAppState = {
    todos: [
        { id: 0, text: 'Learn React', completed: true },
        { id: 1, text: 'learn Redux', completed: false, color: 'purple'},
        { id: 2, text: 'Build something fun!', completed: false, color: 'blue'}
    ],
    filters: {
        states: 'Active',
        colors: ['red', 'blue']
    }
}
```

需要注意的是, **在Redux之外有其他状态值是可以的!**到目前为止,这个示例足够小,以至于我们实际上确实将随有状态都保存在Redux存储中,但是正如我们稍后看到的,某些数据确实不需要保存在Redux中(例如“这个下拉列表是否打开?” “或表单输入的当前值“).

# Designing Actions
**Action**是具有`type`字段的纯JavaScript对象. 如前述所述, **您可以将操作视为应用程序中发生的事情的事件**.

就像我们根据应用程序的需求设计状态结构一样,我们也应该能够列出一些描述正在发生的事情的Action列表:
- 根据用户输入的文本添加新的待办事项条目
- 切换待办事项的完成状态
- 为待办事项选择颜色类别
- 删除待办事项
- 将所有待办事项标记为已完成
- 清楚所有已完成的待办事项
- 选择不同的“已完成”过滤器值
- 添加新的滤色器
- 移除滤色器

我们通常会将该领域正在发生的事情所需的任何额外数据放入其中`action.payload`.这可以是一个数字、 一个字符串或一个内容包含多个字段的对象.

Redux存储并不关心该`action.type`字段的实际文本是什么.但是，您自己的代码将查看`action.type`，看看是否需要更新。此外，在调试时，您将经常查看ReduxDevTools扩展中的操作类型字符串，以查看应用程序中发生了什么。因此，尝试选择可读的动作类型，并清楚地描述正在发生的事情——当你稍后看它们时，更容易理解它们！

根据可能发生的事情列表，我们可以创建一个应用程序将使用的操作列表：

- `{type: 'todos/todoAdded', payload: todoText}`
- `{type: 'todos/todoToggled', payload: todoId}`
- `{type: 'todos/colorSelected, payload: {todoId, color}}`
- `{type: 'todos/todoDeleted', payload: todoId}`
- `{type: 'todos/allCompleted'}`
- `{type: 'todos/completedCleared'}`
- `{type: 'filters/statusFilterChanged', payload: filterValue}`
- `{type: 'filters/colorFilterChanged', payload: {color, changeType}}`

在这种情况下, Action主要有一个额外的数据,所以我们可以直接把它放在`action.payload`字段中. 我们可以将滤色器行为拆分为两个动作,一个用于“添加”,一个用户“删除”, 但在这种情况下,我们将作为一个动作执行,其中包含一个额外的字段, 专门用于显示我们可以将对象作为一个动作有效载荷.

与状态数据一样,**actions应该包含描述所发生情况所需的最少信息.**

# Writing Reducers

现在我们知道了我们的状态结构和我们的action是什么样的,是时候编写我们的第一个Reducers.

**Reducers**是接受crrentState和action 作为参数并返回新`state`结果的函数.换句话说, `(state, action) => newState`.

## creating the Root Reducer
**Redux应用程序实际上只有一个reducer函数**: 稍后您将传递给的"root reducer"函数`createStore`. 那个 root reducer 函数负责处理所有被调度的动作,并计算每次的真个新状态结果应该是什么.

让我们首先在src文件夹中创建一个`reducer.js`文件, 旁边是`index.js`和App.js

每个reducer都需要一些初始状态,所以我们将添加一些虚假的todo 条目来帮组我们开始. 然后,我们可以为reducer函数内部的逻辑写一个大纲:

```javascript
const initialState = {
    todos: [
        { id: 0, text: 'Learn React', completed: true },
        { id: 1, text: 'learn Redux', completed: false, color: 'purple'},
        { id: 2, text: 'Build something fun!', completed: false, color: 'blue'}
    ],
    filters: {
        status: 'All',
        colors: []
    }
}

// 使用initialState作为默认值
export default function appReducer(state = initialState, action) {
    // reducer 通常查看action.type字段来决定发生了什么
    switch(action.type) {
        // 根据不同的类型的action,来决定在这里做些什么
        default: 
        // 如果reducer无法识别action type,或者不关心这个具体操作,返回现有的状态不变
        return state;
    }
}
```
在初始化应用程序时, 可以使用为定义的状态值调用reducer. 如果发生这种情况, 我们需要提供一个初始状态值, **以便其余的reducer代码可以使用. Reducers 通常使用es6默认参数语法来提供初始状态: (state = initialState, action)**

接下来, 让我们添加处理`todos/todoAdded`动作的逻辑.

我们知道我们需要检查当前操作的类型是否与该特定字符串匹配.然后,我们需要返回一个包含所有状态的新对象,即使对于未更改的字段也是如此.

```javascript
function nextTodoId(todos)  {
    const maxId = todos.reduce((maxId, todo) => Math.max(todo.id, maxId), -1)
}

// 使用 initialState 作为默认值
export default function appReducer(state = initialState, action) {
    // reducer 通常会查看 action type 字段来决定发生什么
    switch  ( action.type )  { 
        // 这里做一些基于不同类型的动作
        case  'todos/todoAdded' : { 
        // 我们需要返回一个新的状态对象
        return  { 
            // 它包含所有现有的状态数据
            ...state,
            // 但是有一个新的数组用于 `todos` 字段
            todos: [ 
            // 包含所有旧的 todos 
                ...state.todos, 
                // 和新的 todo 对象
                { 
                    // 为这个例子使用一个自动递增的数字 ID 
                    id: nextTodoId( state.todos ) , 
                    text: action.payload , 
                    completed: false 
                } 
            ] 
        }
        default :
        // 如果这个 reducer 不能识别动作类型，或者没有
        // 关心这个具体的动作，返回现有状态不变
        return state;
    } 
}
```

那是。。。将一个待办事项添加到状态需要做大量工作。为什么这些额外的工作是必要的？

## Rules of Reduces

我们之前说过, **reducer必须始终遵循一些特殊规则**:
- 它们应该只根据`state`和`action`参数计算新的状态值
- 不允许它们修改现有的 `state`.相反,它们必须通过复制现有状态并更改复制的值来进行不可变更新.
- 它们不能做任何异步逻辑或其他“副作用”

> **“副作用”是在从函数返回值之外可以看到的状态或行为的任何变化**. 一些常见的副作用如下:
> - 将值记录到控制台
> - 保存文件
> - 设置异步计时器
> - 发出Ajax HTTP请求
> - 修改存在函数之外的某些状态,或改变函数的参数
> - 生成随机或唯一随机ID(例如`Math.random()`或`Date.now()`)

任何遵循这些规则的函数也被称为**“纯”函数**,即使它没有专门写成reducer函数

但为什么这些规则很重要? 有几个不同的原因:
- Redux的目标之一是让您的代码可预测.如果仅根据输入参数计算函数的输出,则更容易理解该代码的工作原理并对其进行测试.
- 另一个方面,如果一个函数依赖于自身之外的变量,或者行为随机,你永远不知道运行它会发生什么.
- 如果一个函数修改了其他值,包括它的参数,这可能会改变应用程序的工作方式.这可能是常见的错误来源,例如"我更新了状态,但现在我的UI没有在应该更新的时候更新!"
- 一些Redux DevTools 功能依赖于让你的reducer正确地遵循这些规则

关于“不可变更新”的规则尤为重要,值的进一步讨论.

## Reducers and Immutable updates

早些时候, 我们谈到了"mutable"(修改现有的对象/数组值)和“不变形”(将值视为无法更改的东西).

> ### 警告
> 在Redux中,**我们的reducer永远不允许改变原始/当前状态值!**
> ```javascript
> // 非法 - 默认情况下,这将改变状态!
> state.value = 123
> ```

在Redux 中不能改变状态有几个原因:

- 它会导致错误, 例如UI无法正确更新以显示新值
- 这使得更难理解为什么以及如何更新状态
- 它使编写测试变得更加困难
- 它破坏了正确使用“事件旅行调试”的能力
- 它违背了Redux的预期精神和使用模式

那么如果我们不能改变原始状态, 我们如何返回一个更新的状态呢

> ### tips
> Reducers 只能复制原始值,然后他们可以改变这些副本.
> ```javascript
> // 这是安全的, 因为我们复制了一份
> return {
>  ...state,
>   value: 123
> }
> ```

我们已经看到我们可以[手动编写不可变更新](https://redux.js.org/tutorials/fundamentals/part-2-concepts-data-flow#immutability), 通过使用JavaScript的数组/对象扩展运算符和其他返回原始值副本的函数.

当数据嵌套时, 这变得更加困难. **不可变更新的一个关键规则时,您必须复制需要更新的每一层嵌套**.

但是,如果你认为"以这种方式手动编写不可变更新看起来很难记住和正确执行".....是的,你是对的!

手动编写不可变的更新逻辑很困难,并且**在reducer中意外改变状态是Redux用户最常犯的一个错误**.

> ### tips
> **在实际应用程序中, 您不必手动编写这些复杂的嵌套不可变更新**. 在第**8部分: 使用Redux Toolkit的现代Redux中**, 您将学习如何使用Redux Toolkit来简化在reducer中编写不可变更新逻辑的过程.

# Handling Additional Actions
考虑到这一点,让我们为另外几个案例添加reducer逻辑,首先,根据起ID切换代码事项的`completed`字段:
```javascript
export default function appReducer(state = initialState, action) {
    switch(action.type) {
        case 'todos/todoAdded': {
            return {
                ....state,
                todos: [
                    ...state.todos,
                    {
                        id: nextTodoId(state.todos),
                        text: action.payload,
                        completed: false
                    }
                ]
            }
        }
        case 'todos/todoToggled': {
            return {
                // 再次复制整个 state object
                ...state,
                // 这一次,我们需要复制旧的 todos 数组
                todos: state.todos.map(todo => {
                    // 如果这不是我们要找的待办事项,那就别管它
                    if(todo.id !== action.payload) {
                        return todo
                    }

                    // 我们找到了必须改变的待办事项,返回副本:
                    return {
                        ...todo,
                        // 翻转 completed flag
                        completed: !todo.completed
                    }
                })
            }
        }
        default: 
        return state;
    }
}
```

由于我们一直专注于待办事项状态, 让我们添加一个案例来处理"可见行选择更改"操作
```javascript
export default function appReducer(state = initialState, action) {
    switch(action.type) {
        case 'todos/todoAdded': {
            return {
                ...state,
                todos: [
                    ...state.todos,
                    {
                        id: nextTodoId(state.todos),
                        text: action.payload,
                        completed: false
                    }
                ]
            }
        }
        case 'todos/todoToggled': {
            return {
                ...state,
                todos: state.todos.map(todo => {
                    if(todo.id !== action.payload) {
                        return todo
                    }
                    
                    return {
                        ...todo,
                        completed: !todo.completed
                    }
                })
            }
        }
        case 'filters/statusFilterChanged': {
            return {
                // 复制完成的状态
                ...state,
                // 重写 filters value
                filters: {
                    // 复制其他的 filter fields
                    ...state.filters,
                    // 并用新的值替换 status 字段
                    status: action.payload
                }
            }
        }
        default:
            return state
    }
}
```

我们只处理了3个动作,但这已经有点长了. 如果我们试图处理合格reducer函数中的每一个动作,那么将很难阅读所有内容.
者就是为什么**reducer通常被拆分为多个较小的reducer函数**的原因 -—— 以便更容易理解和维护 reducer逻辑.

# Splitting Reducers

作为其中的一部分, **Redux reducer通常会根据他们更新的Redux状态部分进行拆分**. 我们的todo应用状态目前有两个顶级部分: `state.todos`和`state.filters`. 因此, 我们可以将整个root reducer函数拆分为两个较小的reducer `todoReducer`和`filtersReducer`.

那么,这些拆分的reducer函数应该放在哪里呢?

**我们建议根据“功能”**(与应用程序的特定概念或区域相关的代码)来组织您的Redux应用程序文件夹和文件. **特定功能的Redux代码通常编写为单个文件,称为“切片”文件**,其中包含所有reducer逻辑和所有与应用程序状态部分相关的操作代码.

因此, **Redux应用程序状态的特定部分的reducer称为"slice reducer"**. 通常, 一些Action对象将于特定的切片reducer密切相关,因此Action type类型字符串应该以该功能的名称(例如: `'todos'`)开头并描述发生的事件(例如: `'todoAdded'`), 连接在一次称为一个字符串(`'todos/todoAdded'`).

在我们的项目中, 创建新`features`文件夹, 然后`todos`在其中创建一个文件夹. 创建一个名为`todosSlice.js`的新文件, 然后将与todo相关的初始状态剪切并粘贴到该文件中:

```javascript
const initialState = [
    { id: 0, text: 'learn React', completed: true },
    { id: 1, text: 'learn Redux', completed: false, color: 'purple' },
    { id: 2, text: 'Build something fun!', completed: false, color: 'blue' }
]

function nextTodoId(todos) {
    const maxId = todos.reduce((maxId, todo) => Math.max(todo.id, maxId), -1)
    return maxId + 1
}

export default function todosReducer(state = initialState, action) {
    switch(action.type) {
        default: 
            return state
    }
}
```

现在我们可以复制更新todos的逻辑. 但是, 这里有一个重要的区别. **这个文件只需要更新与todos相关的状态————它不再嵌套了!** 这是我们拆分reducer的另一个原因.由于todos状态本身就是一个数组, 因此我们不必在此处复制外部根状态对象. 这使得这个reducer更容易阅读.

这称为**reducer组合**, 它是构建redux应用程序的基本模式.

这是我们处理完这些操作更新后的reducer的样子:
```javascript
export default function todosReducer(state = initialState, action) {switch(action.type) {
    case 'todos/todoAdded': {
        return [
            ...state, 
            {
                id: nextTodoId(state),
                text: action.payload,
                completed: false
            }
        ]
    }
    case 'todos/todoToggled': {
        return state.map(todo => {
            if(todo.id !== action.payload) {
                return todo
            }

            return {
                ...todo,
                completed: !todo.completed
            }
        })
    }
    default:
        return state
}}
```

这有点短,更容易阅读.

现在我们可以对可见行逻辑做同样的事情. 创建`src/features/filters/filtersSlice.js`,然后将所有与过滤器相关的代码移到那里:
```javascript
const initialState = {
    status: 'All',
    colors: []
}

export default function filtersReducer(state = initialState, action) {
    switch(action.type) {
        case 'filters/statusFilterChanged': {
            return {
                // 同样,要复制的嵌套层次少了一层
                ...state,
                status: action.payload
            }
        }
        default: 
            return state
    }
}
```

我们仍然比心复制包含过滤器状态的对象, 但由于嵌套较少,因此更容易阅读正在发生的事情.
> **info**
> 为了使这个页面更短, 我们将跳过展示如何为其他操作编写reducer更新逻辑.
> 根据[上述要求](https://redux.js.org/tutorials/fundamentals/part-3-state-actions-reducers#defining-requirements), **尝试自己编写更新**.
> 如果遇到困难,请参阅本页末尾的代码示例,了解这些reducer的完整实现.