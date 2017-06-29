---
title: React入门
date: 2017-06-23 17:13:02
tags:
---

# React learn

## React

React is a javascript library for building user interface.

## A Simple Component

React components implement a * render() * method that takes input data and return what to display.
Input data that is passed into the component can be accessed via * this.props *.

```javascript
class HelloMessage extends React.Component{
  render () {
    return (<div>Hello, {this.props.name}</div>)
  }
}
ReactDOM.render(
  <HelloMessage name="黄剑枫" />,
  document.getElementById('root')
)

```

## A Statefull Component

In addition to taking input data, a component can maintain internal state data (accessed via this.state).
When component's state data changes, the rendered markup will be updated by re-invoking * render() *.

```

class Timer extends React.Component {
  constructor () {
    super();
    this.state = {
       second: 0,
      timeInterval: null
    };
  }
  render () {
    return (
      <div>{this.state.second}</div>
    );
  }
  // 组件绑定时的钩子函数
  componentDidMount () {
    this.startTimeTick();
  }

  startTimeTick () {
    this.state.timeInterval =
      setInterval(() => {
        this.timeTick();
      }, 1000)
  }

  destroyTimeTick () {
    if (this.state.timeInterval) {
      clearInterval(this.state.timeInterval);
    }
  }

  timeTick () {
    this.setState({
      second: this.state.second + 1
    });
  }

  // 组件从DOM删除之前的钩子函数
  componentWillUnmount () {
    this.destroyTimeTick();
  }

}

ReactDOM.render(
  <Timer />,
  document.getElementById('root')
);

```
# A Application

Using props and state, we can put together a small Todo application.

```javascript

class TodoApp extends React.Component {
  constructor () {
    super();
    this.state = {
      items: [],
      text: ''
    };
  }
  render () {
    return (
      <div>
        <h1>TODO</h1>
        <TodoList items={this.state.items}/>
        <form onSubmit={(e) => {
            this.handleSubmit(e)
          }}>
          <input type="text" onChange={(e) => this.handleChange(e)} value={this.state.text}/>
          <button>Add #{this.state.items.length + 1}</button>
        </form>
      </div>
    )
  }

  handleSubmit (e) {
    e.preventDefault();
    const text = this.state.text;
    this.setState((prevState) => ({
      items: prevState.items.concat(text),
        text: ''
    }));
  }

  handleChange (e) {
    this.setState({
      text: e.target.value
    })
  }
}

class TodoList extends React.Component {
  render () {
    return (
      <ul>
        {
          this.props.items.map((item) => {
            return <li>{item}</li>
          })
        }
       </ul>
    );
  }
}
ReactDOM.render(
  <TodoApp />,
  document.getElementById('root')
);
```

