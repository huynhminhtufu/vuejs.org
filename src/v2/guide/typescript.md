---
title: Hỗ trợ TypeScript
type: guide
order: 404
---

> Trong phiên bản Vue 2.5.0+ chúng tôi đã cải thiện khá nhiều các phần khai báo kiểu để nó hoạt động tốt với object-based API mặc định. Đồng thời có một vài thay đổi nhỏ đòi hỏi phải thực hiện để nâng cấp. Có thể đọc [bài blog này](https://medium.com/the-vue-point/upcoming-typescript-changes-in-vue-2-5-e9bd7e2ecf08) để xem cụ thể hơn.

## Official Declaration trong NPM Packages

Một hệ thống static type (kiểu tĩnh) có thể giúp ngăn chặn khá nhiều lỗi lúc runtime (lúc chạy), đặc biệt là đối với các ứng dụng lớn. Đó là lí do vì sao Vue đã được bom gồm sẵn với [official type declarations](https://github.com/vuejs/vue/tree/dev/types) cho [TypeScript](https://www.typescriptlang.org/) - không những Vue core, mà [vue-router](https://github.com/vuejs/vue-router/tree/dev/types) và [vuex](https://github.com/vuejs/vuex/tree/dev/types) cũng được tích hợp.

Vì những gói này đã được [phát hành trên NPM](https://cdn.jsdelivr.net/npm/vue/types/), phiên bản TypeScript mới nhất sẽ có thể được các resolve type declarations (giải mã khai báo kiểu) trong các package của NPM, điều này có nghĩa rằng khi bạn cài Vue bằng NPM, bạn không cần cài thêm tool nào để có thể sử dụng TypeScript với Vue.

Chúng tôi cũng đã lên kế hoạch để cung cấp một tùy chọn giúp bạn tạo nhanh sẵn một dự án với Vue + TypeScript trong `vue-cli` trong tương lai gần.

## Cấu hình đề xuất

``` js
// tsconfig.json
{
  "compilerOptions": {
    // để đồng bộ với phần hỗ trợ trình duyệt của Vue
    "target": "es5",
    // điều này hỗ trợ kiểm soát chặt chẽ hơn với các thuộc tính dữ liệu trên `this`
    "strict": true,
    // nếu xài webpack 2+ hoặc rollup
    "module": "es2015",
    "moduleResolution": "node"
  }
}
```

Note that you have to include `strict: true` (or at least `noImplicitThis: true` which is a part of `strict` flag) to leverage type checking of `this` in component methods otherwise it is always treated as `any` type.

See [TypeScript compiler options docs](https://www.typescriptlang.org/docs/handbook/compiler-options.html) for more details.

## Development Tooling

For developing Vue applications with TypeScript, we strongly recommend using [Visual Studio Code](https://code.visualstudio.com/), which provides great out-of-the-box support for TypeScript.

If you are using [single-file components](./single-file-components.html) (SFCs), get the awesome [Vetur extension](https://github.com/vuejs/vetur), which provides TypeScript inference inside SFCs and many other great features.

[WebStorm](https://www.jetbrains.com/webstorm/) also provides out-of-the-box support for both TypeScript and Vue.js.

## Basic Usage

To let TypeScript properly infer types inside Vue component options, you need to define components with `Vue.component` or `Vue.extend`:

``` ts
import Vue from 'vue'

const Component = Vue.extend({
  // type inference enabled
})

const Component = {
  // this will NOT have type inference,
  // because TypeScript can't tell this is options for a Vue component.
}
```

## Class-Style Vue Components

If you prefer a class-based API when declaring components, you can use the officially maintained [vue-class-component](https://github.com/vuejs/vue-class-component) decorator:

``` ts
import Vue from 'vue'
import Component from 'vue-class-component'

// The @Component decorator indicates the class is a Vue component
@Component({
  // All component options are allowed in here
  template: '<button @click="onClick">Click!</button>'
})
export default class MyComponent extends Vue {
  // Initial data can be declared as instance properties
  message: string = 'Hello!'

  // Component methods can be declared as instance methods
  onClick (): void {
    window.alert(this.message)
  }
}
```

## Augmenting Types for Use with Plugins

Plugins may add to Vue's global/instance properties and component options. In these cases, type declarations are needed to make plugins compile in TypeScript. Fortunately, there's a TypeScript feature to augment existing types called [module augmentation](https://www.typescriptlang.org/docs/handbook/declaration-merging.html#module-augmentation).

For example, to declare an instance property `$myProperty` with type `string`:

``` ts
// 1. Make sure to import 'vue' before declaring augmented types
import Vue from 'vue'

// 2. Specify a file with the types you want to augment
//    Vue has the constructor type in types/vue.d.ts
declare module 'vue/types/vue' {
  // 3. Declare augmentation for Vue
  interface Vue {
    $myProperty: string
  }
}
```

After including the above code as a declaration file (like `my-property.d.ts`) in your project, you can use `$myProperty` on a Vue instance.

```ts
var vm = new Vue()
console.log(vm.$myProperty) // This should compile successfully
```

You can also declare additional global properties and component options:

```ts
import Vue from 'vue'

declare module 'vue/types/vue' {
  // Global properties can be declared
  // on the `VueConstructor` interface
  interface VueConstructor {
    $myGlobal: string
  }
}

// ComponentOptions is declared in types/options.d.ts
declare module 'vue/types/options' {
  interface ComponentOptions<V extends Vue> {
    myOption?: string
  }
}
```

The above declarations allow the following code to be compiled:

```ts
// Global property
console.log(Vue.$myGlobal)

// Additional component option
var vm = new Vue({
  myOption: 'Hello'
})
```

## Annotating Return Types

Because of the circular nature of Vue's declaration files, TypeScript may have difficulties inferring the types of certain methods. For this reason, you may need to annotate the return type on methods like `render` and those in `computed`.

```ts
import Vue, { VNode } from 'vue'

const Component = Vue.extend({
  data () {
    return {
      msg: 'Hello'
    }
  },
  methods: {
    // need annotation due to `this` in return type
    greet (): string {
      return this.msg + ' world'
    }
  },
  computed: {
    // need annotation
    greeting(): string {
      return this.greet() + '!'
    }
  },
  // `createElement` is inferred, but `render` needs return type
  render (createElement): VNode {
    return createElement('div', this.greeting)
  }
})
```

If you find type inference or member completion isn't working, annotating certain methods may help address these problems. Using the `--noImplicitAny` option will help find many of these unannotated methods.
