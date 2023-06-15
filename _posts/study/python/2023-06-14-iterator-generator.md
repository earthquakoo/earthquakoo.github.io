---
layout: post
title:  "iterator & generator"
subtitle:   "iterator, generator"
categories: study
tags: python
comments: true
---

1. iterable
2. iterator
3. Generator

---

## iterable
Iterable 객체란 반복 가능한 객체를 뜻합니다. 대표적으로 iterable한 타입으로는 list, dict, set, str, tuple 등이 있습니다. 

또한 `__iter__()` 나 `__getitem__()` 메소드로 정의된 class는 모두 iterable 하다고 할 수 있습니다.

## iterator
Iterator 객체는 값을 차례대로 꺼낼 수 있는 객체로 `next()` 메소드로 데이터를 순차적으로 호출 가능한 객체입니다. 만약 `next()`로 다음 데이터를 불러올 수 없을 경우(가장 마지막 데이터인 경우) StopIteration exception을 발생시킵니다. 

그렇다면 'Iterable한 객체들은 모두 iterator인가' 라는 의문점이 생깁니다. 결론부터 말하자면 iterable 이라고 해서 반드시 iterator는 아닙니다.

```python
a = [1,2,3]
next(a)
```
```terminal
Traceback (most recent call last):
  File "C:\Users\cream\Desktop\sigan\asd.py", line 2, in <module>
    next(a)
TypeError: 'list' object is not an iterator
```

List는 iterable이지만, 위와 같이 `next()` 메소드로 호출해도 동작하지 않고 iterator가 아니라는 에러 메시지를 확인할 수 있습니다. 만약 iterable을 iterator로 변환하고 싶다면, `iter()`라는 built-in function을 사용하면 됩니다.

```python
a = [1,2,3]
print(type(a))

b = iter(a)
print(type(b))
```
```terminal
<class 'list'>
<class 'list_iterator'>
```

```python
print((next(b)))
print((next(b)))
print((next(b)))
print((next(b)))
```
```terminal
1
2
3
Traceback (most recent call last):
  File "C:\Users\cream\Desktop\sigan\asd.py", line 10, in <module>        
    print((next(b)))
StopIteration
```

위처럼 list를 iterator로 만들면 `next()`를 이용해 element를 하나씩 꺼낼 수 있습니다. 그리고 마지막 element를 호출한 이후에 `next()`를 호출하면 StopIteration 이라는 exception이 발생하는 것을 확인할 수 있습니다.

하지만, list나 tuple 같은 iterable한 객체를 사용할 때 굳이 `iter()` 함수를 사용하지 않고도 for 문을 사용하여 순차적으로 접근이 가능했습니다. 이것은 for 문을 돌 때 python 내부에서 임시로 list를 iterator로 자동 변환해주었기 때문입니다.

> for loop & iterator

그렇다면 for 문 내부에서는 실제로 어떤 일이 일어나는지 살펴보겠습니다.

```python
l = [1,2,3,4]

for element in l:
    print(element, end=" ")
```
```terminal
1 2 3 4
```

위의 예제는 기본적인 for loop문으로 list의 element를 하나씩 반환하는 코드입니다.

```python
l = [1,2,3,4]
list_iterator = iter(l)

while True:
    try:
        element = next(list_iterator)
        print(element, end=" ")
    except StopIteration:
        break
```
```terminal
1 2 3 4
```

위의 예제는 기본적인 for loop문의 동작 모습을 구현한 것입니다. iterable한 객체를 `iter`를 통해 iterator로 동작하도록 했습니다. 이후 동작은 iterator와 동일하게 모든 element가 소진될 때까지 반복합니다.

> Building custom iterator

위에서 살펴보았듯이 iterable한 객체를 `iter`로 iterator를 만들 수 있는 것을 확인했고 iterator의 실제 동작을 확인할 수 있었습니다. 그렇다면 사용자가 정의한 iterator도 만들 수 있을 것 같습니다. 

실제 구현을 하기 위해서는 자기 자신의 객체를 기억해 리턴하는 `__iter__` 메소드와 다음 element를 반환하는 `__next__` 메소드를 구현해야합니다.

```python
class square:
    def __init__(self, limit=0):
        self.limit = limit

    def __iter__(self):
        self.n = 1
        return self # 자기 자신 객체 반환

    def __next__(self):
        if self.n <= self.limit:
            result = self.n ** 2
            self.n += 1
            return result # 다음 element 반환
        else: 
            raise StopIteration


sequence = square(5)
i = iter(sequence)
print(next(i))
print(next(i))
print(next(i))
print(next(i))
print(next(i))
print(next(i))
```
```terminal
1
4
9
16
25
Traceback (most recent call last):
  File "C:\Users\cream\Desktop\sigan\asd.py", line 25, in <module>        
    print(next(i))
  File "C:\Users\cream\Desktop\sigan\asd.py", line 15, in __next__        
    raise StopIteration # raise StopIteration exception
StopIteration
```

## Generator

위에서 iterator를 실제로 구현해서 만들어보았습니다. 하지만 매번 iterator를 사용하기 위해 직접 만드는 것은 귀찮은 일입니다. 그런 저희를 위해서 python에서는 **generator**라는 **iterator maker**를 사용할 수 있게 해놨습니다.

Generator는 iterator와 같은 루프 작용을 컨트롤하기 위해 쓰여지는 함수 또는 루틴입니다. Generator는 list를 리턴하는 함수와 비슷하며, 호출할 수 있는 parameter를 가지고 있고 연속적인 값을 만들어냅니다. 

하지만 한번에 모든 값을 포함한 list를 만들어서 리턴하는 대신에 `yield` 구문을 이용해 한 번 호출될 때마다 하나의 값만 리턴하고, 다시 호출될 때 마지막으로 실행한 라인과 함께 중단된 위치에서 다시 시작하게 됩니다.

제가 이해한 바로는 **generator는 iterator와 같은 역할을 하는 함수**입니니다. 

```python
def my_generator():
    a = 10
    yield 1 # step 2 - pause
    yield 2 # step 4 - pause
    yield 3 # step 6 - pause
    
print(type(my_generator))
print(type(my_generator()))

gen_iterator = my_generator()
print(next(gen_iterator)) # step 1
print(next(gen_iterator)) # step 3
print(next(gen_iterator)) # step 5
print(next(gen_iterator)) # step 7
```
```terminal
<class 'function'>
<class 'generator'>
1
2
3
Traceback (most recent call last):
  File "C:\Users\cream\Desktop\sigan\asd.py", line 14, in <module>
    print(next(gen_iterator))
StopIteration
```

또한 generator는 iterator와 같기 때문에 for 문을 직접 돌 수 있습니다.

```python
for e in my_generator():
	print(e)
```
```terminal
1
2
3
```

> generator를 사용하는 이유

Generator와 list로 만든 객체가 수많은 데이터를 보유하고 있다고 가정할 때, 효율성에서 차이가 발생합니다.
Generator의 경우 모든 값의 순서를 객체를 만든 순간 기억한 상태로 동작하기 전까지는 메모리를 할당하지 않습니다. 하지만 list는 동작하는 순간 모든 값을 메모리에 올려버리기 때문에 메모리 효율성에서 차이가 발생합니다.