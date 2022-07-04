---
title:  "타입스크립트 Decorator (@Decorator)"
search: false
categories: 
  - TypeScript
toc: true  
last_modified_at: 2022-06-29T18:06:00-05:00
tags:
  - TypeScript
  - JavaScript
  - Decorator
---

# 타입스크립트 데코레이터
## Intro 
안녕하세요. 전제 사원입니다.
타입스크립트 데코레이터 공부하면서 정리한 내용입니다.

- 데코레이터
- 데코레이터 팩토리
- 데코레이터 합성
- 데코레이터 평가
- 클래스 데코레이터
- 메서드 데코레이터
- 접근자 데코레이터
- 프로퍼티 데코레이터
- 매개변수 데코레이터
- 메타데이터
<hr>

## 데코레이터
데코레이터는 클래스 선언, Method, Accessor, property, parameter에   
첨부할 수 있는 특수한 종류의 선언입니다.
- @Expression 형식 [(at icon) + Function name]을 사용합니다.
<hr>

## 데코레이터 팩토리
데코레이터가 선언에 적용되는 방식을 원하는대로 바꾸고 싶을 때..  
그럴 때 작성하는 것으로 런타임에 호출할 표현식을 반환하는 함수 입니다.

```ts
function Apple(value: string) { //데코레이터 팩토리
  return function (target) { //데코레이터
    // 위에서 가져온 value와 target으로 어떠한 작업을 수행
  }
}
```
<hr>

## 데코레이터 합성
- 단일행 :   
    ```ts 
    @A @B function
    ```
- 다중행 :  
    ```ts
    @A
    @B
    function
    ```

위에서 선언한 @A @B function은 A(B(function))과 같습니다.
1. 위에서 아래로 평가하며, 
    ```
    A call
    B call
    ```
2. 아래에서 위로 함수를 호출합니다.
    ```
    B Complate
    A Complate
    ```
<hr>

## 데코레이터 평가 
- D : Decorator
1. Method, Accessor 또는 Property D가 다음에 오는 Parameter D 는  
각 Instance member에 적용됩니다.
- Class > Object or method

2. Method, Accessor 또는 Property D가 다음에 오는 Parameter D 는  
각 Static member(정적 멤버)에 적용됩니다.
- Class > Static Object

3. Parameter D 는 생성자에 적용됩니다.
- Constructor

4. Class D 는 클래스에 적용됩니다.
- Class

<hr>

## 클래스 데코레이터 (Class Decorator)
클래스 선언 직전에 선언 합니다.  
클래스 정의를 감시, 수정, 교체 하는데 사용 가능합니다.  
* 선언 파일이나 주변 컨텍스트에서 사용할 수 없습니다.
    - 컨텍스트 (Context) : 예) 선언 클래스

클래스 데코레이터는 적용하는 클래스의 생성자를 유일한 인수로 받습니다.
- constructor

클래스 데코레이터가 값을 반환하면 생성자 함수로 컨버팅 됩니다.

예제 Class
```ts
@classDecorator
class sealed {
  data: string;
  constructor(message : string) {
    this.data = message;
  }

  method() {
    return "Hello, " + this.data;
  }
}
```

예제 Decorator
```ts
function classDecorator(constructor: Function) {
  Object.seal(constructor);
  Object.seal(constructor.prototype);
}
```
- 예제에서는 Object.seal() 기능을 이용해서 객체를 밀봉하였습니다.
    - Object.seal() : 객체 밀봉
        - 새로운 속성 추가 x
        - 모든 속성을 설정 불가능 상태로 변경
        - [Object.freeze와 다른 점] 쓰기 가능한 속성의 값은 밀봉 후에도 변경 가능

<hr>

## 메서드 데코레이터 (Method decorator)
메서드 선언 직전에 선언합니다.  
메서드 정의를 감시, 수정, 교체 하는데 사용 가능합니다.  
* 선언 파일이나 주변 컨텍스트에서 사용할 수 없습니다.  

클래스 데코레이터와 달리 런타임에 세 개의 인수와 함께 함수로 호출됩니다.
1. 정적 멤버에 대한 클래스의 생성자 함수 또는 인스턴스 멤버에 대한 클래스의 프로토타입
2. 멤버의 이름
3. 멤버의 프로퍼티 설명자 (property descriptor)

메서드 데코레이터가 값을 반환하면, 메서드의 프로퍼티 설명자로 사용됩니다.

예제 Class
```ts
class example {
    a: string = "Hello";
    get b(): string {
        return `${this.a} World`;
    }
    @decorator
    method(c: string): void {
        console.log(c);
    }
}
```
예제 Decorator
```ts
function decorator() {
    return function (
        target: any,
        propertyKey: string, 
        descriptor: propertyDescriptor
    ) {
        console.log(target); // {b: [Getter], method: [Function (anonymous)]}
        console.log(propertyKey); // d
        console.log(descriptor); 
        /**
         * {
         *  value: [Function (anonymous)]
         *  writable: true,
         *  enumerable: true,
         *  configurable: true
         * }
         */ 
    }
}
```
@decorator 가 실행되면서 예제 코드의 console 결과가 나오는 것입니다.

