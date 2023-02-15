# Java ArrayList retainAll() 方法

[![Java ArrayList](https://www.runoob.com/images/up.gif) Java ArrayList](https://www.runoob.com/java/java-arraylist.html)

retainAll() 方法用于保留 arraylist 中在指定集合中也存在的那些元素，也就是删除指定集合中不存在的那些元素。

retainAll() 方法的语法为：

```
arraylist.retainAll(Collection c);
```

**注：**arraylist 是 ArrayList 类的一个对象。

**参数说明：**

- collection - 集合参数

### 返回值

如果 arraylist 中删除了元素则返回 true。

如果 arraylist 类中存在的元素与指定 collection 的类中元素不兼容，则抛出 ClassCastException 异常。

如果 arraylist 包含 null 元素，并且指定 collection 不允许 null 元素，则抛出 NullPointerException 。

### 实例

保留指定集合中的元素：

## 实例

**import** java.util.ArrayList;
**class** Main {
  **public** **static** **void** main(String[] args){

​    *// 创建一个动态数组*
​    ArrayList<String> sites = **new** ArrayList<>();
​    
​    sites.add("Google");
​    sites.add("Runoob");
​    sites.add("Taobao");

​    System.out.println("ArrayList 1: " + sites);

​    *// 创建另一个动态数组*
​    ArrayList<String> sites2 = **new** ArrayList<>();

​    *// 往动态数组中添加元素*
​    sites2.add("Wiki");
​    sites2.add("Runoob");
​    sites2.add("Google");
​    System.out.println("ArrayList 2: " + sites2);

​    *// 保留元素*
​    sites.retainAll(sites2);
​    System.out.println("保留的元素: " + sites);
  }
}

执行以上程序输出结果为：

```
ArrayList 1: [Google, Runoob, Taobao]
ArrayList 2: [Wiki, Runoob, Google]
保留的元素: [Google, Runoob]
```

在上面的实例中，我们创建了个名为 sites 和 sites2 的动态数组。

注意这一行：

```
sites.retainAll(sites2);
```

实例中，我们传入了数组 sites2 作为 retainAll() 方法的参数。该方法从 sites 删除了不存在于 sites2 的元素。

ArrayList 和 HashSet 共有元素：

## 实例

**import** java.util.ArrayList;
**import** java.util.HashSet;

**class** Main {
  **public** **static** **void** main(String[] args) {
    *// 创建一个数组*
    ArrayList<Integer> numbers = **new** ArrayList<>();

​    *// 往数组中添加元素*
​    numbers.add(1);
​    numbers.add(2);
​    numbers.add(3);
​    System.out.println("ArrayList: " + numbers);

​    *// 创建一个HashSet对象*
​    HashSet<Integer> primeNumbers = **new** HashSet<>();

​    *// 往 HashSet添加元素*
​    primeNumbers.add(2);
​    primeNumbers.add(3);
​    primeNumbers.add(5);
​    System.out.println("HashSet: " + primeNumbers);

​    *// 在arraylist中保留公共的元素*
​    numbers.retainAll(primeNumbers);
​    System.out.println("共有元素: " + numbers);
  }
}

执行以上程序输出结果为：

```
ArrayList: [1, 2, 3]
HashSet: [2, 3, 5]
共有元素: [2, 3]
```

在以上实例中，我们创建了一个名为 numbers 的 ArrayList 和一个名 primeNumbers 的 HashSet。

注意这一行：

```
numbers.retainAll(primeNumbers);
```

代码中，retainAll() 方法在 numbers 动态数组中删除了所有不存在于 primeNumbers 中的元素。