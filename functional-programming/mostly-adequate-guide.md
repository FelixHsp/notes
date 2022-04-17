# 函数式编程指北

本文为学习 [*mostly-adequate-guide*](https://github.com/MostlyAdequate/mostly-adequate-guide/blob/master/SUMMARY.md) 及其 [译文](https://llh911001.gitbooks.io/mostly-adequate-guide-chinese/content/) 过程中的记录和总结。
## 第 1 章 我们在做什么？
本章主要是通过一个例子初步介绍函数式编程。
``` typescript
/*
 * 一个创建鸟群的方法，入参n为该鸟群中海鸥的个数
 */
var Flock = function(n) {
  this.seagulls = n;
};

/*
 * 定义一个鸟群合并方法
 */
Flock.prototype.conjoin = function(other) {
  this.seagulls += other.seagulls;
  return this;
};

/*
 * 定义一个繁殖方法，假设m和n只海鸥经过繁殖后鸟群中海鸥数量变为m*n
 */
Flock.prototype.breed = function(other) {
  this.seagulls = this.seagulls * other.seagulls;
  return this;
};

var flock_a = new Flock(4);
var flock_b = new Flock(2);
var flock_c = new Flock(0);

/*
 * 1、4只海鸥的鸟群和0只海鸥的鸟群合并
 * 2、再去和2只海鸥的鸟群繁殖
 * 3、再去合并一个4只海鸥和2只海鸥繁殖后的鸟群
 * 由于 flock_a 在运算过程中被改变了，最终得到的数量是32而非期待的16
 */
var result = flock_a.conjoin(flock_c).breed(flock_b).conjoin(flock_a.breed(flock_b)).seagulls;
```

试试另一种更函数式的写法：
``` typescript
var conjoin = function(flock_x, flock_y) { return flock_x + flock_y };
var breed = function(flock_x, flock_y) { return flock_x * flock_y };

var flock_a = 4;
var flock_b = 2;
var flock_c = 0;

/*
 * 计算结果无问题，但是函数嵌套严重
 */
var result = conjoin(breed(flock_b, conjoin(flock_a, flock_c)), breed(flock_a, flock_b));
```

回顾下数学中的二元运算：
```typescript

/**
 * 结合律（assosiative）
 */
add(add(x, y), z) == add(x, add(y, z));

/**
 * 交换律（commutative）
 */
add(x, y) == add(y, x);

/**
 * 同一律（identity）
 */
add(x, 0) == x;

/**
 * 分配律（distributive）
 */
multiply(x, add(y,z)) == add(multiply(x, y), multiply(x, z));
```

根据上述二元计算中的性质改写函数式的写法：
```typescript
var add = function(x, y) { return x + y };
var multiply = function(x, y) { return x * y };

var flock_a = 4;
var flock_b = 2;
var flock_c = 0;

var result = add(multiply(flock_b, add(flock_a, flock_c)), multiply(flock_a, flock_b));

/**
 * 再对结果进行整合
 */
add(multiply(flock_b, add(flock_a, flock_c)), multiply(flock_a, flock_b));

/**
 * 应用同一律，去掉多余的加法操作（add(flock_a, flock_c) == flock_a）
 */
add(multiply(flock_b, flock_a), multiply(flock_a, flock_b));

/**
 * 再应用分配律
 */
multiply(flock_b, add(flock_a, flock_a));
```

## 第 2 章 一等公民的函数
本章主要是介绍函数的无意义包裹问题。
``` typescript
/**
 * bad
 */
const hi = name => `Hi ${name}`;
const greeting = name => hi(name);
hi("Felix"); // "Hi Felix"

/**
 * good
 */
const hi = name => `Hi ${name}`;
const greeting = hi;
greeting("Felix"); // "Hi Felix"


/**
 * bad
 */
const getServerStuff = callback => ajaxCall(json => callback(json));

/**
 * good
 */
const getServerStuff = ajaxCall;


/**
 * bad
 */
const BlogController = {
  index(posts) { return Views.index(posts); },
  show(post) { return Views.show(post); },
  create(attrs) { return Db.create(attrs); },
  update(post, attrs) { return Db.update(post, attrs); },
  destroy(post) { return Db.destroy(post); },
};

/**
 * good
 */
const BlogController = {
  index: Views.index,
  show: Views.show,
  create: Db.create,
  update: Db.update,
  destroy: Db.destroy,
};


/**
 * bad
 */
httpGet('/post/2', json => renderPost(json));
httpGet('/post/2', (json, err) => renderPost(json, err));

/**
 * good
 */
httpGet('/post/2', renderPost);
```