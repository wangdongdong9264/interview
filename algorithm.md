# 算法

## 快排

时间复杂度

  最好的情况：每一次base值都刚好平分整个数组，`O(nlogn)`

  最坏的情况：每一次base值都是数组中的最大/最小值，`O(n2)`

空间复杂度

  快速排序是递归的，需要借助栈来保存每一层递归的调用信息，所以空间复杂度和递归树的深度一致

  最好的情况：每一次base值都刚好平分整个数组，递归树的深度 `O(logn)`

  最坏的情况：每一次base值都是数组中的最大/最小值，递归树的深度 `O(n)`

稳定性

  快速排序是不稳定的，因为可能会交换相同的关键字。

  快速排序是递归的，
  
  特殊情况：left > right, 直接退出。

```js

function quickSort(arr, low, high) {
  var key = arr[low] // 第一位数字
  var start = low // 起始索引
  var end = high // 末位索引
  while(end > start) {
    while (end > start && arr[end] >= key) end--
    if (arr[end] < key) {
      var temp = arr[end]
      arr[end] = arr[start]
      arr[start] = temp
    }
    while (end > start && arr[start] <= key) start++
    if (arr[start] > key) {
      var temp = arr[start]
      arr[start] = arr[end]
      arr[end] = temp
    }
  }
  if (start > low + 1) {
    this.quickSort(arr, low, start - 1)
  }
  if (end < high - 1) {
    this.quickSort(arr, end + 1, high)
  }
}
var arr = [49, 38, 65, 97, 76, 13, 27, 49]
console.time('quickSort')
quickSort(arr, 0, arr.length-1)
console.timeEnd('quickSort')
console.log(arr)

```

## 斐波那契数列

斐波那契：由0和1开始，之后的斐波那契数列每一项都等于前两项之和。
斐波那契数列示例：1、1、2、3、5、8、13、21、34

```js

function fibonacci(nub) {
  let n = nub && parseInt(nub);
  let n1 = 1; // 初始 n = 1时候的值为1
  let n2 = 1; // 初始 n = 2时候的值为1
  let f;    // 声明变量sum 接受第 n 个的斐波那契数
  
  // n 等于 1 或 n 等于 2 的时候直接返回1
  if(n == 1 || n == 2) {
    return 1;
  }
  for(let i = 2; i < n; i++) {
    f = n1 + n2;
    n1 = n2;
    n2 = f;
  } 
  return f
}
let sum = fibonacci(8) 
console.log(8) // 21

```
