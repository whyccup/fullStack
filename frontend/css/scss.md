# scss语法

1. 嵌套规则 父级{子集}
1. 变量和运算 
    1. 变量
        1. 变量以美元符号开头，赋值方法与 CSS 属性的写法一样。

            ```
            $width: 5em;
            // 直接使用即调用变量：
            #main {
              width: $width;
            }
            ```

        1. 支持块级作用域，嵌套规则内定义的变量只能在嵌套规则内使用（局部变量），不在嵌套规则内定义的变量则可以额在任何地方使用(全局变量，变量提升)。
            1. 将局部变量装换为全局变量可以添加！global声明：

              ```
              #main {
                $width: 5em !global;
                width: $width;
              }
              #sidebar {
                width: $width;
              }
              ```

            编译为

              ```
              #main {
                width: 5em;
              }
              #sidebar {
                width: 5em;
              }
              ```
    1. 数据类型： 支持6种主要的数据类型：
        1. 数字，1，2px, 3em
        1. 字符串, 有引号无引号，“foo”, 'bar', baz
        1. 颜色，blue，#04a3f9, rgba(255, 0, 0, 0, 0.5)
        1. 布尔型，true，false
        1. 空值，null
        1. 数组（list），用空格或逗号作分割符，1.5em 1em 0 2em, a,b,c
        1. maps, 相当于js的 object，(key1: val1, key2: value2)
    1. 运算
        1. 支持数字的加减乘除，取整，求摩运算等(+ , -, *, /, %)
        1. **以下三种情况 / 将被视为除法**
            1. 若值或值得一部分，是变量或者函数的返回值
            1. 若值被圆括号包裹
            1. 若值是算术表达式的一部分

              ```
              p {
                font: 10px/8px;             // Plain CSS, no division
                $width: 1000px;
                width: $width/2;            // Uses a variable, does division
                width: round(1.5)/2;        // Uses a function, does division
                height: (500px/2);          // Uses parentheses, does division
                margin-left: 5px + 8px/2px; // Uses +, does division
              }
              ```

              编译为

              ```
              p {
                font: 10px/8px;
                width: 500px;
                height: 250px;
                margin-left: 9px; 
              }
              ```
        1. 圆括号可以用来影响运算的顺序, 与js运算相同，小括号内部的算式优先运算。
        1. 若使用变量，同时又要确保/不做除法运算而是完整地编译到CSS文件中，需要用#{}插值语句将变量包裹
          ```
          p {
            $font-size: 12px;
            $line-height: 30px;
            font: #{$font-size}/#{$line-height};
          }
          ```

          编译为 

          ```
          p {
            font: 12px/30px
          }
          ```
        1. 颜色值运算
            1. 每两个数字分段进行计算，也就是分红色，绿色，以及蓝色的值。
            1. color: #010203 + #040506; 计算 01 + 04 = 05，02 + 05 = 07，03 + 06 = 09，然后编译为 color: #050709; 
            1. 其他运算符同理
            1. **若其一颜色包含alpha通道，则另一颜色也必须拥有alpha值才能进行运算，且alpha值不进入计算**
        1. +可用于连接字符串
            1. cursor: e + -resize; => cursor: e-resize;
            1. **且结果是否有引号，完全取决于运算符左侧有无引号**
        1. 运算表达式与其他值连用时，用空格做连接符
            1. margin: 3px + 4px auto; => margin: 7px auto;
        1. 在有引号的文本字符串中使用#{} 插值语句可以添加动态的值（类似于vue的双花括号）
            1. content: "I ate #{5 + 10} pies!"; => content: "I ate 15 pies!";
            1. 若插入空值（null），则视作插入了空字符
                1. content: "I ate #{$value} pies!" => content: "I ate pies!";
1. 继承

    ```
    .classA: {
      width: 16px;
    }
    .classB: {
      @extend .classA
      height: 20px;
    }
    ```
1. mixin, 即重用代码块
    1. @mixin 定义一个代码块
    1. @include 调用刚才定义的代码块
    1. 遵从嵌套规则，也就是有块级作用域

        ``` 
        @mixin left(opt1, opt2) {
          padding: opt1;
          margin: opt2;
        }
        .box {
          @inclidu left(10px, 20px)
        }
        ```
1. 颜色亮度调整函数 lighten(颜色，百分比) 更亮, darken(颜色，百分比) 更暗。
1. 插入文件 @import 命令插入外部文件，.scss和css都可
1. 条件语句 @if @else
  
    ```
    .box {
      @if 1+1>1 {
        width: 100px;
      } @else {
        width: 200px;
      }
    }
    ```
1. 循环语句 @for @while @each
  
    ```
    @for $i from 1 to 10{
      .border-#{$i}{
        border:#{$i}px solid red;
      }
    }

    @while $i>0{
      .item-#{$i}{
          width:2em*$i;
        }
      $i:$i-2; // 每次循环-2
    }

    @each $member in a,b,c,d {
      .#($member) {
        background-image: url("/image/#{$member}.jpg");
      }
    }
    ```
1. 自定义函数 @function

    ```
    @function name ($n) {
      @return $n*2;
    }

    .box{width: mame(1)}
    ```