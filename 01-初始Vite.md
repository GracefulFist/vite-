# 1-初始 Vite 

## 1.1 概念：

Vite 是一个构建工具。

前置知识，什么是构建工具？

## 1.2 什么叫做构建工具？

**浏览器只认识：HTML、CSS、JS 文件。**

而在实际开发中，我们会引入各种的语法工具：

1. 为了项目的语法规范：会使用到 TS，我们可以利用 tsc 将 .ts 文件转化为 .js 文件
2. 遇到 React/Vue ,我们要使用 react-compiler / vue-compiler ，将我们写的 .jsx / .vue 文件转化为 render 函数
3. 使用 less / sass / post-css /component-style , 我们就需要去安装 less-loader / sass-loader 等一系列的编译工具。
4. 语法降级，我们要使用 babel ，将 es 的新语法转化为浏览器认识的语法
5. 体积更小，我们会使用 uglifyjs，将代码转化为体积更小性能更高的代码
6. ........

这里陈列引入的各种语法工具，想要达到其所期待的效果，都必须要借助各种的 loader 来实现展示出相应的功能。

前提：没有构建工具的话。在代码开发的实际过程中，当我们修改其中的任意一份文件，想要看到结果，我们必须手动的执行各种 loader 的指令，而且哪怕你修改一个变量名，你都需要**一步不错的**重新的执行一遍所有的命令。问题就来了：**稍微修改一点点代码，就会变得非常麻烦。**

而且，每条命令都不能出错，举个列子：

> .tsx 文件 ---> tsc ---> .jsx 文件 ---> react-complier ---> .js 文件

每一步所执行的命令都不能出错，不能出现要使用，比如 tsc 将  .tsx 文件 转化为 .jsx 文件，使用 tsc 命令，但却使用了 react 命令，那么会直接报错，你必须从头再来，你会感到非常的麻烦。

需求来了：你希不希望这时候有个工具将所有你需要的工具都集成在一起，而你只需要关注代码的修改/编写，只要文件一变化，这个东西就会自动的帮我们执行所有的流程。

这个东西就叫做**构建工具**。

> 打包：将我们写的浏览器不认识的代码，交给构建工具进行编译处理的过程叫做打包。打包完成以后会给我们一个浏览器认识的文件。

## 1.3 一个构建工具到底承担了哪些脏活累活？

1. 模块化开发支持：直接从 node_modules 中导入文件 + 多种模块化支持（import 和 require）
2. 处理代码兼容性：比如 babel 降级，less / sass 、ts 等语法转化（语法转化不是构建工具所做的，而构建工具将这些语法对应的工具集集成起来做一个自动化的处理）
3. 提高项目性能：压缩文件，代码分割
4. 优化开发体验：
   1. 构建工具会监听修改的文件，当文件发生变化的时候，构建工具会调用相应的集成工具进行重新打包，重新在浏览器中运行。（整个过程叫做 热更新，又叫做 hot replacement）
   2. 开发服务器：跨域问题。

## 1.4 总结：

​	构建工具他让我们不用每次都关心代码如何在浏览器中如何运行，我们只需要首次给构建工具提供一个配置文件当然，这个配置文件并不是必须的，如果你不给他，他会有默认的配置文件提供）当我们有了这个集成的配置文件之后，我们就可以在下次更新的时候，调用一次对应的命令就好了。如果我们再结合热更新，我们就更不需要管任何东西了，这就是构建工具所做的事。

​	构建工具**让我们不用在关心生产的代码和代码如何在浏览器当中运行，我们只需要关心我们怎么开发的爽就怎么写就好了。**

了解知识点：

市面上有哪些构建工具？

1. webpack
2. vite 
3. parcel
4. esbuild
5. rollup
6. grunt
7. gulp

# 2.Vite 相对于 webpack 的优势

## 2.1介绍

Vite 的官方文档：https://www.vitejs.net/guide/why.html#the-problems

开发遇到的痛点：

> 然而，当我们开始构建越来越大型的应用时，需要处理的 JavaScript 代码量也呈指数级增长。包含数千个模块的大型项目相当普遍。我们开始遇到性能瓶颈 —— 使用 JavaScript 开发的工具通常需要很长时间（甚至是几分钟！）才能启动开发服务器，即使使用 HMR（热更新），文件修改后的效果也需要几秒钟才能在浏览器中反映出来。如此循环往复，迟钝的反馈会极大地影响开发者的开发效率和幸福感。

