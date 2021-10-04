# 1.两数之和

[题目链接](https://leetcode-cn.com/problems/two-sum/)

**题解：**

遍历数组，将数组深拷贝给一个变量，获得目标值减去当前正在遍历的数的差，在变量中查找是否存在。

```js
var twoSum = function(nums, target) {
    for(let i in nums){
        let arr = [...nums];
        let x=target -nums[i];
        arr[i]=null;
        if(arr.indexOf(x)!==-1)
        return [i,arr.indexOf(x)];
    }
};
```

**优化：**

利用map()对象

```js
var twoSum = function(nums, target) {
    let map = new Map();
    for (let i = 0,len = nums.length; i < len; i++) {
        let key = target - nums[i];
        if (map.has(key)) {
            return [map.get(key), i];
        }
        map.set(nums[i], i);
    }
};
```

# 2.两数相加

[题目链接](https://leetcode-cn.com/problems/add-two-numbers/)

**题解：**

如果只是单纯的将他们装换成数字相加运算，则会超出number限制，会有测试用例不通过。（可以使用位运算）

另一种思路就是，进行单个位数的相加，若果超过10就进位，由于链表的设计使得这种方法很简单。

```js
var addTwoNumbers = function(l1, l2) {
  const res = new ListNode();
  let num1 = num2 = ans = sum = 0,
      flag = false,
      cur = res;
  while (l1 !== null || l2 !== null) {
    num1 = 0;
    num2 = 0;
    if (l1 !== null) {
      num1 = l1.val;
      l1 = l1.next;
    }

    if (l2 !== null) {
      num2 = l2.val;
      l2 = l2.next;
    }

    // 求和
    sum = num1 + num2; 
    // 上次进位了, sum + 1
    if (flag) sum++; 

    // 判断是否发生进位
    flag = sum >= 10;
    // 给结果新增一位
    cur.next = new ListNode(sum % 10)
    cur = cur.next;
  }
  // 最后一次运算也发生了进位, 则最后再新增一位 1
  if (flag) {
    cur.next = new ListNode(1);
  }
  return res.next;
};
```

