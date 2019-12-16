### 循环的时候缓存长度
在一个`for`循环中，不要每次都去访问数据的长度属性，在这之前缓存它
```
var myLength = myArray.length;
 
for ( var i = 0; i < myLength; i++ ) {
 
    // do stuff
 
}
```