---
layout: post
title:  "연습시즌"
date:   2017-07-15 23:48:29
author: Heeveloper
categories: Algorithm
tags: BOJ 백준온라인 9023 TrainingSeason 연습시즌 dp 동적계획법 메모이제이션 memoization
excerpt: Training Season
mathjax : false
---

* content
{:toc}

## 1. Problem
 BOJ Probelm 9023<br>
 [(https://www.acmicpc.net/problem/9023)](https://www.acmicpc.net/problem/9023)
<br>

---
## 2. My Approach
* 문제를 이해하면 어떤 편법도 쓰일 수 없고, 결국 모든 경우의 수를 찾아 비교해야 한다는 것을 알 수 있다.
* 따라서 동적계획법을 사용해야 했고, 재귀함수는 각 팀의 일정 하루하루를 가리키는 인덱스 p, q를 인자로 받고 휴식요금 계산을 위해 각 팀의 휴식여부도 전달한다.
* 재귀 함수 내에선 하루의 일정을 짜서 비교하는 데, 경우는 다음으로 나눌 수 있다.
> * 훈련을 하는 경우
> > * 두 팀의 경기장이 같은 경우
> > * 두 팀의 경기장이 다른 경우
> * 휴식을 하는 경우
> > * A팀이 쉬는 경우
> > * B팀이 쉬는 경우

* 각 경우에 맞게 인덱스를 증가시켜 재귀함수를 호출함으로써 일정을 짠다.
* 기저사례는 다음 2가지로 나눌 수 있다.
> * 두 팀 모두 모든 일정을 소화했을 때
> * 한 팀만 일정을 소화했을 때
> > 일정이 남은 한 팀이 훈련을 할 동안, 일정이 끝난 팀은 남은 기간동안 휴식만을 취한다.

* 그 때 그 때 최소의 비용이 드는 값을 반환한다.

---
## 3. Implementation

~~~c++
int cache[101][101][2][2];

int Schedule(int t1, int t2, int r1, int r2){
    int& ret = cache[t1][t2][r1][r2];
    if(ret != -1) return ret;

    // Base case
    if(t1 == team1.size() && t2 == team2.size())
        return ret = 0;
    if(t1 == team1.size())
        return ret = ((int)team2.size() - t2)*(C+d) + D;
    if(t2 == team2.size())
        return ret = ((int)team1.size() - t1)*(C+d) + D;

    ret = 0;
    // 경기
    if(team1[t1] == team2[t2])
        ret = C + Schedule(t1+1, t2+1, 0, 0);
    else
        ret = 2*C + Schedule(t1+1, t2+1, 0, 0);
    // A팀 휴식
    if(r1)
        ret = min(ret, d+C+Schedule(t1, t2+1, 1, 0));
    else
        ret = min(ret, D+d+C+Schedule(t1, t2+1, 1, 0));
    // B팀 휴식
    if(r2)
        ret = min(ret, d+C+Schedule(t1+1, t2, 0, 1));
    else
        ret = min(ret, D+d+C+Schedule(t1+1, t2, 0, 1));

    return ret;
}
~~~

---
## 4. Feedback
* 문제가 꽤나 복잡해보인다고 쫄지 말자. 이번 문제처럼 쓸데없이 설명만 긴 문제일 확률이 다분하다.
* 모든 경우의 수를 계산해야하는 문제가 주어지면, 제일 첫 번째로 하부 문제를 어떻게 나눌 건지가 관건이다. 문제를 정확하게 이해하고 발생하는 경우를 파악하고 정리하자.
* 메모이제이션으로 사용되는 배열이 2차원을 넘어서는 것에 주저하지 말자. `cache`는 기학학적인 의미의 배열이 아닌 재귀함수를 기록하는 도구일뿐이다.

  ---
