### 其他内容

#### 命名空间

> 这篇文章描述了如何在TypeScript里使用命名空间（之前叫做“内部模块”）来组织你的代码

代码示例

```javascript
    interface StringValidator {
        isAcceptable(s: string): boolean;
    }

    let lettersRegexp = /^[A-Za-z]+$/;
    let numberRegexp = /^[0-9]+$/;

    class LettersOnlyValidator implements StringValidator {
        isAcceptable(s: string) {
            return lettersRegexp.test(s);
        }
    }

    class ZipCodeValidator implements StringValidator {
        isAcceptable(s: string) {
            return s.length === 5 && numberRegexp.test(s);
        }
    }

    // Some samples to try
    let strings = ["Hello", "98052", "101"];

    // Validators to use
    let validators: { [s: string]: StringValidator; } = {};
    validators["ZIP code"] = new ZipCodeValidator();
    validators["Letters only"] = new LettersOnlyValidator();

    // Show whether each string passed each validator
    for (let s of strings) {
        for (let name in validators) {
            let isMatch = validators[name].isAcceptable(s);
            console.log(`'${ s }' ${ isMatch ? "matches" : "does not match" } '${ name }'.`);
        }
    }
```

`使用命名空间的验证器`
> 把所有与验证器相关的类型都放到一个叫做Validation的命名空间里。 因为我们想让这些接口和类在命名空间之外也是可访问的，所以需要使用 `export`


```javascript
    namespace Validation {
        export interface StringValidator {
            isAcceptable(s: string): boolean;
        }

        const lettersRegexp = /^[A-Za-z]+$/;
        const numberRegexp = /^[0-9]+$/;

        export class LettersOnlyValidator implements StringValidator {
            isAcceptable(s: string) {
                return lettersRegexp.test(s);
            }
        }

        export class ZipCodeValidator implements StringValidator {
            isAcceptable(s: string) {
                return s.length === 5 && numberRegexp.test(s);
            }
        }
    }

    // Some samples to try
    let strings = ["Hello", "98052", "101"];

    // Validators to use
    let validators: { [s: string]: Validation.StringValidator; } = {};
    validators["ZIP code"] = new Validation.ZipCodeValidator();
    validators["Letters only"] = new Validation.LettersOnlyValidator();

    // Show whether each string passed each validator
    for (let s of strings) {
        for (let name in validators) {
            console.log(`"${ s }" - ${ validators[name].isAcceptable(s) ? "matches" : "does not match" } ${ name }`);
        }
    }
```

`分离到多文件`
> 现在，我们把Validation命名空间分割成多个文件。 尽管是不同的文件，它们仍是同一个命名空间，并且在使用的时候就如同它们在一个文件中定义的一样。 因为不同文件之间存在依赖关系，所以我们加入了引用标签来告诉编译器文件之间的关联

- Validation.ts

```javascript
    namespace Validation {
        export interface StringValidator {
            isAcceptable(s: string): boolean;
        }
    }
```

- LettersOnlyValidator.ts

```javascript
    /// <reference path="Validation.ts" />
    namespace Validation {
        const lettersRegexp = /^[A-Za-z]+$/;
        export class LettersOnlyValidator implements StringValidator {
            isAcceptable(s: string) {
                return lettersRegexp.test(s);
            }
        }
    }
```
- ZipCodeValidator.ts

```javascript
    /// <reference path="Validation.ts" />
    namespace Validation {
        const numberRegexp = /^[0-9]+$/;
        export class ZipCodeValidator implements StringValidator {
            isAcceptable(s: string) {
                return s.length === 5 && numberRegexp.test(s);
            }
        }
    }
```
- Test.ts

```javascript
    /// <reference path="Validation.ts" />
    /// <reference path="LettersOnlyValidator.ts" />
    /// <reference path="ZipCodeValidator.ts" />

    // Some samples to try
    let strings = ["Hello", "98052", "101"];

    // Validators to use
    let validators: { [s: string]: Validation.StringValidator; } = {};
    validators["ZIP code"] = new Validation.ZipCodeValidator();
    validators["Letters only"] = new Validation.LettersOnlyValidator();

    // Show whether each string passed each validator
    for (let s of strings) {
        for (let name in validators) {
            console.log(`"${ s }" - ${ validators[name].isAcceptable(s) ? "matches" : "does not match" } ${ name }`);
        }
    }
```

