---
layout: post
title:  "출전 순서 정하기"
date:   2017-07-15 23:42:29
author: Heeveloper
categories: Algorithm
tags: 프로그래밍대회에서배우는알고리즘문제해결전략 Algospot 알고스팟 MATCHORDER 출전순서정하기 탐욕법 greedy
excerpt: MATCH ORDER
mathjax : false
---

* content
{:toc}

## 1. Problem
![screenshot](/img/matchorder_problem.png)
<br>

---
## 2. My Approach
* 탐욕법(Greedy-Argorithm)을 적용할 수 있는 기본적인 문제라고 소개되어 있다.
* 하지만 문제를 이해하고 나면 간단한 알고리즘만으로도 해결이 가능한 정도의 난이도다.
* 국내 선수들의 승수를 최대화 하기위해선 러시아 선수들의 레이팅보다 높은 국내 선수들 중 가장 낮은 레이팅의 선수를 배치한다.

---
## 3. Implementation
* 비교연산과 구현을 수월하기 위해서 각 선수단들을 레이팅 오름차순으로 정렬하여 낮은 레이팅의 선수들부터 위치시킨다.
* 반복문을 통해 러시아 선수를 기준(i)으로 해당 선수보다 레이팅이 높은 한국 선수가 나타날 때까지 인덱스(j)를 증가시켜준다.
> 이 때 나타난 한국 선수는 러시아 i번째 선수보다 레이팅이 높은 선수들 중 제일 낮은 레이팅의 선수다.

* 모든 러시아 선수들을 비교했거나, 비교할 한국 선수가 없는 경우 반복문을 종료한다.

~~~c++
sort(RUS.begin(),RUS.end());
sort(KOR.begin(),KOR.end());
int j=0, ret=0;
for(int i=0;i<RUS.size();i++){
    while(j<KOR.size() && RUS[i]>KOR[j])
        j++;
    if(j < KOR.size()){
        j++;
        ret++;
    }
    else break;
}
~~~

---
## 4. Textbook Approach
* 본인의 알고리즘은 문제해결의 원칙을 이해하고 같은 의미의 쉬운 알고리즘을 구현한 반면, 책에선 문제해결의 실제 방식과 동일한 매커니즘을 적용한 알고리즘을 구현했다.
* 한국 선수들을 오름차순으로 정렬한다 (`multiset` 사용: Analysis 참고)
* 해당 러시아 선수보다 높은 레이팅의 선수가 없으면 가장 낮은 레이팅의 한국 선수를 버린다(해당 러시아 선수와 매칭).
* 해당 러시아 선수보다 높은 레이팅의 선수가 있으면 그런 선수들 중 가장 낮은 레이팅의 한국 선수를 버린다(해당 러시아 선수와 매칭).

---
## 5. Implementation
~~~c++
int order(const vector<int>& russian, const vector<int>& korean){
  int n = russian.size(), wins = 0;
  //아직 남아 있는 선수들의 레이팅
  multiset<int> ratings(korean.begin(), korean.end());
  for(int rus=0; rus< n; ++rus){
    //  가장 레이팅이 높은 한국 선수가 이길 수 없는 경우
    //  가장 레이팅이 낮은 선수와 경기시킨다.
    if(*ratings.rbegin() < russian[rus])
      ratings.erase(ratings.begin());
      //이 외의 이길 수 선수 중 가장 레이팅이 낮은 선수와 경기 시킨다.
      else{
        ratings.erase(ratings.lower_bound(russian[rus]));
        ++wins;
      }
  }
  return wins;
}
~~~


---
## 6. Analysis
* 책의 알고리즘에서 소개된 `multiset`은 컨테이너는 기존의 셋(set) 컨테이너와 달리 중복된 key를 여러개 가질 수 있다는 차이가 있다. 그외 부분은 set 컨테이너와 동일하며 사용방법 또한 유사하다.
* `multiset` 사용하는 경우
> 1. 저장된 데이터들을 정렬해야 할 경우.
> 2. 많은 자료들을 저장하고, 특정 자료에 대해 검색을 빠르게 해야 할 경우.
> 3. map이랑 동일, key만 저장하고 key에 대해 정렬할 때 set을 사용.
> 4. set과 동일하나, key 값이 중복된 원소를 가질 수 있는 경우.

* 때문에 `multiset`으로 인해 이길 수 있는 가장 레이팅이 낮은 선수를 찾는 작업과 선택한 선수의 레이팅을 삭제하는 작업 등 모두 O(lgn)에 수행되어 진다.
* 따라서 책의 알고리즘의 전체 시간 복잡도는 O(nlgn)이 된다.

---
## 7. Feedback
* 탐욕적 알고리즘(Greedy-Argorithm) 레시피
> 1. 문제의 답을 만드는 과정을 여러 조각으로 나눈다.
> 2. 각 조각마다 어떤 우선순위로 선택을 내려야 할지 결정한다. 이에 대한 직관을 얻기 위해서는 예제 입력이나 그 외의 작은 입력을 몇 개 손으로 풀어보는 것이 효율적이다.
> 3. 어떤 방식이 동작할 것 같으면 두 가지의 속성을 증명해 보자.
> > a) 탐욕적 선택 속성 : 항상 각 단계에서 우리가 선택한 답을 포함하는 최적해가 존재함을 보이면 된다. 이 증명은 대개 우리가 선택한 답과 다른 최적해가 존재함을 가정하고, 이것을 조작해서 우리가 선택한 답을 포함하는 최적해로 바꿀 수 있음을 보이는 형태로 이뤄진다.<br><br>
> > b) 최적 부분 구조 : 각 단계에서 항상 최적의 선택만을 했을 때 전체 최적해를 구할 수 있는지 여부를 증명한다. 다행히도 대개의 경우 이 속성이 성립하는지 아닌지는 자명하게 알 수 있다.

  ---
