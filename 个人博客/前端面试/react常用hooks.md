# useCallback
useCallback可以缓存函数。
## 参考
const cacheFn = useCallback(fn,dependencies);

## 参数[](https://www.imywh.com/blog/6454d9acd25a316a6a815f14#%E5%8F%82%E6%95%B0)

fn：要缓存的函数值。它可以接受任何参数并返回任何值。React会在初始化渲染期间将你的函数返回

（而不是调用！！！）。在下一次渲染时，如果自上次渲染以来没有改变，React将提供相同的函数。

dependencies：代码中引用的所有变量。包括props、state以及在组件主体中声明的所有变量和函

数。

## 应用场景[](https://www.imywh.com/blog/6454d9acd25a316a6a815f14#%E5%BA%94%E7%94%A8%E5%9C%BA%E6%99%AF)

父组件传递一个函数给子组件，当父组件中其他state或者props改变时，传递给子组件所依赖的

state/props并没有发生改变时，就不用重新渲染子组件。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/260a8800663946d080e2401747f71e02~tplv-k3u1fbpfcp-watermark.image?)
**代码示例**

```js
//子组件：ShippingForm
import React from "react";

interface IProps {
    onClick: () => void;
}

const ShippingForm: React.FC<IProps> = React.memo((props) => {
    console.log("ShippingForm渲染");
    return <>
        <button onClick={props.onClick}>子组件按钮</button>
    </>
});

export default ShippingForm;

//父组件：App
import { useCallback, useState } from "react";
import ShippingForm from "./components/ShippingForm";

function App() {
    console.log('父组件渲染');
    const [count, setCount] = useState(0);
    const [otherCount, setOtherCount] = useState(100);

    const handleShippingClick = useCallback(() => {
        setCount(count + 1);
    }, [count]);
  
    /**
    * 如果改为这样，每次otherCount改变时，ShippingForm就会重新渲染
    */
     const handleShippingClick = () => {
        setCount(count + 1);
    },;

    const parentClick = () => {
        setOtherCount(otherCount + 100);
    }

    return (
        <div className="App">
            <h1>父组件Count：{otherCount}</h1>
            <button onClick={parentClick}>父组件按钮</button>
            <h1>子组件Count: {count}</h1>
            <ShippingForm onClick={handleShippingClick}></ShippingForm>
        </div>
    );
}

export default App;
```
# useMemo[](https://www.imywh.com/blog/6454d9acd25a316a6a815f14#useMemo)

## 应用场景[](https://www.imywh.com/blog/6454d9acd25a316a6a815f14#%E5%BA%94%E7%94%A8%E5%9C%BA%E6%99%AF-)

-   重新渲染之间缓存计算结果。
-   跳过昂贵的重新计算。
-   跳过组件的重新渲染。
-   记住另一个钩子的依赖关系。
-   记忆函数。

## 参考[](https://www.imywh.com/blog/6454d9acd25a316a6a815f14#%E5%8F%82%E8%80%83-)

const cachedValue = useMemo(calculateValue, dependencies)

## 参数[](https://www.imywh.com/blog/6454d9acd25a316a6a815f14#%E5%8F%82%E6%95%B0-)

calculateValue：计算要缓存的值的函数，她应该是纯函数，应该不带任何参数，并且应该返回任何类型的值。React将在初始渲染期间调用你的函数。在下一次渲染时，如果上次渲染以来没有更改，React将再次返回相同的值，否则，他将调用，返回其结果，并存储它以便以后可以重用。

dependencies：代码中引用的所有反应值的列表。反应值包括 props、state 以及直接在组件主体中声明的所有变量和函数。

## 对比有无useMemo的效率(for循环一亿次)

