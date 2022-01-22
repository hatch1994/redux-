---
title: 'Redux基础教程,第1部分 - redux 概述'
date: '2022-01-19'
---

# 什么是Redux

它有助于首先理解这个"Redux"是什么.它有什么用? 它帮助我们解决了哪些问题? 我们又为什么要使用它?


*Redux 是一种模式和库,用于管理和更新应用程序状态,使用称为“动作”的事件.* 它充当需要在整个应用程序中使用过程中的状态集中存储,其规则确保状态只能以可预测的方式更新.

## 为什么要使用Redux

Redux 帮助您管理“全局”状态————应用程序的许多部分都需要的状态.


*Redux 提供的模式和工具使您更容易理解何时、何地、为什么以及如何更新应用程序中的状态,以及当这些更改发生时您的应用程序逻辑将如何表现.* Redux指导您编写可预测和可测试的代码,者有助于让您确信您的应用程序将按预期工作

## 我什么时候应该使用Redux

Redux可帮组您处理共享状态管理,但与任何工具一样,它也需要权衡取舍.有更多的概念要学习,也有更多的代码要编写.它还为您的代码添加了一些间接性,并要求您遵循某些限制.这是短期和长期生产力之间的权衡.

Redux在以下情况下更有用:

* 您在应用程序的许多地方都需要大量的应用程序状态
* 应用程序状态会随着时间的推移而频繁更新
* 更新改状态的逻辑可能很复杂
* 该应用程序具有中型或大型代码库,并且可能由许多人开发

*并非所有应用程序都需要Redux.花一些时间考虑您正在构建的应用程序类型,并决定哪些工具最能帮助解决您正在处理的问题.*

