---
layout: post
title: "[프로그래머스/swift] 프로그래머스 완전범죄 DP 를 이용한 풀이"
date: 2025-07-11 22:04:50 +0900
categories: [프로그래머스, DP, BFS]
tags: [swift, DP, BFS, 프로그래머스]
---

## 문제
[프로그래머스_완전범죄](https://school.programmers.co.kr/learn/courses/30/lessons/389480)

### 문제 설명
- 도둑 A, B가 물건을 훔치려고 한다.
- A, B가 물건을 훔쳤을 때 남기는 흔적 수, 경찰에 붙잡히는 조건을 줄 때 둘 다 안잡히고, A도둑이 남긴 흔적의 누적 개수가 최소인 값을 return하는 함수를 작성

## 문제 분석
- 현재 A, B의 흔적을 저장하는 Node 객체 생성
- 한 물건을 훔칠 때마다 Node 는 업데이트 될것임!
- 처음 물건부터 마지막 물건까지
	- A가 훔쳤을 경우의 Node
	- B가 훔쳤을 경우의 Nod
- 위 노드들을 순차적으로 탐색한 결과 중
	- A 가 경찰에게 붙잡히지 않고,
	- B 가 경찰에게 붙잡히지 않고,
- 위 두 조건을 모두 만족하는 결과에서 A의 흔적이 가장 작은 결과를 반환

### 문제 코드
```swift
import Foundation

class Node {
    let remA: Int
    let remB: Int
    
    init(remA: Int, remB: Int) {
        self.remA = remA
        self.remB = remB
    }
    
    func newNode(a: Int = 0, b: Int = 0) -> Node {
        let newNode = Node(remA: remA + a, remB: remB + b)
        return newNode
    }
}

func solution(_ info:[[Int]], _ n:Int, _ m:Int) -> Int {
    var stack: [Node] = [ Node(remA: 0, remB: 0) ]
    
    for item in info {
        // 스택에서 값을 모두 뺸 후 저장
        var tempStack: [Node] = []
        while !stack.isEmpty {
            tempStack.append(stack.popLast()!)
        }
        
        for node in tempStack {
            if node.remA + item[0] < n {
                stack.append(node.newNode(a: item[0]))
            }
            if node.remB + item[1] < m {
                stack.append(node.newNode(b: item[1]))
            }
        }
        if tempStack.isEmpty {
            break
        }
    }
    // 답이 없으면 -1 리턴
    if stack.isEmpty {
        return -1
    }
    // stack에서 최소값 탐색
    var result = n
    for node in stack {
        if node.remA < result {
            result = node.remA
        }
    }
    return result
}
```

### 결과
> 실패!
> 시간초과가 많이 발생했다


### 결과 분석
- 시간초과가 뜬 것이면 역시 스택에서 값을 모두 꺼내고, 새로운 스택에 저장하는 비효율적인 자료구조를 사용해서 그런 것인가?
	- Queue를 사용하여 풀이 해 보았지만, 동일하게 시간초과가 발생함
- 모든 경우를 탐색하는 방식의 비효율이 문제인가?
	- DP (Dynamic Programming)의 개념을 알게 되었다!

### 본 문제에 DP 적용
- i 번째 물건까지 훔쳤을 경우, Node 가 동일하다면 어떤 경로로 왔던 동일한 노드라고 생각하여도 무방함!
	- 스택에 새로운 노드를 추가할 때 해당 노드가 스택에 있는지를 확인하여 없는 경우에만 삽입!

### DP 를 활용한 문제 해결 코드
```swift
import Foundation

struct Node: Equatable {
    let remA: Int
    let remB: Int
    
    init(remA: Int, remB: Int) {
        self.remA = remA
        self.remB = remB
    }
    func newNode(a: Int = 0, b: Int = 0) -> Node {
        let newNode = Node(remA: remA + a, remB: remB + b)
        return newNode
    }
}

func solution(_ info:[[Int]], _ n:Int, _ m:Int) -> Int {
    var stack: [Node] = [ Node(remA: 0, remB: 0) ]
    
    for item in info {
        // 스택에서 값을 모두 뺸 후 저장
        var tempStack: [Node] = []
        while !stack.isEmpty {
            tempStack.append(stack.popLast()!)
        }
        
        for node in tempStack {
            let newNodeA = node.newNode(a: item[0])
            let newNodeB = node.newNode(b: item[1])
            
            if newNodeA.remA < n && !stack.contains(newNodeA) {
                stack.append(newNodeA)
            }
            if newNodeB.remB < m && !stack.contains(newNodeB) {
                stack.append(newNodeB)
            }
        }
        if stack.isEmpty {
            return -1
        }
    }
    // stack에서 최소값 탐색
    var result = n
    for node in stack {
        if node.remA < result {
            result = node.remA
        }
    }
    return result
}
```


## 결론
- 경우의 수를 탐색하는 경우, (본 문제에서는 BFS) 경우의 수가 많아지게 된다면 중복을 최소화하는 DP의 적용 방안을 생각해 보는 것이 좋다!



