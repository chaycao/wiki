# foreach
foreach 在 jdk1.5 加入，当作 for 语法的一个增强。

能使用的类型：数组、java.lang.Iterable

- 数组：转换为对数组中每个元素的循环引用
- Iterable：调用Iterator()返回的迭代器hasNext()、next()遍历