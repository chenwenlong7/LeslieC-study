## 1. d.ts(声明文件)

### [什么是声明语句](http://ts.xcatliu.com/basics/declaration-files.html#什么是声明语句)

假如我们想使用第三方库 jQuery，一种常见的方式是在 html 中通过 `<script>` 标签引入 jQuery，然后就可以使用全局变量 `$` 或 `jQuery` 了。

我们通常这样获取一个 `id` 是 `foo` 的元素：

```js
$('#foo');
// or
jQuery('#foo');
```

但是在 ts 中，编译器并不知道 `$` 或 `jQuery` 是什么东西[1](https://github.com/xcatliu/typescript-tutorial/tree/master/examples/declaration-files/01-jquery)：

```ts
jQuery('#foo');
// ERROR: Cannot find name 'jQuery'.
```

这时，我们需要使用 `declare var` 来定义它的类型[2](https://github.com/xcatliu/typescript-tutorial/tree/master/examples/declaration-files/02-declare-var)：

```ts
declare var jQuery: (selector: string) => any;

jQuery('#foo');
```

上例中，`declare var` 并没有真的定义一个变量，只是定义了全局变量 `jQuery` 的类型，仅仅会用于编译时的检查，在编译结果中会被删除。它编译结果是：

```js
jQuery('#foo');
```

除了 `declare var` 之外，还有其他很多种声明语句，将会在后面详细介绍。

### [什么是声明文件](http://ts.xcatliu.com/basics/declaration-files.html#什么是声明文件)

通常我们会把声明语句放到一个单独的文件（`jQuery.d.ts`）中，这就是声明文件[3](https://github.com/xcatliu/typescript-tutorial/tree/master/examples/declaration-files/03-jquery-d-ts)：

```ts
// src/jQuery.d.ts

declare var jQuery: (selector: string) => any;
// src/index.ts

jQuery('#foo');
```

声明文件必需以 `.d.ts` 为后缀。

一般来说，ts 会解析项目中所有的 `*.ts` 文件，当然也包含以 `.d.ts` 结尾的文件。所以当我们将 `jQuery.d.ts` 放到项目中时，其他所有 `*.ts` 文件就都可以获得 `jQuery` 的类型定义了。

```autoit
/path/to/project
├── src
|  ├── index.ts
|  └── jQuery.d.ts
└── tsconfig.json
```

假如仍然无法解析，那么可以检查下 `tsconfig.json` 中的 `files`、`include` 和 `exclude` 配置，确保其包含了 `jQuery.d.ts` 文件。

这里只演示了全局变量这种模式的声明文件，假如是通过模块导入的方式使用第三方库的话，那么引入声明文件又是另一种方式了，将会在后面详细介绍。

### [第三方声明文件](http://ts.xcatliu.com/basics/declaration-files.html#第三方声明文件)

当然，jQuery 的声明文件不需要我们定义了，社区已经帮我们定义好了：[jQuery in DefinitelyTyped](https://github.com/DefinitelyTyped/DefinitelyTyped/tree/master/types/jquery/index.d.ts)。

我们可以直接下载下来使用，但是更推荐的是使用 `@types` 统一管理第三方库的声明文件。

`@types` 的使用方式很简单，直接用 npm 安装对应的声明模块即可，以 jQuery 举例：

```bash
npm install @types/jquery --save-dev
```

可以在[这个页面](https://microsoft.github.io/TypeSearch/)搜索你需要的声明文件。

### [书写声明文件](http://ts.xcatliu.com/basics/declaration-files.html#书写声明文件)

当一个第三方库没有提供声明文件时，我们就需要自己书写声明文件了。前面只介绍了最简单的声明文件内容，而真正书写一个声明文件并不是一件简单的事，以下会详细介绍如何书写声明文件。

在不同的场景下，声明文件的内容和使用方式会有所区别。

库的使用场景主要有以下几种：

- [全局变量](http://ts.xcatliu.com/basics/declaration-files.html#quan-ju-bian-liang)：通过 `<script>` 标签引入第三方库，注入全局变量
- [npm 包](http://ts.xcatliu.com/basics/declaration-files.html#npm-bao)：通过 `import foo from 'foo'` 导入，符合 ES6 模块规范
- [UMD 库](http://ts.xcatliu.com/basics/declaration-files.html#umd-ku)：既可以通过 `<script>` 标签引入，又可以通过 `import` 导入
- [直接扩展全局变量](http://ts.xcatliu.com/basics/declaration-files.html#zhi-jie-kuo-zhan-quan-ju-bian-liang)：通过 `<script>` 标签引入后，改变一个全局变量的结构
- [在 npm 包或 UMD 库中扩展全局变量](http://ts.xcatliu.com/basics/declaration-files.html#zai-npm-bao-huo-umd-ku-zhong-kuo-zhan-quan-ju-bian-liang)：引用 npm 包或 UMD 库后，改变一个全局变量的结构
- [模块插件](http://ts.xcatliu.com/basics/declaration-files.html#mo-kuai-cha-jian)：通过 `<script>` 或 `import` 导入后，改变另一个模块的结构

### [全局变量](http://ts.xcatliu.com/basics/declaration-files.html#全局变量)

全局变量是最简单的一种场景，之前举的例子就是通过 `<script>` 标签引入 jQuery，注入全局变量 `$` 和 `jQuery`。

使用全局变量的声明文件时，如果是以 `npm install @types/xxx --save-dev` 安装的，则不需要任何配置。如果是将声明文件直接存放于当前项目中，则建议和其他源码一起放到 `src` 目录下（或者对应的源码目录下）：

```autoit
/path/to/project
├── src
|  ├── index.ts
|  └── jQuery.d.ts
└── tsconfig.json
```

如果没有生效，可以检查下 `tsconfig.json` 中的 `files`、`include` 和 `exclude` 配置，确保其包含了 `jQuery.d.ts` 文件。

全局变量的声明文件主要有以下几种语法：

- [`declare var`](http://ts.xcatliu.com/basics/declaration-files.html#declare-var) 声明全局变量
- [`declare function`](http://ts.xcatliu.com/basics/declaration-files.html#declare-function) 声明全局方法
- [`declare class`](http://ts.xcatliu.com/basics/declaration-files.html#declare-class) 声明全局类
- [`declare enum`](http://ts.xcatliu.com/basics/declaration-files.html#declare-enum) 声明全局枚举类型
- [`declare namespace`](http://ts.xcatliu.com/basics/declaration-files.html#declare-namespace) 声明（含有子属性的）全局对象
- [`interface` 和 `type`](http://ts.xcatliu.com/basics/declaration-files.html#interface-he-type) 声明全局类型

#### [`declare var`](http://ts.xcatliu.com/basics/declaration-files.html#declare-var)

在所有的声明语句中，`declare var` 是最简单的，如之前所学，它能够用来定义一个全局变量的类型。与其类似的，还有 `declare let` 和 `declare const`，使用 `let` 与使用 `var` 没有什么区别：

```ts
// src/jQuery.d.ts

declare let jQuery: (selector: string) => any;
// src/index.ts

jQuery('#foo');
// 使用 declare let 定义的 jQuery 类型，允许修改这个全局变量
jQuery = function(selector) {
    return document.querySelector(selector);
};
```

而当我们使用 `const` 定义时，表示此时的全局变量是一个常量，不允许再去修改它的值了[4](https://github.com/xcatliu/typescript-tutorial/tree/master/examples/declaration-files/04-declare-const-jquery)：

```ts
// src/jQuery.d.ts

declare const jQuery: (selector: string) => any;

jQuery('#foo');
// 使用 declare const 定义的 jQuery 类型，禁止修改这个全局变量
jQuery = function(selector) {
    return document.querySelector(selector);
};
// ERROR: Cannot assign to 'jQuery' because it is a constant or a read-only property.
```

一般来说，全局变量都是禁止修改的常量，所以大部分情况都应该使用 `const` 而不是 `var` 或 `let`。

需要注意的是，声明语句中只能定义类型，切勿在声明语句中定义具体的实现[5](https://github.com/xcatliu/typescript-tutorial/tree/master/examples/declaration-files/05-declare-jquery-value)：

```ts
declare const jQuery = function(selector) {
    return document.querySelector(selector);
};
// ERROR: An implementation cannot be declared in ambient contexts.
```

#### [`declare function`](http://ts.xcatliu.com/basics/declaration-files.html#declare-function)

`declare function` 用来定义全局函数的类型。jQuery 其实就是一个函数，所以也可以用 `function` 来定义：

```ts
// src/jQuery.d.ts

declare function jQuery(selector: string): any;
// src/index.ts

jQuery('#foo');
```

在函数类型的声明语句中，函数重载也是支持的[6](https://github.com/xcatliu/typescript-tutorial/tree/master/examples/declaration-files/06-declare-function)：

```ts
// src/jQuery.d.ts

declare function jQuery(selector: string): any;
declare function jQuery(domReadyCallback: () => any): any;
// src/index.ts

jQuery('#foo');
jQuery(function() {
    alert('Dom Ready!');
});
```

#### `declare class`[](http://ts.xcatliu.com/basics/declaration-files.html#declare-class)

当全局变量是一个类的时候，我们用 `declare class` 来定义它的类型[7](https://github.com/xcatliu/typescript-tutorial/tree/master/examples/declaration-files/07-declare-class)：

```ts
// src/Animal.d.ts

declare class Animal {
    name: string;
    constructor(name: string);
    sayHi(): string;
}
// src/index.ts

let cat = new Animal('Tom');
```

同样的，`declare class` 语句也只能用来定义类型，不能用来定义具体的实现，比如定义 `sayHi` 方法的具体实现则会报错：

```ts
// src/Animal.d.ts

declare class Animal {
    name: string;
    constructor(name: string);
    sayHi() {
        return `My name is ${this.name}`;
    };
    // ERROR: An implementation cannot be declared in ambient contexts.
}
```

#### [`declare enum`](http://ts.xcatliu.com/basics/declaration-files.html#declare-enum)

使用 `declare enum` 定义的枚举类型也称作外部枚举（Ambient Enums），举例如下[8](https://github.com/xcatliu/typescript-tutorial/tree/master/examples/declaration-files/08-declare-enum)：

```ts
// src/Directions.d.ts

declare enum Directions {
    Up,
    Down,
    Left,
    Right
}
// src/index.ts

let directions = [Directions.Up, Directions.Down, Directions.Left, Directions.Right];
```

与其他全局变量的类型声明一致，`declare enum` 仅用来定义类型，而不是具体的值。

`Directions.d.ts` 仅仅会用于编译时的检查，声明文件里的内容在编译结果中会被删除。它编译结果是：

```js
var directions = [Directions.Up, Directions.Down, Directions.Left, Directions.Right];
```

其中 `Directions` 是由第三方库定义好的全局变量。

#### [`declare namespace`](http://ts.xcatliu.com/basics/declaration-files.html#declare-namespace)

`namespace` 是 ts 早期时为了解决模块化而创造的关键字，中文称为命名空间。

由于历史遗留原因，在早期还没有 ES6 的时候，ts 提供了一种模块化方案，使用 `module` 关键字表示内部模块。但由于后来 ES6 也使用了 `module` 关键字，ts 为了兼容 ES6，使用 `namespace` 替代了自己的 `module`，更名为命名空间。

随着 ES6 的广泛应用，现在已经不建议再使用 ts 中的 `namespace`，而推荐使用 ES6 的模块化方案了，故我们不再需要学习 `namespace` 的使用了。

`namespace` 被淘汰了，但是在声明文件中，`declare namespace` 还是比较常用的，它用来表示全局变量是一个对象，包含很多子属性。

比如 `jQuery` 是一个全局变量，它是一个对象，提供了一个 `jQuery.ajax` 方法可以调用，那么我们就应该使用 `declare namespace jQuery` 来声明这个拥有多个子属性的全局变量。

```ts
// src/jQuery.d.ts

declare namespace jQuery {
    function ajax(url: string, settings?: any): void;
}
// src/index.ts

jQuery.ajax('/api/get_something');
```

注意，在 `declare namespace` 内部，我们直接使用 `function ajax` 来声明函数，而不是使用 `declare function ajax`。类似的，也可以使用 `const`, `class`, `enum` 等语句[9](https://github.com/xcatliu/typescript-tutorial/tree/master/examples/declaration-files/09-declare-namespace)：

```ts
// src/jQuery.d.ts

declare namespace jQuery {
    function ajax(url: string, settings?: any): void;
    const version: number;
    class Event {
        blur(eventType: EventType): void
    }
    enum EventType {
        CustomClick
    }
}
// src/index.ts

jQuery.ajax('/api/get_something');
console.log(jQuery.version);
const e = new jQuery.Event();
e.blur(jQuery.EventType.CustomClick);
```

##### [嵌套的命名空间](http://ts.xcatliu.com/basics/declaration-files.html#嵌套的命名空间)

如果对象拥有深层的层级，则需要用嵌套的 `namespace` 来声明深层的属性的类型[10](https://github.com/xcatliu/typescript-tutorial/tree/master/examples/declaration-files/10-declare-namespace-nesting)：

```ts
// src/jQuery.d.ts

declare namespace jQuery {
    function ajax(url: string, settings?: any): void;
    namespace fn {
        function extend(object: any): void;
    }
}
// src/index.ts

jQuery.ajax('/api/get_something');
jQuery.fn.extend({
    check: function() {
        return this.each(function() {
            this.checked = true;
        });
    }
});
```

假如 `jQuery` 下仅有 `fn` 这一个属性（没有 `ajax` 等其他属性或方法），则可以不需要嵌套 `namespace`[11](https://github.com/xcatliu/typescript-tutorial/tree/master/examples/declaration-files/11-declare-namespace-dot)：

```ts
// src/jQuery.d.ts

declare namespace jQuery.fn {
    function extend(object: any): void;
}
// src/index.ts

jQuery.fn.extend({
    check: function() {
        return this.each(function() {
            this.checked = true;
        });
    }
});
```

#### [`interface` 和 `type`](http://ts.xcatliu.com/basics/declaration-files.html#interface-和-type)

除了全局变量之外，可能有一些类型我们也希望能暴露出来。在类型声明文件中，我们可以直接使用 `interface` 或 `type` 来声明一个全局的接口或类型[12](https://github.com/xcatliu/typescript-tutorial/tree/master/examples/declaration-files/12-interface)：

```ts
// src/jQuery.d.ts

interface AjaxSettings {
    method?: 'GET' | 'POST'
    data?: any;
}
declare namespace jQuery {
    function ajax(url: string, settings?: AjaxSettings): void;
}
```

这样的话，在其他文件中也可以使用这个接口或类型了：

```ts
// src/index.ts

let settings: AjaxSettings = {
    method: 'POST',
    data: {
        name: 'foo'
    }
};
jQuery.ajax('/api/post_something', settings);
```

`type` 与 `interface` 类似，不再赘述。

##### [防止命名冲突](http://ts.xcatliu.com/basics/declaration-files.html#防止命名冲突)

暴露在最外层的 `interface` 或 `type` 会作为全局类型作用于整个项目中，我们应该尽可能的减少全局变量或全局类型的数量。故最好将他们放到 `namespace` 下[13](https://github.com/xcatliu/typescript-tutorial/tree/master/examples/declaration-files/13-avoid-name-conflict)：

```ts
// src/jQuery.d.ts

declare namespace jQuery {
    interface AjaxSettings {
        method?: 'GET' | 'POST'
        data?: any;
    }
    function ajax(url: string, settings?: AjaxSettings): void;
}
```

注意，在使用这个 `interface` 的时候，也应该加上 `jQuery` 前缀：

```ts
// src/index.ts

let settings: jQuery.AjaxSettings = {
    method: 'POST',
    data: {
        name: 'foo'
    }
};
jQuery.ajax('/api/post_something', settings);
```

#### [声明合并](http://ts.xcatliu.com/basics/declaration-files.html#声明合并)

假如 jQuery 既是一个函数，可以直接被调用 `jQuery('#foo')`，又是一个对象，拥有子属性 `jQuery.ajax()`（事实确实如此），那么我们可以组合多个声明语句，它们会不冲突的合并起来[14](https://github.com/xcatliu/typescript-tutorial/tree/master/examples/declaration-files/14-declaration-merging)：

```ts
// src/jQuery.d.ts

declare function jQuery(selector: string): any;
declare namespace jQuery {
    function ajax(url: string, settings?: any): void;
}
// src/index.ts

jQuery('#foo');
jQuery.ajax('/api/get_something');
```

关于声明合并的更多用法，可以查看[声明合并](http://ts.xcatliu.com/advanced/declaration-merging.html)章节。

## 2.import type 和 import 的区别


如先前介绍， TypeScript 3.8 引入了新的语法 -- 「 仅仅导入 / 导出声明 」。

该语法在使用时为类型解析过程增加了确定性。

现在，编译器（无论是tsc，babel还是其他）都将能够查看单个文件，并取消导入或导出（如果它是TypeScript类型）。

`import type ... from`让编译器知道您要导入的内容绝对是一种类型。

`export type ... from`一样， 仅用作导出。

```javascript
// src/lib-type-re-export.ts
export type { Track, Playlist } from "./types";
export type { CreatePlaylistRequestParams } from "./api";
export { createPlaylist } from "./api";

// 会被编译为：

// dist/lib-type-re-export.js
export { createPlaylist } from "./api";
```



**参考：**

[TypeScript 入门教程](http://ts.xcatliu.com/basics/declaration-files.html#%E4%B9%A6%E5%86%99%E5%A3%B0%E6%98%8E%E6%96%87%E4%BB%B6)