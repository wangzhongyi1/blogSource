# 前端开发规范

## 条件判断
- if 多条件判断
```js
// 冗余
if (value === 1  || value === 'one' || value === 'two') {}

// 推荐1
if ([1, 'one', 'two'].indexOf(value) >= 0) {}

// 推荐2（es6）
if ([1, 'one', 'two'].includes(value)) {}
```

- 合并条件 简化判断
```js
// before:
if (fruit === 'apple') {
  return true;
} else if (fruit === 'cherry') {
  return true;
} else  if (fruit === 'peach') {
  return true;
} else {
  return false;
}

// after:
const redFruits = ['apple', 'cherry', 'peach'];
return redFruits.includes(fruit);
```

- 合并条件表达式
```js
// 当检查条件各不相同，最终行为却一致时，应该使用“逻辑或”和“逻辑与”将它们合并成一个条件表达式。从而使条件检查用意更清晰

// before:
if (obj.a < 2) return 0;
if (obj.b > 10) return 0;
if (obj.c === 'school') return 0;

// after:
function fn() {
  return obj.a < 2 || obj.b > 10 || obj.c === 'school';
}
if (fn()) return 0;
```

- 把条件分支语句提炼成函数
```js
// 复杂的条件分支语句时导致程序难以阅读和理解的重要原因，而且容易导致形成一个庞大的函数
// 有时可以将条件分支语句提炼成语义化的函数，使代码更加直观，逻辑清晰

// before:
// 根据不用季节决定打折力度
function getPrice(price) {
  var d = new Date;
  if (d.getMonth() >= 6 && d.getMonth() <= 9) { // 夏天
    return price * 0.8;
  }
  return price;
}

// after:
// 是否夏天
function isSummer() {
  var d = new Date;
  return d.getMonth() >= 6 && d.getMonth() <= 9;
}
function getPrice(price) {
  if (isSummer()) {
    return price * 0.8;
  }
  return price;
}
```

## 使用三元运算符优化条件判断
- 当有不包含复杂逻辑时的 if...else，可以使用三元运算符来简化
```js
// before:
let test;
if (x > 100) {
  test = true;
} else {
  test = false;
}

// after:
let test = x > 10 ? true : false;
// 或简写：
let test = x > 10;
```

- 当使用三元嵌套时，可能会造成理解的复杂度，可以增加适当的注释
```js
const x = 300;
const test = x > 100
              ? 'greater 100'
              : x < 50
                ? 'less 50'
                : 'between 50 and 100';
```

## 提前让函数退出代替嵌套条件分支/用卫语句(Guard Clauses)替换嵌套的条件语句
条件表达式通常有两种风格。

第一种风格是：两个条件分支都属于正常行为。

第二种风格则是：只有一个条件分支是正常行为，另一个分支则是异常的情况。

这两类条件表达式有不同的用途，这一点应该通过代码表现出来。如果两条分支都是正常行为，就应该使用形如if...else...的条件表达式；如果某个条件极其罕见，就应该单独检查该条件，并在该条件为真时立刻从函数中返回。这样的单独检查常常被称为“卫语句”（guard clauses）。

以卫语句取代嵌套条件表达式的精髓就是：给某一条分支以特别的重视。如果使用if-then-else结构，你对if分支和else分支的重视是同等的。这样的代码结构传递给阅读者的消息就是：各个分支有同样的重要性。卫语句就不同了，它告诉阅读者：“这种情况不是本函数的核心逻辑所关心的，如果它真发生了，请做一些必要的整理工作，然后退出。”

“每个函数只能有一个入口和一个出口”的观念，现今的编程语言都会强制保证每个函数只有一个入口，至于“单一出口”规则，其实不是那么有用。保持代码清晰才是最关键的：如果单一出口能使这个函数更清楚易读，那么就使用单一出口；否则就不必这么做。

卫语句就是把复杂的条件表达式拆分成多个条件表达式，减少嵌套。

- 使用时机：一般来说，当出现if语句时就可以考虑是否使用卫语句 无论if是仅出现一次，还是多次
- 为什么用：一般来说卫语句使用是为了两个事情，1）减少智力成本 2）结构优化
- 应用场景（哪里用）：
1. 长if语句
```js
// 仅需将代码重构下即可，逻辑完全不变
// before:
if(list.length>0){
  list.forEach(item=>item)
  list.map(item=>item)
  list = [...list]
  // 就是假装逻辑特别长
  // 就是假装逻辑特别长
  // 就是假装逻辑特别长
}
// 结束了 else没有处理


// after:
if(list.length<=0) return
list.forEach(item=>item)
list.map(item=>item)
list = [...list]
// 就是假装逻辑特别长
// 就是假装逻辑特别长
// 就是假装逻辑特别长

处理思路：就是将if取反后处理else逻辑即可
```
2. if-else嵌套地狱
```js
// before:
function getPayAmount() {
    let result;
    if (isDead){
        result = deadAmount();
    } else {
        if (isSeparated){
            result = separatedAmount();
        } else {
            if (isRetired){
                result = retiredAmount();
            }else{
                result = normalPayAmount();
            }
        }
    }
    return result;
}


// after:
function getPayAmount() {
    if (isDead) return deadAmount();
    if (isSeparated) return separatedAmount();
    if (isRetired) return retiredAmount();
    return normalPayAmount();
}
处理后，多层次判断结构变为单层次结构，作用域更加明显，逻辑由嵌套变为线性处理
处理思路：
选中最外层需要被替换的条件逻辑，将其替换为卫语句。
```

## 合并重复的条件片段
如果一个函数体内有一些条件分支语句，而这些条件分支语句内部分散了一些重复的代码，那么就有必要进行合并去重工作
```js
// before:
function main(currPage) {
  if (currPage <= 0) {
    currPage = 0;
    jump(currPage); // 跳转
  } else if (currPage >= totalPage) {
    currPage = totalPage;
    jump(currPage);
  } else {
    jump(currPage);
  }
}

// after:
function main(currPage) {
  if (currPage <= 0) {
    currPage = 0;
  } else if (currPage >= totalPage) {
    currPage = totalPage;
  }
  jump(currPage); // 把jump函数独立出来
}
```

## numm、undefined和空值检查并设置默认值
- 当创建新的变量或者调试API提供的数据时，需要检查数据是否为null、undefined和空值。
```js
// before:
let test2 = ''
if (test !== null || test !== undefined || test !== '') {
  test2 = test;
}

// after:
const test2 = test || ''; // 短路或设置默认值
// 新语法，空值合并运算符(??)，当左侧操作数为空或未定义时返回右侧操作数，否则返回左侧操作数
const foo = null ?? 'default string';
console.log(foo); // 'default string'
```

## if 存在的简写
```js
// before:
if (test === true) or if (test !== '') or if (test !== null) {}

// after:
if (test) {}
```

## if 执行函数/短路运算符(&&)
```js
// before:
if (state) {
  show();
}

// after:
state && show();

// before:
let num = 0;
if (state) {
  num = 3;
}

// after:
let num = 0;
state && (num = 3);
```
