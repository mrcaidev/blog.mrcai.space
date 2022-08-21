---
title: Web 性能优化：图片
description: 图片是导致 Web 性能问题的主要原因，其对性能的影响甚至达到了 JavaScript 的两倍以上。我们可以从多个方面来优化图片的性能。
topic: Web Performance
createdAt: 2022/8/12
---

高质量的图片能够显著提高网站的吸引力，让用户对网站留下深刻的印象，同时也能够让观点的传达更加清晰直观。

然而，根据 [Web Almanac 2021](https://almanac.httparchive.org/en/2021/page-weight) 的统计，图片也是导致网页性能问题的罪魁祸首。它在页面上的重量，甚至是第二名 JavaScript 的两倍以上。

![各类文件在页面重量中的占比中位数](https://s2.loli.net/2022/08/12/rdjiEWF6oQM83kH.png)

这个问题出现的原因，其实很容易理解：JS 再怎么复杂，在传输的时候它也只是一串代码。比如我用 Next.js 写的博客，那么大一个框架，最后构建出的 First Load JS 顶天也只有 100 多 KB，然而一张图片随随便便就几甚至几十 MB。

我们既想要提供高质量的图片，又不想对性能造成损失，这就要求对图片进行多方面的、深度的优化。

## 适当降低图片质量

简单粗暴，从根源的图片内容上解决问题。

它的原理很简单：一张图片包含的信息量越少，或者越容易被压缩，图片的性能就越好。

1. **虚化**：如果我们只是需要强调图片里的某一个部分，那么就可以考虑虚化背景部分。在保留重点的同时，降低整张图片的信息量。
2. **调低导出质量**：在使用编辑软件导出图片时，可以适当调低图片质量到 75% 左右。这个比例对人类来说，几乎看不出任何区别；但对计算机来说，压缩率有很大的提升。
3. **压缩工具**：[Squoosh](https://squoosh.app) 之类的图片优化工具，可以进一步压缩图片。
4. **缩放**：将图片先缩小为原尺寸的 87%，然后再放大为此时尺寸的 115%。一来一去，尺寸几乎不变，但许多像素在缩放时消失了。

经过这四道工序，一张图片的大小可以变为原来的 3% 左右，而用户的观感几乎不受影响。

## 选择正确的图片格式

|       用途       |    格式    |
| :--------------: | :--------: |
|     普通图片     | WebP, JPEG |
|  复杂计算机图形  | PNG, JPEG  |
|   有透明度图片   | PNG, WebP  |
| 图标等可绘制图形 |    SVG     |
|     动态图片     |   Video    |

## 其它优化

- 调整宽度在 320-1920px 之间。
- 能不透明，就不要透明。
- 删掉 SVG 里所有不必要的点线面。
- 在构建流程中集成 [sharp](https://github.com/lovell/sharp)、[imagemin](https://github.com/imagemin/imagemin) 等工具。

## 响应式图片

如果我们想要为 1920x1080 的显示器提供 1920px 宽的图片，那么手机也同样会接收到这幅 1920px 宽的图片——这对移动端的性能造成了损失。

`<img>` 原生的 `srcset` 属性，可以为不同尺寸的设备提供不同尺寸的图片。`size` 属性可以进一步基于这些图片，实现媒体查询。

```html
<img
  srcset="image-small.jpg 320w            <== 给 <=320px 的设备
          image-medium.jpg 600w           <== 给 <=600px 的设备
          image-large.jpg 1200w           <== 给 <=1200px 的设备
          image-full.jpg 1920w            <== 给 <=1920px 的设备
          "
  size="(min-width:1200px) 1200px, 100vw  <== >=1200px 的设备是 1200px，否则是 100vw"
  src="image.jpg"
  alt="A nice image."
/>
```

提供 4-5 个可选项就足够了，过犹不及。

## 懒加载

直到用户马上滚动到图片的地方，才加载这张图片。使用 `<img>` 原生的 `loading` 属性即可实现。

```html
<img src="image.png" alt="A nice image." loading="lazy" />
```

只需要加这一个属性，就可以给性能带来巨大的提升。`loading="lazy"` 应当成为每张图片的默认值。

_注意：首屏的 LCP 图片不建议使用懒加载，否则反而会降低性能！_