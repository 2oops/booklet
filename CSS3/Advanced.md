# Css3进阶

1. `float`和`absolute`是兄弟关系

2. ```css
   .cover { //无固定width/height容器内的绝对定位元素拉伸
   	position: absolute; 
   	left: 0; top: 0; right: 0; bottom: 0;
   	background-color: #fff;
   	opacity: .5; filter: alpha(opacity=50);
   }
   
   .overlay { //没有宽度和高度声明实现的全屏自适应效果
       position: absolute;
   	left: 0; top: 0; right: 0; bottom: 0;
   	background-color: #000;
   	opacity: .5; filter: alpha(opacity=50);
   }
   ```

3. ```html
   <!doctype html> //高度自适应的九宫格效果
   <html>
   <head>
   <meta charset="utf-8">
   <title>高度自适应的九宫格效果</title>
   <style>
   html, body { height: 100%; margin: 0; }
   .page {
       position: absolute;
   	left: 0; top: 0; right: 0; bottom: 0;
   }
   .list {
   	float: left;
   	height: 33.3%; width: 33.3%;
   	position: relative;
   }
   .list:before {
   	content: '';
   	position: absolute;
   	left: 10px; right: 10px; top: 10px; bottom: 10px;
   	border-radius: 10px;
   	background-color: #cad5eb;
   }
   .list:after {
   	content:attr(data-index);
   	position: absolute;
   	height: 30px;
   	left: 0; right: 0; top: 0; bottom: 0;
   	margin: auto;
   	text-align: center;
   	font: 24px/30px bold 'microsoft yahei';
   }
   </style>
   </head>
   
   <body>
   <div class="page">
   	<div class="list" data-index="1"></div>
       <div class="list" data-index="2"></div>
       <div class="list" data-index="3"></div>
       <div class="list" data-index="4"></div>
       <div class="list" data-index="5"></div>
       <div class="list" data-index="6"></div>
       <div class="list" data-index="7"></div>
       <div class="list" data-index="8"></div>
       <div class="list" data-index="9"></div>
   </div>
   </body>
   </html>
   ```

4. ```css
   .image {
       position: absolute; left: 0; right: 0; width: 50%; //width>拉伸
       margin:auto;//添加改行后实现绝对定位居中布局
   }
   ```

5. ```css
   .content {  //整体布局
       position: absolute;
       top: 48px;
       bottom: 52px;
       left: 252px;//如果有侧边栏
       overflow: auto;
   }
   ```

6. ```
   .overlay {
       position: absolute;
       top: 0;right: 0;bottom: 0;left: 0;
       background-color: rgba(0,0,0,.5);
       z-index: 9;
   }
   
   <div class="page"></div>
   <div class="overlay"></div>
   ```

   ------

7. `relative`

   1. 限制定位`left,right,top,bottom`，限制`z-index`(z-index设置为auto时在IE7+对内部元素不起限制作用)，限制`overflow`,relative限制不了`position:fixed`
   2. relative相对自身定位，`left，right`同时存在时绝对定位实现拉伸，相对定位则斗争。(left>right,top>bottom)
   3. 层叠上下文
   4. relative最小化影响原则：relative最小化，尽量避免使用relative

8. `margin`

   1. ```css
      .list {
          margin-top: 15px;
          margin-bottom: 15px;
      }
      ```

   2. relative中使用百分比margin以宽度为准，absolute中使用百分比margin以上一定位元素为准

   3. ```css
      margin-left: auto;
      margin-right: auto;//两者都为auto时实现居中效果，但图片（block）不居中
      ```

   4. `width/height`限制了absolute元素自动填满容器

   5. ```css
      position: absolute;
      margin: auto;//自动平分被变更的尺寸空间实现居中
      ```

   6. margin负值可实现两端对齐`margin-left: -20px; margin-right: 20px;`

   7. 负值下实现等高布局`margin-bottom: -20px; padding-bottom: 20px;`必须使用`overflow: hidden`限制

   8. 负值下实现两栏自适应

   9. margin重叠  `margin-start,margin-end`，设置`direction: ltl`，`writing-mode: vertical-lr;`

