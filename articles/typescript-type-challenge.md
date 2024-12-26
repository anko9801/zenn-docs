---
title: "Type challenge に必要な知識"
emoji: "📌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["TypeScript"]
published: false
---

## もし

if 文 for 文
```js
function test(env: any[]) => {
  if (env === 'production') {
  }
  for ()
}
```

```js
type test<T extends any[]> = 
```
と対応する

Essence
- ジェネリクスに extends をつけると、型に制約を設けることができる
- keyof T は T のプロパティ名のユニオン型を返す

Essence
- infer は、「ここに入る型って何？」ってのを推論してくれる便利な機能

## JavaScript のデータ型
| データ型 | | |
|---|---|---|
| null | |
| undefined | |
| boolean | |
| number | |
| bigint | |
| string | |
| symbol | |
| オブジェクト | key は string | symbol |
| 配列 | key は number |
| キー付きコレクション | Map, Set, WeakMap, WeakSet |

```ts
type Primitive = number | string | boolean | bigint | symbol | undefined | null;
type Builtin = Function | Date | Error | RegExp;
interface User {
    name: string;
    age: number;
    private: boolean;
}
type User = {
    name: string;
    age: number;
    private: boolean;
};
type IsPositiveFunc = (arg: number) => boolean;
interface IsPositiveFunc {
  (arg: number): boolean;
}
```



オブジェクト型は key, value で構成され、配列は key が number なオブジェクト型といえる。
hashable な型 `string | number | symbol` しか key にならない

```ts
T[]
[]
[...T[]]
```

Keyof Type Operator
Indexed Access Types は次の性質を持ちます。
`...` はタプル内で1回しか使えない

```ts
K1 | ... | Kn == keyof T
P == keyof P // primitive type P
T[A] | T[B] == T[A | B]
T[0] | ... | T[n] == T[number]
```

### Conditional Types
継承関係となる

```ts
(A | B) | C == A | (B | C)
```

```ts
T extends U ? X : Y
```

```ts
A extends A | B
```

### Mapped Types

`NewKeyType` では `never` とすることでそのカラムを消すことが出来ます。逆に増やす方法は存在しません。

```ts
type MappedTypeWithNewProperties<Type> = {
  [Property in keyof Type as NewKeyType]: Type[Properties];
};
{ [P in T]: K<P> };
interface SquareConfig {
  color?: string;
  width?: number;
  [propName: string]: any;
}
type DeepNonNullable<T> = T extends Builtin
  ? NonNullable<T>
  : { [key in keyof T]-?: DeepNonNullable<T[key]> };
```

### Template Literal Types

```ts
infer
```


### 再帰構造
再帰回数の上限
- 終了判定条件をうまく設定することが重要

次のように再帰構造は書くことができません。これは不可能となる条件が難しいので実装されてないようです。

```ts
type A = T<A>
type A = A[] | T

// NG
type Data = number | string | Data[] | Record<string, Data>;
// OK
type Data = number | string | { [key: number]: Data } | { [key: string]: Data };
```


```ts
type Getters<Type> = {
    [Property in keyof Type as `get${Capitalize<string & Property>}`]: () => Type[Property]
};
type First<T extends any[]> = T extends [] ? never : T[0];
type Partial<T> = { [P in keyof T]?: T[P] | undefined; }
type Required<T> = { [P in keyof T]-?: T[P]; }
type Readonly<T> = { readonly [K in keyof T]: T[K] };
type Readonly2<T, K extends keyof T = keyof T> = { readonly [k in K]: T[k] } & Omit<T, K>;
type DeepReadonly<T> = {
  readonly [K in keyof T]: T[K] extends Record<string, unknown> | Array<unknown>
    ? DeepReadonly<T[K]>
    : T[K];
};
type Record<K extends string | number | symbol, T> = { [P in K]: T; }
type Extract<T, U> = T extends U ? T : never
type Exclude<T, U> = T extends U ? never : T
type NonNullable<T> = T & {}
type Pick<T, K extends keyof T> = { [key in K]: T[key] };
type Omit<T, K extends keyof T> = { [key in keyof T as key extends K ? never : key]: T[key]; };
type TupleToUnion<T extends any[]> = T[number];
type TupleToObject<T extends readonly any[]> = { [K in T[number]]: K };
type Parameters<T extends (...args: any) => any> = T extends (...args: infer P) => any ? P : never
type ReturnType<T extends (...args: any) => any> = T extends (...args: any) => infer R ? R : never
type InstanceType<T extends abstract new (...args: any) => any> = T extends abstract new (...args: any) => infer R ? R : any
type ConstructorParameters<T extends abstract new (...args: any) => any> = T extends abstract new (...args: infer P) => any ? P : never
type ThisParameterType<T> = T extends (this: infer U, ...args: never) => any ? U : unknown
type OmitThisParameter<T> = unknown extends ThisParameterType<T> ? T : T extends (...args: infer A) => infer R ? (...args: A) => R : T
```
