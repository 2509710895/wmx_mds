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