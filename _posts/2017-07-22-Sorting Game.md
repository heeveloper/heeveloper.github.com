---
layout: post
title:  "Sorting Game"
date:   2017-07-20 15:28:22
author: Heeveloper
categories: Algorithm
tags: 프로그래밍대회에서배우는알고리즘문제해결전략 Algospot 알고스팟 SORTGAME graph 너비우선탐색 bfs
excerpt: SORTGAME
mathjax : false
---

* content
{:toc}

## 1. Problem
![screenshot](/img/sortgame_problem.png)
<br>

---
## 2. My Approach
* 주어진 숫자 배열(N)이 정렬되었는 지 확인한다.
* 정렬돼 있지 않으면 다음과 같은 순서대로 바꿔서 큐에 저장한다.
> * 배열의 인덱스 i(=0~N-R)부터 위치한 R(=뒤집을 연속된 2~N개의 숫자)개의 숫자들의 순서를 뒤집는다.
> * 즉 처음엔 2개의 숫자들을 바꿔주어 큐에 저장. (인덱스 0과 1, 1과 2, 2와 3,..)
> * 다음엔 3개의 숫자들을 바꿔주어 큐에 저장. (인덱스 0~2, 1~3,..)

* 큐에 저장된 배열 중 정렬된 것이 있는지 모두 검사한다.
* 없으면 뒤집혀진 배열들을 한 번 더 뒤집어 큐에 저장한다.
* 즉 한 번 뒤집은 경우를 모두 검사한 뒤에야 두 번 뒤집은 경우를 검사하게 된다.
* 따라서 정렬된 배열이 등장했을 때 그 때까지 뒤집은 횟수가 최소로 뒤집은 횟수가 된다.

---
## 3. Implementation

* 두 개의 큐를 선언한다.
> 1. 정렬되었는 지 확인할 큐 : 뒤집혀진 숫자 배열들이 저장.
> 2. 뒤집혀질 큐 : 정렬 확인 결과 정렬되어 있지 않아 뒤집혀질 배열 저장.

* 위의 내용을 별도의 최적화 없이 있는 그대로 구현하면 걸리는 시간도 문제이지만 뒤집어진 배열들을 저장하는 큐의 크기도 기하급수적으로 증가한다...

~~~c++
//
//  main.cpp
//  SortGame
//
//  Created by  Heejun Lee on 7/23/17.
//  Copyright © 2017  Heejun Lee. All rights reserved.
//

#include <iostream>
#include <vector>
#include <algorithm>
#include <queue>

using namespace std;

int T, N;
queue<pair<vector<int>,int>> Q;
queue<vector<int>> checked;

bool IsSorted(vector<int>& nums){
    for(int i=1;i<nums.size();i++)
        if(nums[i-1]>nums[i])
            return false;
    return true;
}

vector<int> reverse2(vector<int>& nums, int length, int start){
    vector<int> ret;
    for(int i=0;i<start;i++)
        ret.push_back(nums[i]);
    for(int i=start+length-1;i>=start;i--)
        ret.push_back(nums[i]);
    for(int i=start+length;i<nums.size();i++)
        ret.push_back(nums[i]);
    return ret;
}

int sort(vector<int>& nums){
    int count = 0;
    vector<int> sorted = nums;
    sort(sorted.begin(),sorted.end());
    Q.push(make_pair(nums, count));
    while(!Q.empty()){
        while(!Q.empty()){
            vector<int> here = Q.front().first;
            count = Q.front().second;
            if(here == sorted) return count;
            checked.push(here);
            Q.pop();
        }
        int k=0;
        while(!checked.empty()){
            for(int i=2;i<nums.size()+1;i++)
                for(int j=0;j<nums.size()-i+1;j++)
                    Q.push(make_pair(reverse2(checked.front(), i, j), count+1));
            checked.pop();
            k++;
        }
    }
    return count;
}

int main(int argc, const char * argv[]) {
    cin >> T;
    while(T--){
        cin >> N;
        vector<int> nums = vector<int>(N,0);
        for(int i=0;i<N;i++)
            cin >> nums[i];
        cout << sort(nums) << endl;

        while(!Q.empty())
            Q.pop();
        while(!checked.empty())
            checked.pop();
    }
    return 0;
}
~~~

* 시간 초과 이전에 메모리 초과로 **Runtime Error** 가 발생한다.

---
## 4. Textbook Approach

