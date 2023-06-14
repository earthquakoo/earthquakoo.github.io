---
layout: post
title:  "Decorator(First-class function, closure)"
subtitle:   "decorator"
categories: study
tags: python
comments: true
---

1. First Class Function
2. Closure
3. Decorator

---

데코레이터(decorator)를 이해하기 위해선 몇 가지 과정을 거쳐야합니다. 그 첫번째 과정으로 first class function에 대해서 알아보겠습니다.

## First Class Function

프로그래밍 언어는 해당 언어의 함수들이 다른 변수처럼 다루어질 때 first-class function 일급 함수를 가진다고 합니다. 예를 들어, first-class function을 가진 언어에서 함수는 다른 함수들에 매개변수로 전달될 수 있고, 다른 함수에 의해 반환될 수 있으며, 변수에 값으로서 할당될 수 있습니다.

간단한 예제를 통해 알아보도록 하겠습니다.

```python
def square(x):
    return x * x

f = square

print(square)
print(f)
```
```terminal
<function square at 0x000001FE1433D1F0>
<function square at 0x000001FE1433D1F0>
```

함수 `square` 을 정의하고 변수 `f`에 할당하여 출력해보니 동일한 메모리 값이 저장된 `square` 함수 오브젝트가 할당되어 있는 것을 볼 수 있습니다.

```python
def square(x):
    return x * x

print(square(5))

f = square

print(f(5))
```
```terminal
25
25
```

당연하게도 변수 `f`를 이용해 함수를 호출하는 것도 동일하게 가능합니다. 위처럼 함수를 변수에 할당할 수 있고 함수가 할당된 변수를 함수처럼 다룰 수 있을 때 first-class function을 가진다고 할 수 있습니다.

또한 First-class function을 지원하면 함수를 변수에 할당할 수 있을 뿐만 아니라, 인자(arguments)로써 함수로 전달하거나 함수의 리턴값으로도 사용할 수 있습니다.

```python
def square(x):
    return x * x

def cube(x):
    return x * x * x

def quad(x):
    return x * x * x * x

def my_map(func, arg_list):
    result = []
    for i in arg_list:
        result.append(func(i))
    return result

num_list = [1, 2, 3, 4, 5]

squares = my_map(square, num_list)
cubes = my_map(cube, num_list)
quads = my_map(quad, num_list)

print(squares)
print(cubes)
print(quads)
```
```terminal
[1, 4, 9, 16, 25]
[1, 8, 27, 64, 125]
[1, 16, 81, 256, 625]
```

이처럼 first class function을 이용하면 이미 정의된 여러 함수를 간단히 재활용할 수 있습니다.

## Closure

Closure은 어떤 함수를 함수 자신이 가지고 있는 환경과 함께 저장한 레코드입니다. 또한 함수가 가진 자유 변수(free variable)를 closure가 만들어지는 당시의 값과 레퍼런스에 맵핑해주는 역할을 한다고 합니다.

> 중첩 함수(Nested function) or 내부 함수(Inner function)

먼저 closure을 이해하기 전에 중첩 함수를 알고 넘어가야 합니다. 중첩 함수 또는 내부 함수는 말 그대로 함수 내에 또 다른 함수가 있는 것을 말합니다.

```python
def outer():
    print("여긴 외부 함수 영역입니다.")
    def inner():
        print("여긴 내부 함수 영역입니다.")
    inner()

outer()
inner()
```
```terminal
여긴 외부 함수 영역입니다.
여긴 내부 함수 영역입니다.

Traceback (most recent call last):
  File "<pyshell#82>", line 1, in <module>
    inner()
NameError: name 'inner' is not defined
```

`outer`함수 내에서는 `inner`함수를 호출할 수 있는데 `outer` 함수의 외부에서는 내부 함수인 `inner` 함수를 호출할 수 없습니다. 이는 `outer` 함수 내에서 선언되었으니 `outer` 함수 내에서만 호출이 가능하다는 것을 의미합니다.

> 클로저(Closure)

```python
def start_at(x):
    def increment_by(y):
        return x + y
    return increment_by
 
 
closure1 = start_at(1)
closure2 = start_at(2)
print("closure1:", closure1(3))
print("closure2:", closure2(4))
```
```terminal
closure1: 4
closure2: 6
```

함수 `start_at`을 살펴보면 내부 함수인 `increment_by`를 반환합니다. 

```python
closure1 = start_at(1)
closure2 = start_at(2)
```

여기서 외부 함수인 `start_at`에 매개변수 `x`로 1과 2를 넘겼습니다. 그리고 `closure1` 과 `closure2`에는 함수 객체가 담긴 것을 예상할 수있습니다. 여기서 **외부 함수인 `start_at`의 역할이 끝남과 동시에 매개변수 `x`의 값도 메모리상에서 사라지는게 아닐까** 라고 생각해볼 수 있습니다. 하지만 내부 함수인 `increment_by`는 외부 함수에서 사용되는 변수 `x`의 값을 그대로 기억하고 있는 것을 볼 수 있습니다.

