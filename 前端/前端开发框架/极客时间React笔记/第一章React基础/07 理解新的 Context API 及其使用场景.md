# 理解 Context API 及其使用场景
React 16.3 新特性:Context API
**解决组件之间通信问题**

下面结点通过使用 Context API 获得共享一些全局状态

![](http://ww1.sinaimg.cn/large/006rAlqhly1g0n8jehuvij30ix08b405.jpg)
1. 根节点提供所有上下文数据
2. 其他节点通过 Context API 访问数据

```react
import React from "react";

// 英文字符串
const enStrings = {
  submit: "Submit",
  cancel: "Cancel"
};
// 中文字符串
const cnStrings = {
  submit: "提交",
  cancel: "取消"
};
const LocaleContext = React.createContext(enStrings);

// 切换语言的 button
class LocaleProvider extends React.Component {
  // 默认状态的 locale 为中文
  state = { locale: cnStrings };
  // 切换语言的方法
  toggleLocale = () => {
    const locale =
      this.state.locale === enStrings
        ? cnStrings
        : enStrings;
        // 设置刷新 locale 数据
    this.setState({ locale });
  };
  render() {
    return (
      <LocaleContext.Provider value={this.state.locale}>
        <button onClick={this.toggleLocale}>
          切换语言
        </button>
        {this.props.children}
      </LocaleContext.Provider>
    );
  }
}

// 展示组件
class LocaledButtons extends React.Component {
  render() {
    return (
      <LocaleContext.Consumer>
        {/* 函数作为子组件 */}
        {locale => (
          <div>
            <button>{locale.cancel}</button>
            &nbsp;<button>{locale.submit}</button>
          </div>
        )}
      </LocaleContext.Consumer>
    );
  }
}
// 导出组件
export default () => (
  <div>
    <LocaleProvider>
      <div>
        <br />
        <LocaledButtons />
      </div>
    </LocaleProvider>
    <LocaledButtons />
  </div>
);

```
> consumer 在 Provider 的下级
>
> 为什么不使用外部数据？ 外部数据并不像内部状态自动监听数据变化并刷新。



# 小结

1. Context API 的使用方法
2. 使用场景