当涉及到多文件时，我们必须确保所有编译后的代码都被加载了。 我们有两种方式

第一种方式，把所有的输入文件编译为一个输出文件

`tsc --outFile sample.js Test.ts`
或单独地指定每个文件。
`tsc --outFile sample.js Validation.ts LettersOnlyValidator.ts ZipCodeValidator.ts Test.ts`

第二种方式，我们可以编译每一个文件（默认方式），那么每个源文件都会对应生成一个JavaScript文件。 然后，在页面上通过 `<script>`标签把所有生成的JavaScript文件按正确的顺序引进来

```javascript
    <script src="Validation.js" type="text/javascript" />
    <script src="LettersOnlyValidator.js" type="text/javascript" />
    <script src="ZipCodeValidator.js" type="text/javascript" />
    <script src="Test.js" type="text/javascript" />
```

`别名`

```javascript
    namespace Shapes {
        export namespace Polygons {
            export class Triangle { }
            export class Square { }
        }
    }

    import polygons = Shapes.Polygons;
    let sq = new polygons.Square(); // Same as "new Shapes.Polygons.Square()"
```

`外部命名空间`

```javascript
   declare namespace D3 {
        export interface Selectors {
            select: {
                (selector: string): Selection;
                (element: EventTarget): Selection;
            };
        }

        export interface Event {
            x: number;
            y: number;
        }

        export interface Base extends Selectors {
            event: Event;
        }
    }

    declare var d3: D3.Base; 
```

#### 声明合并

`合并接口`

```javascript
   // 接口的非函数的成员应该是唯一的。如果它们不是唯一的，那么它们必须是相同的类型。如果两个接口中同时声明了同名的非函数成员且它们的类型不同，则编译器会报错。 
   interface Box {
        height: number;
        width: number;
    }

    interface Box {
        scale: number;
    }

    let box: Box = {height: 5, width: 6, scale: 10}; 
```

```javascript
    // 对于函数成员，每个同名函数声明都会被当成这个函数的一个重载。 同时需要注意，当接口 A与后来的接口 A合并时，后面的接口具有更高的优先级。
    interface Cloner {
        clone(animal: Animal): Animal;
    }

    interface Cloner {
        clone(animal: Sheep): Sheep;
    }

    interface Cloner {
        clone(animal: Dog): Dog;
        clone(animal: Cat): Cat;
    }

    interface Cloner {
        clone(animal: Dog): Dog;
        clone(animal: Cat): Cat;
        clone(animal: Sheep): Sheep;
        clone(animal: Animal): Animal;
    }
```

```javascript
    // 这个规则有一个例外是当出现特殊的函数签名时。 如果签名里有一个参数的类型是 单一的字符串字面量（比如，不是字符串字面量的联合类型），那么它将会被提升到重载列表的最顶端。
    interface Document {
        createElement(tagName: any): Element;
    }
    interface Document {
        createElement(tagName: "div"): HTMLDivElement;
        createElement(tagName: "span"): HTMLSpanElement;
    }
    interface Document {
        createElement(tagName: string): HTMLElement;
        createElement(tagName: "canvas"): HTMLCanvasElement;
    }

    interface Document {
        createElement(tagName: "canvas"): HTMLCanvasElement;
        createElement(tagName: "div"): HTMLDivElement;
        createElement(tagName: "span"): HTMLSpanElement;
        createElement(tagName: string): HTMLElement;
        createElement(tagName: any): Element;
    }
```

`合并命名空间`

对于命名空间的合并，模块导出的同名接口进行合并，构成单一命名空间内含合并后的接口。

对于命名空间里值的合并，如果当前已经存在给定名字的命名空间，那么后来的命名空间的导出成员会被加到已经存在的那个模块里。

