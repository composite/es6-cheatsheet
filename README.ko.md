# es6-cheatsheet

이 치트시트는 ES2015 [ES6] 에 대한 팁, 사용법, 실습을 위한 코드 예제가 들어 있습니다.
참여는 언제나 환영합니다! 번역 참역 및 오역 수정도 환영합니다 (역주의 변명)

## Table of Contents

- [var 와 let / const](#var-versus-let--const)
- [IIFEs 를 Blocks 으로 데체](#replacing-iifes-with-blocks)
- [화살표 함수](#arrow-functions)
- [문자열](#strings)
- [변수 분해](#destructuring)
- [모듈](#modules)
- [인자](#parameters)
- [클래스](#classes)
- [기호](#symbols)
- [Maps](#maps)
- [WeakMaps](#weakmaps)
- [프로미즈](#promises)

## var versus let / const

> `var` 외에 우리는 이제 더 강력한 값을 지칭하기 위해 `let` 과 `const` 지시자로 접근할 수 있습니다.
 `var` 와는 다르게, `let` 과 `const` 구문은 어느 상위 레벨이던 호이스팅 현상이 발생할 수 없습니다.

`var` 사용 예제:

```javascript
var snack = 'Meow Mix';

function getFood(food) {
    if (food) {
        var snack = 'Friskies';
        return snack;
    }
    return snack;
}

getFood(false); // undefined
```

그러나, `var` 대신 `let` 구문을 통해 지켜보면,

```javascript
let snack = 'Meow Mix';

function getFood(food) {
    if (food) {
        let snack = 'Friskies';
        return snack;
    }
    return snack;
}

getFood(false); // 'Meow Mix'
```

위와 같은 구문으로 리팩토링을 고려할 때, `var` 구문을 통한 변수의 활용 범위를 조심하세요.
`var` 를 `let` 으로 바꿀 경우에 예상치 못한 상황이 발생할 수 있습니다.

> **참고**: `let` 과 `const` 는 블록 단위 스코프입니다. 즉, 블록 내 식별자가 정의되기 전 참조할 경우
`ReferenceError` 예외가 발생합니다.

```javascript
console.log(x);

let x = 'hi'; // ReferenceError: x 는(은) 정의되지 않았습니다.
```

> **실습**: 예전 환경에서 `var` 선언 중심으로 된 코드를 리팩토링할 때 조심해야 합니다.
새로운 작업 환경에서 작업할 때, `let` 지시문을 통해 언제나 변경 가능한 변수를 선언하고,
`const` 지시문을 통해 다시 정의할 수 없는 변수를 선언할 수 있습니다.

<sup>[(차례로 돌아가기)](#table-of-contents)</sup>

## Replacing IIFEs with Blocks

> 보통 **즉시 실행 함수 구문(Immediately Invoked Function Expressions)** 은
스코프 내에서 값을 정의하기 위해 쓰입니다.
ES6에서는 이제 굳이 함수 기반의 구문을 쓰지 않아도 블록 기반의 구문을 통해
스코프를 분리할 수 있게 됐습니다.

```javascript
(function () {
    var food = 'Meow Mix';
}());

console.log(food); // 참조 오류
```

ES6 블록 구문 사용:

```javascript
{
    let food = 'Meow Mix';
}

console.log(food); // 참조 오류
```

<sup>[(차례로 돌아가기)](#table-of-contents)</sup>

## Arrow Functions

가끔 함수 안에 함수에서 `this` 지시문에 대한 컨텍스트 참조를 유지하고 싶어합니다.
예를 들면,

```javascript
function Person(name) {
    this.name = name;
}

Person.prototype.prefixName = function (arr) {
    return arr.map(function (character) {
        return this.name + character; // 정의되지 않은 'name' 속성을 가져올 수 없습니다.
    });
};
```

한가지 보통 사용하는 해결 방법은 `this` 컨텍스트를 변수로 담아 참조하는 것이죠.

```javascript
function Person(name) {
    this.name = name;
}

Person.prototype.prefixName = function (arr) {
    var that = this; // this가 가리키는 참조를 저장
    return arr.map(function (character) {
        return that.name + character;
    });
};
```

이제 이런 식으로도 `this`를 참조할 수 있습니다.:

```javascript
function Person(name) {
    this.name = name;
}

Person.prototype.prefixName = function (arr) {
    return arr.map(function (character) {
        return this.name + character;
    }, this);
};
```

아니면 컨텍스트 바인딩 메소드를 제공하기도 하죠.

```javascript
function Person(name) {
    this.name = name;
}

Person.prototype.prefixName = function (arr) {
    return arr.map(function (character) {
        return this.name + character;
    }.bind(this));
};
```

**화살표 함수(Arrow Functions)** 를 사용하면, 컨텍스트 참조가 담긴 `this`가
컨텍스트 위치를 바꾸지 않고도 아래와 같이 바로 재작성 가능합니다.

```javascript
function Person(name) {
    this.name = name;
}

Person.prototype.prefixName = function (arr) {
    return arr.map(character => this.name + character);
};
```

> **실습**: **화살표 함수** 를 `this` 컨텍스트 참조를 유지하고 싶은 함수 구문에 사용해 보세요.

물론 화살표 함수는 좀 더 복잡한 수행을 위해 함수 구문처럼 사용할 수 있습니다.

```javascript
var squares = arr.map(function (x) { return x * x }); // 함수 구문
```

```javascript
const arr = [1, 2, 3, 4, 5];
const squares = arr.map(x => x * x); // 화살표 구문을 통한 단순화된 구현
```

> **실습**: **화살표 함수** 를 기존에서 간편하게 구현 가능한 함수 구문을 대체해 보세요.

<sup>[(차례로 돌아가기)](#table-of-contents)</sup>

## Strings

ES6에서는 표준 라이브러리 기능이 향상됐습니다. 문자열을 좀 더 효율적으로 사용할 수 있는
몇가지 메소드가 추가됐습니다. 예를 들면, `.includes()` 메소드와 `.repeat()` 등이 있습니다.

### .includes( )

```javascript
var string = 'food';
var substring = 'foo';

console.log(string.indexOf(substring) > -1);
```
`> -1` 결과를 반환하여 문자열이 들어있는 지 확인하는 부분을
`.includes()` 메소드를 통해 부울 값으로 간편하게 확인할 수 있습니다.

```javascript
const string = 'food';
const substring = 'foo';

console.log(string.includes(substring)); // true
```

### .repeat( )

```javascript
function repeat(string, count) {
    var strings = [];
    while(strings.length < count) {
        strings.push(string);
    }
    return strings.join('');
}
```

ES6에서 더 간편하게 문자열을 반복하는 방법을 접근할 수 있도록 구현했습니다.

```javascript
// String.repeat(numberOfRepetitions)
'야옹'.repeat(3); // '아옹야옹야옹'
```

### Template Literals

**템플릿 리터럴(Template Literals)** 을 통하여, 문자열 안에 있는 특수문자를 굳이 이스케이프하지 않아도 작성할 수 있게 됐습니다.

```javascript
var text = "이 문자열에는 \"쌍따옴표\"를 이스케이프하여 넣었습니다.";
```

```javascript
let text = `이 문자열에는 "쌍따옴표"를 이스케이프하여 넣었습니다.`;
```

**템플릿 리터럴** 에서는 또한 보간 기능을 지원하여, 문자열 안에 변수를 직접 삽입하여 치환할 수 있습니다.

```javascript
var name = 'Tiger';
var age = 13;

console.log('내가 키우는 고양이 이름은 ' + name + ' 이고 ' + age + ' 살 입니다.');
```

간단하게 하면,

```javascript
const name = 'Tiger';
const age = 13;

console.log(`내가 키우는 고양이 이름은 ${name} 이고 ${age} 살 입니다.`);
```

ES5 에서는 행간을 표현할 때 이렇게 합니다.

```javascript
var text = (
    'cat\n' +
    'dog\n' +
    'nickelodeon'
);
```

또는,

```javascript
var text = [
    'cat',
    'dog',
    'nickelodeon'
].join('\n');
```

**템플릿 리터럴** 에서는 행간을 유지할 수 있어, 굳이 명시적으로 구분자를 주지 않아도
간편하게 아래와 같이 표현할 수 있습니다.

```javascript
let text = ( `cat
dog
nickelodeon`
);
```

**템플릿 리터럴** 에서는 구문 작성도 지원합니다. 이렇게요.

```javascript
let today = new Date();
let text = `지금 시각은 ${today.toLocaleString()} 입니다.`;
```

<sup>[(차례로 돌아가기)](#table-of-contents)</sup>

## Destructuring

**변수 분해(Destructuring)** 기능을 통하여 변수나 객체(때론 더 깊게)에 있는 각 값들을
각 변수로 저장하는 간단한 구문을 제공합니다.

### Destructuring Arrays

배열을 변수로 분해

```javascript
var arr = [1, 2, 3, 4];
var a = arr[0];
var b = arr[1];
var c = arr[2];
var d = arr[3];
```

```javascript
let [a, b, c, d] = [1, 2, 3, 4];

console.log(a); // 1
console.log(b); // 2
```

### Destructuring Objects

객체를 변수로 분해

```javascript
var luke = { occupation: 'jedi', father: 'anakin' };
var occupation = luke.occupation; // 'jedi'
var father = luke.father; // 'anakin'
```

```javascript
let luke = { occupation: 'jedi', father: 'anakin' };
let {occupation, father} = luke;

console.log(occupation); // 'jedi'
console.log(father); // 'anakin'
```

<sup>[(차례로 돌아가기)](#table-of-contents)</sup>

## Modules

Prior to ES6, we used libraries such as [Browserify](http://browserify.org/)
to create modules on the client-side, and [require](https://nodejs.org/api/modules.html#modules_module_require_id)
in **Node.js**. With ES6, we can now directly use modules of all types
(AMD and CommonJS).

### Exporting in CommonJS

```javascript
module.exports = 1;
module.exports = { foo: 'bar' };
module.exports = ['foo', 'bar'];
module.exports = function bar () {};
```

### Exporting in ES6

With ES6, we have various flavors of exporting. We can perform
**Named Exports**:

```javascript
export let name = 'David';
export let age  = 25;​​
```

As well as **exporting a list** of objects:

```javascript
function sumTwo(a, b) {
    return a + b;
}

function sumThree(a, b, c) {
    return a + b + c;
}

export { sumTwo, sumThree };
```

We can also export a value simply by using the `export` keyword:

```javascript
export function sumTwo(a, b) {
    return a + b;
}

export function sumThree(a, b, c) {
    return a + b + c;
}
```

And lastly, we can **export default bindings**:

```javascript
function sumTwo(a, b) {
    return a + b;
}

function sumThree(a, b, c) {
    return a + b + c;
}

let api = {
    sumTwo,
    sumThree
};

export default api;
```

> **Best Practices**: Always use the `export default` method at **the end** of
the module. It makes it clear what is being exported, and saves time by having
to figure out what name a value was exported as. More so, the common practice
in CommonJS modules is to export a single value or object. By sticking to this
paradigm, we make our code easily readable and allow ourselves to interpolate
between CommonJS and ES6 modules.

### Importing in ES6

ES6 provides us with various flavors of importing. We can import an entire file:

```javascript
import 'underscore';
```

> It is important to note that simply **importing an entire file will execute
all code at the top level of that file**.

Similar to Python, we have named imports:

```javascript
import { sumTwo, sumThree } from 'math/addition';
```

We can also rename the named imports:

```javascript
import {
    sumTwo as addTwoNumbers,
    sumThree as sumThreeNumbers
} from 'math/addition';
```

In addition, we can **import all the things** (also called namespace import):

```javascript
import * as util from 'math/addition';
```

Lastly, we can import a list of values from a module:

```javascript
import * as additionUtil from 'math/addtion';
const { sumTwo, sumThree } = additionUtil;
```

When importing the default object we can choose which functions to import:

```javascript
import React from 'react';
const { Component, PropTypes } = React;
```

> **Note**: Values that are exported are **bindings**, not references.
Therefore, changing the binding of a variable in one module will affect the
value within the exported module. Avoid changing the public interface of these
exported values.

<sup>[(차례로 돌아가기)](#table-of-contents)</sup>

## Parameters

In ES5, we had varying ways to handle functions which needed **default values**,
**indefinite arguments**, and **named parameters**. With ES6, we can accomplish
all of this and more using more concise syntax.

### Default Parameters

```javascript
function addTwoNumbers(x, y) {
    x = x || 0;
    y = y || 0;
    return x + y;
}
```

In ES6, we can simply supply default values for parameters in a function:

```javascript
function addTwoNumbers(x=0, y=0) {
    return x + y;
}
```

```javascript
addTwoNumbers(2, 4); // 6
addTwoNumbers(2); // 2
addTwoNumbers(); // 0
```

### Rest Parameters

In ES5, we handled an indefinite number of arguments like so:

```javascript
function logArguments() {
    for (var i=0; i < arguments.length; i++) {
        console.log(arguments[i]);
    }
}
```

Using the **rest** operator, we can pass in an indefinite amount of arguments:

```javascript
function logArguments(...args) {
    for (let arg of args) {
        console.log(arg);
    }
}
```

### Named Parameters

One of the patterns in ES5 to handle named parameters was to use the **options
object** pattern, adopted from jQuery.

```javascript
function initializeCanvas(options) {
    var height = options.height || 600;
    var width  = options.width  || 400;
    var lineStroke = options.lineStroke || 'black';
}
```

We can achieve the same functionality using destructuring as a formal parameter
to a function:

```javascript
function initializeCanvas(
    { height=600, width=400, lineStroke='black'}) {
        // ...
    }
    // Use variables height, width, lineStroke here
```

If we want to make the entire value optional, we can do so by destructuring an
empty object:

```javascript
function initializeCanvas(
    { height=600, width=400, lineStroke='black'} = {}) {
        // ...
    }
```

### Spread Operator

We can use the spread operator to pass an array of values to be used as
parameters to a function:

```javascript
Math.max(...[-1, 100, 9001, -32]); // 9001
```

<sup>[(차례로 돌아가기)](#table-of-contents)</sup>

## Classes

Prior to ES6, we implemented Classes by creating a constructor function and
adding properties by extending the prototype:

```javascript
function Person(name, age, gender) {
    this.name   = name;
    this.age    = age;
    this.gender = gender;
}

Person.prototype.incrementAge = function () {
    return this.age += 1;
};
```

And created extended classes by the following:

```javascript
function Personal(name, age, gender, occupation, hobby) {
    Person.call(this, name, age, gender);
    this.occupation = occupation;
    this.hobby = hobby;
}

Personal.prototype = Object.create(Person.prototype);
Personal.prototype.constructor = Personal;
Personal.prototype.incrementAge = function () {
    return Person.prototype.incrementAge.call(this) += 20;
};
```

ES6 provides much needed syntactic sugar for doing this under the hood. We can
create Classes directly:

```javascript
class Person {
    constructor(name, age, gender) {
        this.name   = name;
        this.age    = age;
        this.gender = gender;
    }

    incrementAge() {
      this.age += 1;
    }
}
```

And extend them using the `extends` keyword:

```javascript
class Personal extends Person {
    constructor(name, age, gender, occupation, hobby) {
        super(name, age, gender);
        this.occupation = occupation;
        this.hobby = hobby;
    }

    incrementAge() {
        super.incrementAge();
        this.age += 20;
        console.log(this.age);
    }
}
```

> **Best Practice**: While the syntax for creating classes in ES6 obscures how
implementation and prototypes work under the hood, it is a good feature for
beginners and allows us to write cleaner code.

<sup>[(차례로 돌아가기)](#table-of-contents)</sup>

## Symbols

Symbols have existed prior to ES6, but now we have a public interface to using
them directly. One such example is to create unique property keys which will
never collide:

```javascript
const key = Symbol();
const keyTwo = Symbol();
const object = {};

object[key] = 'Such magic.';
object[keyTwo] = 'Much Uniqueness';

// Two Symbols will never have the same value
>> key === keyTwo
>> false
```

<sup>[(차례로 돌아가기)](#table-of-contents)</sup>

## Maps

**Maps** is a much needed data structure in JavaScript. Prior to ES6, we created
**hash** maps through objects:

```javascript
var map = new Object();
map[key1] = 'value1';
map[key2] = 'value2';
```

However, this does not protect us from accidentally overriding functions with
specific property names:

```javascript
> getOwnProperty({ hasOwnProperty: 'Hah, overwritten'}, 'Pwned');
> TypeError: Property 'hasOwnProperty' is not a function
```

Actual **Maps** allow us to `set`, `get` and `search` for values (and much more).

```javascript
let map = new Map();
> map.set('name', 'david');
> map.get('name'); // david
> map.has('name'); // true
```

The most amazing part of Maps is that we are no longer limited to just using
strings. We can now use any type as a key, and it will not be type-casted to
a string.

```javascript
let map = new Map([
    ['name', 'david'],
    [true, 'false'],
    [1, 'one'],
    [{}, 'object'],
    [function () {}, 'function']
]);

for (let key of map.keys()) {
    console.log(typeof key);
    // > string, boolean, number, object, function
}
```

> **Note**: Using non-primitive values such as functions or objects won't work
when testing equality using methods such as `map.get()`. As such, stick to
primitive values such as Strings, Booleans and Numbers.

We can also iterate over maps using `.entries()`:

```javascript
for (let [key, value] of map.entries()) {
    console.log(key, value);
}
```

<sup>[(차례로 돌아가기)](#table-of-contents)</sup>

## WeakMaps

In order to store private data in < ES5, we had various ways of doing this.
One such method was using naming conventions:

```javascript
class Person {
    constructor(age) {
        this._age = age;
    }

    _incrementAge() {
        this._age += 1;
    }
}
```

But naming conventions can cause confusion in a codebase and are not always
going to be upheld. Instead, we can use WeakMaps to store our values:

```javascript
let _age = new WeakMap();
class Person {
    constructor(age) {
        _age.set(this, age);
    }

    incrementAge() {
        let age = _age.get(this) + 1;
        _age.set(this, age);
        if (age > 50) {
            console.log('Midlife crisis');
        }
    }
}
```

The cool thing about using WeakMaps to store our private data is that their
keys do not give away the property names, which can be seen by using
`Reflect.ownKeys()`:

```javascript
> const person = new Person(50);
> person.incrementAge(); // 'Midlife crisis'
> Reflect.ownKeys(person); // []
```

A more practical example of using WeakMaps is to store data which is associated
to a DOM element without having to pollute the DOM itself:

```javascript
let map = new WeakMap();
let el  = document.getElementById('someElement');

// Store a weak reference to the element with a key
map.set(el, 'reference');

// Access the value of the element
let value = map.get(el); // 'reference'

// Remove the reference
el.parentNode.removeChild(el);
el = null;

value = map.get(el); // undefined
```

As shown above, once the object is is destroyed by the garbage collector,
the WeakMap will automatically remove the key-value pair which was identified
by that object.

> **Note**: To further illustrate the usefulness of this example, consider how
jQuery stores a cache of objects corresponding to DOM elements which have
references. Using WeakMaps, jQuery can automatically free up any memory that
was associated with a particular DOM element once it has been removed from the
document. In general, WeakMaps are very useful for any library that wraps DOM
elements.

<sup>[(차례로 돌아가기)](#table-of-contents)</sup>

## Promises

Promises allow us to turn our horizontal code (callback hell):

```javascript
func1(function (value1) {
    func2(value1, function (value2) {
        func3(value2, function (value3) {
            func4(value3, function (value4) {
                func5(value4, function (value5) {
                    // Do something with value 5
                });
            });
        });
    });
});
```

Into vertical code:

```javascript
func1(value1)
    .then(func2)
    .then(func3)
    .then(func4)
    .then(func5, value5 => {
        // Do something with value 5
    });
```

Prior to ES6, we used [bluebird](https://github.com/petkaantonov/bluebird) or
[Q](https://github.com/kriskowal/q). Now we have Promises natively:

```javascript
new Promise((resolve, reject) =>
    reject(new Error('Failed to fulfill Promise')))
        .catch(reason => console.log(reason));
```

Where we have two handlers, **resolve** (a function called when the Promise is
**fulfilled**) and **reject** (a function called when the Promise is **rejected**).

> **Benefits of Promises**: Error Handling using a bunch of nested callbacks
can get chaotic. Using Promises, we have a clear path to bubbling errors up
and handling them appropriately. Moreover, the value of a Promise after it has
been resolved/rejected is immutable - it will never change.

Here is a practical example of using Promises:

```javascript
var fetchJSON = function(url) {
    return new Promise((resolve, reject) => {
        $.getJSON(url)
            .done((json) => resolve(json))
            .fail((xhr, status, err) => reject(status + err.message));
    });
};
```

We can also **parallelize** Promises to handle an array of asynchronous
operations by using `Promise.all()`:

```javascript
var urls = [
    'http://www.api.com/items/1234',
    'http://www.api.com/items/4567'
];

var urlPromises = urls.map(fetchJSON);

Promise.all(urlPromises)
    .then(function (results) {
        results.forEach(function (data) {
        });
    })
    .catch(function (err) {
        console.log('Failed: ', err);
    });
```