@decorator를 좀 바꿔본다면
```ts
function decorator ( target: any, propertyKey: string, descriptor: PropertyDescriptor): void {
    const method = descriptor.value;
    descriptor.value = function () {
        try {
            method();   
        } catch (error) {
            console.log("error 핸들링 로직");
        }
    }
}
```
이렇게 바꾸고 반영해서 실행한다면!!

예제 class 중
```ts
...
@decorator
method(c: string): void {
    console.log(c);
    throw new Error();
}
...
//데코레이터
...
new example().method("오..");

```
console
```
오..
error 핸들링 로직
```

이라는 결과가 나온다.

<hr>

## 접근자 데코레이터 (Accessor Decorator)
접근자 선언 직전에 선언  
접근자 정의를 감시, 수정, 교체 하는데 사용 가능합니다.
* 문서 순서대로 지정한 첫 번째 접근자에 적용해야 합니다.
    - 각각의 선언이 아닌 결합한 property descriptor에 적용되기 때문입니다.

메서드 데코레이터와 같이 세개의 인수와 함께 함수로 호출합니다.  
접근자 데코레이터가 값을 반환하면 프로퍼티 설명자로 사용됩니다.

예제 class (typescript handbook 中)
```ts
class Point {
    private _x: number;
    private _y: number;

    constructor(x: number, y: number, public size: string = "100") {
        this._x = x;
        this._y = y;
    }

    @configurable(false)
    get x() { return this._x }

    @configurable(false)
    get y() { return this._x }
}
```

예제 decorator
```ts
function configurable(value: boolean) {
    return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
        descriptor.configurable = value;
    }
}
```

```ts
// point 객체 인스턴스 생성
const point = new Point(100, 150);

// 속성 제거 가능
delete point.size;

// [오류] delete 연산자의 피연산자는 읽기 전용 속성일 수 없습니다.
delete point.x;

```
이 코드에서 생성된 객체 인스턴스의 접근 제어자 속성을 제거하려 시도하면 오류가 발생합니다.

메서드 데코레이터와 동일하나 decription의 configurable 속성을 바꿉니다.

<hr>

## 프로퍼티 데코레이터 (Property decorator)
프로퍼티 선언 직전에 선언합니다.  

앞선 메서드 & 접근자 데코레이터와 달리 두 개의 인수와 함께 함수로 호출합니다.
1. 정적 멤버에 대한 클래스의 생성자 함수 또는 인스턴스 멤버에 대한 클래스의 프로토타입
2. 멤버의 이름

예제 class
```ts
class Car {
    private name: string;
    private price: number;
    private type: string;

    constructor(name: string, price: number) {
        this.name = name;
        this.price = price;
    }

    public toString() {
        return `${this.name}, ${this.type}, ${this.price}`
    }
}
```

객체를 주입할 때 사용할 간단한 컨테이너를 정의하고 객체를 넣어둡니다.
```ts
class Container {
    private static map: {[key: string]: any} = {};

    static add(key: string, value: string) {
        Container.map[key] = value;
    }

    static get(key: string): string {
        return Container.map[key];
    }
}

Container.add('myType', 'Classic');
console.log(Container.get('myType')); // 'Classic'
```

이제 예제에서 사용할 데코레이터를 정의합니다.
예제 decorator
```ts
function Inject(param: string) {
    return function (target: any, propertyKey: string) {
        console.log(target); // {toString: f, constructor: f}
        console.log(propertyKey); // type
        target[propertyKey] = Container.get(param); //target 오브젝트에 property 값 할당
    }
}
```

이렇게 정의한 데코레이터는 클래스를 정의할 때 사용할 수 있습니다.

```ts
class Car {
    private name: string;
    private price: number;
    @Inject('myType') //값 주입
    private type: string;
}

let myCar = new Car('AMG GT', 15000);
console.log(myCar.toString()); // AMG GT, Classic, 15000
```

## 매개변수 데코레이터 (Parameter decorator)
매개 변수 선언 직전에 선언 합니다.  
세 개의 인수와 함께 함수로 호출 합니다.
1. 프로퍼티 데코레이터와 동일
2. ''
3. 함수 매개 변수 목록에 있는 서수 색인 (Ordinal index)

매개 변수 데코레이터의 반환값은 무시 됩니다.

정리하면 파라미터 데코레이터는 옵저빙(감시), 값 변경이 안되기에 metadata를 정의할 때 사용합니다. 그렇기에 reflect metadata랑 같이 사용합니다.

## 데코레이터 호출 순 
```ts
function cd() {
    console.log('class');
    ...
}

function md() {
    console.log('method');
    ...
}

function prod() {
    console.log('property');
    ...
}

function pramd() {
    console.log(parameter);
    ...
}

@cd()
class example {
    @prod()
    property = "property";

    @md()
    test(
        @paramd() param: string
    ) {
        console.log('test');
    }
}
```
결과는
```
property
method
parameter
class
```
순으로 데코레이터 호출 순서는

Property decorator =>  
Method decorator =>  
Parameter decorator =>  
Class decorator 입니다.

<hr>

## 메타데이터 Metadata
- 데이터의 데이터
- 타입스크립트에서는 실험적인 기능 제공을 하기 때문에 따로 tsconfig.json에서 설정을 해주어야 한다.
    - tsconfig.json > compilerOptions > emitDecoratorMetadata : true