```javascript
    namespace Animals {
        export class Zebra { }
    }

    namespace Animals {
        export interface Legged { numberOfLegs: number; }
        export class Dog { }
    }


    // 等同于
    namespace Animals {
        export interface Legged { numberOfLegs: number; }

        export class Zebra { }
        export class Dog { }
    }

```
除了这些合并外，你还需要了解非导出成员是如何处理的。 非导出成员仅在其原有的（合并前的）命名空间内可见。这就是说合并之后，从其它命名空间合并进来的成员无法访问非导出成员。

```javascript
    namespace Animal {
        let haveMuscles = true;

        export function animalsHaveMuscles() {
            return haveMuscles;
        }
    }

    namespace Animal {
        export function doAnimalsHaveMuscles() {
            return haveMuscles;  // Error, because haveMuscles is not accessible here
        }
    }
```

#### 装饰器

`基础使用`
> 有一个`@sealed`装饰器，我们会这样定义`sealed`函数：

```javascript
    function sealed(target) {
        // do something with "target" ...
    }
```

`装饰器工厂`

```javascript
    function color(value: string) { // 这是一个装饰器工厂
        return function (target) { //  这是装饰器
            // do something with "target" and "value"...
        }
    }
```

`装饰器组合`

在TypeScript里，当多个装饰器应用在一个声明上时会进行如下步骤的操作：

- 由上至下依次对装饰器表达式求值。
- 求值的结果会被当作函数，由下至上依次调用。

```javascript
    function f() {
        console.log("f(): evaluated");
        return function (target, propertyKey: string, descriptor: PropertyDescriptor) {
            console.log("f(): called");
        }
    }

    function g() {
        console.log("g(): evaluated");
        return function (target, propertyKey: string, descriptor: PropertyDescriptor) {
            console.log("g(): called");
        }
    }

    class C {
        @f()
        @g()
        method() {}
    }
    /*
        控制台打印
        f(): evaluated
        g(): evaluated
        g(): called
        f(): called
    */
```

`类装饰器`
> 类装饰器表达式会在运行时当作函数被调用，类的`构造函数`作为其唯一的参数。

```javascript
    @sealed
    class Greeter {
        greeting: string;
        constructor(message: string) {
            this.greeting = message;
        }
        greet() {
            return "Hello, " + this.greeting;
        }
    }

    function sealed(constructor: Function) {
        Object.seal(constructor);
        Object.seal(constructor.prototype);
    }
```
`方法装饰器`

方法装饰器表达式会在运行时当作函数被调用，传入下列3个参数：

- 1、对于`静态成员`来说是类的`构造函数`，对于`实例成员`是类的`原型对象`。
- 2、成员的名字。
- 3、成员的属性描述符。

如果方法装饰器`返回一个值`，它会被用作方法的`属性描述符`。

```javascript
    class Greeter {
        greeting: string;
        constructor(message: string) {
            this.greeting = message;
        }

        @enumerable(false)
        greet() {
            return "Hello, " + this.greeting;
        }
    }

    function enumerable(value: boolean) {
        return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
            descriptor.enumerable = value;
        };
    }
```

`访问器装饰器`
> TypeScript不允许同时装饰一个成员的get和set访问器, 装饰器的规则同`方法装饰器`

```javascript
    class Point {
        private _x: number;
        private _y: number;
        constructor(x: number, y: number) {
            this._x = x;
            this._y = y;
        }

        @configurable(false)
        get x() { return this._x; }

        @configurable(false)
        get y() { return this._y; }
    }

    function configurable(value: boolean) {
        return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
            descriptor.configurable = value;
        };
    }
```

`属性装饰器`

属性装饰器表达式会在运行时当作函数被调用，传入下列2个参数：

- 对于静态成员来说是类的构造函数，对于实例成员是类的原型对象。
- 成员的名字。

```javascript
    class Greeter {
        @format("Hello, %s")
        greeting: string;

        constructor(message: string) {
            this.greeting = message;
        }
        greet() {
            let formatString = getFormat(this, "greeting");
            return formatString.replace("%s", this.greeting);
        }
    }

    import "reflect-metadata";

    const formatMetadataKey = Symbol("format");

    function format(formatString: string) {
        return Reflect.metadata(formatMetadataKey, formatString);
    }

    function getFormat(target: any, propertyKey: string) {
        return Reflect.getMetadata(formatMetadataKey, target, propertyKey);
    }
```

