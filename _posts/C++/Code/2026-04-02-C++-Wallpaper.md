---
title: "C++: 바탕화면 정리"
author: Jaeseong Kim
date: 2026-04-02 12:00:00 +0800
categories: [C++, Code]
tags: [C++, Code]
---
https://school.programmers.co.kr/learn/courses/30/lessons/161990
## 조건
* 바탕화면에 파일들이 있다. 공간은 정사각형의 배열이며, 각 공간이 비어있으면 점(.)으로, 파일이 있으면 샵(#)으로 주어진다.
* 좌표는 0부터 시작하며, 파일은 각 좌표 사이에 한 변이 1인 크기의 정사각형으로 존재한다.
* 직사각형의 바탕화면에서 한번에 파일을 드래그해서 선택할 수 있는 좌표값 2개를 하나의 배열로 반환해야한다. 
## 풀이
반복문으로 순회하면서 최솟값과 최댓값을 찾을 수 있으면 쉽게 풀린다.

```c++
#include <string>
#include <vector>

using namespace std;

vector<int> solution(vector<string> wallpaper) {
    vector<int> answer;
    //최솟값 최댓값 선언
    int minX = wallpaper.size();
    int minY = wallpaper[0].size();
    int maxX = 0;
    int maxY = 0;
    //순회하며 파일이 있으면 좌표 비교후 저장
    for(int i = 0; i < wallpaper.size();i++){
        for(int j = 0; j < wallpaper[i].size(); j++){
            if(wallpaper[i][j] == '#'){
                if(i < minX) minX = i;
                if(j < minY) minY = j;
                if(i+1 > maxX) maxX = i+1;
                if(j+1 > maxY) maxY = j+1;
            }
        }
    }
    return {minX,minY,maxX,maxY};
}
```