```python
print("closure1:", closure1(3))
print("closure2:", closure2(4))
```

이러한 행동을 하는 함수를 클로저라고 합니다. **클로저는 내부 함수가 메모리에 존재하지 않는 경우에도 호출될 때 주변환경을 기억합니다.** 외부 함수의 `start_at`과 함께 매개변수 `x`도 사라졌으나, 내부 함수에 쓰인 `x`에는 호출할 때 외부 함수로 넘겨줬던 값이 묶인다는 것을 확인했습니다.

```python
print("closure1.__closure__:", closure1.__closure__)  # closure1 = start_at(1)
# (<cell at 0x0415AFB0: int object at 0x63F464B0>,)
print("closure2.__closure__:", closure2.__closure__)  # closure2 = start_at(2)
# (<cell at 0x0415AEB0: int object at 0x63F464C0>,)
print("closure1.__closure__[0].cell_contents:", closure1.__closure__[0].cell_contents)
# 1
print("closure2.__closure__[0].cell_contents:", closure2.__closure__[0].cell_contents)
# 2
```

위와 같이 `__closure__` 멤버로 직접 확인해볼 수 있습니다다. `__closure__`은 cell로 이루어진 tuple이며, 각 셀에는 `cell_contents`라는 멤버가 있는데 이를 통해 cell에 담긴 값을 확인해볼 수 있습니다. 파이썬에서 cell 객체는 클로저의 자유 변수(free variables)를 저장하기 위해 사용됩니다. 

여기서 자유 변수는 **코드 영역에서는 사용되지만, 전역 변수도 아니며 그 영역 내에서도 정의하지 않는 변수**를 말합니다.

```python
a = 1
 
def outer():
    b = 2
    def inner():
        c = 3
        print(a, b, c)
    inner()
```

위의 예제에서는 함수 `inner` 기준으로 `b`가 자유 변수라고 할 수 있습니다다. `inner`의 코드 영역에서는 `a`는 전역 변수이며, `c`는 지역 변수인데 변수 `b`는 함수 `inner` 내부에서 정의된 것이 아니기 때문입니다.

## Decorator

데코레이터(decorator)란 사전적 의미로는 '장식', '장식하는 사람'이란 뜻을 가지고 파이썬에서도 비슷한 의미로 사용됩니다. 기존의 코드에 여러 기능을 추가하는 파이썬 구문이라고 생각하면 편합니다.

```python
def decorator_function(original_function):
    def wrapper_function():
        return original_function()
    return wrapper_function


def display():  # 2
    print('display 함수가 실행되었습니다.')


decorated_display = decorator_function(display)

decorated_display()
```
```terminal
display 함수가 실행되었습니다.
```

앞서 언급한 first class function과 closure를 살펴보셨다면 충분히 이해가능한 코드입니다. 이제 데코레이터를 쓰는 이유에 대해서 살펴보겠습니다.

```python
def decorator_function(original_function):
    def wrapper_function():
        print('{} 함수가 호출되기전 입니다.'.format(original_function.__name__))
        return original_function()
    return wrapper_function


def display1():
    print('display1 함수가 실행됐습니다.')


def display2():
    print('display2 함수가 실행됐습니다.')


display_1 = decorator_function(display1)
display_2 = decorator_function(display2)

display_1()
display_2()
```
```terminal
display_1 함수가 호출되기전 입니다.
display_1 함수가 실행됐습니다.
display_2 함수가 호출되기전 입니다.
display_2 함수가 실행됐습니다.
```

위와 같이 데코레이터 함수를 만들어 `display1`과 `display2` 함수에 기능을 추가할 수 있습니다. 여기서 데코레이터 기능을 이용해 더 간소화 시켜보겠습니다.

```python
def decorator_function(original_function):
    def wrapper_function():
        print('{} 함수가 호출되기전 입니다.'.format(original_function.__name__))
        return original_function()
    return wrapper_function


@decorator_function
def display1():
    print('display1 함수가 실행됐습니다.')


@decorator_function
def display2():
    print('display2 함수가 실행됐습니다.')


display_1()
display_2()
```
```bash
display_1 함수가 호출되기전 입니다.
display_1 함수가 실행됐습니다.
display_2 함수가 호출되기전 입니다.
display_2 함수가 실행됐습니다.
```

`@` 문자를 이용해 데코레이터 구문이 더 간소화되고 동일한 결과를 확인할 수 있습니다. 즉, `@decorator_function` 과 `display_1 = decorator_function(display1)` 이 구문은 동일한 역할을 하는 것을 알 수 있습니다.

