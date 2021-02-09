# CSS居中对齐总结
### 水平居中
#### 布局(div):
		1.margin 0 auto (普遍适用div)
		2.父盒子设置flex 子盒子margin:auto(弹性盒子,这个直接水平垂直全搞定,普遍适合各种元素)
#### div中的文本
		1.text-align:center (本质是控制元素的位置)
### 垂直居中
#### 布局(div):
		1.父盒子设置flex align-items: center;(本质是多个子元素垂直排列,只有一个则显得是垂直居中效果)
		2.2.父盒子设置flex 子盒子margin:auto(弹性盒子,这个直接水平垂直全搞定)
#### div中的文本:
		1.div盒子设置height: 130px;line-height: 130px;