`参数装饰器`

参数装饰器表达式会在运行时当作函数被调用，传入下列3个参数：

- 对于静态成员来说是类的构造函数，对于实例成员是类的原型对象。
- 成员的名字。
- 参数在函数参数列表中的索引。


```javascript
    class Greeter {
        greeting: string;

        constructor(message: string) {
            this.greeting = message;
        }

        @validate
        greet(@required name: string) {
            return "Hello " + name + ", " + this.greeting;
        }
    }
```
`装饰器执行顺序`
- 有多个参数装饰器时：从最后一个参数依次向前执行
- 方法和方法参数中参数装饰器先执行。
- 类装饰器总是最后执行
- 方法和属性装饰器，谁在前面谁先执行。因为参数属于方法一部分，所以参数会一直紧紧挨着方法执行
- 类比React组件的componentDidMount 先上后下、先内后外
```javascript
    namespace e {
        function Class1Decorator() {
            return function (target: any) {
                console.log("类1装饰器");
            }
        }
        function Class2Decorator() {
            return function (target: any) {
                console.log("类2装饰器");
            }
        }
        function MethodDecorator() {
            return function (target: any, methodName: string, descriptor: PropertyDescriptor) {
                console.log("方法装饰器");
            }
        }
        function Param1Decorator() {
            return function (target: any, methodName: string, paramIndex: number) {
                console.log("参数1装饰器");
            }
        }
        function Param2Decorator() {
            return function (target: any, methodName: string, paramIndex: number) {
                console.log("参数2装饰器");
            }
        }
        function PropertyDecorator(name: string) {
            return function (target: any, propertyName: string) {
                console.log(name + "属性装饰器");
            }
        }

        @Class1Decorator()
        @Class2Decorator()
        class Person {
            @PropertyDecorator('name')
            name: string = 'zhufeng';
            @PropertyDecorator('age')
            age: number = 10;
            @MethodDecorator()
            greet(@Param1Decorator() p1: string, @Param2Decorator() p2: string) { }
        }
    }
    /**
    name属性装饰器
    age属性装饰器
    参数2装饰器
    参数1装饰器
    方法装饰器
    类2装饰器
    类1装饰器
    */
```
`装饰器的应用`

[通过ts装饰器封装controller路由请求](https://github.com/Henry-Diasa/ts-decorator-router)

#### Mixins

```javascript
    // Disposable Mixin
    class Disposable {
        isDisposed: boolean;
        dispose() {
            this.isDisposed = true;
        }

    }

    // Activatable Mixin
    class Activatable {
        isActive: boolean;
        activate() {
            this.isActive = true;
        }
        deactivate() {
            this.isActive = false;
        }
    }

    class SmartObject implements Disposable, Activatable {
        constructor() {
            setInterval(() => console.log(this.isActive + " : " + this.isDisposed), 500);
        }

        interact() {
            this.activate();
        }

        // Disposable
        isDisposed: boolean = false;
        dispose: () => void;
        // Activatable
        isActive: boolean = false;
        activate: () => void;
        deactivate: () => void;
    }
    applyMixins(SmartObject, [Disposable, Activatable]);

    let smartObj = new SmartObject();
    setTimeout(() => smartObj.interact(), 1000);

    ////////////////////////////////////////
    // In your runtime library somewhere
    ////////////////////////////////////////

    function applyMixins(derivedCtor: any, baseCtors: any[]) {
        baseCtors.forEach(baseCtor => {
            Object.getOwnPropertyNames(baseCtor.prototype).forEach(name => {
                derivedCtor.prototype[name] = baseCtor.prototype[name];
            });
        });
    }
```

#### 定义Module

- jquery.d.ts
```javascript
    // Es6 模块化
    declare module 'jquery' {
        interface JqueryInstance {
            html: (html: string) => JqueryInstance;
        }
        // 混合类型
        function $(readyFunc: () => void): void;
        function $(selector: string): JqueryInstance;
        namespace $ {
            namespace fn {
                class init {}
            }
        }
        export = $;
    }
```

- index.ts

```javascript
    import $ from 'jquery';

    $(function() {
        $('body').html('<div>123</div>');
        new $.fn.init();
    });

```