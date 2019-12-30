### 第 151 题：用最简洁代码实现 indexOf 方法

1. 方法一：使用正则表达式进行匹配
2. 方法二：for 循环遍历，配合裁剪截取到字段和传入的进行匹配

### 第 150 题：二分查找如何定位左边界和右边界

1. 左加右不加，找右缩左，找左缩右

```js
function binarySearch(arr, target) {
  let left = 0;
  let right = arr.length - 1;
  while (left <= right) {
    const min = (left + right) >>> 1;
    if (arr[min] === target) {
      right = min - 1; // 找左边界二分法
    } else if (arr[min] > target) {
      // 数据在左侧
      right = min - 1;
    } else {
      left = min + 1;
    }
  }
  return left;
}
```

### 第 149 题：babel 怎么把字符串解析成 AST，是怎么进行词法/语法分析的？

1. input => tokenizer => tokens，先对输入代码进行分词，根据最小有效语法单元，对字符串进行切割
2. tokens => parser => AST，然后进行语法分析，会涉及到读取、暂存、回溯、暂存点销毁等操作。
3. AST => transformer => newAST，然后转换生成新的 AST
4. newAST => codeGenerator => output，最后根据新生成的 AST 输出目标代码

### 第 148 题： webpack 中 loader 和 plugin 的区别是什么

1. loader，它是一个转换器，将 A 文件进行编译成 B 文件，比如：将 A.less 转换为 A.css，单纯的文件转换过程。
2. plugin 是一个扩展器，它丰富了 webpack 本身，针对是 loader 结束后，webpack 打包的整个过程，它并不直接操作文件，而是基于事件机制工作，会监听 webpack 打包过程中的某些节点，执行广泛的任务
