## 自定义滚动条组件
想要实现自定义的滚动条组件替换浏览器默认的滚动条，大体需要考虑以下几个方面：
1. 屏蔽浏览器默认的滚动条。
2. 不能影响原有元素的布局样式。
3. 实现原有浏览器默认的滚动功能。
   1. 实现元素内部超出内容的滚动效果。
   2. 实现自定义的拖拽条。
4. 要考虑滚动条组件或者原生滚动条的嵌套。

### 方案一（element-ui）
#### 屏蔽浏览器默认滚动条
1. 首先计算出浏览器默认滚动条的宽度。

    可通过向页面添加隐藏的 `div` 元素获得：
    ```js
    let scrollbarWidth;

    function getScrollBarWidth() {
      if (scrollbarWidth !== undefined) return scrollbarWidth;

      let element = document.createElement('div');
      element.className = 'ln-scrollbar-temp';
      element.style.visibility = 'hidden';
      element.style.width = '100px';
      element.style.position = 'absolute';
      element.style.top = '-9999px';
      element.style.overflow = 'scroll';

      document.body.appendChild(element);
      scrollbarWidth = element.offsetHeight;
      element.parentNode.removeChild(element);
    }
    ```

2. 将元素用两个容器（`div`）嵌套包裹，第一个容器设置 `overflow` 为 `hidden`，第二个容器设置 `overflow` 为 `scroll`，`margin-right` 和 `margin-bottom` 设置为 `-{scrollbarWidth}px`，这样滚动条就会被父元素遮住。

#### 消除因为元素被包裹导致的布局问题
包裹容器的 `position` 样式设置为 `relative`。

#### 实现滚动功能
该方案不需要实现元素如何滚动，因为我们只是遮住了滚动条，滚动功能依旧存在，我们需要做的是实现自定义滚动条随着滚动移动到指定位置。

这里以纵向滚动条为例，横向滚动条也是同样的原理：

1. 计算滚动条高度。

    ```js
    (clientHeight / scrollHeight) + '%'
    ```

2. 实现滚动条随着滚动移动到指定位置。

    可通过监听第二个容器的滚动事件（`onScroll`），计算出滚动条的偏移量（translateY）。
    ```js
    (scrollTop / clientHeight) + '%'
    ```

#### 嵌套滚动的处理

此方案不需要考虑，因为使用的是默认的滚动行为。

### 方案二（perfect-scrollbar）
#### 屏蔽浏览器默认滚动条

直接设置 `overflow` 为 `hidden`，禁用浏览器的默认滚动行为。

#### 消除因为元素被包裹导致的布局问题
该方案并不会为元素增加包裹容器，所以不会有布局的干扰问题。

#### 实现滚动功能
因为禁用了浏览器默认的滚动行为，所以我们需要自己实现滚动行为：
* 元素超出内容的滚动显示。
* 滚动条的高度计算。
* 滚动条的位置计算。

这里以纵向滚动条为例，横向滚动条也是同样的原理：

1. 实现自定义的滚动行为。

    可通过监听元素的 `wheel` 事件，每次调整元素的 `scrollTop` 的值，增加或者减少。

2. 计算滚动条高度。

    ```js
    (clientHeight / scrollHeight) + '%'
    ```

3. 实现滚动条随着滚动移动到指定位置。

    可通过监听第二个容器的滚动事件（`onScroll`），计算出滚动条的偏移量（translateY）。
    ```js
    (scrollTop / clientHeight) + '%'
    ```

#### 嵌套滚动的处理

需要考虑以下几点：
1. 是否是在子滚动元素触发的 `wheel` 事件，如果是需要检查能否响应：
    * 如果子元素已经滚动到顶端，并且还在继续向该方向滚动，则应该响应，否则不能响应。

### 总结

#### 方案一
优点：
* 使用的是浏览器默认的滚动行为，需用考虑滚动功能的实现，嵌套滚动的处理。

缺点：
* 需要使用额外的两个容器包裹滚动元素，某些情况下可能需要调整布局。

#### 方案二
优点：
* 容器包裹滚动元素，不需要考虑布局干扰问题。

缺点：
* 需要自己完全实现滚动行为，还得考虑嵌套滚动元素的复杂情况。
