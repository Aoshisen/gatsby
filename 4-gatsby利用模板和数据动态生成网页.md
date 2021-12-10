# 通过拿到的本地数据动态的生成HTML文件

## 准备工作

- 插件安装

```bash
#安装解析markdown的插件
npm install --save gatsby-transformer-remark
```

- 在配置文件中引入插件

```js
// gatsby-config.js
module.exports = {
  siteMetadata: {
    title: `Pandas Eating Lots`,
  },
  plugins: [
    `gatsby-transformer-remark`,
    },
  ],
}
```

- 重启一下项目然后打开 GraphiQL(新的插件安装后在此页面就会多两个东西)

![](.\assets\QQ截图20210902161423.png)

- 新增查询语句查询我们的markdown文件并展示到页面上（这里的操作和之前在allFile里面取到数据再渲染到页面上面的逻辑差不多）

```js
// /src/pages/another.js
import React from "react"
import Layout from "../components/layout"
import { graphql } from "gatsby"
import { css } from "@emotion/react"
import { rhythm } from "../utils/typography"

const AnotherPage = ({ data }) => (
  <Layout>
    {/* <h1>Amazing Pandas Eating Things</h1>
    <div>
      <img
        src="https://2.bp.blogspot.com/-BMP2l6Hwvp4/TiAxeGx4CTI/AAAAAAAAD_M/XlC_mY3SoEw/s1600/panda-group-eating-bamboo.jpg"
        alt="Group of pandas eating bamboo"
      />
    </div> */}
    <div>
      <h1
        css={css`
          display: inline-block;
          border-bottom: 1px solid;
        `}
      >
        Amazing Pandas Eating Things
      </h1>
      <h4>{data.allMarkdownRemark.totalCount} Posts</h4>
      {data.allMarkdownRemark.edges.map(({ node }) => (
        <div key={node.id}>
          <h3
            css={css`
              margin-bottom: ${rhythm(1 / 4)};
            `}
          >
            {node.frontmatter.title}{" "}
            <span
              css={css`
                color: #bbb;
              `}
            >
              — {node.frontmatter.date}
            </span>
          </h3>
          <p>{node.excerpt}</p>
        </div>
      ))}
    </div>
  </Layout>
)

export default AnotherPage
export const query = graphql`
  query {
    allMarkdownRemark(sort: { fields: [frontmatter___date], order: DESC }) {
      totalCount
      edges {
        node {
          id
          frontmatter {
            title
            date(formatString: "DD MMMM, YYYY")
          }
          excerpt
        }
      }
    }
  }
`
//但是到这里还查询不到页面markdown的数据 还需要在创建gatsby-node.js 里面给markdown生成可以查询的节点
```

## 重头戏(以编程的方式利用数据创建页面)

> 但是从现阶段的成果来看我们只是完成了一些markdown 的基本数据的提取，根据一个markdown建立一个对应的页面的需求现在应该只是完成了相当小的一部分（只是完成了列出相应markdown的需求，后面可以根据列出来的目录跳转到对应的基于markdown生成的页面当中）

### 生成页面的"路径"（path）或者slug

> ***注意**: 通常，数据源会直接为内容提供一个 slug 或路径名——使用这些系统之一（例如CMS）时，你不需要像创建 Markdown 文件那样自己创建 slug。*

