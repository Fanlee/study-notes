# createElement

主要接收三个参数：

- type：要创建的react元素类型，可以是标签名称，如==‘div’==， =='span'==，也可以是React组件类型

  ```javascript
  // <span></span>
  React.createElement("span", null);
  // <B/>
  React.createElement(B, null);
  ```

  

- config： 标签上面的属性集合，没有任何属性则为null

  ```javascript
  // <div title="2" style={{background:'red'}}></div>
  React.createElement("div", {
    title: '2',
    style: {
        background: 'red'
    }
  });
  
  ```

  

- children：第三个参数后的所有参数为当前创建React元素的子节点， 若是当前元素节点的textContent则为字符串，否则为新的React.createElement 创建的元素

  ```javascript
  /*
  <ul>
    <li>1</li>
    <li>2</li>
    <li>3</li>
  </ul>
  */
  
  React.createElement(
    "ul", 
    null,
    React.createElement("li", null, "1"),
    React.createElement("li", null, "2"),
    React.createElement("li", null, "3")
  )
  ```

  

```javascript
function createElement(type, config, children) {
  var propName; 
  // 标签上的属性集合
  var props = {};
  var key = null;
  var ref = null;
  var self = null;
  var source = ull;
  
  // config 不为 null 时，说明标签上有属性，将属性添加到 props 中
  // 其中，key 和 ref 为 react 提供的特殊属性，不加入到 props 中，而是用 key 和 ref 单独记录
  if (config != null) {
    // 验证ref的合法性
    if (hasValidRef(config)) {
      ref = config.ref;
    }
    // 验证key的合法性
    if (hasValidKey(config)) {
      key = '' + config.key;
    }
		// self 和 source是开发环境下对代码在编译器中位置等信息进行记录，用于开发环境下调试
    self = config.__self === undefined ? null : config.__self;
    source = config.__source === undefined ? null : config.__source;
    
    // 将 config 中除 key、ref、__self、__source 之外的属性添加到 props 中
    for (propName in config) {
      if (
        hasOwnProperty.call(config, propName) &&
        !RESERVED_PROPS.hasOwnProperty(propName)
      ) {
        props[propName] = config[propName];
      }
    }
  }

  const childrenLength = arguments.length - 2;
  // 只有一个子节点则直接添加到 props 的 children 属性上
  if (childrenLength === 1) {
    props.children = children;
  }
  // 多个子节点则将子节点 push 到一个数组中然后将数组赋值给 props 的 children
  else if (childrenLength > 1) {
    const childArray = Array(childrenLength);
    for (let i = 0; i < childrenLength; i++) {
      childArray[i] = arguments[i + 2];
    }
    props.children = childArray;
  }

  // 如果 type.defaultProps 存在，遍历 type.defaultProps 的属性，如果 props 不存在该属性，则添加到 props 上
  if (type && type.defaultProps) {
    const defaultProps = type.defaultProps;
    for (propName in defaultProps) {
      if (props[propName] === undefined) {
        props[propName] = defaultProps[propName];
      }
    }
  }

  return ReactElement(
    type,
    key,
    ref,
    self,
    source,
    ReactCurrentOwner.current,
    props,
  );
}
```

```javascript
const ReactElement = function(type, key, ref, self, source, owner, props) {
  const element = {
    // 用于表示是否为 ReactElement
    $$typeof: REACT_ELEMENT_TYPE,
    // 用于创建真实 dom 的相关信息
    type: type,
    key: key,
    ref: ref,
    props: props,
    _owner: owner,
  };

  return element;
};
```

