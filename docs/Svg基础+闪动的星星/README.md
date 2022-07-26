### **闪动的星星**

```jsx
import { useEffect, useRef } from 'react';

const SVG_NS = 'http://www.w3.org/2000/svg';
const XLINK_NS = 'http://www.w3.org/1999/xlink';

const MoonSvg = () => {
  const svgRef = useRef();
  const starRef = useRef();
  const starGroupRef = useRef();

  useEffect(() => {
    renderStar();
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, []);

  const use = (originEl) => {
    const originElId = originEl.id;
    let _use = document.createElementNS(SVG_NS, 'use');
    _use.setAttributeNS(XLINK_NS, 'xlink:href', `#${originElId}`);
    return _use;
  };

  const random = (min, max) => {
    return min + (max - min) * Math.random();
  };

  const renderStar = () => {
    let starCount = 500;
    const starEl = starRef.current;
    const starGroup = starGroupRef.current;

    let star;
    while (starCount--) {
      star = use(starEl);
      star.setAttribute('opacity', random(0.1, 0.4));
      star.setAttribute(
        'transform',
        'translate(' +
          random(-400, 400) +
          ',' +
          random(-300, 50) +
          ')' +
          'scale(' +
          random(0.1, 0.6) +
          ')'
      );
      starGroup.appendChild(star);
    }
  };
  return (
    <svg
      ref={svgRef}
      className="svg-container"
      width="100%"
      height="70%"
      viewBox="-400 -300 800 650"
      preserveAspectRatio="XMidYMid slice"
      style={{ background: '#001122' }}
    >
      <defs>
        {/* 绘制星星 */}
        <polygon
          ref={starRef}
          id="star"
          points="0 -10 2 -2 10 0 2 2 0 10 -2 2 -10 0 -2 -2"
          fill="white"
        >
          {/* 添加两个动画，一个是颜色变化，一个是大小变化 */}
          <animate
            attributeName="fill"
            attributeType="XML"
            from="#fff"
            to="#bfbfbf"
            dur="0.3"
            repeatCount="indefinite"
          />
          <animateTransform
            attributeName="transform"
            attributeType="XML"
            type="scale"
            from="0.5"
            to="1"
            dur="1"
            repeatCount="indefinite"
          />
        </polygon>
      </defs>
      <g id="real">
        {/* 星星的group */}
        <g id="start-group" ref={starGroupRef}></g>
        {/* 绘制月亮 */}
        <g id="moon-group">
          {/* 通过蒙版的方式叠加出月牙形 */}
          <mask id="moon-mask">
            <circle cx="-250" cy="-150" r="100" fill="white"></circle>
            <circle cx="-200" cy="-200" r="100" fill="black"></circle>
          </mask>
          <circle
            cx="-250"
            cy="-150"
            r="100"
            fill="yellow"
            mask="url(#moon-mask)"
          ></circle>
        </g>
        {/* 灯塔 */}
        <g id="light-tower" transform="translate(250, 0)">
          <defs>
            <linearGradient id="tower" x1="0" y1="0" x2="1" y2="0">
              <stop offset="0" stop-color="#999"></stop>
              <stop offset="1" stop-color="#333"></stop>
            </linearGradient>
            <radialGradient id="light" cx="0.5" cy="0.5" r="0.5">
              <stop offset="0" stop-color="rgba(255, 255, 255, 0.8)"></stop>
              <stop offset="1" stop-color="rgba(255, 255, 255, 0)"></stop>
            </radialGradient>
            <clipPath id="light-clip">
              {/* 绘制灯塔发出的光 */}
              <polygon
                points="0 0 -400 -15 -400 15"
                fill="rgba(255, 0, 0, 0.5)"
              >
                {/* 设置光旋转 */}
                <animateTransform
                  attributeName="transform"
                  attributeType="XML"
                  type="rotate"
                  from="0"
                  to="360"
                  dur="10s"
                  repeatCount="indefinite"
                ></animateTransform>
              </polygon>
              {/* 灯塔发亮点 */}
              <circle cx="0" cy="0" r="2"></circle>
            </clipPath>
          </defs>
          {/* 下面的灯塔 */}
          <polygon points="0 0 5 50 -5 50" fill="url(#tower)"></polygon>
          {/* 灯塔照射的范围 */}
          <ellipse
            cx="0"
            cy="0"
            rx="300"
            ry="100"
            fill="rgba(255, 255, 255, 0.5)"
            fill="url(#light)"
            clip-path="url(#light-clip)"
          ></ellipse>
        </g>
      </g>
      {/* 折射的月亮 星星 灯塔 */}
      <g id="reflact" transform="translate(0 50)" mask="url(#fading)">
        <defs>
          <linearGradient id="fade" x1="0" y1="0" x2="4" y2="0">
            <stop offset="0" stop-color="rgba(255, 255, 255, 0.3)"></stop>
            <stop offset="0.5" stop-color="rgba(255, 255, 255, 0)"></stop>
          </linearGradient>
          {/* 蒙版 */}
          <mask id="fading">
            <rect
              x="-400"
              y="0"
              width="800"
              height="300"
              fill="url(#fade)"
            ></rect>
          </mask>
        </defs>
        {/* 直接使用上面的星星 月亮分组 */}
        <use xlinkHref="#real" transform="scale(1, -1) translate(0 -50)" />
      </g>
      {/* 分割线 */}
      <line x1="-400" y1="50" x2="400" y2="50" stroke="white"></line>
    </svg>
  );
};

