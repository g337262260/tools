## 组件生命周期函数

### 1. 组件实例化阶段

- defaultProps:

  设置组件的初始值，例如color,width等，可以通过this.props获取相应的值

- constructor（props）:

  这里通过this.props可以获取defaultProps设置的默认属性值，同时这里用于初始化控件的可变化的变量，通过this.state设置变量的初始值，通过this.setState()函数修改变量的值，调用render()函数重新渲染页面，得到新的页面

- componentWillMount:

  组件将要被加载到视图之前调用

- render:

  第一次调用，用于渲染页面

- componentDidMount:

  在调用了render方法，组件加载完成并被成功渲染出来之后，所要执行的后续操作，一般都会在这个函数中进行，比如经常要面对的网络请求等加载数据操作，因为UI渲染是异步的，所以在这个函数里面进行网络请求，能够避免出现UI错误。 

### 2.组件运行时阶段

​	组件的属性prop和状态state任何一个改变都可能会触发render()函数渲染页面 

- componentWillReceiveProps:

  指父元素对组件的props进行了修改 

- shouldComponentUpdate:

  一般用于优化性能，通过业务逻辑判断返回true或false，来决定页面是否进行重新绘制，默认返回true,执行后面两个周期函数 

- componentWillUpdate:

  组件刷新钱调用

- componentDidUpdate:

  组件更新后调用

### 3.页面卸载阶段

- componentWillUnmount

  一般用于清理工作，移除事件监听，取消定时器等。。

