---
title: markdown编辑器(React)
date: 2017-06-29 14:37:02
tag:
---

*dangerouslySetInnerHTML* is React's replacement for using innerHTML in the browser DOM.
You can set HTML directly from React, but you have to type out *dangerouslySetInnerHTML* and pass an object with a *__html* key, to remind yourself that is dangerous.

[codepen地址](https://codepen.io/suifengfengye/pen/MorPBY)

```
class MarkdownEditor extends React.Component {
  constructor () {
    super();
    this.state = {
      text: ''
    };
  }
  render () {
    return (
      <div className="markdown-editor">
        <p>Input</p>
        <textarea onChange={(e) => {this.handleChange(e)}}></textarea>
        <p>Output</p>
        <div dangerouslySetInnerHTML={this.getRawMarkup()}></div>
      </div>
    );
  }
  handleChange (e) {
    this.setState({
      text: e.target.value
    });
  }

  getRawMarkup () {
    const md = new Remarkable();
    return { __html: md.render(this.state.text) }
  }

}


ReactDOM.render(
  <MarkdownEditor />,
  document.getElementById('root')
);
```