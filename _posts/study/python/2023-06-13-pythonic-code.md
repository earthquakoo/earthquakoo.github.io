---
layout: post
title:  "Pythonic Code"
subtitle:   "Pythonic Code"
categories: study
tags: python
comments: true
---

1. split, join
2. list comperhension
3. enumerate
4. zip
5. lambda, map
6. generator
7. argument

---

## split, join
- `s.split()` : 문자열의 값을 특정 구분자로 나누어 list 형태로 반환
- `s.join()` : `split()` 과 반대로 list를 구분자로 연결해 하나의 문자열로 합쳐서 반환

```python
s = "jin jinwoo earthquakoo"
print(s.split())

l = ['jin', 'jinwoo', 'earthquakoo']
print(''.join(l))
```
```terminal
['jin', 'jinwoo', 'earthquakoo']

jin jinwoo earthquakoo
```

## list comperhension
- 기존의 list를 사용해서 다른 list를 만드는 기법

```python
a = []

for i in range(10):
    a.append(i)
print(a)

l = [i for i in range(10)]
print(l)

l = [i for i in range(10) if i % 2 == 0]
print(l)
```
```terminal
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
[0, 2, 4, 6, 8]
```

## enumerate
- `enumerate()` : list의 element를 접근할 때 index를 부여해서 추출하는 방법으로 index와 element를 동시에 접근하면서 반복문 돈다.

```python
l = ['jin', 'jinwoo', 'earthquakoo']

for i, e in enumerate(l):
    print(i, e)
```
```terminal
0 jin
1 jinwoo
2 earthquakoo
```

## zip
- `zip()` : 두 개의 list를 병렬적으로 접근하여 추출하는 방법으로 두 list의 데이터를 엮어 tuple로 만든다.

```python
a = ['jin', 'jinwoo', 'earthquakoo']
b = ['good man', 'nice guy', 'bad guy']

for list_a, list_b in zip(a, b):
    print(list_a, list_b)
```
```terminal
jin good man
jinwoo nice guy
earthquakoo bad guy
```

## lambda, map
- `lambda` : return할 값을 한 줄 정도의 statement로 작성(python3 부터는 권장하지 않음)

```python
def num_add(x, y):
    return x + y

print(num_add(1, 2))

num_add = lambda x, y : x + y
print(num_add(3, 4))
```
```terminal
3
7
```

- `map()` : list의 element들을 지정된 함수로 처리. 두 개 이상의 list에 적용 가능하고 `if filter`도 사용 가능하다.
- list comperhension 형태로 표현하는 것이 좋다.

```python
a = [1,2,3]

num_square = lambda x : x ** 2
print(list(map(num_square, a)))

num_sum = lambda x, y : x + y
print(list(map(num_sum, a, a)))
```
```terminal
[1, 4, 9]
[2, 4, 6]
```

## generator
- iterable object를 특수한 형태로 사용하는 함수
- element가 사용되는 시점에 값을 메모리에 반환
	- yield를 사용해 한 번에 하나의 element만 반환
- 많은 데이터를 쓸 때 메모리를 절약할 수 있다.

```python
import sys

def general_list(value):
    result = []
    for i in range(value):
        result.append(i)
    return result

result1 = general_list(50)
print(result1)
print(type(result1))
print(sys.getsizeof(result1))
```
```terminal
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49]
<class 'list'>
472
```

```python
def generator_list(value):
    result = []
    for i in range(value):
        yield i

result2 = generator_list(50)
print(type(result2))
print(sys.getsizeof(result2))
```
```terminal
<class 'generator'>
112
```

## variable-length argument
- 개수가 정해지지 않은 변수를 함수의 parameter로 사용
- asterisk를 사용해서 함수의 parameter로 표시
- 입력된 값은 함수 내에서 tuple type으로 사용

```python
def variable_length_argument(a, b, *args):
    return a + b + sum(args)

print(variable_length_argument(1,2,3,4,5,6,7,8,9,10))
```
```terminal
55
```

## keyword variable-length argument
- parameter 이름을 따로 지정하지 않고 입력
- asterisk를 두 개 사용하여 함수의 parameter로 표시
- 입력된 값은 함수 내에서 dict type 으로 사용
- variable-length argument와 같이 사용할 경우 가변인자 다음에 사용

```python
def keyword_variable_length_argument(**kwargs):
    return kwargs

print(keyword_variable_length_argument(a = 1, b = 2, c = 3))
```
```terminal
{'a': 1, 'b': 2, 'c': 3}
```

## unpacking a container
```python
def unpacking_container(a, *args):
    print(a, *args)
    print(a, args)
    print(type(args))

unpacking_container(1, *(2,3,4,5,6))
```
```terminal
1 2 3 4 5 6
1 (2, 3, 4, 5, 6)
<class 'tuple'>
```