- 需要学习怎么使用Gatsby的两个API（[`onCreateNode`](https://www.gatsbyjs.cn/docs/node-apis/#onCreateNode)和[`createPages`](https://www.gatsbyjs.cn/docs/node-apis/#createPages)）

> 在根目录下创建gatsby-node.js

```js
// /gatsby-node.js
//重启或者停止开发服务器，就会在终端控制台显示出新创建的节点记录
exports.onCreateNode = ({ node }) => {
  console.log(node.internal.type)
}
//修改函数使得其只记录MarkdownRemark节点
exports.onCreateNode = ({ node }) => {
  if (node.internal.type === `MarkdownRemark`) {
    console.log(node.internal.type)
  }
}
// 使用对应的markdown的名称来创建页面路径,需要遍历父节点，拿到文件名称
exports.onCreateNode = ({ node, getNode }) => {
  if (node.internal.type === `MarkdownRemark`) {
    const fileNode = getNode(node.parent)
    console.log(`\n`, fileNode.relativePath)
  }
}
```

- 然后重启开发服务器应该可以看到打印的消息

![](.\assets\QQ截图20210902172057.png)

> 拿到文件名称之后就可以通过文件名创建文件的路径

```js
//通过文件名创建slug的路径可以通过 gatsby-source-filesystem 插件来实现
//因为这里是node 的环境所以需要使用require引入相应的插件
const { createFilePath } = require(`gatsby-source-filesystem`)

exports.onCreateNode = ({ node, getNode }) => {
  if (node.internal.type === `MarkdownRemark`) {
    console.log(createFilePath({ node, getNode, basePath: `pages` }))
  }
}

```

- 重启开发服务器就能看到之前打印的消息改变了

![](.\assets\QQ截图20210902173327.png)

> 将生成的slug（路径）直接添加到MarkDownRemark 节点里

```js
//通过createNodeField创建对应的节点
const { createFilePath } = require(`gatsby-source-filesystem`)
exports.onCreateNode = ({ node, getNode, actions }) => {
  //把创建节点的方法解构出来
  const { createNodeField } = actions
  if (node.internal.type === `MarkdownRemark`) {
     //得到需要创建的文件的路径
    const slug = createFilePath({ node, getNode, basePath: `pages` })
    //通过创建节点的方法根据创建的路径创建对应的节点
    createNodeField({
      node,
      name: `slug`,
      value: slug,
    })
  }
}
```

- 重启开发服务器，打开或刷新 GraphiQL

![](.\assets\QQ截图20210902174652.png)

- 然后创建页面

```js
//gatsby-node.js
const { createFilePath } = require(`gatsby-source-filesystem`)

exports.onCreateNode = ({ node, getNode, actions }) => {
  const { createNodeField } = actions
  if (node.internal.type === `MarkdownRemark`) {
    const slug = createFilePath({ node, getNode, basePath: `pages` })
    createNodeField({
      node,
      name: `slug`,
      value: slug,
    })
  }
}
//++++++ 默认导出
exports.createPages = async ({ graphql, actions }) => {
  // **Note:** The graphql function call returns a Promise
  // see: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise for more info
  const result = await graphql(`
    query {
      allMarkdownRemark {
        edges {
          node {
            fields {
              slug
            }
          }
        }
      }
    }
  `)
  console.log(JSON.stringify(result, null, 4))
}
//++++
```

- 创建一个页面还需要一个模板

```jsx
// src/templates/blog-post.js
import React from "react"
import Layout from "../components/layout"

export default () => {
  return (
    <Layout>
      <div>Hello blog post</div>
    </Layout>
  )
}
//后期替换
import React from "react"
import { graphql } from "gatsby"
import Layout from "../components/layout"

export default ({ data }) => {
  const post = data.markdownRemark
  return (
    <Layout>
      <div>
        <h1>{post.frontmatter.title}</h1>
        <div dangerouslySetInnerHTML={{ __html: post.html }} />
      </div>
    </Layout>
  )
}

export const query = graphql`
  query($slug: String!) {
    markdownRemark(fields: { slug: { eq: $slug } }) {
      html
      frontmatter {
        title
      }
    }
  }
`
```

- 更新gatsby-node.js使得其能正常渲染

```js
//需要使用到路径处理相关(/gatsby-node.js)
const path = require(`path`)
const { createFilePath } = require(`gatsby-source-filesystem`)


exports.onCreateNode = ({ node, getNode, actions }) => {
  const { createNodeField } = actions
  if (node.internal.type === `MarkdownRemark`) {
    const slug = createFilePath({ node, getNode, basePath: `pages` })
    createNodeField({
      node,
      name: `slug`,
      value: slug,
    })
  }
}

exports.createPages = async ({ graphql, actions }) => {
  //++++
  const { createPage } = actions
  const result = await graphql(`
    query {
      allMarkdownRemark {
        edges {
          node {
            fields {
              slug
            }
          }
        }
      }
    }
  `)
//++++根据数据选择对应模板渲染
  result.data.allMarkdownRemark.edges.forEach(({ node }) => {
    createPage({
      path: node.fields.slug,
      component: path.resolve(`./src/templates/blog-post.js`),
      context: {
        // Data passed to context is available
        // in page queries as GraphQL variables.
        slug: node.fields.slug,
      },
    })
  })
 //++++
}
```

- 好了页面添加上了 美滋滋

![img](https://www.gatsbyjs.cn/static/b1a3666651a4de0eaf754938d4c9e4fb/ca3c3/new-pages.png)

- 之前首页的目录链接到我们通过模板生成的页面里面

```js
import React from "react"
import { css } from "@emotion/core"
//++++
import { Link, graphql } from "gatsby"
import { rhythm } from "../utils/typography"
import Layout from "../components/layout"

export default ({ data }) => {
  return (
    <Layout>
      <div>
        <h1
          css={css`
            display: inline-block;
            border-bottom: 1px solid;
          `}
        >
          Amazing Pandas Eating Things
        </h1>
        <h4>{data.allMarkdownRemark.totalCount} Posts</h4>
        {data.allMarkdownRemark.edges.map(({ node }) => (
          <div key={node.id}>
   //++++                                   
            <Link
              to={node.fields.slug}
              css={css`
                text-decoration: none;
                color: inherit;
              `}
            >
              <h3
                css={css`
                  margin-bottom: ${rhythm(1 / 4)};
                `}
              >
                {node.frontmatter.title}{" "}
                <span
                  css={css`
                    color: #bbb;
                  `}
                >
                  — {node.frontmatter.date}
                </span>
              </h3>
              <p>{node.excerpt}</p>
            </Link>
//++++
          </div>
        ))}
      </div>
    </Layout>
  )
}

export const query = graphql`
  query {
    allMarkdownRemark(sort: { fields: [frontmatter___date], order: DESC }) {
      totalCount
      edges {
        node {
          id
          frontmatter {
            title
            date(formatString: "DD MMMM, YYYY")
          }
            //++
          fields {
            slug
          }
            //++
          excerpt
        }
      }
    }
  }
`
```

### 结语

- 这篇文章是通过官方教程书写，文笔略为粗糙，官方的说明也很细致，附上[地址](https://www.gatsbyjs.cn/tutorial/part-seven/#creating-slugs-for-pages)
- 关于根目录下的gatsby-node.js文件的理解，以及导出的createPages和onCreateNode方法。
- - 这两个默认导出的方法可以看作是网站创建过程中的两个生命周期函数，先创建Node节点，然后再根据Node 创建出对应的页面。所以修改这两个默认的导出函数，实现以编程的方式利用数据创建页面。