> ### 想知道更多?
> 
> 如果您还不确定Redux是否适合您的应用程序,这些资源会提供更多知道:
>
> * [何时(何时不)使用Redux](https://changelog.com/posts/when-and-when-not-to-reach-for-redux)
> * [Redux之道,第1部分-实现和意图](https://blog.isquaredsoftware.com/2017/05/idiomatic-redux-tao-of-redux-part-1/)
> * [Redux常见问题解答:我应该什么时候使用Redux?](https://redux.js.org/faq/general#when-should-i-use-redux)
> * [您可能不需要Redux](https://medium.com/@dan_abramov/you-might-not-need-redux-be46360cf367)
>

## Redux库和工具

Redux是一个小型的独立JS库.但是,它通常与其他几个包一起使用:

### React

Redux 可以与任何 UI 框架集成，并且最常与 React 一起使用。React-Redux是我们的官方包，它允许您的 React 组件通过读取状态片段和调度操作来更新存储来与 Redux 存储进行交互。

### Redux

Redux Toolkit是我们推荐的编写 Redux 逻辑的方法。它包含我们认为对于构建 Redux 应用程序至关重要的包和功能。[Redux Toolkit](https://react-redux.js.org/) 构建在我们建议的最佳实践中，简化了大多数 Redux 任务，防止常见错误，并使编写 Redux 应用程序更容易。

### Redux DevTools

[Redux DevTools Extension](https://github.com/zalmoxisus/redux-devtools-extension)显示了 Redux 存储中状态随时间变化的历史记录。这使您可以有效地调试您的应用程序，包括使用强大的技术，如“时间旅行调试”。

# Redux

既然您知道Redux是什么,让我们简要地看一下组成Redux应用程序的部分以及它是如何工作的.

>
> ### 信息
>
> 本页的其余描述仅关注Redux核心库(redux 包).在完成本教材的其余部分时,我们将讨论其他与Redux相关的包.
>

## Redux

每个Redux应用程序的中心是*store*. “存储”是保存应用程序全局状态的容器.

store是一个JavaScript对象,具有一些特殊的功能和能力,使其不同于普通的全局对象:

* 你绝不能直接修改或更改保存在Redux存储中的状态
* 相反,导致状态更新的唯一方法是创建一个描述“应用程序中发生的事情”的普通操作对象,然后将操作分派到存储以告诉它发生了什么.
* 当一个Action被调用时,store运行root reducer函数,并让它根据旧状态和action计算新状态
* 最后, store通知订阅者状态已更新,以便UI可以使用新数据进行更新.

# Redux 核心示例

让我们看一个Redux应用程序的最小工作示例

### 一个小型计算器的应用程序:

```javascript
<!DOCTYPE html>
<html>
    <head>
        <title>Redux basic example</title>
        <script src="https://unpkg.com/redux@latest/dist/redux.min.js"></script>
    </head>
    <body>
        <div>
            <p>
                Clicked: <span id="value">0</span> items
                <button id="increment"> + </button>
                <button id="decrement"> - </button>
                <button id="incrementIfOdd"> 每次添加奇数 </button>
                <button id="incrementAsync"> 异步添加 </button>
            </p>
        </div>
        <script>
            // 定义应用的初始状态值
            const initialState = { value: 0 }

            // 创建一个 “reducer” 函数来决定新的状态值
            // 应该是应用程序发生一些什么时
            function counterReducer(state = initialState, action) {
                // “reducer”通常会查看发生的 "action" 类型
                // 决定如何更新状态值
                switch (action.type) {
                    case: "counter/incremented":
                        return { ...state, value: state.value + 1 };
                    case: "counter/decremented":
                        return { ...state, value: state.value - 1 };
                    default:
                        // 如果 “reducer” 不关心这个 “action”类型,
                        // 原样返回现有状态
                        return state;
                }
            }

            // 使用 “createStore”函数创建一个新的Redux store
            // 并且 使用 “counterReducer” 来进行逻辑更新
            const store = Redux.createStore(counterReducer);

            const valueEl = document.querySelector('#value');

            // 每当store状态发生变化时,通过以下方式更新UI
            // 读取最新的存储状态并显示新数据
            function render() {
                const state = store.getState();
                valueEl.innerHTML = state.value.toString();
            }

            // 使用初始化状态更新UI
            render();

            // 订阅, 将来状态发生变化时重绘
            store.subscribe(render);

            // 通过“分派”操作对象处理用户输入,
            // 描述应用程序中发生了什么
            document.querySelector('#increment')
                .addEventListener('click', function() {
                    store.dispatch({ type: 'counter/incremented' });
                });

            document.querySelector('#decrement')
                .addEventListener('click', function() {
                    store.dispatch({ type: 'counter/decremented'})
                });

            document.querySelector('#incrementIfOdd')
                .addEventListener('click', function() {
                    // 我们可以根据状态来编写逻辑
                    if(store.getState().value % 2 !== 0) {
                        store.dispatch({ type: 'counter/incremented' })
                    }
                });

            document.querySelector('#incrementAsync')
                .addEventListener('click', function() {
                    // 我们也可以编写与store交互的异步逻辑
                    setTimeout(function() {
                        store.dispatch({ type: 'counter/incremented' });
                    }, 1000)
                });

        </script>
    </body>
</html>
```

因为Redux是一个独立的JS库, 没有依赖关系,所以这个例子是通过只为Redux库加载单个脚本标签来编写的, 并且使用基本的JS和HTML来编写UI.在实践中,通常通过从`NPM 安装Redux`包来使用Redux,并且使用像[React](https://reactjs.org/)这样的库来创建UI.

> ## 信息
>
> 第5部分: UI和React展示了如何一起使用Redux和React.
>
让我们把这个例子分解成不同的部分,看看发生了什么.

State, Actions, 和 Reducers

我们首先定义一个初始状态值来描述应用程序:

```javascript
// 定义应用程序的初始状态值
const initialState = {
    value: 0
}
```

对于这个应用程序,我们将使用计数器的当前值跟踪一个数字.

Redux 应用程序通常有一个JS对象作为状态的根部分,该对象内部还有其他值.

然后, 我们定义了一个**reducer**函数.reducer接收两个参数,当前参数`state`和`action`.当redux应用程序启动时,我们还没有任务状态,所以我们提供了initialState这个reducer的默认值:

```javascript
// 创建一个 “reducer” 函数来决定新的状态值
// 应该是应用程序发生一些什么时
function counterReducer(state = initialState, action) {
    // “reducer”通常会查看发生的 "action" 类型
    // 决定如何更新状态值
    switch (action.type) {
        case: "counter/incremented":
            return { ...state, value: state.value + 1 };
        case: "counter/decremented":
            return { ...state, value: state.value - 1 };
        default:
            // 如果 “reducer” 不关心这个 “action”类型,
            // 原样返回现有状态
            return state;
    }
}
```
action对象总是有一个`type`字段,它是您提供的一个字符串,用作动作的唯一名称.应该是一个可读的`type`名称,以便查看此代码的任何人都了解它的含义.在这种情况下,我们使用单词“counter”作为我们action type的前半部分,后半部分是对“发生了什么的描述”.在这种情况下,我们的“计算器”是“递增的”,所以我们将action type写为`'counter/incremented'`.


根据action type,我们要么需要返回一个全新的对象作为新的`state`结果,要么返回现有的`state`对象, 当然这发生在type 匹配不上,没有任何变化时.
请注意,我们通过复制现有状态并更新副本来不可变地更新状态,而不是直接修改原始对象.

## Store

现在我们有了一个reducer函数,我们可以通过调用Redux库API创建一个store实例.`createStore`
```javascript
// 使用 “createStore”函数创建一个新的Redux store
// 并且 使用 “counterReducer” 来进行逻辑更新
const store = Redux.createStore(counterReducer);
```
我们将reducer函数传递给createStore,它使用reducer函数生成初始状态,并计算任何未来的更新.

## UI

在任何应用程序中,用户界面都会在屏幕上显示现有状态,当用户执行某项操作时,应用程序将更新其数据,然后使用这些值重新绘制UI.
```javascript
const valueEl = document.querySelector('#value');

// 每当store状态发生变化时,通过以下方式更新UI
// 读取最新的存储状态并显示新数据
function render() {
    const state = store.getState();
    valueEl.innerHTML = state.value.toString();
}

// 使用初始化状态更新UI
render();

// 订阅, 将来状态发生变化时重绘
store.subscribe(render);
```
在这个小例子中,我们只使用了一些基本的HTML元素作为我们的UI,其中一个`<div>`显示当前值.

因此,我们编写了一个函数,该函数知道如何使用该`store.getState()`方法从Redux存储中获取最新状态,然后获取该值并更新UI以显示它.

Redux store允许我们调用`store.subscribe()`并传递订阅者回调函数,该函数将在每次更新store时调用. 因此, 我们可以将我们render函数作为订阅者回调传递,并且知道每次store更新时,我们都可以将UI更新为最新的值.

Redux 本身是一个可以在任何地方使用的独立库.这也意味着它可以与任何UI层一起使用.

## Dispatching Actions

最后, 我们需要通过创建描述所发生事件的操作对象并将他们分派到store来响应用户输入.当我们调用`store.dispatch(action)`时,store 运行reducer,计算更新状态,并运行订阅者更新UI.
```javascript
 // 通过“分派”操作对象处理用户输入,
// 描述应用程序中发生了什么
document.querySelector('#increment')
    .addEventListener('click', function() {
        store.dispatch({ type: 'counter/incremented' });
    });

document.querySelector('#decrement')
    .addEventListener('click', function() {
        store.dispatch({ type: 'counter/decremented'})
    });

document.querySelector('#incrementIfOdd')
    .addEventListener('click', function() {
        // 我们可以根据状态来编写逻辑
        if(store.getState().value % 2 !== 0) {
            store.dispatch({ type: 'counter/incremented' })
        }
    });

document.querySelector('#incrementAsync')
    .addEventListener('click', function() {
        // 我们也可以编写与store交互的异步逻辑
        setTimeout(function() {
            store.dispatch({ type: 'counter/incremented' });
        }, 1000)
    });
```
在这里, 我们将调度使reducer从当前计数器值加1或减1的操作.

我们还可以编写仅在某个条件为真时才调度action的代码,或者编写一些延迟后调度动作的异步代码.

## Data Flow

我们可以通过用这张图总结Redux应用程序的数据流.它代表如何:

* action 是 响应用户交互(如点击)而调度的
* store运行reducer函数来计算新状态
* UI 读取最新状态展现新值

(如果这些部分还不是很清楚,请不要担心!在阅读本教程的其余部分时,请记住这张图片,您会看到这些部分是如何组合在一起的.)

![Redux 数据流图](https://redux.js.org/assets/images/ReduxDataFlowDiagram-49fa8c3968371d9ef6f2a1486bd40a26.gif)

# 你学到了什么

那个counter 例子🌰 是很小的, 但它确实展示了一个真正的Redux应用程序的所有工作部分,**我们将在以下部分中讨论的所有内容都扩展了这些基础部分**

考虑到这一点,让我们回顾一下到目前为止我们学到了什么:
> ### 概括
>- **Redux是一个用于管理全局应用程序状态的库**
>   - Redux通常与React-redux库一起使用,用户将Redux和React集成在一起
>   - Redux Toolkit 是编写Redux逻辑的推荐方式
>- **Redux使用了几种类型的代码**
>   - action是带有`type`字段的普通对象,并在应用程序中描述“发生了什么”
>   - Reducers是根据先前状态+action计算新状态值的函数
>   - 每当分派操作时,Redux存储都会运行 root reducer