```js
//App.tsx
import { useCallback, useState } from "react";
import MemoComp from "./component/MemoComp";

function App() {

    const [count, setCount] = useState(1);
    const [age, setAge] = useState(18);

    const handleCount = useCallback(() => {
        setCount(count + 1);
    }, [count]);

    const handleAge = useCallback(() => {
        setAge(age + 1);
    }, [age]);

    return (
        <div className="App">
            <button onClick={handleCount}>改变count</button>
            <button onClick={handleAge}>改变age</button>
            <MemoComp count={count} age={age}></MemoComp>
        </div>
    );
}

export default App;

//MemoComp.tsx
import { useMemo } from "react";

interface IProps {
    count: number,
    age: number
}

const MemoComp: React.FC<IProps> = (props) => {
    const startTime = Date.now();
    const doubleCount = useMemo(() => {
        let result = 0;
        for(let i = 0; i < 100000000; i++) {
            result += i;
        }
        return result + props.count;
    }, [props.count]);
    console.log('结果：', doubleCount);
    console.log('耗时：', Date.now() - startTime);

    return <h1>年龄：{props.age}</h1>
}

export default MemoComp;
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/005fcf3e42c847f69d342d33a835f1f7~tplv-k3u1fbpfcp-watermark.image?)

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/19ab18e5a41a4b809910d5d8b498950e~tplv-k3u1fbpfcp-watermark.image?)

```js
//不使用 useMemo 保存计算结果
const doubleCount = () => {
    let result = 0;
    for(let i = 0; i < 100000000; i++) {
        result += i;
    }
    return result + props.count;
};
```

# useContext
## 应用场景[](https://www.imywh.com/blog/6454d9acd25a316a6a815f14#%E5%BA%94%E7%94%A8%E5%9C%BA%E6%99%AF--)

组件间的数据传递（超过三层级以上推荐使用），如下图父组件要传递一个{name: 'hello', age: 18}这样一个对象到孙组件。如果不通过层层传递，如何将数据传递到孙组件。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8eda8e9ebf0643d4bbc3eeef085617a5~tplv-k3u1fbpfcp-watermark.image?)
## 代码示例

```js
// App.tsx
import { createContext, useState } from "react";
import ChildrenComp_1 from "./component/ChildrenComp_1";

interface IPerson {
    name: string;
    age: number
}

export const DataContext = createContext<IPerson | undefined>(undefined);

function App() {

    const [person, setPerson] = useState<IPerson | undefined>({
        name: 'hello',
        age: 18
    });

    return (
        <div className="App">
            <DataContext.Provider value={person}>
                <ChildrenComp_1></ChildrenComp_1>
            </DataContext.Provider>
        </div>
    );
}

export default App;

//ChildrenComp_1.tsx
import ChildrenComp_2 from "../ChildrenComp_2";

const ChildrenComp_1: React.FC = () => {
    return <>
        <h1>ChildrenComp_1</h1>
        <ChildrenComp_2></ChildrenComp_2>
    </>
}

export default ChildrenComp_1;

//ChildrenComp_2.tsx
import { useContext } from "react";
import { DataContext } from "../../App"

const ChildrenComp_2: React.FC = () => {

    const data = useContext(DataContext);

    return <>
        <h1>ChildrenComp_2</h1>
        <h2>name：{data?.name}</h2>
        <h2>age：{data?.age}</h2>
    </>
}

export default ChildrenComp_2;
```
# useEffect
## 应用场景[](https://www.imywh.com/blog/6454d9acd25a316a6a815f14#%E5%BA%94%E7%94%A8%E5%9C%BA%E6%99%AF---)

-   组件挂载的时候运行某些函数。
-   组件卸载的时候运行某些函数。
-   组件依赖的props/state改变时运行某些函数。

**参考**

useEffect(setup, dependencies)

**参数**

-   setup：具有效果逻辑的函数。设置函数还可以选择 return 清理函数。当你的组件第一次被添加到DOM中时，React将运行你的setup函数。每次依赖更改后会重新渲染，React将首先使用旧值运行清理函数，然后使用新值运行您的设置函数。从DOM中删除组件后，React会最后一次运行清理函数。
-   dependencies：代码中引用的所有反应值的列表。反应值包括props、state以及在组件主体中声明的所有变量和函数。

**dependencies区别**

-   如果dependencies不传。
-   如果dependencies为空数组[]。
-   如果dependencies传递依赖项。


```js
import { useEffect, useState } from "react";

const EffectComp:React.FC = () => {

    const [count, setCount] = useState(0);

    const handleClick = () => {
        setCount(count + 1);
    }
    //如果useEffect dependencies不传，当组件挂载时会运行setup函数和清理函数，每次状态改变，先运行清理函数再运行setup函数，组件卸载的时候运行最后一次清理函数。
    useEffect(() => {
        console.log("函数运行");
        return () => {
            console.log("清理函数运行");
        }
    });

    return <>
        <h1>EffectComp：{count}</h1>
        <button onClick={handleClick}>增加按钮</button>
    </>
}

