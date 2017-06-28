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

