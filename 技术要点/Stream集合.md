## 1. 创建流
```java
//collection.stream()和collection.parallelStream()，集合实例调用实例方法获取stream
List<String> list = new ArrayList<>();
Stream<String> stream = list.stream(); //获取顺序流
Stream<String> parallelStream = list.parallelStream(); //获取并行流

//Arrays.stream()，数组调用Arrays.stream()静态方法获取stream
Integer[] nums = new Integer[10];
Stream<Integer> stream = Arrays.stream(nums);

//调用Stream中的静态方法
//Stream.of()
Stream<Integer> ofStream = Stream.of(1,2,3,4,5,6);
Stream<String> ofStream = Stream.of("a", "b", "c", "d", "e", "f");
Stream<Bubble> ofStream = Stream.of(new Bubble(1), new Bubble(2), new Bubble(3), new Bubble(4), new Bubble(5), new Bubble(6));

//Stream.iterate()，iterate中第一个参数可以理解为seed种子，seed被作为参数代入iterate方法中的第二个参数lambda中，可以看出是无限流，最后需要limit作限制
Stream<Integer> iterateStream = Stream.iterate(0, (n) -> n + 2).limit(10);

//Stream.generate()，可以理解为和Stream.iterate差不多，也是无限流，只是参数形式不太一样
Stream<Double> generateStream = Stream.generate(Math::random).limit(2);

//bufferedReader.lines()，将读取的所有行内容转为流
BufferedReader reader = new BufferedReader(new FileReader("..."));
Stream<String> linesStream = reader.lines();

//pattern.splitAsStream()将字符串分隔成流
Pattern pattern = Pattern.compile(";");
Stream<String> splitStream = pattern.splitAsStream("a;b;c;d");
```

## 2. 流操作
```java
Stream<Integer> stream = Stream.of(1,2,3,4,5,6,7,8,9,10);

//filter()，按照filter中的lambda参数过滤出符合条件的stream元素集合
stream.filter(num -> num >= 5);

//limit(n)，获取n个元素的stream元素集合
stream.limit(2);

//skip(n)，跳过n个元素，配合limit(n)能实现分页
stream.skip(10);

//distinct()，根据流中元素的hashCode()和equals()去除重复元素
stream.distinct()

//map()，接受一个lambda方法，该方法被应用到每个元素上，可以修改原本stream中的每个元素
stream.map(i -> i + 2);

//flatMap()，接受一个lambda方法，该方法被应用到每个元素上，并且必须返回stream，flatMap会把lambda得到的多个stream合并成一个大stream
stream.flatMap(it -> it.getList().stream());

//sorted()，自然排序，流中元素需实现Comparable接口
stream.sorted()
//sorted(Comparator com)，流中元素按照Comparator排序器排序
stream.sorted((o1, o2) -> {return o1.getField() - o2.getField()})

//全部元素符合条件才返回true
allMatch
//全部元素都不符合条件才返回true
noneMatch
//只要有一个元素符合条件才返回true
anyMatch
//返回流中第一个元素
findFirst
//返回流中任意元素
findAny
//流中元素总个数
count
//流中元素最大值
max
//流中元素最小值
min
```

## 3. 流生成
```java
stream.collect()
//stream流中的元素进行汇总，转成集合
Collectors.toList()
Collectors.toSet()
Collectors.toCollection()
Collectors.toMap()
Collectors.collectingAndThen()
Collectors.joining()
Collectors.counting()
Collectors.summarizingDouble() Collectors.summarizingLong() Collectors.summarizingInt()
Collectors.averagingDouble() Collectors.averagingLong() Collectors.averagingInt()
Collectors.summingDouble() Collectors.summingLong() Collectors.summingint()
Collectors.maxBy() Collectors.minBy()
Collectors.groupingBy()
Collectors.partitioningBy()
```