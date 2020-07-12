#### 参考
* [React Hooks 系列之5 useCallback](https://juejin.im/post/5ec2465a5188256d841a552a)
* [详解 useCallback & useMemo](https://juejin.im/post/5e78884c6fb9a07c8679220d)

#### useCallback
1. useCallback的回调函数在组件每次render时都会被创建（不管依赖数组是否变化），useCallback的性能优化不体现于是否创建新函数。
2. 当依赖函数变化时，将返回新创建的内联函数，否则返回缓存的旧函数。在子组件使用React.memo（props值不变，则不重新render子组件）的情况下，useCallback可以避免“父组件需要重新render，但依赖数组不变”场景下，子组件的render。
3. 许多文章都有提到，创建内联函数是廉价的，这也不是useCallback致力于优化的点，考虑到inputs的新旧比较本身就有一定花销，与React.memo搭配才是使用useCallback的正确姿势。
4. 所有回调函数中引用的值都应该出现在依赖项数组中。


#### source code
```
// 将 callback 与 deps 放入缓存
function mountCallback<T>(callback: T, deps: Array<mixed> | void | null): T {
  const hook = mountWorkInProgressHook();
  const nextDeps = deps === undefined ? null : deps;
  hook.memoizedState = [callback, nextDeps];
  return callback;
}

// 组件 render 时执行 updateCallback：读取缓存，判断依赖数组是否变化，若变化，则返回新创建的内联函数 callback，否则返回缓存中的函数。返回结果下一步就被传递给子组件
function updateCallback<T>(callback: T, deps: Array<mixed> | void | null): T {
  const hook = updateWorkInProgressHook();
  const nextDeps = deps === undefined ? null : deps;
  const prevState = hook.memoizedState;
  if (prevState !== null) {
    if (nextDeps !== null) {
      const prevDeps: Array<mixed> | null = prevState[1];
      if (areHookInputsEqual(nextDeps, prevDeps)) {
        return prevState[0];
      }
    }
  }
  hook.memoizedState = [callback, nextDeps];
  return callback;
}
```


#### good cases
1. 分离-多个函数，传递给多个子组件
* 不使用 useCallback
```
export default function App() {
  const [count1, setCount1] = useState(0);
  const [count2, setCount2] = useState(0);

  const handleClickButton1 = () => {
    setCount1(count1 + 1);
  };

  const handleClickButton2 = () => {
    setCount2(count2 + 1);
  };
  const handleClickButton3 = () => {
    setCount3(count3 + 1);
  };

  return (
    <div>
      <div>
        <Button onClickButton={handleClickButton1}>Button1</Button>
      </div>
      <div>
        <Button onClickButton={handleClickButton2}>Button2</Button>
      </div>
       <div>
        <Button onClickButton={handleClickButton3}>Button3</Button>
      </div>     
    </div>
  );
}
```
```
const Button = ({ onClickButton, children }) => {
  console.log("Button render");
  return (
    <>
      <button onClick={onClickButton}>{children}</button>
      <span>{Math.random()}</span>
    </>
  );
};

export default React.memo(Button);
```
handleClickButton1 被触发后，会导致传递给三个组件的函数引用都发生变化。于是三个子组件都重新render。
* 使用 useCallback
```
export default function App() {
  const [count1, setCount1] = useState(0);
  const [count2, setCount2] = useState(0);

  const handleClickButton1 = useCallback(() => {
    setCount2(count1 + 1);
  }, [count1]);

  const handleClickButton2 = useCallback(() => {
    setCount2(count2 + 1);
  }, [count2]);
  
  const handleClickButton3 = useCallback(() => {
    setCount2(count3 + 1);
  }, [count3]);

  return (
    <div>
      <div>
        <Button onClickButton={handleClickButton1}>Button1</Button>
      </div>
      <div>
        <Button onClickButton={handleClickButton2}>Button2</Button>
      </div>
       <div>
        <Button onClickButton={handleClickButton3}>Button3</Button>
      </div>     
    </div>
  );
}
```
handleClickButton1 被触发后，只会导致传递给第一个子组件的引用发生变化。所以只有第一个子组件重新render了。这就是useCallback的好处。



#### bad cases
* 每次父组件 render 都会給子组件一个新的函数引用（trigger handleClickButton1->render App->update count1->recreate and return new function reference），与不用useCallback相比，还增加了比较新旧 inputs 的消耗。 
```
export default function App() {
  const [count1, setCount1] = useState(0);

  const handleClickButton1 = useCallback(() => {
    setCount1(count1 + 1);
  }, [count1]);

  return (
    <div>
      <div>
        <Button onClickButton={handleClickButton1}>Button1</Button>
      </div>   
    </div>
  );
}
```
