# aHooks 学习记录

# useRequest 使用

```TypeScript
// 不带参数
export const queryPreSolarization: () => Promise<any> = async () => {
    const [d, e] = await defaultAxios.put(
        '/crm/data/query/predict_solarization',
        {},
    );
    if (d === null || d.data === null || e) {
        return null;
    }
    return d.data;
};

const { data: predictData, loading: predictLoading } = useRequest(
    queryPreSolarization,
);

// 带参数
export const queryOppSolarization: (
    customerType: number,
) => Promise<any> = async customerType => {
    const [d, e] = await defaultAxios.put(
        '/crm/data/query/opportunity_solarization',
        {
            customerType,
        },
        { timeout: 100000 }, //可能要删
    );
    if (d === null || d.data === null || e) {
        if (e.message.includes('timeout')) {
            message.error('请求超时，请刷新页面');
        }
        return null;
    }
    return d.data;
};

const { data: opportunityData, loading: opportunityLoading } = useRequest(
    () => queryOppSolarization(customerType),
    {
        // 当 customerType 发生改变时，会重新发起请求，把值赋给data: opportunityData
        refreshDeps: [customerType],
    },
);
```