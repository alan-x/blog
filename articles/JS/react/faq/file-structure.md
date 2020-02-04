### 文件结构
**有一个推荐的 React 项目结构吗？**

React 对你如何放置文件到文件夹没有啥一件。有一些常见的方式在生态系统中，你可以会想要了解。


### 根据特性或者路由分组

组织项目的一个常见方式是将 CSS，JS，和测试根据功能或者路由放在同一个文件夹。
```jsx harmony
common/
  Avatar.js
  Avatar.css
  APIUtils.js
  APIUtils.test.js
feed/
  index.js
  Feed.js
  Feed.css
  FeedStory.js
  FeedStory.test.js
  FeedAPI.js
profile/
  index.js
  Profile.js
  ProfileHeader.js
  ProfileHeader.css
  ProfileAPI.js
```
"特性"的定义不是普遍的，这取决于你选择的粒度。如果你不能提取一系列的顶级文件夹，你可以询问你产品的用户，它由哪几部分组成，并且使用他们的心智模型作为蓝图。

### 根据文件类型分组
另一个组织项目流行的方式是分组类似的文件，比如：
```jsx harmony
api/
  APIUtils.js
  APIUtils.test.js
  ProfileAPI.js
  UserAPI.js
components/
  Avatar.js
  Avatar.css
  Feed.js
  Feed.css
  FeedStory.js
  FeedStory.test.js
  Profile.js
  ProfileHeader.js
  ProfileHeader.css
```

一些人更倾向于更加深入，并且分离组件到不同的文件夹，依赖于他们在应用中的角色。比如，[Atomic Design]()是基于这个原则构建的设计原则。记住对待这类方法论为有帮助的例子，而不是严格的规则去遵守更有帮助。


### 避免太多嵌套

JavaScript 项目中太深的文件夹嵌套有很多痛点。编写相对引入变得很困难，或者更新引入当文件被移动的时候。除非你有非常令人信服的理由去使用深层文件夹结构，考虑在一个项目中限制你自己去最多三到四层嵌套文件夹。当然，这只是一个建议，这可能和你的项目无关。

### 不要过分思考

如果你刚开始一个项目，[不要花费超过五分钟]()在选择一个文件结构。选择前面的任意一个方式（或者使用你自己的）并开始编码！你可能想要重新思考它，在你编写了一些真实代码之后。


如果你觉得完全被困住了，先保持所有的文件到单一的文件夹。最终它将会变得足够大，你讲想要从其他文件中分离一些文件。到那时，你将会有足够的认知去知道哪些文件你最经常在一起编辑。通常，保持常常一起改变的文件距离更近。这个原则叫做"colocation"。

随着项目变大，在实践中他们通常使用混合使用前面的两种方式。因此，选择"对的"一个通常不是非常重要。





















