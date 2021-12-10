## gatsby 中如何使用模块化的 css 样式

### 全局的公共样式

```bash
#进入到项目的根目录新建styles文件夹并创建全局样式
makdir styles
cd styles
touch global.css
```

```js
// /src/gatsby-browser.js
//创建好的全局样式如何作用到全局
//在项目的根目录下创建gatsby-browser.js
import "./src/styles/global.css";

//但是，当处理仅在 Node.js 环境中运行的文件时（如 gatsby-node.js），则需要使用 require。
```

### 单个页面的样式

```js
// 建立一个container.module.css(注意这里一定要用.module.css结尾)(src/components/container.js)
.container {
  margin: 3rem auto;
  max-width: 600px;
}
```

```js
// 并在components下面创建一个新的布局组件（src/components/cpmtainer.js）
import React from "react";
//import containerStyles from "./container.module.css"(官方这里貌似是有问题的，改写一下)
import * as containerStyles from "./container.module.css";

export default ({ children }) => (
  <div className={containerStyles.container}>{children}</div>
);
```

### css-in-js 的使用(官方推荐的 Typography.js 处理 css)

- 如何在项目中使用

```bash
#像往常一样建立好一个项目，安装以下包来在项目中支持Typography
npm install --save gatsby-plugin-typography react-typography typography typography-theme-fairy-gates
```

- 在根目录下面的配置文件里面配置插件(gatsby-config.js)

```js
//gatsby-config.js
module.exports = {
  plugins: [
    {
      resolve: `gatsby-plugin-typography`,
      options: {
        pathToConfigModule: `src/utils/typography`,
      },
    },
  ],
};
```

- 建立好了配置文件还不够，我们还需要再去初始化一下这个插件

```js
//src/utils/typography.js
import Typography from "typography";
import fairyGateTheme from "typography-theme-fairy-gates";

const typography = new Typography(fairyGateTheme);

export const { scale, rhythm, options } = typography;
export default typography;
```

- 但是现在我们还是不能愉快的进行 css 代码的编写（这可太麻烦了，还不如我自己写 css，然后再去代码里面引入）

```bash
#官方教程推荐 typography-theme-kirkham gatsby-plugin-emotion @emotion/core
npm install --save gatsby-plugin-typography typography react-typography typography-theme-kirkham gatsby-plugin-emotion @emotion/core
```

- 重复一下上面的步骤

```js
//再去初始一下(src/utils/typography.js)
import Typography from "typography";
import kirkhamTheme from "typography-theme-kirkham";
const typography = new Typography(kirkhamTheme);
export default typography;
export const rhythm = typography.rhythm;
```

```js
//gatsby-config.js
module.exports = {
  plugins: [
    `gatsby-plugin-emotion`,
    {
      resolve: `gatsby-plugin-typography`,
      options: {
        pathToConfigModule: `src/utils/typography`,
      },
    },
  ],
};
```

```js
//在组件里面使用(但是我还是感受不到这些主题和样式有什么很方便的地方)
Copysrc/components/layout.js: copy code to clipboard
import React from "react"
import { css } from "@emotion/core"
import { Link } from "gatsby"
import { rhythm } from "../utils/typography"
export default ({ children }) => (
  <div
    css={css`
      margin: 0 auto;
      max-width: 700px;
      padding: ${rhythm(2)};
      padding-top: ${rhythm(1.5)};
    `}
  >
    <Link to={`/`}>
      <h3
        css={css`
          margin-bottom: ${rhythm(2)};
          display: inline-block;
          font-style: normal;
        `}
      >
        Pandas Eating Lots
      </h3>
    </Link>
    <Link
      to={`/about/`}
      css={css`
        float: right;
      `}
    >
      About
    </Link>
    {children}
  </div>
```

- css-in-js 为什么值得推荐

[阮一峰的 css-in-js 简介](https://www.ruanyifeng.com/blog/2017/04/css_in_js.html)