起因：我们的项目越大，构建工具（webpack）所要处理的 js 代码就越多。（这个是跟 webpack 的构建过程（工作流程）有关）

造成的结果：构建工具需要很长时间才能启动开发服务器。（启动开发服务器 ---> 把项目跑起来）

```bash
# 启动开发服务器
npm run dev
npm run serve
npm start 

yarn start
yarn dev
```

问题：

​	启动开发服务器的时间，随着 js 代码的增多，相应的打包时间越长。

webpack 对此能不能有足够的改进？答案是：不能。这会涉及到 webpack 的大动脉，没法去改变。

## 2.2 为什么 webpack 很难改变这个问题（启动开发服务器）？

原因：我们都知道 webpack 支持多种模块化开发：你的工程可能不只会跑在浏览器端。

```js
// 这一段代码是要放到浏览器端运行的
const lodash = require('lodash') // commonjs 语法规范
import Vue form 'vue' // es6 modules 语法规范
```

上述两段模块化的代码，是可以在 webpack 作为构建工具的项目中存在同时存在的。webpack 是允许我们这样书写的。

上述模块化代码，经过 webpack 转化后呈现出这样的结果

```js
// 他会转化为 webpack 自己封装的一个函数，也就是将这两种模块化代码做了一个统一的操作
const lodash = webpack_require('lodash')
const Vue = webpack_require('Vue')
```

对于，这块转化的过程现在不需要进行了解。但是，你要了解的是 webpack 这个转化的过程，是在开发服务器中进行的，在浏览器端是不能进行  js 修改文件的，在服务器端可以去**修改存在模块化开发的代码段的**。

也就是，webpack 想要完成模块化的转换，他就必须要去读取每一个文件，来寻找并进行替换。

总结：

​	因为 webpack 支持多种模块化开发，他就必须一开始就得统一模块化代码，所以，他就需要将所有的依赖全部读一遍。

延伸：

​	Vite 会不会将 webpack 干掉或取消？

答案是不会。它们的侧重点不一样。因为 webpack 考虑更多的是兼容性，Vite 考虑的是开发体验。Vite 则是基于 es module，这就要求开发者必须使用 es module 语法，Vite 也就不需要去读取每一个文件，按需导入即可，也就不用花费太多的时间启动开发服务器了。

# 3.Vite 脚手架和 Vite 的区别

Vite 官方搭建第一个 Vite 脚手架：https://www.vitejs.net/guide/#trying-vite-online

这两者之间的关系是：Vite 脚手架当中内置了 Vite。

很多人容易存在的误区：认为官网中 yarn create 构建项目的过程也是 Vite 在做的事。这其实是 Vite 脚手架帮我们做的。

当我们敲击了命令 `yarn create vite` 后，

1. 首先帮我们全局安装一个东西：`create-vite(vite 的脚手架)` 
2. 直接运行 `create-vite` bin 目录下的一个执行配置文件。

官网里面是直接叫我们利用脚手架搭建工程，而不叫我们使用 vite 。

# 4.Vite 项目初体验

vite : 开箱即用（out of box），即你不用做任何的配置就可以使用 Vite 来帮你进行构建工作。

在默认的情况下，我们的 es module 导入资源只能通过相对路径或者是绝对路径，才能找到相应的资源。

问题：

​	我们现在的最佳实践就是node_modules，那么 es 官方在我们导入绝对路径或者相对路径资源的时候，不默认帮我们去找node_modules 中的资源文件？

因为 es module 语法导入，是通过 HTTP 协议导入的资源文件，如果根据 node_modules 找到了 lodash，那 lodash 里边又引用了其它资源文件，那么就必须又要导入其它资源，而其它资源又引入了其它资源......那么这个对网络资源的消耗是极为恐怖的，特别损耗性能。所以，es 官方不敢支持node_modules寻找。而 commonjs 则是应用于服务端的，而服务端导入文件是本地文件读取的，就不需要考虑网络性能的损耗。

当 Vite 遇到 `import _ from 'lodash'` 这句语法之后他会做些什么呢？这个在下章进行讲解。

# 5.Vite 的依赖预构建

Vite 的预加载

在上一章中，我们接触到了 Vite 可以支持 node_modules 形式的模块化处理，那么我们就会产生好奇：当 Vite 遇到 `import _ from 'lodash'` 这句话时做了一个什么处理，可以找到这个文件。

当 Vite 在构建项目的过程中，如果遇到非绝对路径或者非相对路径，就会尝试开启路径补全。

````js
import _ from "lodash"
````

