# Ant Design 学习记录

# Ant Table

```TypeScript
const columns: any = [
    {
        // 列头
        title: '指标项',
        // 列对应的字段
        dataIndex: 'name',
        // 宽度
        width: 130,
        ellipsis: true,
        // 当左右滑动表格时，列固定在左边
        fixed: 'left',
    },
    {
        title: '今日新增商机情况',
        // 可实现多级列头
        children: [
            {
                title: '20商机个数',
                dataIndex: 'today_20_count',
                // 数据展示方法
                render: renderNum,
            },
            // {
            //     title: '20商机金额',
            //     dataIndex: 'today_20_amount',
            //     render: renderMoney,
            // },
            {
                title: '40商机个数',
                dataIndex: 'today_40_count',
                render: renderNum,
            },
            {
                title: '40商机金额',
                dataIndex: 'today_40_amount',
                render: renderMoney,
            },
            {
                title: '60商机个数',
                dataIndex: 'today_60_count',
                render: renderNum,
            },
            {
                title: '60商机金额',
                dataIndex: 'today_60_amount',
                render: renderMoney,
            },
            {
                title: '80商机个数',
                dataIndex: 'today_80_count',
                render: renderNum,
            },
            {
                title: '80商机金额',
                dataIndex: 'today_80_amount',
                render: renderMoney,
            },
        ],
    },
    {
        title: '本周新增商机情况',
        children: [
            {
                title: '20商机个数',
                dataIndex: 'week_20_count',
                render: renderNum,
            },
            // {
            //     title: '20商机金额',
            //     dataIndex: 'week_20_amount',
            //     render: renderMoney,
            // },
            {
                title: '40商机个数',
                dataIndex: 'week_40_count',
                render: renderNum,
            },
            {
                title: '40商机金额',
                dataIndex: 'week_40_amount',
                render: renderMoney,
            },
            {
                title: '60商机个数',
                dataIndex: 'week_60_count',
                render: renderNum,
            },
            {
                title: '60商机金额',
                dataIndex: 'week_60_amount',
                render: renderMoney,
            },
            {
                title: '80商机个数',
                dataIndex: 'week_80_count',
                render: renderNum,
            },
            {
                title: '80商机金额',
                dataIndex: 'week_80_amount',
                render: renderMoney,
            },
        ],
    },
    {
        title: '本月新增商机情况',
        children: [
            {
                title: '20商机个数',
                dataIndex: 'month_20_count',
                render: renderNum,
            },
            // {
            //     title: '20商机金额',
            //     dataIndex: 'month_20_amount',
            //     render: renderMoney,
            // },
            {
                title: '40商机个数',
                dataIndex: 'month_40_count',
                render: renderNum,
            },
            {
                title: '40商机金额',
                dataIndex: 'month_40_amount',
                render: renderMoney,
            },
            {
                title: '60商机个数',
                dataIndex: 'week_60_count',
                render: renderNum,
            },
            {
                title: '60商机金额',
                dataIndex: 'month_60_amount',
                render: renderMoney,
            },
            {
                title: '80商机个数',
                dataIndex: 'month_80_count',
                render: renderNum,
            },
            {
                title: '80商机金额',
                dataIndex: 'month_80_amount',
                render: renderMoney,
            },
        ],
    },
    {
        title: '本季度新增商机情况',
        children: [
            {
                title: '20商机个数',
                dataIndex: 'quarter_20_count',
                render: renderNum,
            },
            // {
            //     title: '20商机金额',
            //     dataIndex: 'quarter_20_amount',
            //     render: renderMoney,
            // },
            {
                title: '40商机个数',
                dataIndex: 'quarter_40_count',
                render: renderNum,
            },
            {
                title: '40商机金额',
                dataIndex: 'quarter_40_amount',
                render: renderMoney,
            },
            {
                title: '60商机个数',
                dataIndex: 'quarter_60_count',
                render: renderNum,
            },
            {
                title: '60商机金额',
                dataIndex: 'quarter_60_amount',
                render: renderMoney,
            },
            {
                title: '80商机个数',
                dataIndex: 'quarter_80_count',
                render: renderNum,
            },
            {
                title: '80商机金额',
                dataIndex: 'quarter_80_amount',
                render: renderMoney,
            },
        ],
    },
];
```

## Table组件 每一条记录的组成部分事件绑定 

```javascript
{    
  title: '产品名称',    
  dataIndex: 'goodName',    
  render: text => <Link to='/index/detail'>{text}</Link>, 
} 
//在render 里渲染的Node里绑定
```

# Ant Form

## Form组件 默认值设置 

```javascript
<Form    
	initialValues={{        
    item1: 3,        
    item2: 2,        
    item3: 1000,        
    item4: 3,        
    item5: 18,    
	}} 
> 
  //item1 是form.item的name属性，后面是默认值
```

# 全局化配置中文 

```javascript
import React from 'react' 
import Main from './components/Main/Main'; 
import { ConfigProvider } from 'antd'; 
import zh_CN from 'antd/lib/locale-provider/zh_CN'; 
import 'moment/locale/zh-cn'; 
import 'antd/dist/antd.css' 
import './App.css'; 
export default function App() {  
  return (    
    <div className="App">      
    	<ConfigProvider locale={zh_CN}>        
  			<Main />      
  		</ConfigProvider>    
		</div>  
	) 
} 
```

