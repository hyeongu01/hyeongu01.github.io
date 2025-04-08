---
layout: post
title: "C++ 배열 스타일 총정리: vector vs array vs new"
date: 2025-04-09 00:47:03 +0900
categories: [c++]
tags: [c++, vector, array, new]
---

C++에서 배열을 선언하고 사용하는 방식은 다양합니다.  
이번 글에서는 대표적인 세 가지 배열 스타일인 `std::vector`, `std::array`, 그리고 `new`를 활용한 **동적 할당 방식**을 비교해보겠습니다.

---

## 1. `std::vector` (가변 크기 동적 배열)

```cpp
#include <vector>

std::vector<int> arr(10);        // 크기 10, 값은 0
std::vector<int> arr2(10, -1);   // 크기 10, 모든 값 -1
```

- **런타임에 크기 지정 가능**
- 자동으로 메모리 관리
- STL 함수와 호환 (`sort`, `find`, `push_back`, …)

---

## 2. `std::array` (정적 크기 배열)

```cpp
#include <array>

std::array<int, 5> arr = {1, 2, 3, 4, 5};
```

- **컴파일 타임에 크기 고정**
- C-style 배열보다 안전
- STL 컨테이너처럼 동작

---

## 3. C-style 동적 배열 (`new`)

```cpp
int* arr = new int[n];
std::fill(arr, arr + n, 0);
delete[] arr;
```

- 런타임 크기 지정 가능
- **직접 메모리 해제 필요 (`delete[]`)**
- 범위 검사 없음 → 위험

---

## 2차원 배열 예시 비교

### `vector` 기반 2D 배열

```cpp
int M = 3, N = 4;
std::vector<std::vector<int>> field(M, std::vector<int>(N, -1));
```

### `new` 기반 2D 배열

```cpp
int** arr = new int*[M];
for (int i = 0; i < M; ++i) {
    arr[i] = new int[N];
    std::fill(arr[i], arr[i] + N, -1);
}

// 메모리 해제
for (int i = 0; i < M; ++i) delete[] arr[i];
delete[] arr;
```

---

## 상황별 추천

| 상황                    | 추천 방식     | 이유                         |
| ----------------------- | ------------- | ---------------------------- |
| 크기가 런타임에 정해짐  | `std::vector` | 가변 크기, 안전함            |
| 크기가 고정되어 있음    | `std::array`  | 빠르고 간단                  |
| 성능/메모리 제어가 중요 | `new`         | 완전한 제어 가능 (주의 필요) |

---

## 전체 요약 비교표

| 항목             | `std::vector` | `std::array` | `new` |
| ---------------- | ------------- | ------------ | ----- |
| 크기 동적 지정   | ✅             | ❌            | ✅     |
| 메모리 자동 관리 | ✅             | ✅            | ❌     |
| 범위 체크        | ✅ (`.at()`)   | ✅ (`.at()`)  | ❌     |
| STL 함수 사용    | ✅             | ✅            | ❌     |
| 안전성           | ⭐⭐⭐⭐          | ⭐⭐⭐          | ⭐     |

---

## 📌 마무리

- C++에서는 **대부분의 경우 `std::vector`를 추천**합니다.
- `std::array`는 **고정 크기 배열**을 좀 더 안전하게 사용할 수 있는 방법입니다.
- `new`는 메모리를 직접 다뤄야 하기 때문에 **최적화 상황에서만 조심히 사용하는 것이 좋습니다.**

---

궁금한 점이 있다면 댓글이나 Issue로 남겨주세요 😄
