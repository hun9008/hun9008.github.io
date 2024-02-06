---
layout: post
title: 백준 유기농배추 [c++]
author: Hun
tags:
- PS
- BFS
date: 2024-02-06 23:18 +0800
last_modified_at: 2024-02-06 01:08:25 +0800
toc: true
---

# 유기농배추 실버2

## 문제
차세대 영농인 한나는 강원도 고랭지에서 유기농 배추를 재배하기로 하였다. 농약을 쓰지 않고 배추를 재배하려면 배추를 해충으로부터 보호하는 것이 중요하기 때문에, 한나는 해충 방지에 효과적인 배추흰지렁이를 구입하기로 결심한다. 이 지렁이는 배추근처에 서식하며 해충을 잡아 먹음으로써 배추를 보호한다. 특히, 어떤 배추에 배추흰지렁이가 한 마리라도 살고 있으면 이 지렁이는 인접한 다른 배추로 이동할 수 있어, 그 배추들 역시 해충으로부터 보호받을 수 있다. 한 배추의 상하좌우 네 방향에 다른 배추가 위치한 경우에 서로 인접해있는 것이다.

한나가 배추를 재배하는 땅은 고르지 못해서 배추를 군데군데 심어 놓았다. 배추들이 모여있는 곳에는 배추흰지렁이가 한 마리만 있으면 되므로 서로 인접해있는 배추들이 몇 군데에 퍼져있는지 조사하면 총 몇 마리의 지렁이가 필요한지 알 수 있다. 예를 들어 배추밭이 아래와 같이 구성되어 있으면 최소 5마리의 배추흰지렁이가 필요하다. 0은 배추가 심어져 있지 않은 땅이고, 1은 배추가 심어져 있는 땅을 나타낸다.

## 입력
입력의 첫 줄에는 테스트 케이스의 개수 T가 주어진다. 그 다음 줄부터 각각의 테스트 케이스에 대해 첫째 줄에는 배추를 심은 배추밭의 가로길이 M(1 ≤ M ≤ 50)과 세로길이 N(1 ≤ N ≤ 50), 그리고 배추가 심어져 있는 위치의 개수 K(1 ≤ K ≤ 2500)이 주어진다. 그 다음 K줄에는 배추의 위치 X(0 ≤ X ≤ M-1), Y(0 ≤ Y ≤ N-1)가 주어진다. 두 배추의 위치가 같은 경우는 없다.

## 출력
각 테스트 케이스에 대해 필요한 최소의 배추흰지렁이 마리 수를 출력한다.

# 풀이
- 그래프 탐색 문제로 bfs사용하기로 함.
- 조건대로 입력받고 bfs하면되는데 그냥 bfs 호출 횟수 카운트하면 간단히 해결가능했음.

# 코드
{% highlight c++ %}
#include <iostream>
#include <vector>
#include <queue>
using namespace std;

{% raw %}
int dir[2][4] = {{0, 0, 1, -1}, {1,-1,0,0}};
{% endraw %}
int visited[51][51];
int ground[51][51];
int cnt = 0;

void init()
{
    for(int i = 0; i < 51; i++)
    {
        for(int j = 0; j < 51; j++)
        {
            visited[i][j] = 0;
            ground[i][j] = 0;
        }
    }
}

void bfs(int x, int y, int m, int n){
    queue <pair<int, int> > q;
    q.push(make_pair(x, y));
    visited[x][y] = 1;

    while(!q.empty()){
        int pos_x = q.front().first;
        int pos_y = q.front().second;
        q.pop();
        for(int i = 0; i < 4; i++)
        {
            int nx = pos_x + dir[0][i];
            int ny = pos_y + dir[1][i];
            if(nx >= 0 && nx < m && ny >= 0 && ny < n){
                if(visited[nx][ny] == 0 && ground[nx][ny] == 1){
                    q.push(make_pair(nx, ny));
                    visited[nx][ny] = 1;
                }
            }
        }
    }
    cnt++;
}

int main(){

    int t;
    cin >> t;
    for(int i = 0; i < t; i++)
    {
        int m, n, k;
        init();
        cin >> m >> n >> k;
        for(int j = 0; j < k; j++)
        {
            int t_m, t_n;
            cin >> t_m >> t_n;
            ground[t_m][t_n] = 1;
        }
        for(int e = 0; e < m; e++)
        {
            for(int r = 0; r < n; r++){
                if(ground[e][r] == 1 && visited[e][r] == 0){
                    bfs(e,r,m,n);
                }
            }
        }
        cout << cnt << endl;
        cnt = 0;
    }

    return 0;
}
{% endhighlight %}

