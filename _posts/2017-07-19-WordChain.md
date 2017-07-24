---
layout: post
title:  "단어 제한 끝말잇기"
date:   2017-07-19 23:50:29
author: Heeveloper
categories: Algorithm
tags: 프로그래밍대회에서배우는알고리즘문제해결전략 Algospot 알고스팟 WORDCHAIN 끝말잇기 graph 깊이우선탐색 오일러서킷 EULER dfs
excerpt: WORDCHAIN
mathjax : false
---

* content
{:toc}

## 1. Problem
![screenshot](/img/wordchain_problem.png)
<br>

---
## 2. My Approach
* 입력 받은 단어들간의 시작단어와 끝단어를 비교하여 그래프를 만든다.
> words[i]의 시작단어와  words[j]의 끝단어가 일치하면 인접행렬 adj[i][j] 에 체크한다.

* 임의의 단어를 시작으로 설정하여 연결되는 모든 경우로 `dfs`한다.
* `dfs`
> 1. 연결되는 다음 단어가 존재하여 해당 단어로 이동하여 `dfs`할 때마다 깊이를 의미하는 변수를 하나씩 증가해준다.
> 2. 해당 변수가 총 단어의 갯수에 대응하는 값이 되면 모든 단어가 이어진 것이므로 특별한 값을 반환해준다.
> 3. 위의 반환값으로 인해 `backtracking` 과정에서 해당 경로의 단어들을 기록한다.

---
## 3. Implementation

~~~c++
//
//  main.cpp
//  Wordchain
//
//  Created by  Heejun Lee on 7/21/17.
//  Copyright © 2017  Heejun Lee. All rights reserved.
//
#include <iostream>
#include <vector>
#include <algorithm>

using namespace std;

int T,N;
vector<string> words;
vector<vector<int>> adj;
vector<int> checked;
vector<int> ordered;

void makeGraph(){
    adj = vector<vector<int>>(N,vector<int>(N,0));
    for(int i=0;i<N;i++)
        for(int j=0;j<N;j++){
            if(i==j) adj[i][j] = 0;
            else if(words[i].back() == words[j].front()) adj[i][j] = 1;
        }
}

int dfs(int here, int count){
    checked[here] = 1;
    int ret=0;
    if(count == N-1){
        ordered.push_back(here);
        return 1;
    }
    for(int there=0;there<N;there++)
        if(here != there && adj[here][there] && !checked[there]){
            ret = dfs(there, count+1);
            if(ret)
                ordered.push_back(here);
        }
    return ret;
}

vector<int> topologySort(){
    for(int start=0;start<N;start++){
        checked.clear();
        checked = vector<int>(N,0);
        ordered.clear();
        if(dfs(start, 0) != 0) break;
    }
    if(ordered.empty()) return vector<int>();
    else return ordered;
}

int main() {
    cin >> T;
    while(T--){
        cin >> N;
        words = vector<string>(N);
        for(int i=0;i<N;i++)
            cin >> words[i];
        makeGraph();
        vector<int> ret = topologySort();
        reverse(ret.begin(), ret.end());
        if(ret.empty())
            cout << "IMPOSSIBLE" << endl;
        else{
            for(int i=0;i<ret.size();i++)
                cout << words[ret[i]] + " ";
            cout<<endl;
        }
    }
    return 0;
}

~~~

* 그러나 채점서버를 돌려보면 오답이 나온다.
* 미처 생각하지 못한 생황이 있는 것 같다...

---
## 4. Textbook Approach
* 해밀토니안 경로(Hamiltonian path)와 오일러 트레일
> * 해밀토니안 경로 : 그래프의 모든 정점을 정확히 한 번씩 지나가는 경로
> > 해밀토니안 경로를 찾는 유일한 방법은 `조합탐색`으로, 모든 정점의 배열을 하나하나 시도하며 이들이 경로가 되는지를 확인하는 것이다. 이 방법으로 문제를 풀기 위해서는 최악의 경우 N!개의 후보를 만들어야 하는데, 이 문제에선 단어의 수가 최대 100이기 때문에 이렇게는 절대로 답을 찾을 수 없다.
> * 오일러 트레일 : 오일러 서킷처럼 모든 간선을 정확히 한 번씩 지나가지만, 시작점과 끝점이 다른 경로
> > 시작점과 끝점을 잇는 간선을 하나 추가한 뒤 모든 점이 짝수점이 되려면, 시작점과 끝점을 제외한 모든 점은 짝수점이고 시작점과 끝점은 홀수점이 되어야 한다.

* 따라서 본 문제는 방향 그래프에서 오일러 서킷을 찾는다.
> * 각 정점에 들어오는 간선의 수와 나가는 간선의 수가 같아야 한다.

* 오일러 서킷을 이용해 오일러 트레일을 찾는다.
> * a에서 시작하고 b에서 끝나는 오일러 트레일을 찾기 위해선 간선 (b,a)를 그래프에 추가한 뒤 오일러 서킷을 찾아야 한다.
> * 이 떄 오일러 서킷의 존재 조건이 만족되려면 a에서는 나가는 간선이 들어오는 간선보다 하나 많고, b는 들어오는 간선이 나가는 간선보다 하나 많고, 다른 모든 정점에서는 나가는 간선과 들어오는 간선의 수가 같아야 한다.

