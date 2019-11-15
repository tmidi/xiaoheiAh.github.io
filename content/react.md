---
title: "react"
date: 2019-08-15T13:19:18+08:00
draft: true
---

#### render

render可以返回一个嵌套结构的视图，更形象些的是: `render`返回的是你对你想要渲染内容的描述，react会根据你的描述将对应的内容渲染出来。



#### props传递数据
```js
//返回一个square对象，传入两个参数
class Board extends React.Component {
    renderSquare(i) {
        return <Square value={i} />;
    }
}
```

```js
class Square extends React.Component {
  render() {
    return (
      <button className="square">
        {this.props.value}
      </button>
    );
  }
}
```

