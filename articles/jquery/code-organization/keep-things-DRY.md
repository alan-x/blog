### 保持所有的东西`不要重复你自己`
不要重复你自己；如果你重复你自己，你就做错了。

```
// BAD
if ( eventfade.data( "currently" ) !== "showing" ) {
    eventfade.stop();
}
 
if ( eventhover.data( "currently" ) !== "showing" ) {
    eventhover.stop();
}
 
if ( spans.data( "currently" ) !== "showing" ) {
    spans.stop();
}
 
// GOOD!!
var elems = [ eventfade, eventhover, spans ];
 
$.each( elems, function( i, elem ) {
    if ( elem.data( "currently" ) !== "showing" ) {
        elem.stop();
    }
});
```