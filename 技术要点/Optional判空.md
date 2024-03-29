```java
Optional.empty() //返回一个空的Optional

optional.isEmpty() //判断Optional是否为空，为空则返回true

Optional.of(obj) //返回一个非空的Optional，如果obj为空，则报错NullPointException

Optional.ofNullable(obj) //返回Optional，不管obj是否为空

optional.get() //返回Optional之前的obj值，如果不存在则报错NoSuchElementException

optional.isPreSent() //optional存在值则返回true，否则为null返回false

optional.ifPresent(arg -> {}) //optional存在值则调用其中的方法参数

optional.orElse(obj) //optional存在值则返回，否则返回orElse中的obj

optional.orElseGet(arg -> {}) //optional存在值则返回，否则调用orElseGet中的方法参数

optional.orElseThrow(arg -> {}) //optional存在值则返回，否则调用orElseThrow中的方法参数抛出异常

optional.map(arg -> {}) //根据方法参数中的返回值，返回一个对应的optional，即使用optional封装一遍属性

optional.flatMap(arg -> {}) //根据方法参数中的返回值，返回一个对应的optional，和map不同的是flatMap返回的对象如果已经是optional类型的，则不会再用optional封装一遍

optional.filter(arg -> {}) //方法参数返回true or false，如果返回true则返回一个带值的optional，否则返回一个空的optional
```