---
layout: post
title:  "Coin Change"
date:   2017-07-29 15:28:22
author: Heeveloper
categories: Algorithm
tags: 프로그래밍대회에서배우는알고리즘문제해결전략 Algospot 알고스팟 CoinChange COINS dp 동적계획법
excerpt: COINS
mathjax : false
---

* content
{:toc}

## 1. Problem
![screenshot](/img/coins_problem.png)
<br>

---
## 2. My Approach
* 모든 경우의 수를 구해야 해서 동적계획법을 적용해야 했는데 일반적으로 사용하는 재귀함수의 구현방법이 쉽게 떠오르지 않았다.
* 따라서 `back tracking`을 이용하는 재귀함수보단 단순 반복문으로 주어진 동전으로 환전할 수 있는 금액 0 ~ M까지를 순차적으로 구하되 이전에 구한 기록을 이용하는 방법을 생각했다.
* 문제에 주어진 예제를 통해 접근 방법을 생각해보면 다음과 같다.
> 110원, {10, 50, 100, 500} <br>
> 1. 먼저 10원만 사용해서 만들 수 있는 방법의 수는 다음과 같다<br>
> [0~9]원 = 0 [10] = 1 [11~19] = 0 [20] = 1 ... [50] = 1 ...[100]:1
> 2. 다음 50원을 사용해서 만들 수 있는 방법의 수는 다음과 같다.<br>
> [50]원 = [50] + 1 = 1 + 1 = 2<br>
> [51]원 = [50 + 1] = 0<br>
> [60]원 = [50 + 10] = 2<br>
> [70]원 = [50 + 20] = 2<br>
* 위의 과정을 배열로 나타내면 다음과 같다
> coin = 현재 사용중인 동전<br>
> i = 이전에 사용했던 동전들로 나타낼 수 있었던 금액<br>
> i + coin = 현재 만들 수 있는 금액<br>
> array[i + coin] += array[i]
* 따라서 점화식을 이용한 동적계획법으로 접근 가능하다.

---
## 3. Implementation

~~~c++
//
//  main.cpp
//  CoinChange
//
//  Created by  Heejun Lee on 8/2/17.
//  Copyright © 2017  Heejun Lee. All rights reserved.
//

#include <iostream>

using namespace std;

int T, M, C;
int coins[100];

int main(int argc, const char * argv[]) {
    cin >> T;
    while(T--){
        cin >> M >> C;
        long counts[5000] = {0,};
        for(int i=0;i<C;i++)
            cin >> coins[i];
        for(int i=0;i<C;i++){
            int coin = coins[i];
            counts[coin]++;
            for(int j=1; coin+j<=M; j++){
                counts[coin+j] += counts[j];
            }
        }
        cout << counts[M] % 1000000007 <<endl;
    }
    return 0;
}

~~~


---
## 4. Analysis
시간 복잡도는 반복문에 의해 결정되므로 O(C*M)으로 나타낼 수 있다.<br>
최악의 경우 5000 * 100 번의 반복문이 실행된다. 충분히 짧은 시간이며, 실제로 채점서버에서 4ms만큼 걸렸다.


---
## 5. Feedback
동적계획법을 적용해야 하는 문제 중에선 단순히 최적화 때문이 아니라 재귀함수가 아닌 점화식만을 이용해 해결할 수 있는 경우도 있다.

---
