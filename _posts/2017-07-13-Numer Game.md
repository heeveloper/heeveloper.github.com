---
layout: post
title:  "숫자 게임"
date:   2017-07-13 23:42:29
author: Heeveloper
categories: Algorithm
tags: 프로그래밍대회에서배우는알고리즘문제해결전략 Algospot 알고스팟 NUMERGAME 숫자게임 dp 동적계획법 메모이제이션 memoization
excerpt: NUMBER GAME
mathjax : false
---

* content
{:toc}

## 1. Problem
![screenshot](/img/numbergame_problem.png)
<br>

---
## 2. My Approach
* 자기 턴이 오면 행동할 수 있는 방법은 둘 다 최선을 다한다는 가정하에 4가지로 동일하다.
> 1. 앞의 숫자 1개 뽑기
> 2. 뒤의 숫자 1개 뽑기
> 3. 앞의 숫자 2개 지우기
> 4. 뒤의 숫자 2개 지우기

* 각각 턴이 돌아가면서 현재의 선택이 다음 사람의 선택에 영향을 미치는 이 게임은 재귀함수로 표현하기 적합하다.
* 따라서 넘겨줘야 하는 재귀함수의 인자는 현재 턴의 사람의 선택에 따라 결정되는 나머지 숫자들이라고 생각할 수 있다.
* 메모이제이션의 구현은 `cache`에 접근하는 함수, 즉 재귀함수의 인자로서 결정한다.

---
## 3. Implementation
* 구현의 편의를 주어진 숫자 배열을 전역변수로 선언한 뒤, 재귀함수 `play()`는 인자로 시작과 끝 `index`를 넘겨주어 사실상 나머지 숫자들을 넘겨주는 기능을 하도록 한다.

~~~c++
const int MINIMUM = -99999;

int play(int start, int end){
    if(end - start + 1 <= 0) return 0;
    int& ret = cache[start][end];
    if(ret != MINIMUM) return ret;

    int p[4];
    p[0] = nums[start] - play(start+1, end);
    p[1] = nums[end] - play(start, end-1);
    p[2] = MINIMUM;
    p[3] = MINIMUM;
    if(end - start + 1 >= 2){
        p[2] = -play(start+2, end);
        p[3] = -play(start, end-2);
    }
    int max = MINIMUM;
    for(int i=0;i<4;i++)
        if(max < p[i])
            max = p[i];

    return ret = max;
}
~~~
> * p[ ] 는 각각 4가지 행동을 했을 때 얻을 수 있는 점수차를 의미한다. 따라서 현재 선택한 숫자값에서 다음 턴의 사람이 얻을 수 있는 점수차를 의미하는 다음 play() 를 빼준다.
> * p[2]와 p[3]은 카드를 지웠을 때의 경우로 -play( )앞에 0이 생략되어 있다고 보면 된다.
> * 카드가 2장 이상 남아있지 않은 경우엔 애초에 지우는 행위는 일어나지 않다는 것을 의미하기 위해 0이 아닌 MINIMUM 으로 초기화 해주어야 한다.

---
## 4. Textbook Approach
* 구현방법은 다르나 원리는 같다.
* 결국에 4가지 경우에 따라 재귀적으로 `play()`를 호출해주고 비교해서 현재 순간에서 얻을 수 있는 최대의 점수차를 반환한다.

---
## 5. Implementation
~~~c++
int play(int start, int end){
    if(end-start+1 <= 0) return 0;
    int& ret = cache[start][end];
    if(ret != MINIMUM)
        return ret;

    //카드를 가져가는 경우
    ret = max(nums[start] - play(start+1, end), nums[end] - play(start,end-1));
    //카드를 지우는 경우
    if(end - start + 1 >= 2){
        ret = max(ret, -play(start+2, end));
        ret = max(ret, -play(start, end-2));
    }

    return ret;
}
~~~


---
## 6. Analysis
### Memoization
 * 시험을 해보고 싶은 마음에 `memoization` 부분을 생략하고 채점서버에서 코드를 돌려본 결과 시간초과가 됐다.
 * 중복이 많은 경우일 수록 `memoization` 의 효과를 실감할 수 있다.
 * 반대로 동적계획법(Dynamic Programming)으로 풀어야 하는 문제는 모든 경우의 수를 계산하는 것이 불가피한데, 수행시간을 줄이기 위해선 `memoization`의 사용 역시 필수다. 이 효과를 극대화하기 위해서 문제를 최대한 많은 중복이 일어나는 하부 문제로 나눌 수 있어야 한다.

---
## 7. Feedback
### MEMSET

  여느 때처럼 cache 메모리를 초기화하기 위해서 memeset(cache, MINIMUM, sizeof(cache))를 사용했으나 엉뚱한 값으로 초기화되었다.
  원인을 찾아본 결과 memset은 byte단위로 초기화를 하기 때문에 다음과 같은 주의사항을 정리할 수 있다.
  > 1. 1byte 변수(char, unsigned char 등)를 제외한 변수를 초기화 할 때에는 0이외의 값으로 초기화를 하면 안된다.
  > 2. new, malloc 등을 이용하여 동적으로 배열을 생성하는 변수가 있는 struct, class에서는 memset으로 초기화를 하면 안된다.
  > 3. cstring은 절대 memset으로 초기화를 하면 안된다.
  > 4. virtual function을 가지고 있는 struct, class에서는 절대 memset으로 초기화를 하면 안된다.

  그 동안 -1로 초기화를 할 수 있었던 건 byte단위 초기화가 -1값과 우연히 맞아 떨어져서 가능했던 것이었다.

  Conference : [TistoryStudio memset "함수 주의점"](http://beautyrain.tistory.com/7)

  ---
