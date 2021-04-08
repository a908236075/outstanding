# 第一部分

## 第一章

1.  Java从函数式编程引入的两个核心思想：将方法和 Lambda作为一等值，以及
   在没有可变共享状态时，函数或方法可以有效、安全地并行执行。
2.  只定义一个抽象方法的的接口被称为**函数式接口**;
3.  如果一个类上面标记有**@FunctionalInterface**,就说明他是一个函数式接口,编译器会进行检验,并报错.
4.  函数式接口中的任何一个都**不允许**抛出**受检异常**.

# 第二部分 

## 第四章

1. 常见的流操作

   - filter ——接受一个 Lambda，从流中排除某些元素。在本例中，通过传递 Lambda  d ->
     d.getCalories() > 300 ，选择出热量超过 300卡路里的菜肴。
     
   - map ——接受一个 Lambda，将元素转换成其他形式或提取信息。在本例中，通过传递方
     法引用 Dish::getName ，相当于 Lambda d -> d.getName() ，提取了每道菜的菜名。
     
   - limit ——截断流，使其元素不超过给定数量。

   - collect ——将流转换为其他形式。在本例中，流被转换为一个列表。

   - ~~~java
     List<String> names = menu.stream()
     .filter(dish -> dish.getCalories() > 300)
     .map(Dish::getName)
     .limit(3)
     .collect(toList());
     ~~~

   -  

2. 流只能消费一次

   1. ~~~java
      List<String> title = Arrays.asList("Modern", "Java", "In", "Action");
      Stream<String> s = title.stream();
      s.forEach(System.out::println);
      s.forEach(System.out::println);  //java.lang.IllegalStateException:
      ~~~

3. 使用流

   1. ~~~java
      // distinct 
      numbers.stream()
      .filter(i -> i % 2 == 0)
      .distinct()   // 它会返回一个元素各异
      .forEach(System.out::println);
      ~~~

   2.  不想用filter使元素排序并且只想知道,原始内容中第一个符合的元素,**takeWhile** 操作就是为此而生的！它可以帮助你利用谓词对流进行分片。更妙的是，它会在遭遇第一个不符合要求的元素时停止处理。(即将第一个不符合的元素之前的元素返回)

      1. ~~~java
         // takeWhile  切片
         List<Dish> slicedMenu1
         = specialMenu.stream()
         .takeWhile(dish -> dish.getCalories() < 320)
         .collect(toList());
         ~~~

   3. dropWhile 操作是对 takeWhile 操作的补充。它会从头开始，丢弃所有谓词结果为 false的元素。一旦遭遇谓词计算的结果为 true ，它就停止处理，并返回所有剩余的元素.

      1. ~~~java
         // dropWhile 切片
         List<Dish> slicedMenu2
         = specialMenu.stream()
         .dropWhile(dish -> dish.getCalories() < 320)
         .collect(toList());
         ~~~

   4. 截短流 limit 

   5. 跳过元素 skip

   6. 对流中每一个元素应用函数 map

   7.  flatMap 方法让你把一个流中的每个值都换成另一个流，然后把所有的流连接起来成为一个流。

      1. ~~~java
         List<String> uniqueCharacters =
         words.stream()
         .map(word -> word.split(""))
         .flatMap(Arrays::stream)
         .distinct()
         .collect(toList());
         ~~~

   8. 匹配 anyMatch,allMatch,noneMatch

      1. ~~~java
         // allMatch
         boolean isHealthy = menu.stream()
         .allMatch(dish -> dish.getCalories() < 1000);
         ~~~

   9. 查找元素 findAny  findFirst

      1. ~~~java
         // 方法将返回当前流中的任意元素  findAny
         Optional<Dish> dish =
         menu.stream()
         .filter(Dish::isVegetarian)
         .findAny();
         ~~~

   10. 归约 此类查询需要将流中所有元素反复结合起来，得到一个值，比如一个 Integer 。这样的查询可以被归类为**归约操作**（将流归约成一个值）。用函数式编程语言的术语来说，这称为**折叠**（fold）.

   11. reduce  求和,最大值,最小值

       1. ~~~java
          int sum = numbers.stream().reduce(0, Integer::sum);
          Optional<Integer> max = numbers.stream().reduce(Integer::max);
          Optional<Integer> min = numbers.stream().reduce(Integer::min);
          ~~~

   12. 中间操作和终端操作

       ![image-20210407103343572](./picture/java8实战/image-20210407103343572.png)

   13. 原始类型流特化

       1. 使用map返回的Stream<T>,如果想直接返回我们需要的类型例如int,则使用mapToInt,直接返回int,避免了暗含的装箱成本.

          1. ~~~java
             int calories = menu.stream()
             .mapToInt(Dish::getCalories)
             .sum();		
             ~~~

       2. 转换回对象流  boxed()

          1. ~~~java
             IntStream intStream = menu.stream().mapToInt(Dish::getCalories);
             Stream<Integer> stream = intStream.boxed();
             ~~~

       3. 数值范围

          1. IntStream和LongStream的range和rangeClosed方法.rangeClosed 包含起始值.

   14. 创建流

       1. 由值创建流     Stream<String> stream = Stream.of("Modern ", "Java ", "In ", "Action");

       2. 可空对象创建流 

          1. Stream<String> homeValueStream= Stream.ofNullable(System.getProperty("home"));

       3. 数组创建流   int[] numbers = {2, 3, 5, 7, 11, 13};int sum = Arrays.stream(numbers).sum();

       4. 由文件生成的流

          1. ~~~java
             // 查询文件中有多少个不同的单词
             long uniqueWords = 0;
             try(Stream<String> lines =
             Files.lines(Paths.get("data.txt"), Charset.defaultCharset())){
             uniqueWords = lines.flatMap(line -> Arrays.stream(line.split(" ")))
             .distinct()
             .count();
             }
             catch(IOException e){
             }
             ~~~

          2.  Files.lines 得到一个流，其中的每个元素都是给定文件中的一行

       5. 无限流

## 第六章

1. ### 规约和汇总

   1. 数据流的总数   

      1. long howManyDishes = menu.stream().collect(Collectors.counting());
      2. long howManyDishes = menu.stream().count();

   2. 计算流中的最大值和最小值**maxBy** **minBy**

      1. menu.stream().collect(**maxBy**(dishCaloriesComparator));

   3. 汇总

      1. 求热量的总和	int totalCalories = menu.stream().collect(**summingInt**(Dish::getCalories));
      2. 同样方法可以换成 **summingDouble**,**averagingInt**等方法.
      3. 可以换成 **summarizingInt**,一次就能获取多个值 
         1. IntSummaryStatistics{count=9, sum=4300, min=120,average=477.777778, max=800}

   4. 连接字符串

      1. String shortMenu = menu.stream().map(Dish::getName).collect(joining(", ")); 也可以不加分隔符.

   5. reduce 和collect的比较  reduce适合于不变值,collect适合于在动态的变值规约.

      1. ~~~java
         // reduce
         int totalCalories = menu.stream()
             .collect(reducing(0, Dish::getCalories, (i, j) -> i + j));
         //参数 1 起始值  2 使用的函数将菜肴转换表示其所含热量的int 3 求和 
         ~~~

2. ### 分组

   1. ~~~java
      Map<Dish.Type, List<Dish>> dishesByType =
      menu.stream().collect(groupingBy(Dish::getType));
      ~~~


