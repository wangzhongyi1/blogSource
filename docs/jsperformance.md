# javascript 性能优化

## 加载和执行

1. `</body>` 闭合标签之前，将所有 `<script>` 标签放在页面底部，确保在脚本执行前页面已经完成渲染
2. 合并脚本。下载单个 100KB 的文件比下载 4 个 25KB 的文件更快，因此页面上的 `<script>` 标签越少，加载也就越快，响应也越迅速。
3. 使用无阻塞下载 javascript 的方法，即在页面加载完成后才加载 js 代码，以下有几种无阻塞下载方法：
  - 使用 defer 属性 `<script src="file.js" defer></script>`
  - 使用动态创建的 `<script>` 元素来下载并执行代码
```js
// 核心：监听script元素的 onload 事件
function loadscript(url, callback) {
  let script = document.createElement('script');
  script.onload = function () {
    callback();
  }
  script.src = url;
  document.getElementsByTagName('head')[0].appendChild(script);
}

// 使用：
loadscript('file.js', function () {
  console.log('file is loaded ~');
});
```
  - 使用 XHR 对象下载 javascript 代码并注入页面中
```js
let xhr = new XMLHttpRequest();
xhr.open('get', 'file.js', true);
xhr.onreadystatechange = function () {
  if (xhr.readyState === 4) {
    if (xhr.status === 200 || xhr.status === 304) {
      let script = document.createElement('script');
      script.text = xhr.responseText;
      document.body.appendChild(script);
    }
  }
}
```

## 数据存储

1. 访问字面量和局部变量的速度比访问数组元素和对象成员快，因此尽量使用字面量和局部变量，减少数组和对象成员的使用
2. 尽可能使用局部变量。如果某个跨作用域的值在函数中被引用一次以上，应该把它存储到局部变量中
3. 避免使用 with 语句，它会改变执行环境的作用域链。try-catch 中的 catch 也有同样效果，看实际情况进行运用
4. 属性和方法在原型链中的位置越深，访问的速度越慢

总结：把常用的对象成员、数组元素、跨域变量 保存在局部变量中，有利于提升性能

## DOM

1. 最小化 DOM 的访问次数，如果需要多次访问某个 DOM 节点，使用局部变量存储它的引用
```js
// 不推荐
function innerHTMLLoop() {
  for (let i = 0; i < 5; i++) {
    document.getElementById('myDiv').innerHTML += 'a';
  }
}

// 推荐
function innerHTMLLoop() {
  let content = '';
  for (let i = 0; i < 5; i++) {
    content += 'a';
  }
  document.getElementById('myDiv').innerHTML = content;
}
```

2. 如果需要在迭代中使用 HTML 集合，把它的长度缓存到一个变量中；如果需要多次操作集合，把它拷贝到一个数组中
  - HTML 集合有 document.getElementsByName(), document.getElementsClassName(), document.getElementsByTagName, document.images, document.links, document.forms 等等，返回的是类似数组的列表，但是它以一种“假定实时态”实时存在，即当底层文档对象更新时，它也会自动更新。
```js
// 假定实时态 导致的死循环
let divs = document.getElementsByTagName('div');
for (let i = 0; i < divs.length; i++) {
  document.body.appendChild(document.createElement('div'));
}
```

1. 减少重绘和重排，批量修改样式时，“离线”操作DOM树，并减少访问布局信息的次数
  - 当页面布局和几何属性改变时需要“重排”。下列情况均会发生“重排”
    + 添加或删除可见DOM元素
    + 元素位置发生变化
    + 元素尺寸改变（包括：外边距、内边距、边框厚度、宽度、高度等属性改变）
    + 内容改变，如文本改变或图片被另一个不同尺寸的图片代替
    + 页面渲染器初始化
    + 浏览器窗口改变
  - 完成“重排”后，浏览器会重新绘制受影响的部分到屏幕中，过程称为“重绘”
  - 减少“重排”和“重绘”
    + 使元素脱离文档流
    + 对其应用多次改变
    + 把元素带回文档中
```js
// 第一种方式：隐藏元素，应用修改，重新显示
let ul = document.getElementById('ul');
ul.style.display = 'none';
appendDataToElement(ul, data); // 向ul中添加附加数据
ul.style.display = 'block';


// 第二种方式：使用文档片段在当前 DOM 树之外构建一个子树，再把它拷贝到文档中
let fragment = document.createDocumentFragment();
appendDataToElement(fragment, data);
document.getElementById('app').appendChild(fragment);


// 第三种方式：将原始元素拷贝到一个脱离文档的节点中，修改副本，完成后再替换原始元素
let old = document.getElementById('app');
let clone = old.cloneNode(true);
appendDataToElement(clone, data);
old.parentNode.replaceChild(clone, old);
```

4. 使用事件委托减少事件处理器的数量
  - Event 对象提供了一个属性叫 target，可以返回事件的目标节点，我们称为事件源对象，也就是说，target就可以表示为当前的事件操作的对象，但不是真正操作dom。
```js
ul.addEventListener('click', function (e) {
  e = e || window.event;
  let target = e.target || e.srcElement;
  if (target.nodeName.toLowerCase() === 'li') {
    alert(target.innerHTML);
  }
})；
```

## 算法和流程控制

- 选择时间复杂度低的算法
- 使用 if-else 要确保最可能出现的条件放在首位，条件数量大时，优先使用 switch
- 考虑使用 hash 表存储数据，不仅速度快，而且可读性好

## 部署和构建

- 合并 javascript 文件以减少 HTTP 请求数
- 使用工具压缩 css 和 js 文件体积
- 启用 gzip 压缩
- 使用 CDN 提供 javascript 文件；CDN 不仅可以提升性能，也会为你管理文件的压缩和缓存
