## gatsby中怎么使用数据

### 在非页面组件里面查询数据

```js
//先需要在全局定义一个可被查询到的数据(/gatsby-config.js)
module.exports = {
  siteMetadata: {
    title: `Pandas Eating lots`,
  }
}
```



```js
// (/src/components/layout.js)
import { useStaticQuery, graphql } from "gatsby"
const Layout = ({ children }) => {
  // 目前你只要记住：只有页面可以进行页面查询。
  // 非页面组件（例如 Layout）可以使用 StaticQuery。
  const data = useStaticQuery(
    graphql`
      query {
        site {
          siteMetadata {
            title
          }
        }
      }
    `
  )
  return <div>{data.site.siteMetadata.title}</div>
  }
```

### 在页面组件里面查询数据

```js
import React from "react"
import { graphql } from "gatsby"

//在gatsby里面通过query查询到的数据自动会加到组件的{data}里面，在组件里面直接用就是了
const AboutPage = ({ data }) => (
    <h1>About {data.site.siteMetadata.title}</h1>
)

export default AboutPage
// 页面查询定义在组件之外（一般在页面组件文件的最后），并且只在页面组件中可用。
export const query = graphql`
  query {
    site {
      siteMetadata {
        title
      }
    }
  }
`
```

### 一些高端操作

#### src下面的文件映射到网页上

- 安装数据源插件

```bash
npm install --save gatsby-source-filesystem
```

- 配置数据源插件

```js
//gatsby-config.js
module.exports = {
  siteMetadata: {
    title: `Pandas Eating Lots`,
  },
  plugins: [
    {
      resolve: `gatsby-source-filesystem`,
      options: {
        name: `src`,
        path: `${__dirname}/src/`,
      },
    }
  ]
}
```

- 重启项目然后打开（GraphiQL）

![img](https://www.gatsbyjs.cn/static/88ec3efe94e380d32bc1a20cd82dd8bf/373fb/graphiql-filesystem.png)

- 然后在页面里面筛选（快捷键ctrl+space快速自动补全，但是ctrl+enter 快捷键查询是可以用的）（注：切换默认输入法的快捷键也是ctrl+space，如果快捷键冲突了那么系统就会默认选择输入法的快捷键，那么网页的快捷键就不能使用了）

```js
//复制查询语句到代码里面就可以使用这些数据了

import React from "react"
import { graphql } from "gatsby"
import Layout from "../components/layout"

const MyFile = ({ data }) => {
  console.log(data)
  return (
    <Layout>
      <div>
        <h1>My Site's Files</h1>
        <table>
          <thead>
            <tr>
              <th>relativePath</th>
              <th>prettySize</th>
              <th>extension</th>
              <th>birthTime</th>
            </tr>
          </thead>
          <tbody>
            {data.allFile.edges.map(({ node }, index) => (
              <tr key={index}>
                <td>{node.relativePath}</td>
                <td>{node.prettySize}</td>
                <td>{node.extension}</td>
                <td>{node.birthTime}</td>
              </tr>
            ))}
          </tbody>
        </table>
      </div>{" "}
      <div>Hello world</div>
    </Layout>
  )
}
export default MyFile

//这里是查询语句
export const query = graphql`
  query {
    allFile {
      edges {
        node {
          relativePath
          prettySize
          extension
          birthTime(fromNow: true)
        }
      }
    }
  }
`
```

## 

