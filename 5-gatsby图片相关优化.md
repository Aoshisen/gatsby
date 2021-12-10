## 在gatsby 里面如何通过新一代的图像处理插件处理图片

### 安装新一代的插件以及配套的插件

```bash
# gatsby-plugin-image gatsby-plugin-sharp gatsby-source-filesystem gatsby-transformer-sharp 需要安装的几个插件 ，也许之前已经安装过了那么就直接安装 gatsby-plugin-image 就行了
yarn add gatsby-plugin-image gatsby-plugin-sharp gatsby-source-filesystem gatsby-transformer-sharp

```
- 在配置里面进行全局配置图像插件

```js
//  gatsby-config.js
module.exports = {
  plugins: [
    `gatsby-plugin-image`,
    `gatsby-plugin-sharp`,
    `gatsby-transformer-sharp`,
  ],
}

//或者可以在这个文件里面进行如下配置，定义所有的通过query查询到的图片的默认的样式

module.exports = {
  plugins: [
    `gatsby-plugin-image`,
    `gatsby-plugin-sharp`,
   {
      resolve: `gatsby-plugin-sharp`,
      options: {
        defaults: {
          formats: [`auto`, `webp`],
          placeholder: `dominantColor`,
          quality: 50,
          breakpoints: [750, 1080, 1366, 1920],
          backgroundColor: `transparent`,
          tracedSVGOptions: {},
          blurredOptions: {},
          jpgOptions: {},
          pngOptions: {},
          webpOptions: {},
          avifOptions: {},
        },
      },
    },
  ],
}
```

### 关于GatsbyImage 的一些参数的说明
- layout :（布局）

     默认值 ‘constrained’/CONSTRAINED   确定了图片的大小，并在图片外盒子小于图片的大小的时候进行等比缩放

     - ‘fixed’/FIXED                   确定了图片的大小，当图片外的盒子大小改变的时候不会影响到里面图片的行为
     - ‘fullWidth’/FULL_WIDTH   图片永远占图片外面盒子的100% 宽度

- aspectRatio :（宽高比）
		   
	 默认值是图片本身宽长比   强制图片渲染指定的宽长比的图片 比如aspectRatio:2 宽长比就会使 2：1
	 几个值得注意的点:
	
	1. aspectRatio 可以设置为一个分数
	
	2. 设置了aspectRatio 属性的图像如果为constrained 或者fixed 那么可以根据长或者宽来计算另一个维度
	
	3. 当aspectRatio 属性设置的图像为fullWidth 时，那么就会根据宽长比来进行适当的裁剪
	
- width/height :（宽/高）
				
	默认值就是图片本身 的宽和长。因为通过query 查询到的图像本身就有宽高，所以这两个属性就没有那么重要了，但是如果还是要自定义宽高的话，那么就可以使用这两个属性 值得注意的几个点：
	
	 1. 如果只设置了一个值，那么另一个值会根据aspectRatio 计算得到
	 2. 如果同时设置了两个值，源图像就会根据设置的长宽才见到设置的值
	
- placeholder :（占位符） 

     默认值是 dominantColor/DOMINANT_COLOR 将计算源图像的主色调，然后将其作为纯色背景展示在 图像上

     - placeholder:blurred/BLURRED 图片还没加载出来的时候将图片加载为一个很小的很模糊的图片显示为其模糊的背景
     - placeholder:'traceSVG'/TRACE_SVG  当图片还没加载出来的时候就加载一个由源图像生成的一张svg图来占位，这适用于那种形状简单或有很多透明的图像
     - placeholder：none /NONE 没有占位图，但是没有占位图的话在网页里面显示会很很尴尬，那么可以设置一个静态的颜色来占位 比如placeholder:"#000"

- formats :(格式)

     默认值为 ['auto','webp'] /[AUTO,WEBP]  ,如果想让图片加载为avif 格式那么可以在这里面设置，但是需要考虑浏览器兼容问题 ['auto','webp','avif']/[AUTO,WEBP,AVIF]

     

- transFormOptions 这本身就是一个对象 ，如果要设置，那么传入以下值 会修改 默认的配置

  比如 transFormOptions ={glayscale:true}
  
  - glayscale:false    是否将图片转化为灰度
  - duotone:false      {highlight: string, shadow: string, opacity: number}  添加双色调效果
  - fotate:0            图像旋转的角度
  - trim:false     修剪不重要的像素 [看文档](https://sharp.pixelplumbing.com/api-resize#trim)
  
  - cropFocus:"attention"/ATTENTION   [文档](https://sharp.pixelplumbing.com/api-resize#resize)
  - fit:'cover'/COVER  [文档](https://sharp.pixelplumbing.com/api-resize#resize)

### 如何在文件中使用gatsby-plugin-image 插件

```js
//首先，需要在gatby-config.js 里面配置插件如下
/**
 * Configure your Gatsby site with this file.
 *
 * See: https://www.gatsbyjs.com/docs/gatsby-config/
 */

const path = require(`path`)
module.exports = {
  /* Your site config here */
  plugins: [
    `gatsby-plugin-sass`,

    `gatsby-plugin-less`,

    `gatsby-transformer-sharp`,

    `gatsby-plugin-image`,

    //图片引用的默认路径前缀
    {
      resolve: `gatsby-source-filesystem`,
      options: {
        name: `images`,
        path: path.join(__dirname, `src/image`),
      },
    },

    //路径别名
    {
      resolve: `gatsby-plugin-alias-imports`,
      options: {
        alias: {
          "@components": path.resolve(__dirname, "src/components"),
        },
      },
    },
    {
      resolve: `gatsby-plugin-sharp`,
      options: {
        defaults: {
          formats: [`auto`, `webp`],
          placeholder: `blurred`,
        },
      },
    },
  ],
}

```