export default MoonSvg;
```

### **基本属性**

- viewPort（视窗）width,height 控制，好比是浏览器的显示屏幕
- viewBox(视野)用于控制用户观看 svg 的大小，好比是截图工具的那个框，最终呈现的就是把截图内容全屏显示到 svg 上，因为就会有比例关系 -`preserveAspectRatio="xMidYMid meet"`（视野和视窗的关系）
  - 第 1 个值表示，viewBox 如何与 viewport 对齐；第 2 个值表示，如何维持高宽比（若有）

| 值   | 含义                                 |
| ---- | :----------------------------------- |
| xMin | viewport 和 viewBox 左边对齐         |
| xMid | viewport 和 viewBox x 轴中心对齐     |
| xMax | viewport 和 viewBox 右边对齐         |
| YMin | viewport 和 viewBox 上边缘对齐       |
| YMid | viewport 和 viewBox y 轴中心点对齐。 |
| YMax | viewport 和 viewBox 下边缘对齐。     |

| 值    | 含义                                            |
| ----- | :---------------------------------------------- |
| meet  | 保持纵横比缩放 viewBox 适应 viewport            |
| slice | 保持纵横比同时比例小的方向需要放大填满 viewport |
| none  | 扭曲纵横比以充分适应 viewport                   |

ps:通常设置都是`preserveAspectRatio="xMidYMid meet"`

### **基础图形**

- 矩形  <rect />
  - x ：矩形左上角 x 位置,  默认值为  0
  - y：矩形左上角 y 位置,  默认值为  0width  矩形的宽度,  不能为负值否则报错, 0  值不绘制
  - width
  - height：矩形的高度,  不能为负值否则报错, 0  值不绘制
  - rx：圆角 x 方向半径,  不能为负值否则报错
  - ry：圆角 y 方向半径,  不能为负值否则报错
- 圆形  <circle />
  - r：半径
  - cx：圆心 x 位置,  默认为  0
  - cy：圆心 y 位置,  默认为  0
- 椭圆  <ellipse />
  - rx：椭圆 x 半径
  - ry：椭圆 y 半径
  - cx：圆心 x 位置,  默认为  0
  - cy：圆心 y 位置,  默认为  0
- 直线  <line />
  - x1：起点的 x 位置
  - y1：起点的 y 位置
  - x2：终点的 x 位置
  - y2：终点的 y 位置
- 折线  <polyline />
  - 是点集数列，每个点包含两个数字，数字可以用空格，逗号隔开，一组数字，一个代表横坐标，一个代表纵坐标

```HTML
<polyline points="10 30, 15 40, 20 40, 30 70, 50 80, 90 110, 110 140, 95 150, 100 145"/>
```

- 多边形  <polygon />
  - 和上面一样

```HTML
<polygon points="50 160, 55 180, 70 180, 60 190, 65 205, 50 195, 35 205, 40 190, 30 180, 45 180"/>
```

```HTML
<svg width="400" height="600" viewBox="0 0 200 200" preserveAspectRatio="xMinYMin slice" style="border:1px solid #cd0000;">
        <rect x="10" y="10" width="150" height="150" fill="green"/>
    </svg>
```

### **文档结构**

#### 样式处理

- 内联样式

```HTML
<circle cx="50" cy="40" r="12" style="stroke: red; stroke-width: 3px; fill; #ccccff;"/>
```

- 内部样式表
  - <![CDATA[ 样式内容 ]]>这样的目的是为了告诉XML解析器这只是一段字符串，不是XML结构，不用解析

```HTML
<style type="text/css"><![CDATA[
 circle {
 stroke: red; stroke-width: 3px;
 fill: #ccccff;
 }
 rect { fill: gray; stroke: black; }
]]></style>
```

- 外部样式表
  - 外部样式和普通 css 一样，只需要在每个 svg 文件引入<?xml-stylesheet href="svg.css" type="text/css"?>即可使用

```HTML
// svg.css
circle.special {
 stroke: red; stroke-width: 3px;
 fill: #ccccff;
}
<circle cx="40" cy="40" r="20"/>
<circle cx="60" cy="20" r="10" class="special"/>
```

#### 分组和引用对象

`<g>`元素可以将所有的子元素作为一个组合，并且在此标签上写的所有属性会被子元素继承

`<use>`元素可以把复杂的元素，通过 use 的方式以此绘制，多次引用

`<defs>`元素可以在 svg 中只定义某些内容，但不显示它，但是可以被引用显示

`<image>`可以在 svg 中使用图片

ps：后续将对 Path 进行分享
