# 1.[两数之和](https://leetcode-cn.com/problems/two-sum/)

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

# 2.[两数相加](https://leetcode-cn.com/problems/add-two-numbers/)

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

# 3.[无重复字符的最长子串](https://leetcode-cn.com/problems/longest-substring-without-repeating-characters/)

**题解：**

遍历字符串中的每个字符，从某个字符开始继续向后遍历，直到出现重复字符，记录长度，与最长进行对比。将最大值保存并返回。

```js
var lengthOfLongestSubstring = function(s) {
    let max =0;
    for(let i=0;i<s.length;i++){
        let cur =1;
        let arr =[];
        arr.push(s[i]);
        for(let j=i+1;j<s.length;j++){
            if(arr.indexOf(s[j])===-1){
                arr.push(s[j]);
                cur++;
            }else{
                
                break;
            }
        }
        if(max<cur){
            max=cur;
        }
    }
    return max;
};
```

**优化：**

滑动窗口

我们制作一个窗口，让窗口中的字符串满足题目要求（无重复）
怎么让他满足要求呢？ 那就要滑动窗口了，循环去掉左边第一个元素，直到窗口中元素无重复，此时再扩大窗口

滑动窗口有两个关键点：扩张 + 收缩
首先（右指针）扩张到滑动窗口不满足条件的时候暂停，
（左指针）开始收缩窗口，让窗口满足条件后再进行扩张（右指针）

```js
function lengthOfLongestSubstring(s) {
  let len = s.length;
  let result = 0;

  let set = new Set();
  // 左指针用来收缩窗口
  let left = 0;
  // 右指针用来扩张窗口
  let right = 0;

  while (left < len) {
    // 如果不重复，就不断扩张窗口，元素添加到set中
    while (right < len && !set.has(s[right])) {
      set.add(s[right]);
      right++;
    }
    // 到这里说明有元素重复了，先记录子串长度，然后收缩窗口
    result = Math.max(result, right - left);
    // 收缩窗口
    set.delete(s[left]);
    left++;
  }
  return result;
}
```

