---
title: Antd Table 拖动控制列宽度
description: 介绍如何实现 antd table 组件头部可通过拖拽来改变列的宽度
date: 2023-02-13 12:36:12
categories:
  - Web
tags:
  - Antd
  - React
  - Table
  - Thead
  - 拖动
  - 列宽度
---

# Antd Table 拖动控制列宽度

介绍如何实现 [antd table](https://ant-design.antgroup.com/components/table-cn) 组件头部可通过拖拽来改变列的宽度

## 背景

最近一个项目有一个需求, 使用表格组件时需要让表格头部列的边框可以横向拖拽, 并且拖拽后需要改变该列的宽度, antd 的 table 组件中并没有实现这个功能, 所以需要自己二次封装

## 需求

1. 表格列头部右部边框可横向拖拽
2. 拖拽中显示一条贯穿表格的纵向虚线展示当前拖到的位置
3. 拖拽完成后更新该列的宽度到鼠标松开后的位置

## 封装 table 组件

由于 antd table 并没有提供类似功能, 所以需要将 table 组件二次封装, 以下是封装后的 Table 组件代码

```tsx
// Table.tsx
import { Table as AntdTable, TableProps } from "antd";
import { useState } from "react";

import { DraggableHeaderBorder } from "./ResizableHeader";

/**
 * antd table 的 components api 中, row 字段的函数参数类型没有导出
 * 所以需要自己定义一个, 其中包含 column 中的相关信息
 */
interface ITableRowProps {
  children: {
    key: string;
    props: {
      children: string;
      column: {
        dataIndex: string;
        title: string;
        key: string;
        width?: number;
      };
    };
  }[];
}

export const Table = <T,>({ columns, ...props }: TableProps<T>) => {
  // 创建一个基于传入的 columns 的新的值
  const [mergedColumns, setCol] = useState(columns);

  return (
    <AntdTable<T>
      {...props}
      columns={mergedColumns}
      // components api 用于替换 table 中的原有组件
      components={{
        header: {
          row: (props: ITableRowProps) => {
            return (
              <tr>
                // 便遍历所有 column, 并重新生成自定义的 th
                {props.children.map((item) => {
                  return (
                    <DraggableHeaderBorder
                      key={item.key}
                      title={item.props.children}
                      onChange={(width) => {
                        const key = item.key;
                        setCol(
                          mergedColumns?.map((col) =>
                            col.key === key ? { ...col, width } : col
                          )
                        );
                      }}
                      width={item.props.column.width}
                    />
                  );
                })}
              </tr>
            );
          },
        },
      }}
    />
  );
};
```

其中, DraggableHeaderBorder 是可拖拽组件, 该组件的实现如下:

```tsx
import { FC, memo, useEffect, useRef } from "react";

interface IProps {
  title: string;
  // 用于获取表格组件的高度和top偏移量以定位拖拽时的边框组件位置
  onChange: (width: number) => void;
  // 如果没有宽度则不支持拖拽
  width?: number;
}

// 可拖拽组件通用类
const defaultClasses = "w-1 border-r";
// 拖拽前的类
const beforeDraggingClasses =
  "absolute top-1/2 -translate-y-1/2 right-0 h-4 border-secondary-neutral";
// 拖拽中的类
const draggingClasses =
  "border-dashed fixed border-quaternary-content z-[9999]";

export const DraggableHeaderBorder: FC<IProps> = memo(
  ({ title, width, onChange }) => {
    const thRef = useRef<HTMLTableCellElement>(null); // 用于获取默认分配的列表头宽度
    const borderRef = useRef<HTMLSpanElement>(null); // 可拖拽组件
    const initLeft = useRef<number>(0); // 初始位置
    const endLeft = useRef<number>(0); // 结束位置

    /**
     * 开始拖拽的回调函数
     * @param {MouseEvent} e
     */
    function handleDragStart(e: MouseEvent) {
      e.preventDefault();
      e.stopPropagation();
      // 记录拖拽开始时的位置
      initLeft.current = Math.floor(e.clientX);

      // 找到表格容器父组件, 并获取其高度, 用于确定拖拽时的纵向虚线的高度
      const height =
        (e.target! as HTMLElement).closest(".ant-table-container")
          ?.clientHeight ?? 0;
      const top = e.clientY;
      borderRef.current!.setAttribute(
        "class",
        defaultClasses + " " + draggingClasses
      );

      borderRef.current!.setAttribute(
        "style",
        `left:${initLeft.current}px;top:${top}px;height:${height}px`
      );

      window.addEventListener("mousemove", handleDragOver);
      window.addEventListener("mouseup", handleDragEnd);
    }

    /**
     * 拖拽中的回调函数
     * @param {MouseEvent} e
     */
    function handleDragOver(e: MouseEvent) {
      e.preventDefault();
      e.stopPropagation();
      const x = Math.floor(e.clientX);
      borderRef.current!.style.left = `${x}px`;
    }

    /**
     * 结束拖拽的回调函数
     * @param {MouseEvent} e
     */
    function handleDragEnd(e: MouseEvent) {
      e.preventDefault();
      e.stopPropagation();
      if (borderRef.current) {
        // 记录拖拽结束时的位置
        endLeft.current = borderRef.current.offsetLeft;
        // 恢复之前的样式
        borderRef.current.setAttribute(
          "class",
          defaultClasses + " cursor-col-resize " + beforeDraggingClasses
        );
        borderRef.current.setAttribute("style", "");

        // 定位到新位置并提交变更后宽度
        const diff = endLeft.current - initLeft.current;
        const width = thRef.current!.offsetWidth + diff;
        onChange(width);
      }
      // 鼠标按键抬起时清除鼠标移动的事件监听
      window.removeEventListener("mousemove", handleDragOver);
    }

    useEffect(() => {
      if (borderRef.current && width) {
        // 设置了宽度的col才能拖拽修改宽度
        borderRef.current.addEventListener("mousedown", handleDragStart);
      }

      // 组件销毁时清除事件监听
      return () => {
        if (borderRef.current && width) {
          borderRef.current.removeEventListener("mousedown", handleDragStart);
          window.removeEventListener("mouseup", handleDragEnd);
        }
      };
    }, []);

    return (
      <th
        className="relative h-[32px] !p-3 before:!content-none"
        ref={thRef}
        style={width ? { width } : {}}
      >
        <span className="inline-block w-full">{title}</span>
        <span
          ref={borderRef}
          className={
            defaultClasses +
            " " +
            beforeDraggingClasses +
            `${width ? " cursor-col-resize" : ""}`
          }
        ></span>
      </th>
    );
  }
);
```

该组件生成一个带可拖拽元素的 th 元素, 将其导入到 antd table 的 components api 中的 header.row 字段作为替换的元素即可

随后引入封装后的组件, 使用和 antd table 一样的 api, 即可渲染出能通过拖拽改变列宽度的表格

![antd-table-dynamic-column-width-01](/images/antd-table-dynamic-column-width-01.png)
