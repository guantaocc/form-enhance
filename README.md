## 可配置表单思路

基础结构

```js
[
  {
    _component: "FormRadioGroupField",
    _name: "sex",
    data: [
      {
        text: "男",
        value: "male"
      },
      {
        text: "女",
        value: "female"
      }
    ]
  },
  {
    _component: "FormSelectField",
    _name: "qingjiaType",
    data: [
      {
        text: "事假",
        value: "shijia"
      },
      {
        text: "年假",
        value: "nianjia"
      },
      {
        text: "调休",
        value: "tiaoxiu"
      },
      {
        text: "病假",
        value: "bingjia"
      },
      {
        text: "产假",
        value: "chanjia"
      }
    ]
  },
  {
    _component: "ImageUploader",
    _name: "hospitalMaterial",
    _show: values => ["bingjia", "chanjia"].includes(values.qingjiaType),
    tokenUrl: "http://somecdn.youzan.com"
  }
];
```


## 注册组件

```js
form.registerField('name', component)
```


## slot

```js
[
  {
    _slot: "ImageUploader"
  },
  {
    _slot: "MyFooter"
  }
];
```

## 服务端交互

form.config.js

```js
// "店铺类型"的Select
{
  _component: 'FormSelectField',
  _name: 'shopType',
  _fetch_data: () => {
    return getShopType().then(items => {
      return items.map(item => {
        return {
          text: item.desc,
          value: item.type,
        };
      });
    });
  },
  data: [],
},
```

1、原组件（如 FormSelectField 组件）必须支持 data 属性。因为获取到的数据，会默认塞入到 data 中去。

2、_fetch_data 只会触发一次。


重新渲染

```js
[
  {
    _component: "FormSelectField",
    _name: "province",
    required: "请选择您所在省份",
    data: [{ text: "浙江省", value: 30 }, { text: "广东省", value: 31 }]
  },
  {
    _component: "FormSelectField",
    _name: "city",
    _fetch_data: values => {
      if (!values.province) {
        return Promise.resolve([]);
      } else {
        return fetchCityByProvince(values.province);
      }
    },
    _subscribe: (prevValues, values, restart) => {
      if (values.province !== prevValues.province) {
        // 渲染
        restart();
      }
    },
    required: "请选择您所在城市",
    data: []
  }
];
```

## 样式修改

```js
[
  {
    _component: "FormInputField",
    _name: "age",
    _format: ($component, values) => (
      <div>
        {$component}
        <span>岁</span>
      </div>
    ),
    label: "年龄"
  }
];
```


## 表单回填

```js
一些场景下，比如编辑的时候：需要当从服务端取回数据，并在表单上回显。

这种场景该怎么做呢？

zan-form 提供了一个Form.setValues(values, this) 方法，即可把数据回填回去。

本质上，setValues 方法，是对 elementForm.setFieldsValue() 的一个递归调用，最终使 elementForm.getFormValues() 的值趋向于稳定。

```