开启路径补全后：

```js
import __vite__cjsImport0_lodash from "/node_modules/.vite/deps/lodash.js?v=69c62b60"
```

我们现在只需要关心后边的路径问题。

下面是对上述路径问题的推理过程：

寻找依赖的过程：

目前，我们知道找寻依赖的过程是从当前目录依次向上查找，直到搜寻到根目录（指的是用户目录/user）下或者搜寻到对应依赖为止。

照上述推理，如果打包后的路径是绝对路径，当存在极端情况 node_modules 存在于 /user/node_modules/xxx ,那么浏览器不能读取这个文件（这一点我也不知道为什么，这个之后再说）；而如果打包后的文件是通过相对路径进行引入的， 这个是可以，但会比较麻烦（并没有采用），这个主要是考虑到开发环境和生产环境。在开发环境中，使用相对路径没有任何问题,Vite 会帮我们进行处理的（包括依赖预构建）。但是，一旦到了生产环境，Vite 会将所有的一切交给 rollUp打包工具进行全权处理，rollup将可以理解为 webpack，它会兼容多种场景。在生产环境中的处理过程中，出现了一种比相对路径更好的方式来解决路径问题。

实际上 vite 在考虑另外的一个问题的时候，顺便就解决了这个问题。

问题：

​	当我们使用第三方库的时候，没办法去约束每个开发者导出的方式，有可能使用的是 commonjs,也有可能是 es modules 方式，那么在构建的过程中，你也需要做一个将各种导出的方式转化为 es modules 规范。

解决方案：**依赖预构建**。

什么叫做依赖预构建？

​	当 vite 寻找到对应的依赖后，然后调用 esbuild (是用Go语言对JS进行处理的一个库)，将其它规范全部转化为 es modules 规范，然后将转化后的文件放在当前项目目录下的 /node_modules/.vite/deps/xxx。同时对 esmodule 规范的各个模块进行统一集成。

​	依赖预构建的过程是发生在开发环境中的。



> 网络多包：是指当我使用 es modules 利用HTPP导入一个模块的时候，而这个模块又恰好包含其它多个模块，而多个模块又包含多个模块，那么如果不进行处理的话，所有模块的导入都会利用HTTP请求就会消耗浏览器的网络性能。

依赖预构建解决了3个问题：

1. 不同的第三方包会有不同的导出格式，这个是 vite 没办法规定别人的事
2. 对于路径的处理，我们可以直接使用 /node_modules/.vite/deps/，方便路径重写。 
3. 叫做网络多包传输性能的问题（这也就是原生 es module 不敢支持 node_modules的原因之一），有了依赖预构建以后无论它里面有多少额外的 export 和 import,Vite 都会尽可能的将他们进行集成，最后只生成一个或者几个模块

# 6.Vite 配置文件的语法提示/生产环境和开发环境的配置

vite.config.js 文件的中的语法提示，在 webstorm 中一般有较好的提示，但是到了 VScode 中如果不做一些处理的话，没有良好的代码语法提示。

vite 配置智能提示：https://cn.vitejs.dev/config/#config-intellisense

处理方法：

1. 通过导入 `defineConfig` 方法，来获得良好的代码提示

<img src="D:\8.skill\Vite学习\笔记\vite教程\images\image-20220915211006951.png" alt="image-20220915211006951" style="zoom:70%;" />

原因在于：

首先，`defineConfig` 是一个函数，该函数是用 TS 书写的，它返回一个 UserConfigExport 类型，VScode 就明白该函数返回这个实体对象，就拿到里边的内容（这一块不太明白）；主要还是在于该函数接收一个参数，该参数是被限制类型的，限制的类型为 UserConfigExport ，它就只接受该参数类型里边的成员，那么你在该对象中书写的时候，就会有良好的代码语法提示了。

2. 使用注释来获得提示

```js
/** @type {import('vite').UserConfig} */
export default {
  // ...
}

// 或者这样写
/** @type {import('vite').UserConfig} */
export const viteConfig = {
    //...
}
```

上述方法类似一些大厂的规范代码

```js
/*
/**
 * @param
 * @return{string}
 */
function bar() {}
// 在调用 bar 的时候，会得到 string 的语法提示
```

关于 vite 环境的配置

​	在 webpack 我们会书写环境变量文件，同样的在 vite 中也存在该做法。

1. 首先，先编写 vite 的环境配置文件 vite.config.js

