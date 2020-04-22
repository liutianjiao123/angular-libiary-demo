# Foo

This project was generated with [Angular CLI](https://github.com/angular/angular-cli) version 8.1.3.

## Development server

Run `ng serve` for a dev server. Navigate to `http://localhost:4200/`. The app will automatically reload if you change any of the source files.

## Code scaffolding

Run `ng generate component component-name` to generate a new component. You can also use `ng generate directive|pipe|service|class|guard|interface|enum|module`.

## Build

Run `ng build` to build the project. The build artifacts will be stored in the `dist/` directory. Use the `--prod` flag for a production build.

## Running unit tests

Run `ng test` to execute the unit tests via [Karma](https://karma-runner.github.io).

## Running end-to-end tests

Run `ng e2e` to execute the end-to-end tests via [Protractor](http://www.protractortest.org/).

## Further help

To get more help on the Angular CLI use `ng help` or go check out the [Angular CLI README](https://github.com/angular/angular-cli/blob/master/README.md).
-----------------------------------------------------
##### Angualr创建library库流程及命令

1. ###### 首先安装全局Angular-cli，命令如下：

```bash
npm install @angular/cli -g
```

2. ###### 为了避免Angular库和工作空间命名冲突的问题，在创建工作空间时，建议先不创建初始化src文件目录，即直接创建一个不包含初始化应用的Angular工作区，命令如下：

```bash
// ng new <workspace-name> --createApplication=false

ng new myspace --createApplication=false
```

3. ###### 工作空间创建完成后，继续创建Angular库项目，命令如下：

```bash
cd myspace
// ng genenrate library <library-name>

ng generate library my-lib
```

此命令会在工作区内创建一个project文件目录，并包含有一个my-lib的子目录，它就是我们要生成的my-lib Angular库。

4. ###### 创建测试应用项目，可以直接调用我们建立的Angular库，用这个测试项目可以作为该库的测试或者文档等，创建命令如下：

```bash
// ng generate application <test-app-name>

ng generate application my-lib-test
```

这条命令会在 **project** 文件夹下添加了 **foo-tester** 目录。此外，Angluar CLI 还添加了一个 **foo-tester-e2e** 项目来用做 e2e 测试。

5. ###### 继续用Angular-CLI构建library库，命令如下：

```bash
// ng build <library-name>

ng build my-lib
```

注意，从 v6.1 起，Angluar CLI 始终在生产模式下构建库，所以我们不需要额外指定 `--prod` 选项。

与构建库不同，我们构建一个应用，需要使用 `--prod` 选项，命令如下：

```bash
// ng build <test-app-lib> --prod / (缺省状态为dev方式打包)

ng build my-lib-test --prod
```

打包库是指将生成的发布文件进行打包以生成一个 tgz 文件用以手动分享或发布于 npm。

使用 **npm pack** 指令在根目录下的 package.json 文件中创建一个脚本，该脚本（scripts）用于打包生成的库。

例如：

```json
"scripts": {
  ...
  "build_lib": "ng build example-lib",
  "npm_pack": "cd dist/example-lib && npm pack",
  "package": "npm run build_lib && npm run npm_pack" // "&&"表示串联执行命令；“&”表示并联执行命令
  ...
},
```

现在我们可以通过 

```bash
npm run build_lib
```

指令去构建我们的库了。 该指令会在工作区的 **dist** 目录下创建一个名为 **example-lib** 的子目录。

在完成对于库的构建之后，我们可以使用 

```bash
ng build
```

对主应用进行构建。 该指令会在工作区的 **dist** 目录下创建一个名为 **example-lib-app** 的子目录。

注意，

6. ###### 运行测试项目，库项目无法直接运行，需要通过测试项目进行测试运行，命令如下：

```bash
// ng serve <test-app-lib>

ng serve my-lib-test
```

7. ###### 测试库项目时，分为库项目单元测试和测试应用项目单元测试，其中库项目单元测试命令如下：

```bash
// ng test <library-name>

ng test my-lib
```

测试应用项目单元测试命令如下：

```bash
// ng test <test-app-lib>

ng test my-lib-test
```

以上命令完成后已经实现库项目的创建、运行、测试、打包。

#### PS：

###### package.json 文件

在库构建完成后，当前工作区内至少已经有3个 package.json 文件了。这些 package.json 文件可能会让你觉得困惑为难，所以我们现在理顺这些 package.json 文件的思路。

###### 根目录下的 package.json 文件

这个 package.json 是库工作区的**主 package.json 文件**。我们用该文件来列出主应用和库都需要的依赖项。运行和构建主应用和库所依赖的所有包都必须列举在该文件中。

当我们在开发过程中使用 **npm install** 指令时，新加入的包将会添加到该文件中。

###### 库项目中的 package.json 文件

**库项目中的 package.json 文件**位于 **projects\example -lib** 目录下，其用于告知 **ng-packagr** 将什么信息放入库项目的发布版 package.json 文件中。其 package.json 文件中有三个重要的部分需要注意：

- 名称

这里的名称是指库的名称。未来如果某位用户引入库中的模块，这个名称就是出现在 `from` 部分内的引号中的名称。举例来说大概就是：

```ts
import { ExampleLibModule } from 'example-lib';
```

- 版本号

版本号对于库而言格外重要，版本号能够帮助用户判断他们是否在使用库的最新版本。使用 npm 包的开发者通常会默认你遵循[版本号语义化](https://link.zhihu.com/?target=https%3A//semver.org/)规则。

- 依赖项

此项目中只包含用于运行库所必须的依赖项。因此，你将会看到 **dependencies** 和 **peerDependencies**，但是没有 **devDependencies**。

你同样应该在该 package.json 文件中添加那些常见的 npm 内容比如：License，作者，仓库地址等。

值得注意的是，当你使用 **npm install** 指令时，新安装的包只会被添加到根目录下的 package.json 文件中而不是在库项目的 package.json 文件中。因此，当你安装一个库所需要的包时，你需要将包名称手动添加到库项目的 package.json 文件依赖项中。在后续的文章中我们将会讨论 **dependencies** 和 **peerDependencies** 的区别。

*参考资料：*

[[翻译] Angular Libary 系列之Angular CLI的新功能，不用先初始化工作区就来创建Angular Library](https://github.com/sawyerbutton/angularindepth/blob/master/articles/angular-154.%E7%BF%BB%E8%AF%91-AngularCLI%E7%9A%84%E6%96%B0%E5%8A%9F%E8%83%BD%EF%BC%8C%E4%B8%8D%E7%94%A8%E5%85%88%E5%88%9D%E5%A7%8B%E5%8C%96%E5%B7%A5%E4%BD%9C%E5%8C%BA%E5%B0%B1%E6%9D%A5%E5%88%9B%E5%BB%BAAngular-Library.md)

[[翻译] Angular Libary 系列之 使用 Angular CLI 创建 Library]([https://github.com/sawyerbutton/angularindepth/blob/master/articles/angular-78.%5B%E7%BF%BB%E8%AF%91%5D-Angular-Library-%E7%B3%BB%E5%88%97%E4%B9%8B%E4%BD%BF%E7%94%A8-Angular-CLI-%E5%88%9B%E5%BB%BA-Library.md](https://github.com/sawyerbutton/angularindepth/blob/master/articles/angular-78.[翻译]-Angular-Library-系列之使用-Angular-CLI-创建-Library.md))

[[翻译]Angular Library 系列之 构建和打包](https://zhuanlan.zhihu.com/p/54075148)

[[翻译]Angular Library 系列之 发布](https://zhuanlan.zhihu.com/p/54075336)
