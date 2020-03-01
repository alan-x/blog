[原文地址](https://reactjs.org/docs/typechecking-with-proptypes.html)
### 使用 proptypes 类型检测

注意：从 React v15.5 开始`React.PropTypes`已经移到一个不同的包了。请使用[prop-types 库替代]()。

我们提供[一个 codemod 脚本]()去自动化这个约定。

随着你的应用的增长，你可以使用类型检测捕捉非常多的 bug。对于一些应用，你可以使用类似[Flow]()或者[TypeScript]()之类的
JavaScript 扩展去检测你的整个应用。但是就算你不使用这些，React 也有一些内置的类型检测能力。为了检测组件的属性，你可以赋值指定的`propTypes`属性：
```jsx harmony
import PropTypes from 'prop-types';

class Greeting extends React.Component {
  render() {
    return (
      <h1>Hello, {this.props.name}</h1>
    );
  }
}

Greeting.propTypes = {
  name: PropTypes.string
};

```
`PropTypes`导出一系列的验证器，可以用来确保你的接收的数据有效。在这个例子中，我们使用`PropTypes.string`。当提供了一个无效的属性值，JavaScript 控制台会显示一个警告。为了性能，`PropTypes`只在开发环境检测。

### PropTypes

这是记录提供的不同验证器的例子：
```jsx harmony
import PropTypes from 'prop-types';

MyComponent.propTypes = {
  // You can declare that a prop is a specific JS type. By default, these
  // are all optional.
  optionalArray: PropTypes.array,
  optionalBool: PropTypes.bool,
  optionalFunc: PropTypes.func,
  optionalNumber: PropTypes.number,
  optionalObject: PropTypes.object,
  optionalString: PropTypes.string,
  optionalSymbol: PropTypes.symbol,

  // Anything that can be rendered: numbers, strings, elements or an array
  // (or fragment) containing these types.
  optionalNode: PropTypes.node,

  // A React element.
  optionalElement: PropTypes.element,

  // A React element type (ie. MyComponent).
  optionalElementType: PropTypes.elementType,
  
  // You can also declare that a prop is an instance of a class. This uses
  // JS's instanceof operator.
  optionalMessage: PropTypes.instanceOf(Message),

  // You can ensure that your prop is limited to specific values by treating
  // it as an enum.
  optionalEnum: PropTypes.oneOf(['News', 'Photos']),

  // An object that could be one of many types
  optionalUnion: PropTypes.oneOfType([
    PropTypes.string,
    PropTypes.number,
    PropTypes.instanceOf(Message)
  ]),

  // An array of a certain type
  optionalArrayOf: PropTypes.arrayOf(PropTypes.number),

  // An object with property values of a certain type
  optionalObjectOf: PropTypes.objectOf(PropTypes.number),

  // An object taking on a particular shape
  optionalObjectWithShape: PropTypes.shape({
    color: PropTypes.string,
    fontSize: PropTypes.number
  }),
  
  // An object with warnings on extra properties
  optionalObjectWithStrictShape: PropTypes.exact({
    name: PropTypes.string,
    quantity: PropTypes.number
  }),   

  // You can chain any of the above with `isRequired` to make sure a warning
  // is shown if the prop isn't provided.
  requiredFunc: PropTypes.func.isRequired,

  // A value of any data type
  requiredAny: PropTypes.any.isRequired,

  // You can also specify a custom validator. It should return an Error
  // object if the validation fails. Don't `console.warn` or throw, as this
  // won't work inside `oneOfType`.
  customProp: function(props, propName, componentName) {
    if (!/matchme/.test(props[propName])) {
      return new Error(
        'Invalid prop `' + propName + '` supplied to' +
        ' `' + componentName + '`. Validation failed.'
      );
    }
  },

  // You can also supply a custom validator to `arrayOf` and `objectOf`.
  // It should return an Error object if the validation fails. The validator
  // will be called for each key in the array or object. The first two
  // arguments of the validator are the array or object itself, and the
  // current item's key.
  customArrayProp: PropTypes.arrayOf(function(propValue, key, componentName, location, propFullName) {
    if (!/matchme/.test(propValue[key])) {
      return new Error(
        'Invalid prop `' + propFullName + '` supplied to' +
        ' `' + componentName + '`. Validation failed.'
      );
    }
  })
};
```

### 要求单个子组件
使用`PropTypes.element`，你可以指定只能传递一个子组件到一个组件作为子组件。
```jsx harmony
import PropTypes from 'prop-types';

class MyComponent extends React.Component {
  render() {
    // This must be exactly one element or it will warn.
    const children = this.props.children;
    return (
      <div>
        {children}
      </div>
    );
  }
}

MyComponent.propTypes = {
  children: PropTypes.element.isRequired
};
```

### 默认属性值

你可以为你的`props`定义默认值，通过赋值指定的`defaultPropt`属性：
```jsx harmony
class Greeting extends React.Component {
  render() {
    return (
      <h1>Hello, {this.props.name}</h1>
    );
  }
}

// Specifies the default values for props:
Greeting.defaultProps = {
  name: 'Stranger'
};

// Renders "Hello, Stranger":
ReactDOM.render(
  <Greeting />,
  document.getElementById('example')
);
```

如果你使用像[transform-class-properties]()之类的 Babel 转化，你可以声明`defaultProps`为 React 组件类内的一个静态属性。这个语法还没最终通过，在浏览器内需要一个编译步骤。了解更多，查阅[类域提案]()。
```jsx harmony
class Greeting extends React.Component {
  static defaultProps = {
    name: 'stranger'
  }

  render() {
    return (
      <div>Hello, {this.props.name}</div>
    )
  }
}
```
这个`defaultProps`将会用于确保`this.props.name`将有一个值，如果它没有被父组件指定。`propTypes`类型检测发生在`defaultProps`被解决之前，因此，类型检测也会应用到`defaultProps`。
