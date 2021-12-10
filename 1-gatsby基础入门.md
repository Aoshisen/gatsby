## gatsby如何创建一个站点

### 前期准备

- node
- npm
- vscode
- git

### 从远程拉取一个站点或者使用默认站点模板来创建项目

```bash
#从远程模板拉取
gatsby new hello-world https://github.com/gatsbyjs/gatsby-starter-hello-world
#使用默认模板搭建
gatsby new hello-world
```

### 值得注意的一些问题

```js
     //1-gatsby 配置里面貌似不支持默认导出匿名箭头函数式组件(官方示例写法)类似于如下
export default () => (
  <div style={{ color: `purple`, fontSize: `72px` }}>Hello Gatsby!</div>
)
//如果不修改写法，会引起浏览器和编辑器警告，从而导致热更新失效。可以参考警告改成如下写法
const IndexPage=()=>(<div style={{ color: `purple`, fontSize: `72px` }}>Hello Gatsby!</div>)


     //2-关于模块化的css样式，有两点特别值得注意。首先，模块化的css样式必须后缀名字为.module.css，否则样式能引入但不生效，其次，在组件中引入css样式有点坑。
//官方写法，项目实践中失败
import containerStyles from "./container.module.css" 
//改进写法
import *as containerStyles from "./container.module.css" 


    //3-关于引入图片的问题
//必须用import方法在文件顶部引入图片，用require方法失效
//OK的写法
import Banner1 from "../assets/banner04.jpg"
<img src={Banner1} alt="图片加载失败" />
//notOK 的写法
<img src={require(../assets/banner04.jpg)} alt="图片加载失败" />

```

### 插件参考官网

[数据源插件和渲染查询到的数据](https://www.gatsbyjs.cn/tutorial/part-five/)

