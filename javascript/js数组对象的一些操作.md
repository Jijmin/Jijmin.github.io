### 数组去重排序

```
[...new Set(arr.toString().split(',').map(Number))].sort((a,b)=> a-b)
```
