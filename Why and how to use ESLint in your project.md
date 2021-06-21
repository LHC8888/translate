原文：https://developer.ibm.com/articles/why-and-how-to-use-eslint-in-your-project/

ps: 找不到合适的次翻译lint，所以以个人理解翻译为“检测”

#  为什么要用eslint以及如何去使用？

Npmjs.org 中有成千上完的包，但是他们的代码质量并非都是一样的。检查项目中的直接依赖管理有多好是非常重要的。如果一个依赖包的功能符合你的功能需求，那么这个依赖包的管理方式应该是在你的考虑范围之外，但是如果你有选择的余地，尽量选择管理优秀的依赖包，或者自己进行管理。

Javascript linting 是我评估一个JS项目好坏的习惯之一。

## 为什么要检测Javascript
运行良好的项目都有清晰、一致的自动化编码规范。当我在review一个项目的时候，如果代码看起来就像一个由小孩用一把小斧头造的屋子，那么我并不认为这个项目的代码是实用的。  
没有编码规范也是会阻碍他人贡献代码。依赖一个不欢迎他人贡献的项目本身也是一个风险因素。  
除了检查代码风格之外，lint 工具也能检测出某些具体类型的bug，比如变量的作用域范围。在lint的时候即可发现下面两个错误，对没有定义的变量进行赋值（在JS中，该变量会暴露到全局中并对全局环境造成污染，也很可能会形成难以发现的问题）和使用没有定义的变量。

## 如何配置ESLint

ESLint 是用来检测 NodeJS 包的主要工具，可以被配置为支持多种代码风格。你可以定义你自己的代码风格，但是我会在这个演示如何使用 StrongLoop 风格。StrongLoop 这种代码风格是普通的（以致于不会让我们太多地关注代码风格），并且与许多开源NodeJS的风格很相似。

步骤：  
1. npm install --save-dev eslint eslint-config-strongloop

2. 在 .eslintrc.json 中增加配置项：
```
{ "extends": "strongloop" }
```

3. 确保有 .gitignore  
   注意，也可以使用 ESLint 特定的 .eslintignore 文件，它与 .gitignore 有一样的语法规则，以及相同的内容。当然，为了减少维护的负担，许多项目都只使用 .gitignore

4. 配置完后，在 package.json 中增加下面配置，可以在执行 test 前执行 ESLint，同时忽略 .gitignore 里的文件路径：
```
// package.json
{
  "scripts": {
    "pretest": "eslint --ignore-path .gitignore ."
  }
}
```

如果你执行 npm run test 脚本，那么 pretest 会在其之前执行，posttest会在之后执行

5. 记得提交修改后的 package.json .gitignore .eslintrc.json


## 如何检测历史代码
@@ How to get existing code to lint clean

人们不想使用 ESLint 的原因之一就是在检测历史代码的时候感觉就像在清理 Augean stables。我建议像 Hercules 一样：使用工具来帮助我们。

1. ESLint 可以自动 fix 很多语法问题。这应是第一个你要用来清理历史代码的工具特性
```shell
./node_modules/.bin/eslint --fix --ignore-path .gitignore .

```

2. 如果项目中配置了 pretest scripts，你也可以这样做
```
npm run pretest --fix
```

3. ESLint有几个确定的问题类型不会主动去 fix，这是你可以使用 prettier 来一次性处理完成。prettier 广泛地用于替代 ESLint，也可以用于在提交之前自动格式化代码。它在引导 ESLint也很有用。
```shell
npm i prettier
 ./node_modules/.bin/prettier --single-quote --trailing-comma es5 --print-width 80 --write --no-bracket-spacing **/*.js
```

在运行完 eslint --fix 和 prettier 之后，控制台上应该就少了很多警告信息。即时 prettier 不比 ESLint 常用，但是它可以作为 ESLint 的完善工具（prettier用于自动格式化，ESLint用于？格式执行？和错误检查）。如果你恰好没时间修复错误信息，你可以先关闭 ESLint 校验。项目中拥有一些强制执行的规则校验会更好。

如果你项目的代码风格都一致，你可以发布一个自己的代码风格。

4. 当项目的历史代码清理干净后记得提交

## 自动化linting

有两种级别的自动化：
1. 项目级别范围的规则
2. 个人配置

对于项目级别的检测，因为 ESLint 用来 test用例的，因此不需要做其他配置，即可使用。

对于个人配置，我更喜欢在每次提交都进行 ESlint检测，因此我的上面介绍过的问题会在CI之前被捕获到。我会在 git 的 pre-commit 钩子执行我的 ESLint 检测。如下：
```shell
cp .git/hooks/pre-commit.sample .git/hooks/pre-commit
```

.git/hooks/pre-commit 文件的末尾如下所示：
```
# If there are whitespace errors, print the offending file names and fail.
exec git diff-index --check --cached $against --
```

将其改为：
```
set -e
npm run pretest

# If there are whitespace errors, print the offending file names and fail.
exec git diff-index --check --cached $against --
```

这样就可以使用git的 pre-commit 钩子来对文件进行 ESLint检测了。