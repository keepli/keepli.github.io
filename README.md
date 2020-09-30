# 新增功能
### 1.鼠标点击和打字特效

- 添加的脚本文件在`themes\next\source\js\cursor\ `目录下
-  主题自定义布局文件 `themes\next\layout\_custom\custom.swig `引入脚本（自建文件）
-  主题自定义布局文件`themes\next\layout\_layout.swig` 中 body 末尾引入 `custom.swig `

#### 打字特效脚本
- activate-power-mode.min.js
#### 鼠标点击特效脚本

 -  text.js
 - love.min.js
 - fireworks.js 
 - explosion.min.js

#### 主题配置文件中配置

```yml
# 鼠标点击特效
cursor_effect: 
  enabled: true
  type: fireworks  # fireworks：礼花 | explosion：爆炸 | love：浮出爱心 | text：浮出文字

# 打字特效
typing_effect:
  colorful: true  # 礼花特效
  shake: false  # 震动特效
```
### 2.背景几何

#### 脚本位置

- 在`themes\next\source\lib\canvas-nest`整个`canvas-nest`都为新引入的目录
- 修改了hexo-next中`.gitignore`文件的忽略项，注释了`source/lib/*`
