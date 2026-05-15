# React + AntV


## React + AntV 常见的“坑1”

**问题一：图表/画布容器获取不到或渲染异常**
**原因分析**： React 是声明式渲染，而 AntV 的底层（如 G2, G6, X6）大多需要直接操作 DOM 节点。如果在 DOM 挂载完成前初始化，或者没有正确清理，就会导致容器获取失败、内存泄漏或重复渲染。
**解决方案**： 必须使用 useRef 来获取真实的 DOM 节点，并在 useEffect 中初始化实例，同时在组件卸载（return 函数）时调用 destroy() 销毁实例。
```jsx
import React, { useEffect, useRef } from 'react';
import { Chart } from '@antv/g2';

const MyChart = () => {
  // 1. 使用 useRef 绑定DOM容器和图表实例
  const containerRef = useRef(null);
  const chartRef = useRef(null);

  useEffect(() => {
    if (!containerRef.current) return;

    // 2. 在 DOM 挂载后初始化图表
    const chart = new Chart({
      container: containerRef.current,
      autoFit: true,
      height: 400,
    });

    chart.data([{ year: '1991', value: 3 }, { year: '1992', value: 4 }]);
    chart.interval().position('year*value');
    chart.render();

    // 将实例存入 ref，方便后续更新或销毁
    chartRef.current = chart;

    // 3. 组件卸载时，必须销毁图表实例，防止内存泄漏
    return () => {
      chart.destroy();
      chartRef.current = null;
    };
  }, []);

  return <div ref={containerRef} />;
};
```

**问题二：AntV 图表无法响应 React 的数据更新**
**原因分析**： 很多新手在 useEffect 的依赖项中写错了，或者在数据更新时没有正确调用 AntV 的更新方法（如 changeData），导致 React 状态变了，但图表还是老样子。
**解决方案**： 监听数据变化，并使用 AntV 提供的增量更新 API（如 chart.changeData() 或 graph.changeData()），而不是每次都重新 new 一个实例
```jsx
import React, { useEffect, useRef } from 'react';
import { PivotSheet } from '@antv/s2';

const MyTable = ({ dataConfig, options }) => {
  const containerRef = useRef(null);
  const s2Ref = useRef(null);

  useEffect(() => {
    if (!containerRef.current) return;

    // 初次渲染
    if (!s2Ref.current) {
      const s2 = new PivotSheet(containerRef.current, dataConfig, options);
      s2.render();
      s2Ref.current = s2;
    } else {
      // 数据更新时，使用 changeData 增量更新，性能更好
      s2Ref.current.changeData(dataConfig);
      s2Ref.current.setOptions(options);
    }
  }, [dataConfig, options]); // 监听数据变化

  return <div ref={containerRef} />;
};
```

**问题三：无法在 AntV 节点/图形中直接使用 React 组件**
**原因分析**： AntV 的底层是用 Canvas 或 SVG 绘制的，它不认识 React 的 JSX 语法。如果想在流程图节点（G6/X6）里放一个带点击事件的 React `<Button />`，直接写是无效的。
**解决方案**： 使用 AntV 官方提供的 React 适配层插件（如 @antv/g6-extension-react 或 @antv/x6-react-shape），将 React 组件注册为自定义节点。
```jsx
import React from 'react';
import { register, Graph } from '@antv/g6';
import { ReactNode } from '@antv/g6-extension-react';
import { Button } from 'antd'; // 假设你想在节点里放一个 Antd 按钮

// 1. 定义一个普通的 React 组件作为节点内容
const MyReactNode = ({ datum }) => (
  <div style={{ padding: 10, border: '1px solid blue', background: '#fff' }}>
    <div>{datum.label}</div>
    <Button size="small" onClick={() => alert('React 组件点击生效！')}>点我</Button>
  </div>
);

// 2. 注册自定义 React 节点
register('node', 'react-node', ReactNode);

const MyGraph = () => {
  const containerRef = useRef(null);

  useEffect(() => {
    const graph = new Graph({
      container: containerRef.current,
      data: {
        nodes: [{ id: 'node-1', data: { label: '我是 React 节点' } }],
      },
      node: {
        type: 'react-node', // 使用刚才注册的类型
        style: {
          // 3. 在 component 中渲染你的 React 组件
          component: (datum) => <MyReactNode datum={datum} />,
        },
      },
    });
    graph.render();
  }, []);

  return <div ref={containerRef} style={{ width: 500, height: 500 }} />;
};
```


**问题四：图表不显示或样式错乱**
**原因分析**：容器没有设置明确的宽度和高度（Canvas 默认为 0）。在 Modal（弹窗）或 Tabs（标签页）中渲染时，由于容器初始是隐藏的，AntV 获取不到宽高。  
**解决方案**： 确保容器有宽高；如果在 Modal/Tabs 中，需要在容器变为可见（visible）后，手动调用图表的 changeSize 或 resize 方法。
```jsx
// 假设在 Antd 的 Modal 中
<Modal open={visible} onOk={handleOk}>
  {/* 只有当 visible 为 true 时，才渲染图表组件，确保能获取到 DOM */}
  {visible && <MyChart />} 
</Modal>

// 或者在图表组件内部监听尺寸变化（如使用 ahooks 的 useSize）
import { useSize } from 'ahooks';
// ...
const size = useSize(containerRef);
useEffect(() => {
  if (chartRef.current && size) {
    chartRef.current.changeSize(size.width, size.height);
  }
}, [size]);
```

-------
## React + AntV 常见的“坑2”

1. **容器高度为 0 导致图表渲染空白**  
**解决方案**：确保容器有明确的 min-height，或者在数据加载完成、容器真实渲染后再初始化图表（可以使用条件渲染 {data.length > 0 && <Chart />}）


2. **局部容器变化，图表不自适应：**  
即使设置了 autoFit: true，它通常只能监听 window.resize。如果你的 React 项目有侧边栏折叠、Tabs 切换等导致局部容器尺寸改变的操作，图表依然会错位。
**解决方案**： 结合前面提到的 ResizeObserver。在 useEffect 中监听 containerRef.current 的尺寸变化，手动触发 chartRef.current.resize()。

3. Tooltip 被父元素遮挡
React 项目中经常会有 overflow: hidden 或带滚动条的卡片布局。
**解决方案**： 在图表配置中，将 Tooltip 挂载到 document.body 上：
```json
tooltip: {
  container: document.body, // 脱离当前 React 组件的 DOM 层级限制
}
```