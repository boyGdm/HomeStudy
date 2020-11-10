#React面试常见的问题

##1、redux的中间件的原理是什么？
 dispatch函数的加强版（中间件的理解：是dispatch到store 的过程,dispatch是沟通的桥梁）。
 派发action -- store -- reducer -- store 。
action通常为一个对象，当我们需要传递一个函数时，需要使用中间件 。
```typescript jsx
// action
const getUserInfo = (id) => {
    return function (dispatch, getState, extraArgument){
        return reqGet({id: id})
        .then(res => res.json().data)
        .then(info => {
            dispatch({
                type: "GET_USER_INFO",
                info
            })
        })
        .catch(err => console.log('reqGet error: ' + err));
    }
};

// dispatch action
dispatch(getUserInfo(1));
```
##2、你会把数据统一放到redux中管理么。还是共享数据放在rdux中管理？
为了项目的共处啊管理，我会吧项目中用到的数据统一放到redux里面进行管理，数据放在state和props里面数据散乱不易维护。当某个数据出现问题时不易定位问题。
##3、componentWillReceiveProps的调用时机？
props发生变化的时候。
##4、react的性能优化
pureComponent 的使用。immulate.js的配合使用。主要是为了减少虚拟dom的渲染性能。React里面还有Memo函数的性质是差不多的。
###4.1、 函数组件怎么做性能优化？
函数式组件比普通组件性能高，因为它是一个函数，没有生命周期，相对于类来说，没有构造类的过程，所以它性能高。
但是因为它没有 shoudComoponentUpdate 这个生命周期，所以每当props改变时，函数就会重新执行。
性能优化：
可以通过 React.memo() 对函数式组件进行包装，然后再返回，包装后的函数式组件就带有了shouldComponentUpdate的特性。
```jsx
React.memo(function Test(props){
   return <div>123</div>
})
```
##5、虚拟dom是什么，为什么虚拟dom会提示代码的性能
其实我的理解是 虚拟dom就是一个JS对象的AST抽象树。
##6、webapck中，是借助loader完成的JSX代码的转化，还是babel?
React的ES6转化是babel里面的 - preset-react这个底层来进行转化的。
##7、调用setState后发生了什么？遇到过什么坑。
setState是个异步函数，要到达实时数据变化，只需要，给setState传一个函数。
```jsx
setState((preState) => ({count: preState++ }),() => {
    //dosomething  不要用setTimeout (^_^)
});
```
##8、refs的作用是什么，你在什么业务场景里面用到。  
作用是操作dom ， 图片加载完以后获取图片的宽高 [ window上添加事件监听后，组件销毁前需要移除 ]
ref 推荐使用箭头函数包一下。  
```jsx

<div className='demos' ref={(doms) => { this.element = doms }}></div>
```
```jsx
class Test extends React.Component {
  constructor(props){
    super(props);
    this.state = {
      top: 0
    }
    this.handleWindowScroll = this.handleWindowScroll.bind(this);
  }
 
  handleWindowScroll() {
    this.setState({
      top: document.body.scrollTop
    })
  }
 
  componentDidMount() {
    window.addEventListener('scroll', this.handleWindowScroll);
  }
 
  componentWillUnmount() {
    window.removeEventListener('scroll', this.handleWindowScroll);
  }
 
  render() {
    return <div>{this.state.top}</div>
  }
}
```

##9、高阶组件你是怎么理解的。他的本质是一个什么东西。
个人认为，包括官方也认为，在react中不要用继承，设计模式里面就有一种明确的说法是组合优于继承。
react这种组件式的编程方式，实际上是把一些组件合在一起，是一种组合式的设计模式，一定是优于继承的。所以它的可维护性要比继承好的多。

##10、受控组件和非受控组件的区别。
1、每当表单的状态发生变化时，都会被写入到组件的state中
2、在受控组件中，组件渲染出的状态与它的value或checked prop相对应
3、react受控组件更新state的流程
##11、函数组件和hooks。
### 11.1、 useState
```jsx
const [state, setState] = useState(initialState);
```
###11.2、useEffect
>useEffect接收两个参数，第一个是处理函数，第二是依赖 第一个参数相当于一旦创建就会执行。 如果第二个参数传入的依赖状态改变也会自动执行 如果第二个参数传入的依赖状态为空就相当于值会走一次 如果不传任何状态改变都会触发第一个处理函数的执行 return 出来的这个函数只有销毁的时候才会执行
```jsx
    useEffect(() => {}, [])
```
###11.3、useRef
```jsx
//引入
import React,{useRef} from 'react';
//创建
let newInput = useRef();
//挂载
return <div ref={myNewRef} >...</div>
//执行
useEffect(()=>{
    if(oldOrNew === true){
        newInput.current.focus();
    }
},[])
```
###11.4、 useMemo和useCallback的区别
 useMemo和useCallback都会在组件第一次渲染的时候执行，之后会在其依赖的变量发生改变时再次执行；并且这两个hooks都返回缓存的值，useMemo返回缓存的变量，useCallback返回缓存的函数。
###11.5、useSelector（useSelector用于从Redux存储的state中提取值并订阅该state。这基本上类似于在hooks中实现的mapStateToProps函数）
```jsx
// 以前
import React from 'react';
import { connect } from 'react-redux';

const Component = props => <div title={props.title}>{props.content}</div>;

export default connect(state => ({
  title: state.title, 
  content: state.content
}))(Component)


// 使用redux hooks
import React from 'react';
import { useSelector } from 'react-redux';

const Component = props => {
  const { title, content } = useSelector(state => ({
      title: state.title, 
      content: state.content
  }));
  
  return <div title={title}>{content}</div>;
```
###11.6 useDispatch （解决方案是在useCallback中创建这个匿名函数。）
```jsx
import React from 'react';
import { useCallback, useDispatch } from 'react-redux';
import { increaseCounterAction } from './actions';
import ExpensiveComponent from './ExpensiveComponent';

// 常用的方式
const Component = props => {
  const dispatch = useDispatch();
  return (
    <button onClick={() => dispatch(increaseCounterAction()}>
      Increase the Counter
    </button>
  )
}

// 使用useCallback优化性能
const Component = props => {
  const dispatch = useDispatch();
  const handleIncreaseCounter = useCallback(
    () => dispatch(increaseCounterAction()), 
    [dispatch]
  );
  
  return <ExpensiveComponent onClick={handleIncreaseCounter} />
}
```
###11.7、useStore（useStore用于获取创建的store实例）
```jsx
import React from 'react';
import { useStore } from 'react-redux';
import OtherProvider from './OtherProvider';

const Component = props => {
  const store = useStore();
  
  return <OtherProvider store={store}>{props.children}</OtherProvider>
}
```
##12、this的指向问题你一般都怎么解决的。
