---
layout: post
title: "[프로그래머스/swift] 프로그래머스 충돌위험 찾기 문제풀이"
date: 2025-07-13 01:39:36 +0900
categories: [프로그래머스, swift]
tags: [코딩테스트, swift, 프로그래머스]
---




## 문제
- 문제 바로가기 링크: [프로그래머스: \[PCCP 기출문제\] 3번 / 충돌위험 찾기](https://school.programmers.co.kr/learn/courses/30/lessons/340211?language=swift)
- 문제상황
	- 물류 센터 좌표 리스트: `points` 
	- 운송 루트 (물류 센터 번호) 출발지 -> 도착지 리스트: `routes`
	- 1초마다 로봇이 출발지에서 시작해서 도착지 물류센터로 한칸씩 이동한다.
		- 이동 시에는 최단경로로 이동하고, 최단경로가 여러가지일 경우, 세로로 우선적으로 이동한다
		- 포인트에 도착하면 로봇은 1초후 사라지는 것으로 가정한다.
- 로봇의 충돌 횟수를 반환하라
	- 동시에 일어난 여러개의 충돌도 모두 더해라

### 문제풀이 구상
- `points` 를 저장할 형태를 결정
	- 주어진 대로 그대로 저장하여 사용하면 될듯!
- `routes` 를 해석할 방법을 결정
	- 최단경로를 탐색하는 함수가 필요할듯! -> `getPath()` 함수로 명명하자!
	- `getPath()` 의 반환값은 운송로봇의 위치 리스트
	- 각 `routes` 의 `getPath()` 결과를 비교하여 동일한 인덱스에 동일한 위치가 있는지 카운트!
	- `routes` 가 3개 이상이라면 getPath() 를 여러번 반복해 최종적인 궤적 리스트를 얻는다! -> `readRoute()` 함수로 명명

#### getPath() 함수 구현
```swift
// from 에서 to 까지의 최단 경로로 이동하는 궤적을 반환 (초기 위치 미포함)
func getPath(from: [Int], to: [Int]) -> [[Int]] {
    var result: [[Int]] = []
    
    // y 좌표의 차이만큼 이동
    if to[0] > from[0] {
        for y in (from[0] + 1)...to[0] {
            result.append([y, from[1]])
        }
    } else if to[0] < from[0] {
        for y in 1...(from[0] - to[0]) {
            result.append([from[0] - y, from[1]])
        }
    }
    
    // x 좌표의 차이만큼 이동
    if to[1] > from[1] {
        for x in (from[1] + 1)...to[1] {
            result.append([to[0], x])
        }
    } else {
        for x in 1...(from[1] - to[1]) {
            result.append([to[0], from[1] - x])
        }
    }
    
    return result
}
```

#### readRoute() 함수 구현
```swift
// points 와 route 하나를 받았을 때 route 반환
func readRoute(_ points: [[Int]], _ route: [Int]) -> [[Int]] {
    var result: [[Int]] = []
    
    for index in 0...(route.count - 2) {
        result.append(
            contentsOf: getPath(from: points[route[index]-1], to: points[route[index+1] - 1])
        )
    }
    
    return result
}
```

#### 최종 문제 풀이
- 충돌하는 수를 계산하는 함수가 필요할 것 같아 `countCrash()` 함수를 추가하였다.
```swift
import Foundation

// from 에서 to 까지의 최단 경로로 이동하는 궤적을 반환 (초기 위치 미포함)
func getPath(from: [Int], to: [Int]) -> [[Int]] {
    var result: [[Int]] = []
    
    // y 좌표의 차이만큼 이동
    if to[0] > from[0] {
        for y in (from[0] + 1)...to[0] {
            result.append([y, from[1]])
        }
    } else if to[0] < from[0] {
        for y in 1...(from[0] - to[0]) {
            result.append([from[0] - y, from[1]])
        }
    }
    
    // x 좌표의 차이만큼 이동
    if to[1] > from[1] {
        for x in (from[1] + 1)...to[1] {
            result.append([to[0], x])
        }
    } else if to[1] < from[1] {
        for x in 1...(from[1] - to[1]) {
            result.append([to[0], from[1] - x])
        }
    }
    
    return result
}

// points 와 route 하나를 받았을 때 route 반환
func readRoute(_ points: [[Int]], _ route: [Int]) -> [[Int]] {
    var result: [[Int]] = [ points[route[0] - 1] ]
    
    for index in 0...(route.count - 2) {
        result.append(
            contentsOf: getPath(from: points[route[index]-1], to: points[route[index+1] - 1])
        )
    }
    
    return result
}

// 리스트에서 충돌 횟수 리턴
func countCrash(_ list: [[Int]]) -> Int {
    var res: [[Int]] = []
    
    for i in 0..<list.count {
        for j in (i + 1)..<list.count {
            if list[i] == list[j] && !res.contains(list[i]) {
                res.append(list[i])
            }
        }
    }
    return res.count
}

func solution(_ points:[[Int]], _ routes:[[Int]]) -> Int {
    var paths: [[[Int]]] = []
    var maxLength = 0
    var res = 0
    for route in routes {
        let path = readRoute(points, route)
        if maxLength < path.count {
            maxLength = path.count
        }
        paths.append(path)
    }
    
    for i in 0..<maxLength {
        var list: [[Int]] = []
        
        for path in paths {
            if i < path.count {
                list.append(path[i])
            }
        }
        res += countCrash(list)
    }
    
    return res
}
```

## 결론
- 문제를 보고 필요한 기능들 부터 차근차근 설계하며 접근하였다.
- 추후 필요한 것이 생기면 필요한 기능을 추가하여 해결 하였다.