export default EffectComp;
```
# useLayoutEffect
useLayoutEffect是useEffect的一个版本，在浏览器重新绘制屏幕之前触发。换句话说，会阻止浏览器绘制。

**应用场景**

大多数组件不需要知道它们在屏幕上的位置和大小来决定要呈现的内容。它们只返回一些 JSX。然后浏览器计算它们的布局（位置和大小）并重新绘制屏幕。有时，这还不够。想象一下，悬停时出现在某个元素旁边的工具提示。如果有足够的空间，工具提示应显示在元素上方，但如果不适合，则应显示在下方。为了在正确的最终位置呈现工具提示，您需要知道它的高度（即它是否适合顶部）。

为此，您需要分两次渲染：

1.  在任意位置呈现工具提示（即使位置错误）。
1.  测量其高度并决定放置工具提示的位置。
1.  在正确的位置再次呈现工具提示。

所有这些都需要在浏览器重新绘制屏幕之前发生。您不希望用户看到工具提示在移动。调用以在浏览器重新绘制屏幕之前执行布局测量：useLayoutEffect

**参考**

useLayoutEffect(setup, dependencies?)

# useImperativeHandle[](https://www.imywh.com/blog/6454d9acd25a316a6a815f14#useImperativeHandle)

**应用场景**

-   向父组件暴露一个自定义的ref句柄
-   暴露组件本身的方法

**参考**

useImperativeHandle(ref, createHandle, dependebcies?)

```js
//ImperativeHandleComp组件，向父组件暴露handleFocus方法
import { forwardRef, useImperativeHandle, useRef } from "react";

export interface ImperativeHandleCompRef {
    handleFocus: () => void;
}

const ImperativeHandleComp = forwardRef<ImperativeHandleCompRef>((props, ref) => {
    const inputRef = useRef<HTMLInputElement>(null);

    const handleFocus = () => {
        inputRef.current?.focus();
    }

    useImperativeHandle(ref, () => {
        return {
            handleFocus
        }
    }, []);

    return <>
        <h1>这是ImperativeHandleComp组件</h1>
        <input type="text" placeholder="请输入" ref={inputRef} />
    </>
});

export default ImperativeHandleComp;

//父组件
import { useRef } from "react";
import ImperativeHandleComp, { ImperativeHandleCompRef } from "./component/ImperativeHandleComp";

function App() {

    const ref = useRef<ImperativeHandleCompRef>();

    const hanldeFocus = () => {
        ref.current?.handleFocus();
    }

    return (
        <div className="App">
            <ImperativeHandleComp ref={ref}></ImperativeHandleComp>
            <button onClick={hanldeFocus}>聚焦</button>
        </div>
    );
}

export default App;
```

# useRef[](https://www.imywh.com/blog/6454d9acd25a316a6a815f14#useRef)

**应用场景**

拿到组件的实例

**参考**

const ref = useRef(initialValue)

**返回值**

current：最初，它被设置为你传递的 。之后你可以把它设置为其他值。如果你把 ref 对象作为一个 JSX 节点的 属性传递给 React，React 将为它设置 属性。

**注意事项**

除了 [初始化](https://react.docschina.org/reference/react/useRef#avoiding-recreating-the-ref-contents)外不要在渲染期间写入 或者读取 。这会使你的组件的行为不可预测。

# **useState**[](https://www.imywh.com/blog/6454d9acd25a316a6a815f14#useState)

**参考**

const [state, setState] = useState(initialState);

约定是命名状态变量，例如使用[数组解构。](https://javascript.info/destructuring-assignment)[something, setSomething]

**基于以前的状态更新状态**


```js
function handleClick() {
  setAge(age + 1); // setAge(42 + 1)
  setAge(age + 1); // setAge(42 + 1)
  setAge(age + 1); // setAge(42 + 1)
}

function handleClick() {
  setAge(a => a + 1); // setAge(42 => 43)
  setAge(a => a + 1); // setAge(43 => 44)
  setAge(a => a + 1); // setAge(44 => 45)
}
```
**更新处于状态的对象和数组**

您可以将对象和数组置于状态。在 React 中，state 被认为是只读的，所以你应该替换它而不是改变你现有的对象。例如，如果对象处于状态，请不要改变它：


```js
const [form, setForm] = useState({
    name: 'hello',
    age: 18
});

//错误示例：不能直接改变对象，form.name = 'world';
//正确示例：
setForm({
  ...form,
  name: 'world'
});
```
# 总结[](https://www.imywh.com/blog/6454d9acd25a316a6a815f14#%E6%80%BB%E7%BB%93)

这就是本人在开发中常用的react hooks。

欢迎访问个人博客：https://www.imywh.com