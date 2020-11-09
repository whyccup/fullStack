## css

1. 水平居中
    1. 行内元素水平居中 text-align: center
    1. 块级元素水平居中 margin-left；margin-right auto
    1. flex justify-content center 
1. 垂直居中
    1. 单行行内元素垂直居中 height 和行高(line-height) 相等
    1. 块级元素垂直居中 通过绝对定位元素距离顶部50%，并设置margin-top向上偏移元素高度的一半
    1. flex align-items center
1. 水平垂直居中
    1. 通过margin平移元素整体宽度的一半，使元素水平垂直居中。
    1. 用2D变换，在水平和垂直方向都反向平移宽高的一半，从而使元素水平垂直居中。
    1. flex justify-content center align-items center
1. 伪类
    1. link, visited, hover, active
    1. first-child, last-child
    1. not()
    1. nth-child(n) 从0开始，first正顺序，last逆顺序