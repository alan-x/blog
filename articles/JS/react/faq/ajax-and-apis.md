### Ajax and APIs

### 如何创建一个 AJAX 调用？

你可以使用任何你喜欢的 AJAX 库和 React 一起用。比较流行的是 [Axios]()，[jQuery AJAX]()，和浏览器内置的 [window.fetch]()。

### 应该在组件的哪一个生命周期创建 AJAX 调用？

你应该在 [componentDidMount]() 生命周期方法调用 AJAX 填充输入。这是你可以使用`setState`去更新你的组件，当数据获取到的时候。

### 例子：使用 AJAX 结果去设置本地状态

下面的组件显示如何在`componentDidMount`创建一个 AJAX 调用去更新本地组件状态。

下面例子 API 返回一个 JSON 对象：
```jsx harmony
{
  "items": [
    { "id": 1, "name": "Apples",  "price": "$2" },
    { "id": 2, "name": "Peaches", "price": "$5" }
  ] 
}
```

```jsx harmony
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      error: null,
      isLoaded: false,
      items: []
    };
  }

  componentDidMount() {
    fetch("https://api.example.com/items")
      .then(res => res.json())
      .then(
        (result) => {
          this.setState({
            isLoaded: true,
            items: result.items
          });
        },
        // Note: it's important to handle errors here
        // instead of a catch() block so that we don't swallow
        // exceptions from actual bugs in components.
        (error) => {
          this.setState({
            isLoaded: true,
            error
          });
        }
      )
  }

  render() {
    const { error, isLoaded, items } = this.state;
    if (error) {
      return <div>Error: {error.message}</div>;
    } else if (!isLoaded) {
      return <div>Loading...</div>;
    } else {
      return (
        <ul>
          {items.map(item => (
            <li key={item.name}>
              {item.name} {item.price}
            </li>
          ))}
        </ul>
      );
    }
  }
}
```
