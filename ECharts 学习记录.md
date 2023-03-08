# ECharts 学习记录

# 柱线图

```TypeScript
// 提示框组件
tooltip: {
    // 提示框组件出现的触发方式，axis 为坐标轴触发
    trigger: 'axis',
    axisPointer: {
        // 十字准星指示器
        type: 'cross',
        // 线的样式，type 为cross 才有用
        crossStyle: {
            color: '#999',
        },
    },
},
// 图例组件
legend: {
    // 图例太多，使用滚动展示
    type: 'scroll',
    // 设置翻页按钮大小
    pageIconSize: 10,
    // 图例离容器下侧距离
    bottom: '0',
    // 图例标记的图形宽度
    itemWidth: 10,
    // 图例标记的图形高度
    itemHeight: 10,
    // itemGap: 20,
    // 确定图例的显示
    data: ['20商机个数','40商机个数','40商机金额','60商机个数','60商机金额','80商机个数','80商机金额',],
    // 通过图例控制 折线 柱形 是否显示
    selected: {
        '20商机个数': false,
        // '20商机金额': false,
        '40商机个数': false,
        '40商机金额': false,
        '60商机个数': true,
        '60商机金额': true,
        '80商机个数': false,
        '80商机金额': false,
    },
},
// 直角坐标系内绘图网格
grid: {
    // 组件离容器左侧距离
    left: '0%',
    // 组件离容器右侧距离
    right: '0%',
    // 组件离容器下侧距离
    bottom: '15%',
    // 组件宽度
    width: '100%',
    // 组件高度
    height: '70%',
    // grid 区域是否包括坐标轴的刻度标签，设置为 true 防止标签溢出
    containLabel: true,
},
// 设置直角坐标系的 x 轴
xAxis: [
    {
        // 类目轴 适合离散的类目数据
        type: 'category',
        // 设置x轴的取值范围
        data: xAxis,
        // 鼠标放上柱条出现的样式
        axisPointer: {
            type: 'shadow',
        },
        // 是否显示坐标轴刻度
        axisTick: {
            show: false,
        },
        // 是否显示坐标轴轴线
        axisLine: {
            show: false,
        },
    },
],
// 直角坐标系的y轴
yAxis: [
    {
        // 数值轴，适合连续数据
        type: 'value',
        // 坐标轴在 grid 区域的分隔线
        splitLine: {
            // 分隔线的样式
            lineStyle: {
                // 点线
                type: 'dashed',
            },
        },
        // min: 0,
        // max: 250,
        // interval: 50,
        // alignTicks: true,
        // 坐标轴刻度标签的相关设置
        axisLabel: {
            // 格式化
            formatter: '{value} 万',
        },
    },
],
series: [
    {
        // 系列名字 用于 tooltip 显示，图例筛选
        name: dataSource[0]?.label,
        // 折线图
        type: 'line',
        // 颜色
        color: '#A74FFF',
        // 设置对应的 y轴 ，在多 y 轴时使用
        yAxisIndex:0,
        // 提示框组件
        tooltip: {
            // 格式化显示
            valueFormatter: (value: any) => {
                const text = Number(value);
                if (_.isNaN(text)) return '-';
                return `${text}万 `;
            },
        },
        // 点的图形
        symbol: 'circle',
        // 数据
        data: dataSource[0]?.data,
    },
    {
        name: dataSource[1]?.label,
        // 柱状图
        type: 'bar',
        // 设置对应的 y轴 ，在多 y 轴时使用
        yAxisIndex:1,
        // 堆叠
        stack: 'total',
        // 最大宽度
        barMaxWidth: 50,
        // 颜色
        color: '#0052FF',
        // 柱形颜色渐变
        color: new echarts.graphic.LinearGradient(
            0,//x 设置为1，则左端为终点 100%
            1,//y 设置为1，则上端为终点 100%
            0,//x2 设置为1，则右端为终点 100%
            0,//y2 设置为1，则下端为终点 100%
            [
                {
                    offset: 0,
                    color: '#83ABFF', // 0% 处的颜色
                },
                {
                    offset: 1,
                    color: '#002BFF', // 100% 处的颜色
                },
            ],
            false,
        ),
        tooltip: {
            valueFormatter: (value: any) => {
                const text = Number(value);
                if (_.isNaN(text)) return '-';
                return `${text}万 `;
            },
        },
        data: dataSource[1]?.data,
    },
    dataZoom: {
        // 滑动条的类型
        type: 'slider',
        // 是否显示滑动条
        show: true,
        // 距离底部的距离
        bottom: '3%',
        // 滑动条的高度
        height: 10,
        // 默认展示的开始百分比
        start: start > 0 ? start : 0,
        // 默认展示的结束百分比
        end: 100,
    },
],
```

# 饼图

