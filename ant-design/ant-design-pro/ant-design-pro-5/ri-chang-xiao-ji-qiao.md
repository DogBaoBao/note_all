# 日常小技巧

## 表格传输整数，在编辑页面匹配到下拉框

{% tabs %}
{% tab title=" 字符串" %}
```text
          <Select style={{width: '100%'}}>
            <Option value="1">Java</Option>
            <Option value="2">NodeJs</Option>
            <Option value="3">Golang</Option>
            <Option value="4">Python</Option>
          </Select>
```
{% endtab %}

{% tab title="整数" %}
```
          <Select style={{width: '100%'}}>
            <Option value={1}>Java</Option>
            <Option value={2}>NodeJs</Option>
            <Option value={3}>Golang</Option>
            <Option value={4}>Python</Option>
          </Select>
```
{% endtab %}
{% endtabs %}

value说明

```typescript
export interface OptionCoreData {
    key?: Key;
    disabled?: boolean;
    value: Key;
    title?: string;
    className?: string;
    style?: React.CSSProperties;
    label?: React.ReactNode;
    /** @deprecated Only works when use `children` as option data */
    children?: React.ReactNode;
}

export declare type Key = string | number;
```

可以看到 Key 可以是 字符串或者数字

## tsx, jsx, js 的关系

tsx 编译成 jsx 再编译成 js 最后浏览器解析渲染