두가지의 풀이가 주어진다.<br>
1. 본인의 코드와 비슷하지만 단일 `Queue`와 `map`을 사용하여 메모리가 커지는 것은 막았지만 시간 초과는 여전히 일어난다.<br>
2. 다른 하나는 1이상 8 이하의 모든 N에 대해 정렬된 배열에서 다른 모든 상태까지 도달하는 데 필요한 뒤집기 연산의 수를 너비 우선 탐색을 이용해 미리 계산해 두는 것이다.
> 입력되는 숫자의 범위는 1백만 이하의 정수들이라 매우 많지만, 이 들의 정렬에는 숫자들간의 상대적 크기만 고려되므로 이를 일반화 할 수 있다. 예를 들어 [2, 10, 5, 100]의 숫자 배열은 [0,2,1,3]의 숫자 배열을 정렬하는 것으로 봐도 무방하다.

---
## 5. Implementation

**1. 단일 큐와 map 사용 알고리즘**<br>
* `map<vector<int>, int> distance` 값에 뒤집은 숫자 배열과 그 때까지 뒤집은 갯수를 저장한다.
* 또한 `distance.count( )`를 통해 중복되는 경우가 있을 경우 연산을 생략하여 소요 메모리와 시간을 줄여준다.

~~~c++

int bfs(const vector<int>& perm){
    int n = perm.size();
    //  목표 정점을 미리 계산한다.
    vector<int> sorted = perm;
    sort(sorted.begin(), sorted.end());
    //  방문 목록(큐)과 시작점부터 각 정점까지의 거리
    queue<vector<int>> q;
    map<vector<int>, int> distance;
    //  시작점을 큐에 넣는다.
    distance[perm] = 0;
    q.push(perm);
    while(!q.empty()){
        vector<int> here = q.front();
        q.pop();
        //  목표 정점을 발견했으면 곧장 종료한다.
        if(here == sorted) return distance[here];
        int cost = distance[here];
        //  가능한 모든 부분 구간을 뒤집어 본다.
        for(int i=0;i<n;i++)
            for(int j=i+2; j<=n;j++){
                reverse(here.begin()+i, here.begin()+j);
                if(distance.count(here)==0){
                    distance[here] = cost + 1;
                    q.push(here);
                }
                reverse(here.begin()+i, here.begin()+j);
            }
    }
    return -1;
}
~~~
<br><br>
**2. 미리 계산하는 알고리즘**
* `precalc` : 입력된 숫자 배열의 크기만큼 정렬된 배열을 생성 후, 해당 배열에서 뒤집혀서 만들어질 수 있는 모든 경우의 수를 `map<vector<int>,int> toSort`에 저장해 놓는다.
* `solve` : 입력된 숫자 배열을 상대적 크기에 맞게 0부터 N-1의 값으로 바꾼 뒤 toSort의 해당 배열일 때의 키 값, 즉 뒤집는 횟수를 반환한다.

~~~c++
map<vector<int>, int> toSort;

//[0,..n-1]의 모든 순열에 대해 toSort[]를 계산해 저장한다.
void precalc(int n){
    vector<int> perm;
    for(int i=0;i<n;i++)
        perm[i] = i;
    queue<vector<int>> q;
    q.push(perm);
    toSort[perm] = 0;
    while(!q.empty()){
        vector<int> here = q.front();
        q.pop();
        int cost = toSort[here];
        for(int i=0;i<n;i++){
            for(int j=0;j<n;j++){
                reverse(here.begin()+i, here.begin()+j);
                if(toSort.count(here) == 0){
                    toSort[here] = cost+1;
                    q.push(here);
                }
                reverse(here.begin()+i, here.begin()+j);
            }
        }
    }
}
int solve(const vector<int>& perm){
    //  perm을 [0,...,n-1]의 순열로 변환한다.
    int n = perm.size();
    vector<int> fixed(n);
    for(int i=0;i<n;i++){
        int smaller = 0;
        for(int j=0;j<n;j++)
            if(perm[j]<perm[i])
                ++smaller;
        fixed[i] = smaller;
    }
    return toSort[fixed];
}
~~~

* 상대적 크기의 숫자로 바꾸는 연산도 눈여겨 보자.

---
## 6. Analysis
* 최종적으로 구현된 코드의 시간복잡도는 입력된 숫자 배열의 크기 N에 종속되어, N! 개의 경우의 수를 생성하는 데 걸리는 시간이 지배적이게 된다.
* 처음에 `precalc(8)`로 `toSort`를 초기화 해놓으면 이후 모든 테스트 케이스의 경과시간은 map에서 해당 값만을 찾는

---
## 7. Feedback
* bfs로 최단경로를 구할 때, 별도의 최적화 처리가 없으면 메모리 초과와 시간초과를 유발하기 쉽다.
* 만들어진 그래프가 양방향 그래프일 때 목표값으로부터 반대로 접근하는 방법을 생각해 볼 필요가 있다.
* **map**
> * **Red-Black Tree**를 사용해 키의 순서에 맞게 저장되어 있어, 접근 시 O(lgN)의 시간복잡도를 갖진다.
> * 동적계획법의 `cache`와 같이 쓰일 수 있다.

---
