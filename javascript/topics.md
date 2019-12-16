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