9. padding

   1. 水平padding影响尺寸，垂直padding不影响尺寸，但会影响背景色（占据空间）
   2. 使用百分比`div{padding: 50%}`构建固定比例布局结构
   3. 配合margin实现等高布局`margin-bottom: 10px; padding-bottom: 10px`
   4. 实现两栏自适应布局

10. overflow

    ```
              1. IE8+等浏览器默认`html {overflow: auto}`，想要去除默认滚动条时设置为hidden即可
              2. ![1552487178590](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1552487178590.png)
              3. 滚动条宽度约为17px
              4. 水平居中跳动问题`.container {width: 1150px; margin: 0 auto;}`，修复方法:添加一行：`padding-left: calc(100vw - 100%)` 100%指可用内容宽度，注意减号两边是空格
              5. ![1552487743420](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1552487743420.png)
              6. ![1552487779330](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1552487779330.png)
              7. ![1552571192168](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1552571192168.png)
              8. ![1552571832530](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1552571832530.png)
              9. overflow与`position: relative`结合使用防overflow失效
              10. ![1552572117691](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1552572117691.png)
              11. ![1552572190052](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1552572190052.png)
              12. ![1552572604540](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1552572604540.png)
    ```

11. border

    ```
                  1. `border-width`不支持百分比，但是有三个参数可选thin(1px),medium(3px默认)，thick(5px)
                  2. border-style: dashed/dotted/double/inset/outset/ridge;
                  3. ![1552574715031](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1552574715031.png)
                  4. ![1552574841846](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1552574841846.png)
                  5. ![1552574994490](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1552574994490.png)
                  6. ![1552575201992](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1552575201992.png)
                  7. ![1552575246183](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1552575246183.png)
                  8. ![1552575522139](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1552575522139.png)
                  9. ![1552575668073](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1552575668073.png)
                  10. border可实现等高布局（局限：不支持百分比宽度）
    ```

12. line-height

    ```
            1. 基线之间的距离即行高（实现的居中并非真正的居中）
          
            2. input元素中加入`line-height: inherit`增加可控性
          
            3. ```css
                     body {//body全局数值行高实践
                         font-size: 14px;
                         line-height: 1.4286；
                         //font: 14px/1.4286;
                     }
                 ```
          
            4. 消除图片底部间隙的方法：A  图片块状化`img {display: block;}`   B 图片底线对齐  `img { vertical-align: bottom;}`  C  行高足够小，基线位置上移`.box {line-height: 0;}`
          
            5. ###### ######实际应用######
          
            6. 图片水平居中`.box {line-height: 300px; text-align: center} .box > img {vertical-align: middle;}`
          
            7. ```css
                     //多行文本水平居中
                     .box {
                         line-height: 300px;
                         text-align: center;
                     }
                     .box > text {
                         display: inline-block;
                         line-height: normal;
                         text-align: left;
                         vertical-align: middle;
                     }
                 ```
    ```

13. float

    ```
           1. 浮动的本身设计是为了实现文字环绕效果
          
           2. 伪元素的处理
          
               ```css
               .clearfix:after {
                   content: '';
                   display: none;
                   clear: both;
               }
               .clearfix {
                   *zoom: 1;//兼容IE6,IE7
               }
               ```
          
           3. float和clear均为left时文本行只显示文字长度
    ```

14. z-index

    ```
    1. z-index在position为非static时才有用
    2. 无嵌套时遵循后来居上原则，有嵌套时遵循祖先优先原则，前提是z-index为数值，而不是auto
    3. ![1552921399583](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1552921399583.png)
    4. 对于非浮层元素，避免设置z-index值，z-index值不超过2
    ```

15. vertical-align

    1. 其百分比值是相对于line-height计算的
    2. `vertical-align: middle;`此时图片为近似垂直居中，要实现完全垂直居中加`font-size: 0`即可
    3. `vertical-align: text-top/text-bottom;`前者为盒子的顶部和父级content area 的顶部对齐，后者……底部
    4. 应用：表情图片（或原始尺寸背景图标）与文字的对齐效果--（使用文本底部较为合适，不受行高及其他内联元素影响）
    5. hmtl中的上标和下标（sup,sub)
    6. 兼容IE8+
    7. 实际应用：vertical-align取负值
    8. 图片居中：inline-block化，0宽度100%高度辅助元素，vertical-align: middle
    9. ![1553011503097](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1553011503097.png)