### 解绑元素后操作他们
`DOM`很慢；你会想要尽可能的避免操作他们。`jQuery`在`1.4`版本引入`detatch()`帮助解决这个问题，允许你操作他们的时候从`DOM`上移除元素。
```
var table = $( "#myTable" );
var parent = table.parent();
 
table.detach();
 
// ... add lots and lots of rows to table
 
parent.append( table );
```
