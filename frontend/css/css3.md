# css3特性

1. 边框 border
1. 圆角 border-radius
1. 背景 background
1. 渐变 gradients
    1. Linear Gradients 向下/向上/向左/向右/对角方向 (默认从上到下)
        1. background-image: linear-gradient(direction, color-stop1, color-stop2, ...);
        1. background-image: linear-gradient(#e66465, #9198e5); // 默认从上到下
        1. background-image: linear-gradient(to right, red , yellow); // 从左到右
        1. background-image: linear-gradient(to bottom  right, red , yellow); // 从左上角开始（到右下角）
        1. background-image: linear-gradient(-90deg, red, yellow); // 以角度所指出的射线
        1. background-image: repeating-linear-gradient(red, yellow 10%, green 20%); // 重复的线性渐变
    1. Radial Gradients 从中间向四周渐变
        1. background-image: radial-gradient(red, yellow, green); // 颜色节点均匀分布
        1. background-image: radial-gradient(red 5%, yellow 15%, green 60%); // 颜色节点不均匀分布
        1.  background-image: radial-gradient(circle, red, yellow, green); // shape 参数定义了形状。它可以是值 circle 或 ellipse。其中，circle 表示圆形，ellipse 表示椭圆形。默认值是 ellipse。
        1. repeating-radial-gradient
1. 文字效果
1. 字体
1. 转换 transform 
    1. 2d 
        1. translateX\Y() 以x或y轴方向平移
        1. translate() 以x、y轴方向平移
        1. rotate() 默认以左上角，或 transform-origin 决定的锚点来进行旋转
    1. 3d 
        1. translateZ() 以z轴方向进行平移
        1. translate3d() 以x、y、z轴方向分别进行平移
        1. rotateX\Y() 以 x或y 轴为轴心进行旋转
        1. rotateZ() 以 垂直于x、y轴的z轴为轴心旋转
        1. rotate3d() 以x、y、z的值所交的点，到原点之间连接形成主轴进行旋转
    1. transform-origin
        1. 定义旋转组件锚点位置。
        1. 也就是坐标轴的原点位置，默认在组件的左上角。
        1. 参数|可能的值
            ---|---
            x|left, center, right, length, %(都是相对于自身，起点在左上角)
            y|top, center, bottom, length, %(都是相对于自身，起点在左上角)
            z|length(都是相对于自身，起点在左上角并且z轴指向屏幕，垂直于x、y)
1. 过渡 transition
1. 动画  @keyframes actionName {   from {background: red;}    to {background: yellow;}}
1. 多列 column
1. 用户界面 resize box-sizing outline-offset
1. 图片 img 
1. 按钮 button
1. 分页 float
1. 框大小 box-sizing
    1. 原
        1. width(宽) + padding(内边距) + border(边框) = 元素实际宽度
        1. height(高) + padding(内边距) + border(边框) = 元素实际高度
    1. 修改后
        1. 宽 = 元素实际宽度
        1. 高 = 元素实际高度
        1. 其他只用布局样式之用
1. 弹性盒子 flex
1. 多媒体查询 
    1. @media screen and (max-width: 480px) 在屏幕可视窗口尺寸小于 480 像素的设备上修改