```js
import { defineConfig } from 'vite'
// defineConfig 此时接受一个函数，函数中的参数 command 是属于环境变量，有两个值：build 和 serve
export default defineConfig(({ command }) => {
  if (command === 'build') {
    // 代表生产环境的配置
  } else {
    // 代表开发环境的配置
  }
})
```

2. 接下来，编写 3 个环境配置文件：vite.base.config.js 、vite.dev.config.js 、vite.prod.config.js

```js
// vite.base.config.js 、vite.dev.config.js 、vite.prod.config.js 这三个文件的初始内容
import { defineConfig } from 'vite'

export default defineConfig({})
```

3. 将这些配置文件组合起来

```js
import { defineConfig } from 'vite'
import viteBaseConfig from './vite.base.config.js'
import viteDevConfig from './vite.dev.config.js'
import viteProdConfig from './vite.prod.config.js'

// 策略模式
const envResolver = {
  build: () => {
    console.log('生产环境')
    return { ...viteBaseConfig, viteProdConfig }
  },
  serve: () => {
    console.log('开发环境')
    return { ...viteBaseConfig, viteDevConfig }
  },
}
export default defineConfig(({ command }) => {
  // 返回一个配置对象
  return envResolver[command]()
})
```

# 7.Vite 中环境变量处理

在上一章中，我们了解到的是 Vite 的环境配置文件。而在这一章中，我们会了解到环境变量。简单区分，环境配置文件属于 Vite 所有，而环境变量则不属于 Vite 。

环境变量是指什么？

环境变量，会根据当前代码环境产生值的变化的 **变量** 就叫做环境变量。

小公司会遇到的环境，一般就两个：

1. 开发环境
2. 生产环境

而大公司的环境就比较多了：

1. 开发环境
2. 测试环境
3. 预发布环境
4. 灰度环境
5. 生产环境

使用环境变量的原因，在于我们需要在不同的环境下，某些变量要采用不同的值。如：地图 APP_KEY 的切换。

这时我们面临的情况是：

> 开发环境：APP_LEY  = 110
>
> 测试环境：APP_KEY = 111
>
> 生成环境：APP_KEY = 112

​	每当我们需要在不同环境的上部署代码的时候，我们就需要给 APP_KEY "切换"不同的值。而且我们还很有可能会忘记切换（这在现	实生活中是极为可能的），所以我们希望切换的过程是一个自动化的过程，只需在部署的时候，配置好相应的值即可。

在 Vite 中环境变量的处理：

背景知识：

​	dotenv 库，是 webpack 、Vite 等构建工具用来处理环境变量所借助的第三方库。Vite 之所以能够处理环境变量完全依赖于该库。它的作用：当我们去运行构建命令的时候如 `npm run serve` 、`npm run bulid` 、`yarn dev` 、`yarn build` 的命令的时候，dotenv 会自动在当前目录下寻找 `.env` 文件，找到之后并读取文件，因为读取到的文件是一个字符串，并且是在 node 端读取的，故 node 就可以使用 js 中的split 方法来将文件进行切割，将获取到的 key-value 注入到 process 对象下（process 对象是 node 中对于当前进程的一个对象），但是 Vite 并不会这个时候，就将读取到的 key-value 直接注入到 process 中，因为他考虑到了和其他配置冲突的问题。

主要是和这两个配置可能会发生冲突：

1. root （这个之后再讲）
2. envDir：用来配置当前环境变量的文件地址。也就是说，dotenv 会默认的读取当前目录下的 .env 文件，而我们可以通过 envDir 来配置自定义的当前环境变量的文件地址。

这样一过程，是发生在 node 读取运行 `vite.config.js` 文件之前所做的操作，其实也很好理解，你不能再处理该变量之后，去获取它的值（这段话存疑）。

补充：node 在处理 `vite.config.js` 文件的时候（这个文件只运行在 node 端），你要注意这时你是运行在 node 端的，nodejs 中是没有 es module 规范的，但我们在写 vite.config.js 的时候，却使用的 import 语法，那 node 如何进行处理？首先，他还是读取文件，读取到的还是一段字符串，当他读到 import 语法的时候，会将 import 语法 replace 成 required() 语法，那么他就可以进行导入，导出原理也一样。

访问 process 变量

​	在 `vite.config.js`  中，我们可以访问到这个对象。

```js
......
export default defineConfig(({ command }) => {
  console.log('process', process)
  // 而且我们主要访问 process.env
    console.log('process',process.env)
  // 返回一个配置对象
  return envResolver[command]()
})
```

