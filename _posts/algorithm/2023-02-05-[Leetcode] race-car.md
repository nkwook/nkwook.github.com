---
title: "[Leetcode] 818. Race Car"
excerpt: "test"
tags: [Leetcode, Algorithm]
toc: true
toc_sticky: true    

date: 2023-02-05 
categories: Algorithm 
---

Difficulty | Time  | Succeed?
:---: | --- | :---:
*Hard* |  > 0:45 | X


# Problem description

[Link](https://leetcode.com/problems/race-car/)


# My Wrong Approach

## Idea

> BFS, DP   
> TC: `O(target * log(target))` 

In the worst case, there will be `2 * target` positions and `log(target)` speeds per position.


## Code

```c++
class Solution {
public:
  int racecar(int target) {
    unordered_map < int, unordered_map<int, int> > m;
    // position, speed / instruction length

    queue<tuple<int, int, int>> Q;
    // position, speed, length
    Q.push(make_tuple(0, 1, 0));

    while (!Q.empty()) {

      tuple<int, int, int> t = Q.front();
      Q.pop();
      int position = get<0>(t);
      int speed = get<1>(t);
      int length = get<2>(t);
      if (position == target) {
          return length;
      }


      if (m[position][speed]) {
        continue;
      }
      m[position][speed]=1;

    
      if (position < target){
        Q.push(resultA(position, speed, length));
      }
      Q.push(resultR(position, speed, length));
    }
    return -1;
  }

  tuple<int, int, int> resultA(int p, int s, int l) {
    return make_tuple(p+s, s*2, l+1);
  }

  tuple<int, int, int> resultR(int p, int s, int l) {
    return make_tuple(p, -1, l+1);
  }
};
```

## Why I was wrong

### Integer overflow

```
Line 42: Char 24: runtime error: signed integer overflow: -2147483647 + -2147483648 cannot be represented in type 'int' (solution.cpp)
SUMMARY: UndefinedBehaviorSanitizer: undefined-behavior prog_joined.cpp:51:24
```
[Dynamic analyzer](https://clang.llvm.org/docs/UndefinedBehaviorSanitizer.html) detected integer overflow in `resultA` function. I was confused since I could compile code with `g++` and could generate appropriate answer with test input. However, the dynamic analyzer in leetcode didn't let my erroneous code to run.  
So I resolved the error by adding conditionals to avoid the position variable become huge.

### TLE

I expected `O(nlgn)` time complexity, but TLE occured. I had give condition when giving **R** instruction since when target is bigger than position and the car is heading to target, there is no need to turn speed to negative. 

## Revised Code
```c++
class Solution {
public:
  int racecar(int target) {
    unordered_map < int, unordered_map<int, int> > m;
    // position, speed / instruction length

    queue<tuple<int, int, int>> Q;
    // position, speed, length
    Q.push(make_tuple(0, 1, 0));

    while (!Q.empty()) {

      tuple<int, int, int> t = Q.front();
      Q.pop();
      int position = get<0>(t);
      int speed = get<1>(t);
      int length = get<2>(t);

      if (position == target) {
          return length;
      }


      if (m[position][speed] || position < 0 || position > 2*target) {
        continue;
      }

      m[position][speed]=1;

    
        Q.push(resultA(position, speed, length));

        if(speed <0 || position + speed > target){
            Q.push(resultR(position, speed, length));
        }
        // key idea: determine when to call R, think about whether it is really needed?
    }
    return -1;
  }

  tuple<int, int, int> resultA(int p, int s, int l) {
    return make_tuple(p+s, s*2, l+1);
  }

  tuple<int, int, int> resultR(int p, int s, int l) {
      int newSpeed=-1;
      if (s<0){
          newSpeed=1;
      }
    return make_tuple(p, newSpeed, l+1);
  }
};
```




