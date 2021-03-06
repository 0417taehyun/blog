---
title: "복잡도"
description: "[ 이것이 취업을 위한 코딩 테스트다 with 파이썬 ] 복잡도"
tags:
    - "2021"
    - 알고리즘
---

# 복잡도

!!! note "참고"

    [이것이 취업을 위한 코딩 테스트다 with 파이썬](http://www.kyobobook.co.kr/product/detailViewKor.laf?mallGb=KOR&ejkGb=KOR&barcode=9791162243077)을 참고로 공부한 내용입니다.


## 복잡도(Complexity)

### 정의

**복잡도(Complexity)**는 알고리즘의 성능을 나타내는 척도입니다. 동일한 기능에 대해서 이 복잡도가 낮을수록 좋은 알고리즘입니다.  

크게 그 종류로 **공간 복잡도(Space Complexity)**와 **시간 복잡도(Time Complexity)**로 나눌 수 있습니다.

### 공간 복잡도(Space Complexity)

#### 정의

특정한 크기의 입력에 대하여 해당 알고리즘이 얼마나 많은 메모리를 차지하는 지를 의미합니다. 다시 말해 메모리의 양과 관련된 부분입니다.

#### 확인 방법

파이썬에서 할당된 메모리를 간단하게 확인할 수 있는 방법은 아래 코드와 같습니다.

```python
{!../docs_src/algorithms/coding-test-for-employment/1-big-0/test-1.py!}
```

내장 모듈인 `tracemalloc`과 해당 모듈의 메서드인 `take_snpshot()`, `compate_to()`를 통해 메모리 할당 전과 후의 차이를 비교해볼 수 있습니다. 이때 `compate_to()` 메서드의 두 번째 매개변수는 `key_type`을 의미하며 작성된 `lineno`는 단어 그대로 줄(line)의 번호(No)를 의미합니다.  

위 파일을 실행했을 때 출력되는 결과는 간략하게 아래와 같습니다.  

<div class="termy">
    ```sh
    $ python test-1.py

    test-1.py:5: size=576 B (+576 B), count=1 (+1), average=576 B
    tracemalloc.py:423: size=88 B (+88 B), count=2 (+2), average=44 B
    test-1.py:7: size=64 B (+64 B), count=1 (+1), average=64 B
    tracemalloc.py:560: size=48 B (+48 B), count=1 (+1), average=48 B
    tracemalloc.py:315: size=40 B (+40 B), count=1 (+1), average=40 B
    ```
</div>

세 번째 출력 라인을 통해 `test.py` 파일의 `7`번째 라인에 `64 B` 만큼의 메모리가 할당된 것을 알 수 있습니다. 해당 라인은 `test_list`라는 리스트가 할당된 곳으로 그 값으로 `[1, 'test1', 2, 'test2']`가 할당되었습니다.  

!!! info "정보"

    내장 모듈인 `tracemalloc` 외에도 외부 모듈인 `psutil`, `memory_profiler` 등을 설치하여 간단하게 확인할 수도 있습니다.
    
    해당 코드는 어디까지나 메모리가 할당되는 걸 가시적으로 나타내기 위한 방법이었음을 알아주시기 바랍니다.


### 시간 복잡도(Time Complexity)

#### 정의

특정한 크기의 입력에 대하여 해당 알고리즘이 얼마나 오래 걸리는 지를 의미합니다. 다시 말해 연산의 횟수와 관련된 부분입니다.

#### 확인 방법

파이썬에서 소요된 시간을 간단하게 확인할 수 있는 방법은 아래 코드와 같습니다.

```python
{!../docs_src/algorithms/coding-test-for-employment/1-big-0/test-2.py!}
```

내장 모듈인 `time`을 활용하여 대략적으로 연산을 수행하는데 걸리는 시간을 측정해볼 수 있습니다.

위 파일을 실행했을 때 출력되는 결과는 아래와 같습니다.

<div class="termy">
    ```sh
    $ python test-2.py

    time:  4.0531158447265625e-06
    ```
</div>

!!! info "정보"

    더 간단하게 내장 모듈 중 하나인 `timeit`을 사용하여 측정할 수 있습니다. 위와 같은 방식으로 시간을 측정할 경우 대략적인 시간만 알 수 있을 뿐 이를 테면 로직 한 줄에 대한 작은 단위로 분해하여 시간을 측정하는 데 불편하기 때문입니다.

    더 자세한 내용은 [파이썬 공식문서](https://docs.python.org/ko/3/library/timeit.html)를 확인하시기 바랍니다.

#### 종류

대표적인 시간 복잡도의 종류는 아래와 같습니다.

* $O(1)$ : **상수 시간**이라 하며 입력값에 무관하게 실행 시간이 일정한 경우를 의미합니다. 따라서 매우 빠르고 효율적인 알고리즘인데 상수값 자체가 너무 클 경우 의미가 없을 수도 있습니다. 대표적으로 *해시 테이블의 조회 및 삽입*, 파이썬으로 바꿔 말하면 *딕셔너리의 조회 및 삽입*이 이에 해당합니다.

* $O(logN)$ : **로그 시간**이라 하며 매우 큰 입력값에서 크게 영향을 받지 않는 것이 특징입니다. 대표적으로 *이진 검색*이 이에 해당합니다.

* $O(N)$ : **선형 시간**이라 하며 입력값에 비례하여 실행 시간에 영향을 미칩니다. 대표적으로 *정렬되지 않는 배열에서 최댓값 또는 최솟값을 찾는 경우*가 이에 해당합니다. 모든 입력값(`N`)을 한 번 이상 살펴봐야 하기 때문입니다.

* $O(NlogN)$ : **로그 선형 시간**이라 하며 대표적으로 *병합 정렬* 등 대부분의 *효율좋은 정렬 알고리즘*이 이에 해당합니다. 모든 입력값을 적어도 한 번 이상 비교해야 하는 *비교 기반 정렬 알고리즘*은 이 **로그 선형 시간**보다 빠를 수 없습니다.

    !!! info "정보"

        단, 입력값 자체를 최선의 경우로 바꿔 비교를 생략하는 *팀소트*의 경우 **로그 선형 시간**보다 빠른 **선형 시간**이 나옵니다.

* $O(N^2)$ : **이차 시간**이라 하며 대표적으로 *버블 정렬*과 같은 비효율적인 정렬 알고리즘이 이에 해당합니다.

* $O(2^N)$ : **지수 시간**이라 하며 대표적으로 *피보나치 수를 재귀로 계산하는 알고리즘*이 이에 해당합니다.

* $O(N!)$ : **계승 시간**이라 하며 대표적으로 각 도시를 방문하고 돌아오는 가장 짧은 경로를 찾는 *외판원 문제(TSP_Travelling Slaesman Problem)*를 **브루트 포스(Brute Force)**로 풀이하는 경우가 이에 해당합니다. 가장 느린 알고리즘입니다.


### 결론

보통 효율적인 알고리즘을 구현하면 공간 복잡도와 시간 복잡도 사이에 거래 관계(Trade-Off)가 성립됩니다. 메모리를 조금 더 사용하여 속도를 개선하거나 반대로 조금 느리더라도 메모리를 절약할 수도 있습니다.

!!! info "정보"

    메모리를 더 많이 사용해서 시간을 비약적으로 줄이는 방법을 **메모이제이션(Memoization)**이라 합니다.

실제로 알고리즘 문제를 풀 때는 복잡도(Complexity)라고 했을 때 시간 복잡도(Time Complexity)를 의미합니다.