现在，我们来分析一下上述的代码，我们是先访问的 process 变量，然后才进行配置。设想一下，如果我们在开发或生产配置文件中，进行配置了 `envDir` 属性，并且假设 dotenv 直接将读取到的 key-value 注入到 process 中，那么我们获取到的环境变量就不是我们想要的了。那么，解决办法在于，我们在配置 `envDir` 的时候我们已经知道了环境文件的地址，那么我们就可以采用手动的办法，让 vite 确切的知道该去读取哪里的环境变量文件。

```js
// vite 提供的手动确认方法，loadEnv()
import { loadEnv } form 'vite'
......
export default defineConfig(({ command }) => {
  console.log('process', process)
  // 而且我们主要访问 process.env
    console.log('process',process.env)
  // 返回一个配置对象，为要拿取的环境变量
  const env = loadEnv(mode, process.cwd(), '')
  console.log(env)
  return envResolver[command]()
})
```

process.cwd() 详解：

​	它返回当前 node 进程的工作目录。

loadEnv()

参考链接：https://cn.vitejs.dev/guide/api-javascript.html#loadenv

作用：加载 `envDir` 中的 `.env` 文件，包含文件中以第三个参数为前缀的 key-value 值。

上述，env 变量的值：

![image-20220925173041579](D:\8.skill\Vite学习\笔记\vite教程\images\image-20220925173041579.png)

这一堆的变量为 node 端注入进去的，不用管。使用了 loadEnv 方法确认后，读取到的变量会被注入到 process 对象中。

上述主要是介绍了 dotenv 第三方库和 Vite 的结合使用的过程和注意事项。

接下来我们介绍：环境变量会存储在哪些文件中？

1. .env 文件，所有环境都需要的环境变量
2. .env.development 文件，开发环境需要用到的环境变量文件（默认情况下 Vite 将我们的开发环境取名为 development）
3. .env.production 文件，生产环境需要用到的环境变量文件（默认情况下 Vite 将我们的开发环境取名为 production）
4. 当然我们可以自定义配置环境，......

在 Vite 总如何表示当前运行环境为开发环境或者为生产环境？

​	Vite 中是通过 mode 值的变化，来决定的。当我们运行 `yarn dev` 时，mode 的默认值为 `development` ;当我们运行 `yarn bulid` 时，mode 的默认值为 `production` ;

```js
// yarn dev 相当于运行 yarn dev --mode development 
// yarn bulid 相当于运行 yarn bulid --mode production
export default defineConfig(({ command, mode }) => {
  console.log('process', mode)
  // 返回一个配置对象
  return envResolver[command]()
})
```

​	当然，我们也可以修改 mode 的默认值，当我们运行 `yarn dev --mode develop` ，mode 的值为 develop。同样的，当我们运行 `yarn bulid--mode product`，mode 的值为 product。

我们在返回分析这段代码：

```js
export default defineConfig(({ command, mode }) => {
  // vite 中的 loadEnv 方法
  const env = loadEnv(mode, process.cwd(), '')
  // 返回一个配置对象
  return envResolver[command]()
})
```

当我们执行 loadEnv 方法的时候，发生了什么？

1. 直接找到 .env 文件不解释，并解析其中的环境变量，并放到一个对象中。

2. 会将传进来的 mode 变量进行拼接，```.env[mode]``` ，并根据我们提供的目录去找到该文件并解析，并放到一个对象中

3. 我们可以理解为

   1. ```js
      const baseEnvConfig = 读取 .env 配置
      const developmentEnvConfig = 读取 .env.development 配置
      const lastEnvConfig = { ...baseEnvConfig , ...developmentEnvConfig }  // 相同的key后边的覆盖前面的
      ```

上述是在 `vite.config.js` 中读取到的环境变量，而 vite.config.js 是在 node 端（服务端）被读取的，那他可以访问当 process 对象。但在客户端，我们该如何访问环境变量。

在客户端访问环境变量：

​	如果是客户端，Vite 会将对应的环境变量注入到 import.meta.env 里去。

要注意的是，当  vite 向  import.meta.env 注入环境变量的时候，会做一次拦截，并不是所有环境变量都会注入到  import.meta.env 对象中。

为什么要做一个拦截呢？

​	为了防止我们将不必要的还有隐私性的变量直接送进 import.meta.env，所以它做了一次拦截。如果你的环境变量不是以 Vite 开头的，它就不会帮你注入到客户端去。

​	如果我们想要更改这个前缀，我们可以通过增加 envPrefix 配置