* 본 문제의 답은 오일러 서킷일 수도 있고 오일러 트레일일 수도 있다.
> 각 정점의 차수를 확인하여 오일러 서킷과 트레일을 구분한다.

---
## 5. Implementation
~~~c++
#include <stdio.h>
#include <string.h>
#include <iostream>
#include <vector>
#include <algorithm>

using namespace std;
vector<vector<int>> adj;
vector<string> graph[26][26];
vector<int> indegree, outdegree;

void getEulerCircuit(int u, vector<int>& circuit){
    for (int v = 0; v < adj.size(); ++v){
        while (adj[u][v]>0){
            adj[u][v]--;
            getEulerCircuit(v, circuit);
        }
    }
    circuit.push_back(u);
}

vector<int> getEulerTrailOrCircuit(){
    vector<int> circuit;
    for (int i = 0; i < 26; ++i){
        if (outdegree[i] == indegree[i] + 1){ //    무조건 시작지점인 문자 있는 지 체크 후 그걸로 오일러트레일러 생성
            getEulerCircuit(i, circuit);
            return circuit;
        }
    }
    for (int i = 0; i < 26; ++i){ //    오일러서킷이므로 간선 있는 경우부터 시작
        if (outdegree[i]){
            getEulerCircuit(i,circuit);
            return circuit;
        }
    }
    return circuit;
}


void makeGraph(const vector<string>& words){
    for (int i = 0; i < 26; ++i){
        for (int j = 0; j < 26; ++j){
            graph[i][j].clear();
        }
    }
    adj = vector<vector<int>>(26,vector<int>(26,0));
    indegree = outdegree = vector<int>(26, 0);

    for (int i = 0; i < words.size(); ++i){
        int a = words[i][0] - 'a';
        int b = words[i][words[i].size() - 1] - 'a';
        graph[a][b].push_back(words[i]);
        adj[a][b]++;
        outdegree[a]++;
        indegree[b]++;
    }
}

bool checkEuler(){
    int plus1 = 0, minus1 = 0;
    for (int i = 0; i < 26; ++i){
        int delta = outdegree[i] - indegree[i];
        if (delta < -1 || delta > 1) return false;
        if (delta == 1) plus1++;
        if (delta == -1) minus1++;
    }
    //Euler Trail || Euler Circuit
    return (plus1 == 1 && minus1 == 1) || (plus1==0 && minus1==0);
}

string solve(const vector<string>& words){
    makeGraph(words);
    if (!checkEuler()) return "IMPOSSIBLE";

    vector<int> circuit = getEulerTrailOrCircuit();
    if (circuit.size() != words.size() + 1) return "IMPOSSIBLE";

    reverse(circuit.begin(), circuit.end());
    string ret;
    for (int i = 1; i < circuit.size(); ++i){
        int a = circuit[i - 1], b = circuit[i];
        if (ret.size()) ret += " ";
        ret += graph[a][b].back();
        graph[a][b].pop_back();
    }
    return ret;
}

int main(void){
    int C, N;
    cin >> C;

    for (int test = 1; test <= C; ++test){
        cin >> N;

        vector<string> words = vector<string>(N);
        char ch[11];
        for (int i = 0; i < N; ++i){
            getchar();
            scanf("%s", ch);
            words[i] = ch;
        }
        string str = solve(words);
        printf("%s\n", str.c_str());
    }
    return 0;
}

~~~


---
## 6. Analysis
책의 코드의 시간 복잡도는 오일러 트레일을 찾는 함수에 의해 지배된다.<br>
그래프를 만드는 과정과 답으로 출력할 문자열을 만드는 과정은 모두 단어의 수에 비례하는 시간이 걸리는데, 오일러 트레일을 찾는 함수의 수행 시간은 알파벳의 수 A와 단어의 수 N의 곱에 비례하기 때문이다.<br> 따라서 전체 시간 복잡도는 O(NA)가 된다.


---
## 7. Feedback
깊이 우선 탐색은 그래프의 구조에 관련된 많은 문제를 푸는 데 사용될 수 있다.<br>
깊이 우선 탐색을 수행하면 그 과정에서 그래프의 모든 간선을 한 번씩은 만나게 되는데, 그 중 일부 간선은 처음 발견한 정점으로 연결되어 있어서 우리가 따라갈 테고, 나머지는 무시하게 된다.<br>
그러나 이 간선들을 무시하지 않고 이들에 대한 정보를 수집하면 그래프의 구조에 대해 많은 것을 알 수 있다.<br>
이러한 문제들을 해결하기 위해서 깊이 우선 탐색이 하는 일을 좀 더 자세히 살펴볼 필요성을 느낀다. 시간을 내서 따로 정리를 하도록 해야겠다.


---
