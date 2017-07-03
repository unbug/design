# JavaScript API

[MVP](MVP.md) 里提到， 在 Web 上访问 WebAssembly 的唯一途径是通过显示的 JS API，如下定义。
(在 [将来 :unicorn:][future general], WebAssembly 也可能 通过一个 HTML `<script type='module'>` 
标签加载和直接运行，及其他可以通过 URL-作为 [ES6 模块集成](Modules.md#integration-with-es6-modules) 
的一部分加载 ES6 模块的 Web API.)

WebAssembly JS API 的 TypeScript 声明文件可以在 
[这里](https://github.com/01alchemist/webassembly-types/blob/master/webassembly.d.ts)找到，
它支持自动补全也会使 TypeScript 的编译器收益。

## 陷阱指令

一旦 WebAssembly 的语义标明了一个[陷阱指令](Semantics.md#traps)，就有一个 `WebAssembly.RuntimeError` 对象抛出，
WebAssembly 代码（当前）是无法捕获这些异常的， 因此这些异常将有必要传播到最近的一个非 WebAssembly 的调用者
（不是浏览器就是 JavaScript）那像一个常规的 JavaScript 异常一样被处理。

如果 WebAssembly 通过 import 调用 JavaScript 并且 JavaScript 抛出一个异常，异常自激活的 WebAssembly 传播到其调用者。

由于 JavaScript 异常能被捕获处理，捕获陷阱指令后 JavaScript 还能继续调用 WebAssembly 的 export，陷阱指令则不会，
一般来说就防止了未来代码的执行。

## 堆栈溢出

一旦 WebAssembly 代码出现[堆栈溢出](Semantics.md#stack-overflow)，JavaScript 同样会抛出相同的栈溢出异常。

## `WebAssembly` 对象

`WebAssembly` 对象是全局属性 `WebAssembly` 的值。如 `Math` 和 `JSON` 对象，`WebAssembly` 对象也是一个 JS 对象字面量
 （非构造器或函数）在其命名空间里有以下属性：

### `WebAssembly [ @@toStringTag ]` 对象属性

[`@@toStringTag`](https://tc39.github.io/ecma262/#sec-well-known-symbols) 属性的初始值是字符串 `"WebAssembly"`.

本属性特征属性有 { [[Writable]]: `false`, [[Enumerable]]: `false`, [[Configurable]]: `true` }.

### `WebAssembly` 对象的构造器属相

已有的对象

* `WebAssembly.Module` : [`WebAssembly.Module` 构造器](#webassemblymodule-constructor)
* `WebAssembly.Instance` : [`WebAssembly.Instance` 构造器](#webassemblyinstance-constructor)
* `WebAssembly.Memory` : [`WebAssembly.Memory` 构造器](#webassemblymemory-constructor)
* `WebAssembly.Table` : [`WebAssembly.Table` 构造器](#webassemblytable-constructor)
* `WebAssembly.CompileError` : [NativeError](http://tc39.github.io/ecma262/#sec-nativeerror-object-structure)
   WebAssembly 解码或校验过程中产生了 error
* `WebAssembly.LinkError` : [NativeError](http://tc39.github.io/ecma262/#sec-nativeerror-object-structure)
   WebAssembly 实例化模块时产生了 error （相较于开始函数的陷阱指令产生的）
* `WebAssembly.RuntimeError` : [NativeError](http://tc39.github.io/ecma262/#sec-nativeerror-object-structure)
   WebAssembly 遇到任何[陷阱指令](#traps) 是产生.

### `WebAssembly` 对象的函数属相

#### `WebAssembly.validate`

`validate` 函数的特征为:

```
Boolean validate(BufferSource bytes)
```

如果给定的 `bytes` 参数不是个
[`BufferSource`](https://heycam.github.io/webidl/#common-BufferSource)，会抛出 `TypeError`.

否则， 此函数的**校验**会和 [WebAssembly
specification](https://github.com/WebAssembly/spec/blob/master/interpreter/)中定义的一样，校验成功会返回 `true`，
失败返回 `false` 。

#### `WebAssembly.compile`

`compile` 函数的特征为:

```
Promise<WebAssembly.Module> compile(BufferSource bytes)
```

如果给定的 `bytes` 参数不是个
[`BufferSource`](https://heycam.github.io/webidl/#common-BufferSource)，
返回的 `Promise` 是就被 [rejected](http://tc39.github.io/ecma262/#sec-rejectpromise)
成了 [`TypeError`](https://tc39.github.io/ecma262/#sec-native-error-types-used-in-this-standard-typeerror)。

否侧，此函数会如 [`WebAssembly.Module` 构造器](#webassemblymodule-constructor) 里所描述的开始异步编译 `WebAssembly.Module`。
成功 `Promise` 就被 [fulfilled](http://tc39.github.io/ecma262/#sec-fulfillpromise)
成 `WebAssembly.Module` 对象。失败，`Promise` 就被
[rejected](http://tc39.github.io/ecma262/#sec-rejectpromise) 成 
`WebAssembly.CompileError`。

异步编译逻辑表现为，对给定的 `BufferSource` 在调用 `compile` 过程中进行状态拷贝； `BufferSource` 在 `compile` 
返回前的突变不会影响正在进行的编译。

据 [未来 :unicorn:][future streaming]， 此方法可以被扩展并接受 [stream](https://streams.spec.whatwg.org)，
 因此支持异步，后台运行，流编译。

#### `WebAssembly.instantiate`

`instantiate` 函数会根据参数类型被重载。
若以下重载方案都不匹配，返回的 `Promise` 会被
[rejected](http://tc39.github.io/ecma262/#sec-rejectpromise) 成
 [`TypeError`](https://tc39.github.io/ecma262/#sec-native-error-types-used-in-this-standard-typeerror).

```
dictionary WebAssemblyInstantiatedSource {
   required WebAssembly.Module module;
   required WebAssembly.Instance instance;
};

Promise<WebAssemblyInstantiatedSource>
  instantiate(BufferSource bytes [, importObject])
```

如果给定的 `bytes` 参数不是个
[`BufferSource`](https://heycam.github.io/webidl/#common-BufferSource),
返回的 `Promise` 会被 [rejected](http://tc39.github.io/ecma262/#sec-rejectpromise)
成 [`TypeError`](https://tc39.github.io/ecma262/#sec-native-error-types-used-in-this-standard-typeerror).

此函数会如 [`WebAssembly.Module` 构造器](#webassemblymodule-constructor) 里所描述的开始异步编译 `WebAssembly.Module`，
然后开始队列如 [`WebAssembly.Instance` 构造器](#webassemblyinstance-constructor) 中描述的用 `importObject` 
实例化 `Module` 的结果。实例化后且在进行下一步前任何其他异步任务都可能被运行。成功，
`Promise` 会被 [fulfilled](http://tc39.github.io/ecma262/#sec-fulfillpromise)
一个 `{module, instance}` 对象，含有 `WebAssembly.Module` 和 `WebAssembly.Instance`。
 `module` 和 `instance` 属性特征都是 configurable, enumerable 和 writable。

失败，`Promise` 会被
[rejected](http://tc39.github.io/ecma262/#sec-rejectpromise) 成
`WebAssembly.CompileError`， `WebAssembly.LinkError`， 或 `WebAssembly.RuntimeError`，取决于失败的场景。

异步编译逻辑表现为，对给定的 `BufferSource` 在调用 `compile` 过程中进行状态拷贝； `BufferSource` 在 `compile` 
返回前的突变不会影响正在进行的编译。

```
Promise<WebAssembly.Instance> instantiate(moduleObject [, importObject])
```

以上只会在第一个参数是 `WebAssembly.Module` 实例时应用。

此函数会如 [`WebAssembly.Instance` 构造器](#webassemblyinstance-constructor) 所述异步运行一个任务来通过
`moduleObject` 和 `importObject` 实例化 `WebAssembly.Instance`。实例化后且在进行下一步前任何其他异步任务都可能被运行。成功，
`Promise` 会被 [fulfilled](http://tc39.github.io/ecma262/#sec-fulfillpromise)
一个 `WebAssembly.Instance` 对象。失败，`Promise` 会被
[rejected](http://tc39.github.io/ecma262/#sec-rejectpromise) 成
`WebAssembly.CompileError`， `WebAssembly.LinkError`， 或 `WebAssembly.RuntimeError`，取决于失败的场景。

## `WebAssembly.Module` 对象

`WebAssembly.Module` 对象反应的是编译 WebAssembly 二进制格式模块后的无状态结果，包含一个内部占位:

 * [[Module]] : 一个特别声明的 [`Ast.module`](https://github.com/WebAssembly/spec/blob/master/interpreter/spec/ast.ml#L176)

### `WebAssembly.Module` 构造器

`WebAssembly.Module` 构造器的特征为:

```
new Module(BufferSource bytes)
```

若 NewTarget 是 `undefined`，会抛一个 [`TypeError`](https://tc39.github.io/ecma262/#sec-native-error-types-used-in-this-standard-typeerror)
异常 （即，此构造器不能当函数来叫，只能 `new`）。

如果给定的 `bytes` 参数不是个
[`BufferSource`](https://heycam.github.io/webidl/#common-BufferSource),
会抛出一个 [`TypeError`](https://tc39.github.io/ecma262/#sec-native-error-types-used-in-this-standard-typeerror)
异常。

否则，此函数表现为 `BufferSource` 的异步编译:

1. 字节范围会被 `BufferSource` 分隔是首要解码逻辑，依据了 [BinaryEncoding.md](BinaryEncoding.md)，校验规则依据 [spec/valid.ml](https://github.com/WebAssembly/spec/blob/master/interpreter/spec/valid.ml#L415)。
1. `string` 的值在 `Ast.module` 内解码成 UTF8，依据了 [Web.md](Web.md#names)。
1. 成功，会返回一个 `WebAssembly.Module` 对象，含有 [[Module]] 属性与 `Ast.module` 里设置的相同。
1. 失败，会抛出 `WebAssembly.CompileError` 异常。

### `WebAssembly.Module.prototype [ @@toStringTag ]` 属性

[`@@toStringTag`](https://tc39.github.io/ecma262/#sec-well-known-symbols) 初始值是 `"WebAssembly.Module"` 的值。

本属性特征属性有 { [[Writable]]: `false`, [[Enumerable]]: `false`, [[Configurable]]: `true` }.

### `WebAssembly.Module.exports`

`exports` 函数的特征：

```
Array exports(moduleObject)
```

如 `moduleObject` 不是一个 `WebAssembly.Module`，会抛出 [`TypeError`](https://tc39.github.io/ecma262/#sec-native-error-types-used-in-this-standard-typeerror)
异常。

每次调用此函数它都返回一个新的 `Array`。这样的 `Array` 的产生方式是将每个
[`Ast.export`](https://github.com/WebAssembly/spec/blob/master/interpreter/spec/ast.ml#L152)
`e` 属于 [moduleObject.[[Module]].exports](https://github.com/WebAssembly/spec/blob/master/interpreter/spec/ast.ml#L187)
映射到 `{ name: String(e.name), kind: e.ekind }` 对象，此 `e.name` 是 [decoded as UTF8](Web.md#names)
，`e.ekind` 映射的值是这些字符串 `"function"`, `"table"`, `"memory"`, `"global"` 中的一个。

注意：其他字段，如 `signature` 日后可能会被添加进来。

返回的 `Array` export 顺序与 WebAssembly 二进制的 export 列表相同。

### `WebAssembly.Module.imports`

`imports` 函数的特征：

```
Array imports(moduleObject)
```

如 `moduleObject` 不是一个 `WebAssembly.Module`，会抛出 [`TypeError`](https://tc39.github.io/ecma262/#sec-native-error-types-used-in-this-standard-typeerror)
异常。

每次调用此函数它都返回一个新的 `Array`。这样的 `Array` 的产生方式是将每个
[`Ast.import`](https://github.com/WebAssembly/spec/blob/master/interpreter/spec/ast.ml#L167)
`i` 属于 [moduleObject.[[Module]].imports](https://github.com/WebAssembly/spec/blob/master/interpreter/spec/ast.ml#L203)
映射到 `{ module: String(i.module_name), name: String(i.item_name), kind: i.ikind }` 对象，此
`i.module_name` 和 `i.item_name` 是  [decoded as UTF8](Web.md#names)，
`i.ikind` 映射的值是这些字符串 `"function"`, `"table"`, `"memory"`, `"global"` 中的一个。

注意：其他字段，如 `signature` 日后可能会被添加进来。

返回的 `Array` import 顺序与 WebAssembly 二进制的 import 列表相同。

### `WebAssembly.Module.customSections`

The `customSections` function has the signature:

```
Array customSections(moduleObject, sectionName)
```

If `moduleObject` is not a `WebAssembly.Module`, a [`TypeError`](https://tc39.github.io/ecma262/#sec-native-error-types-used-in-this-standard-typeerror)
is thrown.

Let `sectionNameString` be the result of [`ToString`](https://tc39.github.io/ecma262/#sec-tostring)(`sectionName`).

This function returns a new `Array` every time it is called. Each such `Array` is produced by mapping each
[custom section](BinaryEncoding.md#high-level-structure) (i.e., section with
`id` 0) whose `name` field ([decoded as UTF-8](Web.md#names)) is equal to
`sectionNameString` to an `ArrayBuffer` containing a copy of the section's
`payload_data`. (Note: `payload_data` does not include `name` or `name_len`.).

The `Array` is populated in the same order custom sections appear in the WebAssembly binary.

### Structured Clone of a `WebAssembly.Module`

A `WebAssembly.Module` is a
[cloneable object](https://html.spec.whatwg.org/multipage/infrastructure.html#cloneable-objects)
which means it can be cloned between windows/workers and also
stored/retrieved into/from an [IDBObjectStore](https://w3c.github.io/IndexedDB/#object-store).
The semantics of a structured clone is as-if the binary source, from which the
`WebAssembly.Module` was compiled, were cloned and recompiled into the target realm.
Engines should attempt to share/reuse internal compiled code when performing
a structured clone although, in corner cases like CPU upgrade or browser
update, this may not be possible and full recompilation may be necessary.

Given the above engine optimizations, structured cloning provides developers
explicit control over both compiled-code caching and cross-window/worker code
sharing.

## `WebAssembly.Instance` Objects

A `WebAssembly.Instance` object represents the instantiation of a 
`WebAssembly.Module` into a
[realm](http://tc39.github.io/ecma262/#sec-code-realms) and has one
internal slot:

* [[Instance]] : an [`Instance.instance`](https://github.com/WebAssembly/spec/blob/master/interpreter/spec/instance.ml#L17)
  which is the WebAssembly spec definition of an instance
 * [[Exports]] : the exports object created during instantiation

### `WebAssembly.Instance` Constructor

The `WebAssembly.Instance` constructor has the signature:

```
new Instance(moduleObject [, importObject])
```

If the NewTarget is `undefined`, a [`TypeError`](https://tc39.github.io/ecma262/#sec-native-error-types-used-in-this-standard-typeerror)
exception is thrown (i.e., this
constructor cannot be called as a function without `new`).

If `moduleObject` is not a `WebAssembly.Module`, a [`TypeError`](https://tc39.github.io/ecma262/#sec-native-error-types-used-in-this-standard-typeerror)
is thrown.

Let `module` be the [`Ast.module`](https://github.com/WebAssembly/spec/blob/master/interpreter/spec/ast.ml#L176)
`moduleObject.[[Module]]`.

If the `importObject` parameter is not `undefined` and `Type(importObject)` is
not Object, a [`TypeError`](https://tc39.github.io/ecma262/#sec-native-error-types-used-in-this-standard-typeerror)
is thrown. If the list of 
[`module.imports`](https://github.com/WebAssembly/spec/blob/master/interpreter/spec/ast.ml#L186)
is not empty and `Type(importObject)` is not Object, a [`TypeError`](https://tc39.github.io/ecma262/#sec-native-error-types-used-in-this-standard-typeerror)
is thrown.

Note: Imported JavaScript functions are wrapped as [host function](https://github.com/WebAssembly/spec/blob/master/interpreter/spec/instance.ml#L9) values in the following algorithm. For the purpose of the algorithm, a _new_ [host function](https://github.com/WebAssembly/spec/blob/master/interpreter/spec/instance.ml#L9) value is always generated fresh and considered distinct from any other previously created host function value, including those wrapping the same JavaScript function object.
Consequently, two [closure](https://github.com/WebAssembly/spec/blob/master/interpreter/spec/instance.ml#L7) values are considered equal if and only if:

* Either they are both WebAssembly functions for the same instance and referring to the same function definition.
* Or they are the same host function value.

Let `funcs`, `memories` and `tables` be initially-empty lists of callable JavaScript objects, `WebAssembly.Memory` objects and `WebAssembly.Table` objects, respectively.

Let `imports` be an initially-empty list of [`external`](https://github.com/WebAssembly/spec/blob/master/interpreter/spec/instance.ml#L11) values.

For each [`import`](https://github.com/WebAssembly/spec/blob/master/interpreter/spec/ast.ml#L168)
`i` in `module.imports`:

1. Let `o` be the resultant value of performing
   [`Get`](http://tc39.github.io/ecma262/#sec-get-o-p)(`importObject`, [`i.module_name`](https://github.com/WebAssembly/spec/blob/master/interpreter/spec/ast.ml#L170)).
1. If `Type(o)` is not Object, throw a [`TypeError`](https://tc39.github.io/ecma262/#sec-native-error-types-used-in-this-standard-typeerror).
1. Let `v` be the value of performing [`Get`](http://tc39.github.io/ecma262/#sec-get-o-p)(`o`, [`i.item_name`](https://github.com/WebAssembly/spec/blob/master/interpreter/spec/ast.ml#L171))
1. If `i` is a function import:
   1. If [`IsCallable(v)`](https://tc39.github.io/ecma262/#sec-iscallable) is `false`,
      throw a `WebAssembly.LinkError`.
   1. If `v` is an [Exported Function Exotic Object](#exported-function-exotic-objects):
      1. (The signature of `v.[[Closure]]` is checked against the import's declared
         [`func_type`](https://github.com/WebAssembly/design/blob/master/BinaryEncoding.md#func_type)
         by `Eval.init` below.)
      1. Let `closure` be `v.[[Closure]]`.
   1. Otherwise:
      1. Let `closure` be a new [host function](https://github.com/WebAssembly/spec/blob/master/interpreter/spec/instance.ml#L9) value
         of the given signature and the following behavior:
      1. If the signature contains an `i64` (as argument or result), the host
         function immediately throws a [`TypeError`](https://tc39.github.io/ecma262/#sec-native-error-types-used-in-this-standard-typeerror)
         when called.
      1. Otherwise, the host function calls `v` with an `undefined` receiver
         and WebAssembly arguments coerced to JavaScript arguments
         via [`ToJSValue`](#tojsvalue). The result is returned by coercing
         via [`ToWebAssemblyValue`](#towebassemblyvalue).
   1. Append `v` to `funcs`.
   1. Append `closure` to `imports`.
1. If `i` is a global import:
   1. [Assert](https://tc39.github.io/ecma262/#assert): the global is immutable
      by MVP validation constraint.
   1. If the `global_type` of `i` is `i64` or `Type(v)` is not Number, throw a `WebAssembly.LinkError`.
   1. Append [`ToWebAssemblyValue`](#towebassemblyvalue)`(v)` to `imports`.
1. If `i` is a memory import:
   1. If `v` is not a [`WebAssembly.Memory` object](#webassemblymemory-objects),
      throw a `WebAssembly.LinkError`.
   1. (The imported `Memory`'s `length` and `maximum` properties are checked against the import's declared
      [`memory_type`](https://github.com/WebAssembly/design/blob/master/BinaryEncoding.md#memory_type)
      by `Eval.init` below.)
   1. Append `v` to `memories`.
   1. Append `v.[[Memory]]` to `imports`.
1. Otherwise (`i` is a table import):
   1. If `v` is not a [`WebAssembly.Table` object](#webassemblytable-objects),
      throw a `WebAssembly.LinkError`.
   1. (The imported `Table`'s `length`, `maximum` and `element` properties are checked against the import's declared
      [`table_type`](https://github.com/WebAssembly/design/blob/master/BinaryEncoding.md#table_type)
      by `Eval.init` below.)
   1. Append `v` to `tables`.
   1. Append `v.[[Table]]` to `imports`.
   1. For each index `i` of `v.[[Table]]`:
      1. Let `e` be the `i`the element of `v.[[Table]]`.
   1. If `e` is a [`closure`](https://github.com/WebAssembly/spec/blob/master/interpreter/spec/instance.ml#L7) `c`:
      1. Append the `i`th element of `v.[[Values]]` to `funcs`.

Let `instance` be the result of creating a new
[`instance`](https://github.com/WebAssembly/spec/blob/master/interpreter/spec/instance.ml#L17)
by calling
[`Eval.init`](https://github.com/WebAssembly/spec/blob/master/interpreter/spec/eval.ml#L416)
given `module` and `imports`.
If this terminates with a `Link` error, throw a `WebAssembly.LinkError`; if it causes a trap, throw a `WebAssembly.RuntimeError`; all other exceptions are propagated to the caller.
Among other things, this function performs the following observable steps:

* If, after evaluating the `offset` [initializer expression](Modules.md#initializer-expression)
  of every [Data](Modules.md#data-section) and [Element](Modules.md#elements-section)
  Segment, any of the segments do not fit in their respective Memory or Table, throw a 
  `WebAssembly.LinkError`.

* Apply all Data and Element segments to their respective Memory or Table in the
  order in which they appear in the module. Segments may overlap and, if they do,
  the final value is the last value written in order. Note: there should be no
  errors possible that would cause this operation to fail partway through. After
  this operation completes, elements of `instance` are visible and callable
  through [imported tables](Modules.md#imports), even if `start` fails.

* If a [`start`](Modules.md#module-start-function) is present, it is evaluated.
  Any errors thrown by `start` are propagated to the caller.

The following steps are performed _before_ the `start` function executes:

1. For each table 't' in [`instance.tables`](https://github.com/WebAssembly/spec/blob/master/interpreter/spec/instance.ml#L17):
   1. If there is no element in `tables` whose `table.[[Table]]` is `t`:
      1. Let `table` be a new `WebAssembly.Table` object with [[Table]] set to `t` and [[Values]] set to a new list of the same length all whose entries are `null`.
      1. Append `table` to `tables`.
   1. Otherwise:
      1. Let `table` be the element in `tables` whose `table.[[Table]]` is `t`
   1. (Note: At most one `WebAssembly.Table` object is created for any table, so the above `table` is unique, even if there are multiple occurrances in the list. Moreover, if the item was an import, the original object will be found.)
   1. For each index `i` of `t`:
      1. Let `c` be the `i`th element of `t`
      1. If `c` is a [`closure`](https://github.com/WebAssembly/spec/blob/master/interpreter/spec/instance.ml#L7) `c`:
         1. If there is an [Exported Function Exotic Object](#exported-function-exotic-objects) in `funcs` whose `[[Closure]]` equals `c`:
            1. Let `func` be that function object.
         1. (Note: At most one wrapper is created for any closure, so `func` is uniquely determined. Moreover, if the item was an import that is already an [Exported Function Exotic Object](#exported-function-exotic-objects), then the original function object will be found. For imports that are regular JS functions, a new wrapper will be created.)
         1. Otherwise:
            1. Let `func` be an [Exported Function Exotic Object](#exported-function-exotic-objects) created from `c`.
            1. Append `func` to `funcs`.
         1. Set the `i`th element of `table.[[Values]]` to `func`.

(Note: The table and element function objects created by the above steps are only observable for tables that are either imported or exported.)

Let `exports` be a list of (string, JS value) pairs that is mapped from 
each [external](https://github.com/WebAssembly/spec/blob/master/interpreter/spec/instance.ml#L24) value `e` in `instance.exports` as follows:

1. If `e` is a [closure](https://github.com/WebAssembly/spec/blob/master/interpreter/spec/instance.ml#L12) `c`:
   1. If there is an [Exported Function Exotic Object](#exported-function-exotic-objects) `func` in `funcs` whose `func.[[Closure]]` equals `c`, then return `func`.
   1. (Note: At most one wrapper is created for any closure, so `func` is unique, even if there are multiple occurrances in the list. Moreover, if the item was an import that is already an [Exported Function Exotic Object](#exported-function-exotic-objects), then the original function object will be found. For imports that are regular JS functions, a new wrapper will be created.)
   1. Otherwise:
      1. Let `func` be an [Exported Function Exotic Object](#exported-function-exotic-objects) created from `c`.
      1. Append `func` to `funcs`.
      1. Return `func`.
1. If `e` is a [global](https://github.com/WebAssembly/spec/blob/master/interpreter/spec/instance.ml#L15) `v`:
   1. [Assert](https://tc39.github.io/ecma262/#assert): the global is immutable
      by MVP validation constraint.
   1. If `v` is an `i64`, throw a `WebAssembly.LinkError`.
   1. Return [`ToJSValue`](#tojsvalue)`(v)`.
1. If `e` is a [memory](https://github.com/WebAssembly/spec/blob/master/interpreter/spec/instance.ml#L14) `m`:
   1. If there is an element `memory` in `memories` whose `memory.[[Memory]]` is `m`, then return `memory`.
   1. (Note: At most one `WebAssembly.Memory` object is created for any memory, so the above `memory` is unique, even if there are multiple occurrances in the list. Moreover, if the item was an import, the original object will be found.)
   1. Otherwise:
      1. Let `memory` be a new `WebAssembly.Memory` object created via [`CreateMemoryObject`](#creatememoryobject) from `m`.
      1. Append `memory` to `memories`.
      1. Return `memory`.
1. Otherwise `e` must be a [table](https://github.com/WebAssembly/spec/blob/master/interpreter/spec/instance.ml#L13) `t`:
   1. Assert: There is an element `table` in `tables` whose `table.[[Table]]` is `t`.
   1. Return that `table`.

Let `exportsObject` be a new [frozen](https://tc39.github.io/ecma262/#sec-object.freeze)
plain JS object with [[Prototype]] set to Null and with properties defined
by mapping each export in `exports` to an enumerable, non-writable,
non-configurable data property. Note: the validity and uniqueness checks
performed during [module validation](#webassemblymodule-constructor) ensure
that each property name is valid and no properties are defined twice.

Let `instanceObject` be a new `WebAssembly.Instance` object setting
the internal `[[Instance]]` slot to `instance` and the `[[Exports]]` slot to
`exportsObject`.

Return `instanceObject`.

### `WebAssembly.Instance.prototype [ @@toStringTag ]` Property

The initial value of the [`@@toStringTag`](https://tc39.github.io/ecma262/#sec-well-known-symbols)
property is the String value `"WebAssembly.Instance"`.

This property has the attributes { [[Writable]]: `false`, [[Enumerable]]: `false`, [[Configurable]]: `true` }.

### `WebAssembly.Instance.prototype.exports` property

This is an accessor property whose [[Set]] is Undefined and whose [[Get]]
accessor function performs the following steps:

Let `T` be the `this` value. If `T` is not a `WebAssembly.Instance`, a [`TypeError`](https://tc39.github.io/ecma262/#sec-native-error-types-used-in-this-standard-typeerror)
is thrown.

Return `T.[[Exports]]`.

## Exported Function Exotic Objects

A function with [function index](Modules.md#function-index-space) `index`
from an `Instance` `inst` is reflected to JS via a new kind of *Exported
Function* [Exotic Object](http://tc39.github.io/ecma262/#sec-built-in-exotic-object-internal-methods-and-slots).
Like [Bound Function](http://tc39.github.io/ecma262/#sec-bound-function-exotic-objects) Exotic Object,
Exported Functions do not have the normal function internal slots but instead have:

 * [[Closure]] : the [closure](https://github.com/WebAssembly/spec/blob/master/interpreter/spec/instance.ml#L7)
   (`index`, `inst`)

as well as the internal slots required of all builtin functions:

 * [[Prototype]] : [%FunctionPrototype%](http://tc39.github.io/ecma262/#sec-well-known-intrinsic-objects)
 * [[Extensible]] : `true`
 * [[Realm]] : the [current Realm Record](http://tc39.github.io/ecma262/#current-realm)
 * [[ScriptOrModule]] : [`GetActiveScriptOrModule`](http://tc39.github.io/ecma262/#sec-getactivescriptormodule)

Exported Functions also have the following data properties:

* the `length` property is set to the exported function's signature's arity 
* the `name` is set to [`ToString`](https://tc39.github.io/ecma262/#sec-tostring)(`index`)

WebAssembly Exported Functions have a `[[Call]](this, argValues)` method defined as:

1. Let `sig` be the [`function type`](https://github.com/WebAssembly/spec/blob/master/interpreter/spec/eval.ml#L106)
   of the function's [[Closure]].
1. If `sig` contains an `i64` (as argument or result), a
   [`TypeError`](https://tc39.github.io/ecma262/#sec-native-error-types-used-in-this-standard-typeerror)
   is thrown each time the [[Call]] method is invoked.
1. Let `args` be an empty list of coerced values.
1. Let `inArity` be the number of arguments and `outArity` be the number of results in `sig`.
1. For all values `v` in `argValues`, in the order of their appearance:
  1. If the length of`args` is less than `inArity`, append [`ToWebAssemblyValue`](#towebassemblyvalue)`(v)` to `args`.
1. While the length of `args` is less than `inArity`, append [`ToWebAssemblyValue`](#towebassemblyvalue)`(undefined)` to `args`.
1. Let `ret` be the result of calling [`Eval.invoke`](https://github.com/WebAssembly/spec/blob/master/interpreter/spec/eval.ml#L443)
   passing [[Closure]], and `args`.
1. If `outArity` is 0, return `undefined`.
1. Otherwise, return [`ToJSValue`](#tojsvalue)`(v)`, where `v` is the singular element of `ret`.

`[[Call]](this, argValues)` executes in the [[Realm]] of the callee Exported Function. This corresponds to [the requirements of builtin function objects in JavaScript](https://tc39.github.io/ecma262/#sec-built-in-function-objects).

Exported Functions do not have a [[Construct]] method and thus it is not possible to 
call one with the `new` operator.

## `WebAssembly.Memory` Objects

A `WebAssembly.Memory` object contains a single [linear memory](Semantics.md#linear-memory)
which can be simultaneously referenced by multiple `Instance` objects. Each
`Memory` object has two internal slots:

 * [[Memory]] : a [`Memory.memory`](https://github.com/WebAssembly/spec/blob/master/interpreter/spec/memory.mli)
 * [[BufferObject]] : the current `ArrayBuffer` whose [[ArrayBufferByteLength]]
   matches the current byte length of [[Memory]]

### `WebAssembly.Memory` Constructor

The `WebAssembly.Memory` constructor has the signature:

```
new Memory(memoryDescriptor)
```

If the NewTarget is `undefined`, a [`TypeError`](https://tc39.github.io/ecma262/#sec-native-error-types-used-in-this-standard-typeerror)
exception is thrown (i.e., this constructor cannot be called as a function without `new`).

If `Type(memoryDescriptor)` is not Object, a [`TypeError`](https://tc39.github.io/ecma262/#sec-native-error-types-used-in-this-standard-typeerror)
is thrown.

Let `initial` be [`ToNonWrappingUint32`](#tononwrappinguint32)([`Get`](http://tc39.github.io/ecma262/#sec-get-o-p)(`memoryDescriptor`, `"initial"`)).

If [`HasProperty`](http://tc39.github.io/ecma262/#sec-hasproperty)(`"maximum"`),
then let `maximum` be [`ToNonWrappingUint32`](#tononwrappinguint32)([`Get`](http://tc39.github.io/ecma262/#sec-get-o-p)(`memoryDescriptor`, `"maximum"`)).
If `maximum` is smaller than `initial`, then throw a [`RangeError`](https://tc39.github.io/ecma262/#sec-native-error-types-used-in-this-standard-rangeerror).
Otherwise, let `maximum` be `None`.

Let `memory` be the result of calling 
[`Memory.create`](https://github.com/WebAssembly/spec/blob/master/interpreter/spec/memory.ml#L68)
given arguments `initial` and `maximum`. Note that `initial` and `maximum` are
specified in units of WebAssembly pages (64KiB).

Return the result of [`CreateMemoryObject`](#creatememoryobject)(`memory`).

### CreateMemoryObject

Given a [`Memory.memory`](https://github.com/WebAssembly/spec/blob/master/interpreter/spec/memory.mli#L1)
`m`, to create a `WebAssembly.Memory`:

Let `buffer` be a new `ArrayBuffer` whose
[[[ArrayBufferData]]](http://tc39.github.io/ecma262/#sec-properties-of-the-arraybuffer-prototype-object)
aliases `m` and whose 
[[[ArrayBufferByteLength]]](http://tc39.github.io/ecma262/#sec-properties-of-the-arraybuffer-prototype-object)
is set to the byte length of `m`.

Any attempts to [`detach`](http://tc39.github.io/ecma262/#sec-detacharraybuffer) `buffer` *other* than
the detachment performed by [`m.grow`](#webassemblymemoryprototypegrow) shall throw a 
[`TypeError`](https://tc39.github.io/ecma262/#sec-native-error-types-used-in-this-standard-typeerror)

Return a new `WebAssembly.Memory` instance with [[Memory]] set to `m` and
[[BufferObject]] set to `buffer`.

### `WebAssembly.Memory.prototype [ @@toStringTag ]` Property

The initial value of the [`@@toStringTag`](https://tc39.github.io/ecma262/#sec-well-known-symbols)
property is the String value `"WebAssembly.Memory"`.

This property has the attributes { [[Writable]]: `false`, [[Enumerable]]: `false`, [[Configurable]]: `true` }.

### `WebAssembly.Memory.prototype.grow`

The `grow` method has the signature:

```
grow(delta)
```

Let `M` be the `this` value. If `M` is not a `WebAssembly.Memory`,
a [`TypeError`](https://tc39.github.io/ecma262/#sec-native-error-types-used-in-this-standard-typeerror)
is thrown.

Let `d` be [`ToNonWrappingUint32`](#tononwrappinguint32)(`delta`).

Let `ret` be the current size of memory in pages (before resizing).

Perform [`Memory.grow`](https://github.com/WebAssembly/spec/blob/master/interpreter/spec/memory.mli#L27)
with delta `d`. On failure, a 
[`RangeError`](https://tc39.github.io/ecma262/#sec-native-error-types-used-in-this-standard-rangeerror)
is thrown.

Perform [`DetachArrayBuffer`](http://tc39.github.io/ecma262/#sec-detacharraybuffer)(`M.[[BufferObject]]`).

Assign to `M.[[BufferObject]]` a new `ArrayBuffer` whose
[[[ArrayBufferData]]](http://tc39.github.io/ecma262/#sec-properties-of-the-arraybuffer-prototype-object)
aliases `M.[[Memory]]` and whose 
[[[ArrayBufferByteLength]]](http://tc39.github.io/ecma262/#sec-properties-of-the-arraybuffer-prototype-object)
is set to the new byte length of `M.[[Memory]]`.

Return `ret` as a Number value.

### `WebAssembly.Memory.prototype.buffer`

This is an accessor property whose [[Set]] is Undefined and whose [[Get]]
accessor function performs the following steps:

If `this` is not a `WebAssembly.Memory`, a [`TypeError`](https://tc39.github.io/ecma262/#sec-native-error-types-used-in-this-standard-typeerror)
is thrown. Otherwise return `M.[[BufferObject]]`.

## `WebAssembly.Table` Objects

A `WebAssembly.Table` object contains a single [table](Semantics.md#table)
which can be simultaneously referenced by multiple `Instance` objects. Each
`Table` object has two internal slots:

 * [[Table]] : a [`Table.table`](https://github.com/WebAssembly/spec/blob/master/interpreter/spec/table.mli#L1)
 * [[Values]] : an array whose elements are either `null` or [Exported Function Exotic Object](#exported-function-exotic-objects)

### `WebAssembly.Table` Constructor

The `WebAssembly.Table` constructor has the signature:

```
new Table(tableDescriptor)
```

If the NewTarget is `undefined`, a [`TypeError`](https://tc39.github.io/ecma262/#sec-native-error-types-used-in-this-standard-typeerror)
exception is thrown (i.e., this constructor cannot be called as a function without `new`).

If `Type(tableDescriptor)` is not Object, a [`TypeError`](https://tc39.github.io/ecma262/#sec-native-error-types-used-in-this-standard-typeerror)
is thrown.

Let `element` be the result of calling [`Get`](http://tc39.github.io/ecma262/#sec-get-o-p)(`tableDescriptor`, `"element"`).
If `element` is not the string `"anyfunc"`, a [`TypeError`](https://tc39.github.io/ecma262/#sec-native-error-types-used-in-this-standard-typeerror)
is thrown.
(Note: this check is intended to be relaxed in the
[future :unicorn:][future types] to allow different element types.)

Let `initial` be [`ToNonWrappingUint32`](#tononwrappinguint32)([`Get`](http://tc39.github.io/ecma262/#sec-get-o-p)(`tableDescriptor`, `"initial"`)).

If [`HasProperty`](http://tc39.github.io/ecma262/#sec-hasproperty)(`"maximum"`),
then let `maximum` be [`ToNonWrappingUint32`](#tononwrappinguint32)([`Get`](http://tc39.github.io/ecma262/#sec-get-o-p)(`tableDescriptor`, `"maximum"`)). Otherwise, let `maximum` be None.

If `maximum` is not None and is smaller than `initial`, then throw a [`RangeError`](https://tc39.github.io/ecma262/#sec-native-error-types-used-in-this-standard-rangeerror).

Let `table` be the result of calling 
[`Table.create`](https://github.com/WebAssembly/spec/blob/master/interpreter/spec/table.ml#L68)
given arguments `AnyFuncType`, `initial` and `maximum`.

Let `values` be a new empty array of `initial` elements, all with value
`null`.

Return a new `WebAssemby.Table` instance with [[Table]] set to `table` and
[[Values]] set to `values`.

### `WebAssembly.Table.prototype.length`

This is an accessor property whose [[Set]] is Undefined and whose [[Get]]
accessor function performs the following steps:

Let `T` be the `this` value. If `T` is not a `WebAssembly.Table`, a [`TypeError`](https://tc39.github.io/ecma262/#sec-native-error-types-used-in-this-standard-typeerror)
is thrown.

Return `T.[[Values]].length`.

### `WebAssembly.Table.prototype.grow`

The `grow` method has the signature:

```
grow(delta)
```

Let `T` be the `this` value. If `T` is not a `WebAssembly.Table`,
a [`TypeError`](https://tc39.github.io/ecma262/#sec-native-error-types-used-in-this-standard-typeerror)
is thrown.

Let `d` be [`ToNonWrappingUint32`](#tononwrappinguint32)(`delta`).

Let `ret` be the current length of the table (before resizing).

Perform [`Table.grow`](https://github.com/WebAssembly/spec/blob/master/interpreter/spec/table.ml#L40),
with delta `d`. On failure, a
[`RangeError`](https://tc39.github.io/ecma262/#sec-native-error-types-used-in-this-standard-rangeerror)
is thrown.

Return `ret` as a Number value.

### `WebAssembly.Table.prototype.get`

This method has the following signature

```
get(index)
```

Let `T` be the `this` value. If `T` is not a `WebAssembly.Table`, a [`TypeError`](https://tc39.github.io/ecma262/#sec-native-error-types-used-in-this-standard-typeerror)
is thrown.

Let `i` be the result of [`ToNonWrappingUint32`](#tononwrappinguint32)(`index`).

If `i` is greater or equal than the length of `T.[[Values]]`, a [`RangeError`](https://tc39.github.io/ecma262/#sec-native-error-types-used-in-this-standard-rangeerror) is thrown.

Return `T.[[Values]][i]`.

### `WebAssembly.Table.prototype.set`

This method has the following signature

```
set(index, value)
```

Let `T` be the `this` value. If `T` is not a `WebAssembly.Table`, a [`TypeError`](https://tc39.github.io/ecma262/#sec-native-error-types-used-in-this-standard-typeerror)
is thrown.

If `value` is not an [Exported Function Exotic Object](#exported-function-exotic-objects)
or `null`, throw a [`TypeError`](https://tc39.github.io/ecma262/#sec-native-error-types-used-in-this-standard-typeerror).

Let `i` be the result of [`ToNonWrappingUint32`](#tononwrappinguint32)(`index`).

If `i` is greater or equal than the length of `T.[[Values]]`, a [`RangeError`](https://tc39.github.io/ecma262/#sec-native-error-types-used-in-this-standard-rangeerror) is thrown.

If `value` is `null`, let `elem` be `Uninitialized`;
otherwise, let `elem` be `value.[[Closure]]`.

Set `T.[[Table]][i]` to `elem`.

Set `T.[[Values]][i]` to `value`.

Return `undefined`.

### `WebAssembly.Table.prototype [ @@toStringTag ]` Property

The initial value of the [`@@toStringTag`](https://tc39.github.io/ecma262/#sec-well-known-symbols)
property is the String value `"WebAssembly.Table"`.

This property has the attributes { [[Writable]]: `false`, [[Enumerable]]: `false`, [[Configurable]]: `true` }.

## ToJSValue

To coerce a WebAssembly [`value`](https://github.com/WebAssembly/spec/blob/master/interpreter/spec/values.ml#L9)
to a JavaScript value:

Assert: the WebAssembly value's type is not `i64`.

1. given a WebAssembly `i32` is interpreted as a signed integer, converted (losslessly) to an
  IEEE754 double and then returned as a JavaScript Number
1. given a WebAssembly `f32` (single-precision IEEE754), convert (losslessly) to
   a IEEE754 double, [possibly canonicalize NaN](http://tc39.github.io/ecma262/#sec-setvalueinbuffer),
   and return as a JavaScript Number
1. given a WebAssembly `f64`, [possibly canonicalize NaN](http://tc39.github.io/ecma262/#sec-setvalueinbuffer)
   and return as a JavaScript Number

If the WebAssembly value is optional, then given `None`, return JavaScript value
`undefined`.

## ToWebAssemblyValue

To coerce a JavaScript value to a given WebAssembly [`value type`](https://github.com/WebAssembly/spec/blob/master/interpreter/spec/types.ml#L3),

Assert: the target value type is not `i64`.

1. coerce to `i32` via [`ToInt32(v)`](http://tc39.github.io/ecma262/#sec-toint32)
1. coerce to `f32` by first applying [`ToNumber(v)`](http://tc39.github.io/ecma262/#sec-tonumber)
   and then converting the resulting IEEE754 64-bit double to a 32-bit float using `roundTiesToEven`
1. coerce to `f64` via [`ToNumber(v)`](http://tc39.github.io/ecma262/#sec-tonumber)

If the value type is optional, then given `None`, the JavaScript value is
ignored.

## ToNonWrappingUint32

To convert a JavaScript value `v` to an unsigned integer in the range [0, `UINT32_MAX`]:

Let `i` be [`ToInteger`](http://tc39.github.io/ecma262/#sec-tointeger)(`v`).

If `i` is negative or greater than `UINT32_MAX`, 
[`RangeError`](https://tc39.github.io/ecma262/#sec-native-error-types-used-in-this-standard-rangeerror)
is thrown.

Return `i`.

## Sample API Usage

Given `demo.was` (encoded to `demo.wasm`):

```lisp
(module
    (import "js" "import1" (func $i1))
    (import "js" "import2" (func $i2))
    (func $main (call $i1))
    (start $main)
    (func (export "f") (call $i2))
)
```

and the following JavaScript, run in a browser:

```javascript
var importObj = {js: {
    import1: () => console.log("hello,"),
    import2: () => console.log("world!")
}};
fetch('demo.wasm').then(response =>
    response.arrayBuffer()
).then(buffer =>
    WebAssembly.instantiate(buffer, importObj)
).then(({module, instance}) =>
    instance.exports.f()
);
```

[future general]: FutureFeatures.md
[future streaming]: FutureFeatures.md#streaming-compilation
[future types]: FutureFeatures.md#more-table-operators-and-types