```TypeScript
const colorList = useMemo(() => {
    return colorArr.reduce((pre: any, cur: string[]) => {
        pre.push(
            // 设置渐变
            new echarts.graphic.LinearGradient(
                0,
                1,
                0,
                0,
                [
                    {
                        offset: 0,
                        color: cur[0],
                    },
                    {
                        offset: 0.8,
                        color: cur[1],
                    },
                    {
                        offset: 1,
                        color: cur[1],
                    },
                ],
                false,
            ),
        );
        return pre;
    }, []);
}, [colorArr]);

{
    tooltip: {
        trigger: 'item',
        // position: 'bottom',
        // 设置提示框位置
        // 函数解决当鼠标在左侧 保证提示框在右侧；鼠标在上侧，保证提示框在下侧
        position: function(point, params, dom, rect, size) {
            // 鼠标坐标和提示框位置的参考坐标系是：以外层div的左上角那一点为原点，x轴向右，y轴向下
            // 提示框位置
            let x = 0; // x坐标位置
            let y = 0; // y坐标位置

            // 当前鼠标位置
            const pointX = point[0];
            const pointY = point[1];

            // 外层div大小
            const viewWidth = size.viewSize[0];
            const viewHeight = size.viewSize[1];

            // 提示框大小
            const boxWidth = size.contentSize[0];
            const boxHeight = size.contentSize[1];

            // boxWidth > pointX 说明鼠标左边放不下提示框
            if (pointX <= viewWidth / 2) {
                x = pointX + 10;
            } else {
                // 左边放的下
                x = pointX - boxWidth - 10;
            }

            // boxHeight > pointY 说明鼠标上边放不下提示框
            if (pointY <= viewHeight / 2) {
                y = pointY + 10;
            } else {
                // 上边放得下
                y = pointY - boxHeight - 10;
            }

            return [x, y];
        },
        // 格式化提示框展示 item.marker是对应颜色的圆点
        formatter: (item: any) => {
            let html = `<div style="width:160px;">
                    <div style="width:100%;overflow:hidden;">
                        <div style="float:left;">${item.marker}${item.name}个数</div>
                        <div style="float:right;">${item.value}个</div>
                    </div>
                    <div style="width:100%;overflow:hidden;">
                        <div style="float:left;">${item.marker}${item.name}金额</div>
                        <div style="float:right;">${item.data.amount}万</div>
                    </div>
                    <div style="width:100%;overflow:hidden;">
                        <div style="float:left;">${item.marker}${item.name}百分比</div>
                        <div style="float:right;">${item.percent}%</div>
                    </div>
                </div>`;
            return html;
        },
    },
    // 设置对应扇形的颜色
    color: colorList,
    legend: {
        // type: 'scroll',
        pageIconSize: 10,
        // 图例离下端的距离
        bottom: '0%',
        // 图形宽度
        itemWidth: 8,
        // 图形高度
        itemHeight: 8,
        // 图形字体样式
        textStyle: {
            fontSize: 11,
        },
    },
    grid: {
        left: '0%',
        right: '0%',
        bottom: '15%',
        width: '100%',
        height: '70%',
        containLabel: true,
    },
    series: [
        {
            // name: 'Access From',
            // 饼图
            type: 'pie',
            // 饼图大小
            radius: ['30%', '70%'],
            // 饼图圆心位置
            center: ['50%', '50%'],
            avoidLabelOverlap: false,
            // label: {
            //     show: false,
            //     position: 'center',
            // },
            // emphasis: {
            //     label: {
            //         show: true,
            //         fontSize: 18,
            //         // fontWeight: 'bold',
            //     },
            // },
            // labelLine: {
            //     show: false,
            // },
            itemStyle: {
                // //shadowBlur: 15,
                // shadowColor: '#3DDFDA',
                // shadowColor: [
                //      '#002BFF',
                //      '#A74FFF',
                //      '#2AABE7',
                //      '#3DDFDA',
                // ],
            },

            // data: data,
            data: [
                {
                    // 值
                    value:20,
                    // 对应的图例名称
                    name:"20",
                    // 扇形样式
                    itemStyle:{
                        // 阴影模糊 值越大 模糊范围越大 阴影越不清晰
                        shadowBlur: 15,
                        // 阴影颜色
                        shadowColor: "color",
                        // 阴影在 y 轴的位移
                        shadowOffsetY: 10,
                        // 透明度
                        opacity: 1,
                    };
                    // 设置饼图上展示的值
                    label:{
                        // 是否展示
                        show: true,
                        // 占比小于 5% 位置改为外置
                        position: obj.ratio > 5 ? 'inner' : 'outside',
                        // 格式化
                        formatter: `{c}`,
                        // 字体粗细
                        fontWeight: 'bold',
                        // 字体颜色
                        color: obj.ratio > 5 ? '#fff' : '#000',
                    };
                    // 引导线设置
                    labelLine = {
                        // 逐个进行判断并设置是否展示引导线，否则在原有饼图上切换数据时，引导线会残留，显示错误
                        show: obj.ratio > 5 ? false : true,
                    };
                }
            ],
        },
    ],
};
```