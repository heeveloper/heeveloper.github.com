---
layout: post
title:  "징검다리"
date:   2017-07-26 15:28:22
author: Heeveloper
categories: Algorithm
tags: SCPC 예선문제 징검다리 dp 동적계획법 점화식 메모이제이션 memoization
excerpt: SCPC 2회 예선문제
mathjax : false
---

* content
{:toc}

## 1. Problem
SCPC 2회 예선문제 <징검다리><br>
[Codeground Practice 참고](https://www.codeground.org/)

---
## 2. My Approach
* 출발선에서 마지막 돌다리까지 도달할 수 있는 모든 방법의 수를 구해야 하므로 동적계획법을 떠올렸다.
* 따라서 재귀함수를 구현해야 하는데 다음과 같은 조건을 염두했다.
> * 함정이 있는 돌다리는 따로 표시해두어 접근할 수 없게 해야한다.
> * 이전의 건넌 갯수와 동일하게 건널 수 없으므로 기록해야 한다.

* 따라서 돌다리배열을 만들어 함정을 표시하고, 재귀함수 인자로는 현재 위치와 이전에 건넌 돌다리 갯수를 받아야 한다.
* 다음으로 1개부터 K개까지의 돌다리를 건널 수 있는데, 그 중 이전에 건넌 것과 동일하게 건널 수 없고, 해당 수만큼 건넜을 때 위치할 돌다리가 함정이 없고, 마지막 돌다리를 넘지 않아야 하는 조건이 갖춰졌을 때 해당 수 만큼 건넌다.
* 기저사례
> * 마지막 돌다리에 도달했을 때 그 직전 돌다리에서 마지막 돌다리까지 오는 수는 한 개이므로 1을 반환한다.
> * 마지막 돌다리도 아니고 다음 넘어갈 돌다리도 없으면 0을 반환해야한다.
> * Back tracking 하면서 현재 위치에서 마지막 돌다리까지 가는 경우의 수가 계산된다.

---
## 3. Implementation
* 문제의 조건 중 경우의 수가 1000000009 이상 넘을 경우 이 값으로 나눠주라는 조건을 충족하기 위해 따로 MOD 연산을 처리했다.
* 주어진 위치를 아직 방문하지 않은 것과 주어진 위치에서 마지막까지 도달할 수 없는 것은 각각 -1과 0으로 표시되어 구분된다.

~~~c++
//
//  main.cpp
//  StepStone
//
//  Created by  Heejun Lee on 7/26/17.
//  Copyright © 2017  Heejun Lee. All rights reserved.
//

#include <iostream>
#include <cstring>

using namespace std;

int T,N,K,L;
int stone[50000];
int cache[50000][100];
int MOD=1000000009;

int Go(int num, int prejump){
    int& ret = cache[num][prejump];

    if(ret != -1) return ret;
    int count=0;

    //  Base case
    if(num == N){
        return 1;
    }

    for(int jump=1;jump<=K;jump++)
        if(jump != prejump && stone[num+jump] != -1 && num+jump < N+1){
            count += Go(num+jump, jump)%MOD;
            count %= MOD;
        }

    return ret = count;
}

int main(int argc, const char * argv[]) {
    cin >> T;
    for(int i=1;i<=T;i++){
        memset(stone, 0, sizeof(stone));
        memset(cache, -1, sizeof(cache));
        cin >> N >> K >> L;
        for(int i=0;i<L;i++){
            int mine;
            cin >> mine;
            stone[mine] = -1;
        }
        cout << "Case #" <<i<< endl<< Go(0, 0) << endl;
    }
    return 0;
}
~~~

* 채점결과 제한된 시간내에 제한된 갯수의 input까지만 연산가능하므로 절반의 점수밖에 얻지 못했다.
* 별도의 최적화 과정이 필요했다.

---
## 4. Other Approach
**점화식**을 사용함으로써 중복이 발생하는 경우를 늘려주어 좀 더 효과적인 동적계획법을 구현하였다.
>  **1. i번째 돌다리로 j 거리만큼 점프하여 오는 방법의 수**<br>
> = (i-j)번째 돌다리까지 가는 방법의 수 - (i-j)번째 돌다리로 j 거리만큼 점프하여 온 방법의 수<br>
> **2. i번째 돌다리로 오는 방법의 수**<br>
> = 위의 1번 과정에서 j에 1부터 K까지 넣었을 때 나오는 방법의 수들의 합

---
## 5. Implementation

~~~c++
#include <iostream>
#include <cstring> // using std::memset
#include <vector>

const int MOD = 1000000009;

int T;
int answer[50001];
int dp[50001][101]; // dp[N][K] = N번째 징검다리로 K거리 만큼 점프했을 때 개울을 건너는 방법 수

int main(int const argc, char const** argv)
{
  std::ios::sync_with_stdio(false);
  std::cin.tie(0);

  std::cin >> T;

  for (int TC = 1; TC <= T; ++TC)
  {
    int N, K, L;
    std::cin >> N >> K >> L;

    std::vector<bool> mines(N + 1, false);
    for (int i = 1, x; i <= L; ++i)
      std::cin >> x, mines[x] = true;

    std::memset(dp, 0, sizeof(dp));
    dp[0][0] = answer[0] = 1;

    for (int i = 1; i <= N; ++i)
    {
      answer[i] = 0;
      if(mines[i])
        continue;

      for (int j = 1; j <= K && (i >= j); ++j)
        dp[i][j] = ((answer[i - j] + MOD) - dp[i - j][j]) % MOD, answer[i] = (answer[i] + dp[i][j]) % MOD;
    }

    std::cout << "Case #" << TC << "\n" << answer[N] << "\n";
  }
  return 0;
}
~~~

---
## 6. Analysis
본인의 코드로는 제한 시간 1초를 초과하여 1.000039의 수행시간이 걸렸고, 만점자의 코드는 0.17869초로 약 10배가량 차이났다.<br>
이런 차이는 징검다리의 갯수와 함정의 개수에 비례하여 증가한다.

---
## 7. Feedback
같은 동적계획법을 사용하더라도 문제를 얼마나 더 최적화시키냐에 따라 수행시간이 크게 차이난다.<br>
동적계획법에서 최적화란 결국 반복되는 경우의 수를 극대화하도록 재귀함수를 구현하는 것이다.<br>
오늘 그러한 방법 중 한가지로, **점화식**을 배웠다.<br>
이는 어쩌면 당연히 재귀함수와 붙어 다니는 개념이었다.<br>
> **Recursive Function**<br>
> **Recursive Formula**


---
