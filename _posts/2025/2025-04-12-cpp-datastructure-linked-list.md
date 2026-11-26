---
layout: post
title: "[C++/자료구조] 템플릿 기반 연결 리스트(Linked List) 직접 구현하기"
date: 2025-04-12 12:53:38 +0900
categories: [c++, dataStructure]
tags: [linkedList, dataStructure, c++]
---

## 연결 리스트란?
- 배열의 한계점
	- 배열은 연속된 메모리를 사용하여 선언할 때 크기가 고정되어 추가, 삭제가 어렵다.
- 연결리스트의 장점
	- 노드의 추가, 삭제가 쉽다.
- 기본 동작
	- 추가
	- 삭제
	- 탐색

## 전체 구조 소개
- **파일 구조 설명**
```bash
├── LinkedList
│   ├── Node.h
│   ├── LinkedList.h
│   └── main.cpp
```

- **클래스 설계 방식**
	- Node 클래스
		- Node 클래스는 데이터를 저장하는 데이터 영역, 다음 노드 주소를 저장하는 주소영역으로 구성된다.
		- 메서드: getData(): 데이터 영역에 저장된 데이터를 반환
		- 메서드: getAddr(): 주소영역의 주소값 반환
		- 메서드: setData(): 데이터 영역에 값 저장
		- 메서드: setAddr(): 주소 영역에 주소값 설정
	- LinkedList 클래스
		- Node 클래스의 주소값을 저장하여 연결 리스트의 시작 지점을 저장한다.
		- 메서드: add(): 연결 리스트의 기본 동작 `추가`
		- 메서드: remove(): 연결 리스트의 기본 동작 `삭제`
		- 메서드: search(): 연결 리스트의 기본 동작 `탐색`
		- 메서드: display(): 연결 리스트에 저장된 값을 출력

## Node 클래스 구현
```cpp
#ifndef NODE_H
#define NODE_H
template <typename T>
class Node {
private:
    T data;
    Node<T>* addr;

public:
    T getData() const {
        return data;
    }

    Node<T>* getAddr() const {
        return addr;
    }

    void setData(T data) {
        this->data = data;
    }

    void setAddr(Node<T>* addr) {
        this->addr = addr;
    }
};
#endif
```
{: file='Node.h'}

## LinkedList 클래스 구현
```cpp
#include "Node.h"
#include <iostream>
#include <sstream>
#include <stdexcept>

#ifndef LINKEDLIST_H
#define LINKEDLIST_H
template <typename T>
class LinkedList {
private:
    Node<T>* head;

public:
    LinkedList(): head(nullptr) { }

    ~LinkedList() {
        Node<T>* p = head;
        while (p != nullptr) {
            Node<T>* next = p->getAddr();
            delete p;
            p = next;
        }
    }

    void add(T data) {
        Node<T>* newNode = new Node<T>;
        newNode->setData(data);
        newNode->setAddr(nullptr);
        
        if (head == nullptr) {
            head = newNode;
        } else {
            Node<T>* p = head;
            while (p->getAddr() != nullptr) {
                p = p->getAddr();
            }
            p->setAddr(newNode);
        }
    }

    void remove() {
        Node<T>* tail = head;
        if (tail == nullptr) {
            throw std::out_of_range("there's no Node.");
        } else if (tail->getAddr() == nullptr) {
            delete head;
            head = nullptr;
            return;
        }
        while (tail->getAddr()->getAddr() != nullptr) {
            tail = tail->getAddr();
        }
        delete tail->getAddr();
        tail->setAddr(nullptr);
    }

    bool search(T target) const {
        Node<T>* p = head;
        while (p != nullptr) {
            if (p->getData() == target) {
                return true;
            }
            p = p->getAddr();
        }
        return false;
    }

    void display() {
        std::ostringstream out;
        
        out << "|| Linked List: (size: ";

        std::string outData;
        int size = 0;
        Node<T>* p = head;
        while (p != nullptr) {
            size ++;
            outData += std::to_string(p->getData()) + " -> ";
            p = p->getAddr();
        }
        out << size << ") | " << outData;

        std::string out_str = out.str();
        out_str = out_str.substr(0, out_str.length()-4) + " ||";

        for (int i = 0; i < out_str.length(); i++) {
            std::cout << "=";
        }
        std::cout << std::endl;

        std::cout << out_str << std::endl;
        
        for (int i = 0; i < out_str.length(); i++) {
            std::cout << "=";
        }
        std::cout << std::endl;
    }
};
#endif
```
{: file='LinkedList.h'}

## 연결 리스트 동작 테스트
- 테스트 코드

```cpp
#include <iostream>
#include "LinkedList.h"

int main() {
    LinkedList<int> list;

    std::cout << "=== 연결 리스트 테스트 시작 ===\n\n";

    // 요소 추가 테스트
    std::cout << "[추가 테스트] 10, 20, 30 추가\n";
    list.add(10);
    list.add(20);
    list.add(30);
    list.display();

    // 탐색 테스트
    std::cout << "\n[탐색 테스트] 20 찾기 → ";
    std::cout << (list.search(20) ? "찾았습니다!" : "없습니다.") << "\n";

    std::cout << "[탐색 테스트] 40 찾기 → ";
    std::cout << (list.search(40) ? "찾았습니다!" : "없습니다.") << "\n";

    // 삭제 테스트
    std::cout << "\n[삭제 테스트] 마지막 노드 제거\n";
    list.remove(); // 30 제거
    list.display();

    std::cout << "\n[삭제 테스트] 또 제거\n";
    list.remove(); // 20 제거
    list.display();

    std::cout << "\n[삭제 테스트] 또 제거\n";
    list.remove(); // 10 제거
    list.display();

    // 빈 리스트에서 삭제 시도 (보완 필요 여부 테스트)
    std::cout << "\n[삭제 테스트] 빈 리스트에서 제거 시도\n";
    list.remove(); // 아무것도 없을 때

    // 다시 추가
    std::cout << "\n[추가 테스트] 99 추가\n";
    list.add(99);
    list.display();

    std::cout << "\n=== 연결 리스트 테스트 종료 ===\n";

    return 0;
}
```
{: file='main.cpp'}

- 실행 결과

```bash
=== 연결 리스트 테스트 시작 ===

[추가 테스트] 10, 20, 30 추가
=============================================
|| Linked List: (size: 3) | 10 -> 20 -> 30 ||
=============================================

[탐색 테스트] 20 찾기 → 찾았습니다!
[탐색 테스트] 40 찾기 → 없습니다.

[삭제 테스트] 마지막 노드 제거
=======================================
|| Linked List: (size: 2) | 10 -> 20 ||
=======================================

[삭제 테스트] 또 제거
=================================
|| Linked List: (size: 1) | 10 ||
=================================

[삭제 테스트] 또 제거
============================
|| Linked List: (size: 0) ||
============================

[삭제 테스트] 빈 리스트에서 제거 시도
there's no Node

[추가 테스트] 99 추가
=================================
|| Linked List: (size: 1) | 99 ||
=================================

=== 연결 리스트 테스트 종료 ===
```
{: file='results'}

## 마무리
- 연결 리스트의 기본 동작 방식을 코드로 구현하면서 개념을 더욱 확실하게 익힐 수 있었다.
- `Node.h`, `LinkedList.h` 등으로 구조를 나누어 가독성과 재사용성을 높임.
- 템플릿을 적용해 자료형에 상관 없이 사용할 수 있도록 확장성을 높임.

### 추후 계획
- 양방향 연결 리스트를 구현
