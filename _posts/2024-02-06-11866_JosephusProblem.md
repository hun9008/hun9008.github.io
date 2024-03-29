---
layout: post
title: 백준 요세푸스 문제0 [c++]
author: Hun
tags:
- PS
- DP
date: 2024-02-06 23:18 +0800
last_modified_at: 2024-02-06 01:08:25 +0800
toc: true
---

# 요세푸스 문제0 실버5

## 문제
요세푸스 문제는 다음과 같다.

1번부터 N번까지 N명의 사람이 원을 이루면서 앉아있고, 양의 정수 K(≤ N)가 주어진다. 이제 순서대로 K번째 사람을 제거한다. 한 사람이 제거되면 남은 사람들로 이루어진 원을 따라 이 과정을 계속해 나간다. 이 과정은 N명의 사람이 모두 제거될 때까지 계속된다. 원에서 사람들이 제거되는 순서를 (N, K)-요세푸스 순열이라고 한다. 예를 들어 (7, 3)-요세푸스 순열은 <3, 6, 2, 7, 5, 1, 4>이다.

N과 K가 주어지면 (N, K)-요세푸스 순열을 구하는 프로그램을 작성하시오.

## 입력
첫째 줄에 N과 K가 빈 칸을 사이에 두고 순서대로 주어진다. (1 ≤ K ≤ N ≤ 1,000)

## 출력
예제와 같이 요세푸스 순열을 출력한다.

# 풀이
- 큐를 이용한다.
- 큐에 1~N까지 삽입 후, k번 pop & push함. k번째는 다시 Push하지 않고 출력한다. 큐가 빌때까지 이 과정 반복.

# 코드
{% highlight c++ %}
#include <iostream>
#include <queue>
using namespace std;

int main(){

    int n, k;
    cin >> n >> k;
    queue<int> q;
    
    for(int i = 0; i < n; i++)
    {
        q.push(i + 1);
    }

    cout << "<";
    while(q.size() != 1)
    {
        for(int i = 0; i < k - 1; i++)
        {
            int temp = q.front();
            q.pop();
            q.push(temp);
        }
        int pick = q.front();
        cout << pick << ", ";
        q.pop();
    }

    cout << q.front() << ">" << endl;


    return 0;
}
{% endhighlight %}