---
title: "[백준] 가운데를 말해요"
author: "hvunrnin"
date: 2025-06-05 10:00:00 
categories: [Algorithm, Java]
tags: [Algorithm, Java]
math: true
toc: true
pin: true
---

## 문제
백준 1655. 가운데를 말해요

### 문제
백준이는 동생에게 "가운데를 말해요" 게임을 가르쳐주고 있다. 백준이가 정수를 하나씩 외칠때마다 동생은 지금까지 백준이가 말한 수 중에서 중간값을 말해야 한다. 만약, 그동안 백준이가 외친 수의 개수가 짝수개라면 중간에 있는 두 수 중에서 작은 수를 말해야 한다.

예를 들어 백준이가 동생에게 1, 5, 2, 10, -99, 7, 5를 순서대로 외쳤다고 하면, 동생은 1, 1, 2, 2, 2, 2, 5를 차례대로 말해야 한다. 백준이가 외치는 수가 주어졌을 때, 동생이 말해야 하는 수를 구하는 프로그램을 작성하시오.

### 입력
첫째 줄에는 백준이가 외치는 정수의 개수 N이 주어진다. N은 1보다 크거나 같고, 100,000보다 작거나 같은 자연수이다. 그 다음 N줄에 걸쳐서 백준이가 외치는 정수가 차례대로 주어진다. 정수는 -10,000보다 크거나 같고, 10,000보다 작거나 같다.
N개의 수가 차례대로 주어질 때마다, 지금까지 입력받은 수들 중 가운데 있는 수를 즉시 출력해야 합니다.

### 요악하면 
N개의 수가 차례대로 주어질 때마다, 지금까지 입력받은 수들 중 가운데 있는 수를 즉시 출력하는 문제이다.

## 처음 시도 (단일 PriorityQueue + 중간값 재계산)

안 좋은 풀이인 걸 알았는데 다른 풀이가 생각이 안 나서 일단..

모든 수를 PriorityQueue에 넣고, 중간값을 구하기 위해 절반 정도를 poll()로 꺼낸 뒤 peek한 다음 다시 넣는 방식으로 구현을 했다. 

```java
for(int i=2;i<n;i++){
    pq.add(sc.nextInt());
    int[] tmp = new int[pq.size()/2];
    int size =0;

    if(pq.size()%2==0) size = pq.size()/2-1;
    else size = pq.size()/2;
    for(int j=0;j<size;j++){
        tmp[j]=pq.poll();
    }
    answer.add(pq.peek());
    for(int j=0;j<size;j++){
        pq.add(tmp[j]);
    }
        }
```

이렇게 했더니..

#### 메모리 초과!가 나왔다.

poll()을 반복하면서 PriorityQueue 내부가 계속 재정렬되고, 거기다 꺼낸 값들을 배열에 따로 저장했다가 다시 넣는 과정에서 메모리와 시간이 너무 많이 소모됐다.
특히 입력 개수가 최대 100,000개까지 주어질 수 있는데, 매 입력마다 poll()을 여러 번 하고 다시 add()하는 건 시간 복잡도 면에서도 비효율적이었다.

## 바꾼 방법 (MaxHeap + MinHeap 두 개로 중간값 유지)
그래서 좀 방법을 찾아보고 힙을 두 개 쓰는 방식으로 바꿨다. 생각하지 못한 방법으로 중간값 유지를 할 수 있었다.

- MaxHeap(down)은 중간값 이하의 수들을 담고
- MinHeap(up)은 중간값 초과의 수들을 담는다

-> 이렇게 나눠서 넣으면 항상 MaxHeap의 peek()이 현재까지 입력된 수들의 중간값이 된다.

```java
PriorityQueue<Integer> down = new PriorityQueue<>(Collections.reverseOrder()); // MaxHeap
PriorityQueue<Integer> up = new PriorityQueue<>(); // MinHeap

for (int i = 0; i < n; i++) {
    down.add(sc.nextInt());
    up.add(down.poll());

    if (down.size() < up.size()) {
        down.add(up.poll());
    }

    sb.append(down.peek()).append("\n"); // 중간값 출력
}
```

### 동작 방식 요약
- 새 숫자를 down에 넣는다
- 그중 가장 큰 값을 up으로 이동시킨다
- 만약 up의 크기가 down보다 커졌다면 다시 하나를 down으로 이동시킨다
- 이제 중간값은 항상 down.peek()

이렇게 했더니 시간 초과, 메모리 초과 없이 잘 통과 되었고 시간 복잡도도 O(log N)이라 입력 수가 많아도 문제없었다.

<br>

처음에는 왜 힙을 두 개 써야 하는지 감이 안 잡혔는데, 직접 구현하고 나니까 훨씬 이해가 잘 됐다.

다음부터는 중간값 문제 나오면 바로 MaxHeap + MinHeap을 생각해보기..