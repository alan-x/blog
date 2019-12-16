### 小心匿名函数
到处都是匿名函数是一种痛苦。他们很难调试，维护，测试，或者重用。使用对象字面量去组织和命名你的处理器和回调。
```
// BAD
$( document ).ready(function() {
 
    $( "#magic" ).click(function( event ) {
        $( "#yayeffects" ).slideUp(function() {
            // ...
        });
    });
 
    $( "#happiness" ).load( url + " #unicorns", function() {
        // ...
    });
 
});
 
// BETTER
var PI = {
 
    onReady: function() {
        $( "#magic" ).click( PI.candyMtn );
        $( "#happiness" ).load( PI.url + " #unicorns", PI.unicornCb );
    },
 
    candyMtn: function( event ) {
        $( "#yayeffects" ).slideUp( PI.slideCb );
    },
 
    slideCb: function() { ... },
 
    unicornCb: function() { ... }
 
};
 
$( document ).ready( PI.onReady );
```