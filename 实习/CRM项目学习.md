# CRM 项目学习

## 个人感悟

### 如何快速定位所需要修改的代码

- 根据页面上一些特定的文字，使用 VSCode 的搜索
- 根据需求中发起的链接锁定

## 业务需求

### 选择器

```TypeScript
// 源码
import React, { useEffect, useState } from 'react';
import classname from 'classname';
import _ from 'lodash';
import './index.less';

interface Option {
    title: string;
    value: string | number;
}

export interface SelectorProps {
    label: string;
    defaultValue?: string | number;
    value?: string | number;
    options: Option[];
    // // 单选 or 多选
    // multiple?: boolean;
    onChange?: (selected: string | number) => void;
}

const Selector: React.FC<SelectorProps> = props => {
    const {
        label,
        options,
        // multiple: fasle,
        onChange = () => {},
        value,
        defaultValue = '',
    } = props;

    const currentValue = _.isNil(value) ? defaultValue : value;

    const [selected, setSelected] = useState<string | number>(currentValue);

    useEffect(() => {
        if (_.isNil(value)) return;
        setSelected(value);
    }, [value]);

    const handleSeleted = (value: string | number) => {
        setSelected(value);
        onChange(value);
    };
    return (
        <div className="composite-selector">
            <div className="composite-selector-label">{label}：</div>
            <div className="composite-selector-options">
                {options.map(option => (
                    <div
                        className={classname(
                            'composite-selector-options-item',
                            { active: option.value === selected },
                        )}
                        key={option.value}
                        onClick={() => handleSeleted(option.value)}
                    >
                        {option.title}
                    </div>
                ))}
            </div>
        </div>
    );
};

export default Selector;
// 使用
<Selector
    label="组织筛选"
    options={[
        { title: '展开到区域', value: 0 },
        { title: '展开到城市', value: 1 },
        { title: '展开到团队', value: 2 },
    ]}
    defaultValue={0}
    value={organize}
    onChange={selected => {
        if (typeof selected === 'number') {
            setOrganize(selected);
        }
    }}
/>
```

### Antd 组件粘贴图片

```CSS
import { useEventListener } from 'ahooks';
// 监听粘贴事件
useEventListener('paste', async event => {
    let items = event.clipboardData?.items;
    if (!items) return;
    // 这是一个类数组对象 不是数组，不能用map和forEach遍历,可以进行使用 Array.from() 转化
    for (let i = 0; i < items.length; i++) {
        const item = items[i];
        const pasteFile = item?.getAsFile();

        if (pasteFile && item.type.match(/^image\//i)) {
            // 获取文件对象
            const response = await uploadFile(pasteFile);
            const key = Date.now();
            const fileObject = {
                ...pasteFile,
                uid: key,
                originFileObj: pasteFile,
                percent: 100,
                key: key,
                response: response,
                status: response.status,
                thumbUrl: await getBase64(pasteFile),
            };

            setFileList([...fileList, fileObject]);
        }
    }
});
```

### 埋点

```TypeScript
// 埋点源码
import qs from 'querystring';
// import Cookie from 'tiny-cookie';
import { request } from './utility/request';
import { md5Hash } from './utility/md5';
import { vuuid } from './utility/v-uuid';
import { xor } from './utility/xor';
import { getFromPageParam } from './utility/v-utils';
import { jsonSize } from './utility/jsonSize';
import { composed } from './utility/scheduler';
import * as Cookie from 'tiny-cookie';

const pageSessionID = vuuid();

class Voyager {
    static url = 'https://track.mm.taou.com/v2/track?'; // 请求链接
    static eventList = []; // 事件列表
    static currentDurableEvents = []; // 当前持续事件列表
    static defaultParams = {
        app: 'maimai',
        lang:
            typeof navigator !== 'undefined' &&
            (navigator.language || navigator.userLanguage),
        qilu: 0,
        from_page: getFromPageParam(pageSessionID),
        src_page: getFromPageParam(pageSessionID),
        page: '', // 初始化进行处理
        event_source: 'web',
        mm_app_id: 0,
        duration: 0,
        old_vczh:
            (window.chrome ? !window.chrome.runtime : null) ||
            window.navigator.webdriver, // detect webdriver by qilu
    }; // 默认参数

    constructor(options) {
        let opts = Object.assign({}, { needPageView: true }, options); // 默认needPageView为true
        this.initDefaultParams();
        if (opts.needPageView === true) {
            // 构造函数配置选项
            this.pageViewProcess(opts.pageViewParams || {});
        }
        // 尝试重新发送失败的数据
        this.rensendFailedData();
    }
    /**
     * 处理持续的页面停留事件，对page_view进行单独统计
     * @param {Object} params 额外参数
     * @param {String} event_key 用于区分同样name的多个event, 如果事件不会并行，可以为空
     */
    pageViewProcess(params = {}) {
        this.callPageViewEvent('begin', params);

        // 有关事件的兼容性，参见https://docs.taou.com/pages/viewpage.action?pageId=20688675 兼容性说明
        // 苹果手持设备或者desktop都会执行unload事件
        if (
            /iPhone|iPad|iTouch/i.test(navigator.userAgent) ||
            !this.mobileAndTabletcheck()
        ) {
            window.addEventListener('unload', () => {
                this.callPageViewEvent('end', params);
            });
        }

        document.addEventListener('visibilitychange', () => {
            if (document.visibilityState === 'hidden') {
                this.callPageViewEvent('end', params);
            }

            // fires when app transitions from prerender, user returns to the app / tab.
            if (document.visibilityState === 'visible') {
                // 重新创建新的PageView持续事件
                this.callPageViewEvent('begin', params);
            }
        });
    }
    /**
     * 开始跟踪PageView
     * @param {String} type begin/end
     * @param {Object} params 额外参数
     */
    callPageViewEvent(type, params) {
        // 站内和站外分别进行调用客户端方法和js web sdk
        if (type === 'begin') {
            let durableEvent = this.initDurableEvent('page_view', params);
            // 因为站内已经有客户端的page_view的begin打点，所以去掉站内的客户端的调用
            if (!this.isInApp()) {
                this.track(durableEvent.events[0]);
            }
        } else if (type === 'end') {
            let endEvent = this.getCurrentPageViewEvent();
            // 因为站内已经有客户端的page_view的end打点，所以去掉站内的客户端的调用
            if (endEvent.length) {
                if (!this.isInApp()) {
                    this.track(endEvent[0].events[1]);
                }
            } else {
                throw new Error('get current page view event failed!');
            }
        }
    }
    /**
     * 初始化PageView持续事件(begin/end)配置项
     * @param {String} event_name 事件名称
     * @param {Object} params 额外参数
     * @return {Object} result 返回配置项信息
     */
    initDurableEvent(event_name, params) {
        let event_id = vuuid(); // 页面开始的event_id
        // 页面开始
        let beginOpts = {
            event_id: event_id,
            event_name: event_name,
            event_type: 'begin',
            params,
        };

        let endOpts = Object.assign({}, beginOpts, {
            begin_event_id: event_id,
            event_id: vuuid(),
            event_type: 'end',
        });

        let durableEvent = {
            event_id,
            event_name,
            events: [beginOpts, endOpts],
        };

        Voyager.currentDurableEvents.push(durableEvent); // 将持续事件压入队列中
        return durableEvent;
    }
    /**
     * 获取最新的PageView事件
     */
    getCurrentPageViewEvent() {
        return Voyager.currentDurableEvents
            .reverse()
            .filter(item => item.event_name === 'page_view');
    }

    /**
     * 净化参数，过滤掉保留参数。因为 ios debug 包下传保留字段会 crash 掉
     */
    purifierParams(params) {
        const reserved_params = [
            'event_name',
            'app',
            'event_type',
            'event_id',
            'event_key',
            'begin_event_id',
            'duration',
            'timestamp',
            'event_source',
            'uid',
            'feature',
            'platform',
            'imei',
            'idfa',
            'udid',
            'android_id',
            'gaid',
            'mac_address',
            'brand',
            'model',
            'os_version',
            'network',
            'carrier',
            'qilu',
            'lang',
            'country',
            'battery',
            'app_version_name',
            'app_version_code',
            'edition',
            'mm_app_id',
            'install_id',
            'push_enabled',
            'channel',
            'media_source',
            'agency',
            'store',
            'campaign',
            'launch_id',
            'raw_session_id',
            'time_since_raw_session_begin',
            'session_id',
            'adcode',
            'page',
            'from_page',
            'src_page',
            'browser_id',
            'service',
            'process_id',
            'service_ip',
            'params',
            'sdk_extra',
        ];
        let newParams = {};
        for (let key in params) {
            if (!reserved_params.includes(key)) {
                newParams[key] = params[key];
            } else {
                console.warn(
                    `Parameter ${key} is a reserved field, please change the field`,
                );
            }
        }
        return newParams;
    }
    /**
     * 追踪事件 public方法
     * @param {String} eventName 事件名称
     * @param {String} eventKey 持续事件的key, 用来区分用一个name的不同事件. 可能是feed_id之类的
     *                          ⚠️️️⚠️⚠️ 在event_type为single时，字段不会生效
     */
    trackEvent(event_name, event_key = '', params = {}) {
        if (typeof event_name !== 'string' || event_name === '') {
            throw new Error(
                'eventName is neccessary while call trackEvent fn ',
            );
        }

        if (
            (typeof params.event_type === 'undefined' ||
                params.event_type === 'single') &&
            event_key !== ''
        ) {
            console.warn(
                '%cparameter event_key will not work when event_type=single!!!',
                'background: #222; font-size: 24px; color: yellow',
            );
        }

        let _params = this.purifierParams(params);

        let options = {
            event_name,
            event_key,
            params: _params,
        }; // 初始化选项

        // let { MaiMai_Native } = window;
        // // 站内和站外分别进行调用客户端方法和js web sdk
        // if (this.isInApp()) {
        //   try {
        //     MaiMai_Native &&
        //       MaiMai_Native.log_v2_track_event &&
        //       MaiMai_Native.log_v2_track_event(event_name, JSON.stringify(_params));
        //   } catch (err) {
        //     throw new Error(err);
        //   }
        // } else {
        return this.track(options);
    }
    /**
     * 追踪事件 private方法
     * @param {Object} options 打点选项
     */
    async track(options) {
        if (this.isInApp()) {
            return;
        }

        let opts = Object.assign(
            {},
            options,
            this.getAfData(),
            Voyager.defaultParams,
        ); // 每次事件的配置选项, 合并af数据
        opts.event_id = opts.event_id || vuuid();
        opts.timestamp = new Date().getTime();
        opts.event_type = opts.event_type || 'single'; // event_type为single/begin/end
        if (
            typeof opts.params !== 'undefined' &&
            typeof opts.params === 'object'
        ) {
            // 修复params参数问题，不需要加额外参数，把opts.params和opts进行合并
            try {
                opts = Object.assign({}, opts.params, opts);
                delete opts.params;
            } catch (err) {
                throw new Error(
                    'stringify opts.params ',
                    opts.params,
                    ' failed',
                    err,
                );
            }
        }
        // 计算时长
        if (opts.event_type === 'end') {
            let currentBeiginEvent = Voyager.eventList.filter(
                item => item.event_id === opts.begin_event_id,
            );
            opts.duration = opts.timestamp - currentBeiginEvent[0].timestamp;
        }
        opts.browser_id = this.getBrowserId(); // 浏览器id
        opts.uid = this.getUid(); // 用户u
        // opts.country = this.getCountry(); // 暂时去掉

        if (!this.checkBodySize(opts)) {
            throw new Error('request body is larger than 8M!');
        }
        // wmx:将数据放到
        Voyager.eventList.push(opts);
        composed(opts, async (err, res) => {
            if (err) {
                throw new Error(err);
            }
            await this.send({ events: res });
        });

        return null;
    }
    /**
     *  持续事件开始
     *  @param {String} eventName 持续事件的名称
     *  @param {String} eventKey 持续事件的key, 用来区分用一个name的不同事件. 可能是feed_id之类的
     *  @param {Object} params 事件的自定义参数, json {"key1": "value1","key2": "value2"}
     *  @param {Boolean} endWithRawSession 如果为true，则在rawSession结束时，自动结束该Event
     */
    beginTrackEvent(event_name, event_key, params = {}, end_with_raw_session) {
        // 站内和站外分别进行调用客户端方法和js web sdk
        // if (this.isInApp()) {
        //   MaiMai_Native &&
        //     MaiMai_Native.log_v2_begin_event &&
        //     MaiMai_Native.log_v2_begin_event(
        //       event_name,
        //       event_key,
        //       JSON.stringify(params),
        //       end_with_raw_session
        //     );
        // } else {
        let options = {
            event_name,
            event_key,
            params,
            end_with_raw_session,
        };
        return this.track(options);
        // }
    }
    /**
     *  持续事件结束
     *  @param {String} eventName 持续事件的名称
     *  @param {String} eventKey 持续事件的key, 用来区分用一个name的不同事件. 可能是feed_id之类的
     *  @param {Object} params 事件的自定义参数, json {"key1": "value1","key2": "value2"}
     */
    endTrackEvent(event_name, event_key, params = {}) {
        // 站内和站外分别进行调用客户端方法和js web sdk
        // if (this.isInApp()) {
        //   MaiMai_Native &&
        //     MaiMai_Native.log_v2_end_event(
        //       event_name,
        //       event_key,
        //       JSON.stringify(params)
        //     );
        // } else {
        let options = {
            event_name,
            event_key,
            params,
        };
        return this.track(options);
        // }
    }
    // 检查发送数据SIZE，是否小于8M
    checkBodySize(json) {
        return jsonSize(json) <= 8 * 1024 * 1024;
    }
    // 获取参数k
    getK() {
        return this.getUid() || this.getBrowserId();
    }
    // 获取u
    getUid() {
        try {
            // let {
            //   share_data: {
            //     auth_info: { u },
            //   },
            // } = window;
            // return u === -1 ? '' : u.toString();
            return Cookie.get('crmuid') || '';
        } catch (err) {
            return '';
        }
    }
    // 获取浏览器指纹
    getBrowserId() {
        return Cookie.get('_buuid') || vuuid();
    }
    // 获取af数据，基本针对增长的投放
    getAfData() {
        let query = qs.parse(window.location.search.substr(1));
        let { regfr } = query;
        if (!regfr) return {};
        let arry = regfr.split('_');
        let result = /sem_(baidu|sogou|shenm)/i.exec(regfr);
        if (result && arry.length > 4) {
            let media_source = 'smsearch_int'; // 这几个搜索渠道都是怎么起的名字，醉了
            if (result[1] === 'baidu') {
                media_source = 'baidusearch_int';
            } else if (result[1] === 'sogou') {
                media_source = 'sogousearch';
            }
            let { length } = arry;
            return {
                agency: arry[length - 4],
                campaign: arry[arry.length - 2],
                media_source,
            };
        }
        return {};
    }
    isInApp() {
        return navigator.userAgent.match(/MaiMai/i);
    }
    // 获取国家信息
    async getCountry() {
        try {
            let { country } = await request('https://ipinfo.io', {
                method: 'GET',
            });
            return country;
        } catch (err) {
            throw new Error(
                'cannot get country info throughout ipinfo.io',
                err,
            );
        }
    }
    async send(opts) {
        if (!Array.isArray(opts.events) || !opts.events.length) return;

        let body = xor(JSON.stringify(opts), '42');
        let k = this.getK();
        let s = md5Hash(body, k);
        let q = this.parseWebSDKConfig();

        let result = await request(
            Voyager.url + `app=maimai&${q}k=${k}&s=${s}`,
            {
                body,
            },
        );

        // 在没有进入到发送队列的请求进行额外处理
        if (result === false) {
            let img = new Image();
            img.src = `/sdk/voyager/send?app=maimai&${q}k=${k}&s=${s}&body=${encodeURIComponent(
                body,
            )}`;
            img.onerror = () => {
                let cacheData = localStorage.getItem('sendFailedEvents');
                // 请求失败时，将数据存储到本地，下次进行重新上传, 存入数组
                if (cacheData) {
                    try {
                        let newData = JSON.parse(cacheData);
                        newData.events = [...newData.events, ...opts.events]; // 两个数组进行合并
                        localStorage.setItem(
                            'sendFailedEvents',
                            JSON.stringify(newData),
                        );
                    } catch (err) {
                        throw new Error('save cacheData failed!');
                    }
                } else {
                    localStorage.setItem(
                        'sendFailedEvents',
                        JSON.stringify(opts),
                    );
                }
            };
        }
        return result;
    }
    /**
     * 重发失败数据
     */
    async rensendFailedData() {
        const failedData = localStorage.getItem('sendFailedEvents');
        let allData = {};
        try {
            allData = JSON.parse(failedData);
        } catch (err) {
            allData = {};
        }

        if (allData?.events?.length) {
            const { events } = allData;
            try {
                // send every start 5 requests to avoid too large request data
                this.send({ events: events.splice(0, 5) });
                // 只重发一次，随机立马删除缓存数据
                // do set action again if any data exists after splicing data
                if (events.length) {
                    localStorage.setItem(
                        'sendFailedEvents',
                        JSON.stringify({ events }),
                    );
                } else {
                    localStorage.removeItem('sendFailedEvents');
                }
            } catch (err) {
                throw new Error(
                    'parse localStorage sendFailedEvents failed ' + err,
                );
            }
        }
    }
    // 解析websdkconfig参数
    parseWebSDKConfig() {
        let queryString = '';
        let query = qs.parse(
            decodeURIComponent(window.location.search.substr(1)),
        );
        if (query && query.web_sdk_config) {
            try {
                let queryObj = JSON.parse(query.web_sdk_config);
                if (typeof queryObj !== 'string') {
                    Object.entries(queryObj).forEach(([k, v]) => {
                        queryString += `${k}=${v}&`;
                    });
                }
            } catch (err) {
                throw new Error('failed to parse query.web_sdk_config', err);
            }
        }
        return queryString;
    }
    /**
     * 初始化通用/默认参数
     */
    initDefaultParams() {
        Voyager.defaultParams = Object.assign({}, Voyager.defaultParams, {
            page: this.getPageParams(),
            raw_session_id: vuuid(),
            page_session_id: pageSessionID,
        });
    }
    /**
     * 获取通用page参数，仅用于浏览器环境
     * 其它场景下，继承的子类可以覆盖该方法
     */
    getPageParams() {
        return typeof window !== 'undefined' ? window.location.href : '';
    }
    /**
     * 检测是手机或者平板电脑
     * from detectmobilebrowsers.com
     */
    mobileAndTabletcheck() {
        let check = false;
        (function(a) {
            if (
                /(android|bb\d+|meego).+mobile|avantgo|bada\/|blackberry|blazer|compal|elaine|fennec|hiptop|iemobile|ip(hone|od)|iris|kindle|lge |maemo|midp|mmp|mobile.+firefox|netfront|opera m(ob|in)i|palm( os)?|phone|p(ixi|re)\/|plucker|pocket|psp|series(4|6)0|symbian|treo|up\.(browser|link)|vodafone|wap|windows ce|xda|xiino|android|ipad|playbook|silk/i.test(
                    a,
                ) ||
                /1207|6310|6590|3gso|4thp|50[1-6]i|770s|802s|a wa|abac|ac(er|oo|s\-)|ai(ko|rn)|al(av|ca|co)|amoi|an(ex|ny|yw)|aptu|ar(ch|go)|as(te|us)|attw|au(di|\-m|r |s )|avan|be(ck|ll|nq)|bi(lb|rd)|bl(ac|az)|br(e|v)w|bumb|bw\-(n|u)|c55\/|capi|ccwa|cdm\-|cell|chtm|cldc|cmd\-|co(mp|nd)|craw|da(it|ll|ng)|dbte|dc\-s|devi|dica|dmob|do(c|p)o|ds(12|\-d)|el(49|ai)|em(l2|ul)|er(ic|k0)|esl8|ez([4-7]0|os|wa|ze)|fetc|fly(\-|_)|g1 u|g560|gene|gf\-5|g\-mo|go(\.w|od)|gr(ad|un)|haie|hcit|hd\-(m|p|t)|hei\-|hi(pt|ta)|hp( i|ip)|hs\-c|ht(c(\-| |_|a|g|p|s|t)|tp)|hu(aw|tc)|i\-(20|go|ma)|i230|iac( |\-|\/)|ibro|idea|ig01|ikom|im1k|inno|ipaq|iris|ja(t|v)a|jbro|jemu|jigs|kddi|keji|kgt( |\/)|klon|kpt |kwc\-|kyo(c|k)|le(no|xi)|lg( g|\/(k|l|u)|50|54|\-[a-w])|libw|lynx|m1\-w|m3ga|m50\/|ma(te|ui|xo)|mc(01|21|ca)|m\-cr|me(rc|ri)|mi(o8|oa|ts)|mmef|mo(01|02|bi|de|do|t(\-| |o|v)|zz)|mt(50|p1|v )|mwbp|mywa|n10[0-2]|n20[2-3]|n30(0|2)|n50(0|2|5)|n7(0(0|1)|10)|ne((c|m)\-|on|tf|wf|wg|wt)|nok(6|i)|nzph|o2im|op(ti|wv)|oran|owg1|p800|pan(a|d|t)|pdxg|pg(13|\-([1-8]|c))|phil|pire|pl(ay|uc)|pn\-2|po(ck|rt|se)|prox|psio|pt\-g|qa\-a|qc(07|12|21|32|60|\-[2-7]|i\-)|qtek|r380|r600|raks|rim9|ro(ve|zo)|s55\/|sa(ge|ma|mm|ms|ny|va)|sc(01|h\-|oo|p\-)|sdk\/|se(c(\-|0|1)|47|mc|nd|ri)|sgh\-|shar|sie(\-|m)|sk\-0|sl(45|id)|sm(al|ar|b3|it|t5)|so(ft|ny)|sp(01|h\-|v\-|v )|sy(01|mb)|t2(18|50)|t6(00|10|18)|ta(gt|lk)|tcl\-|tdg\-|tel(i|m)|tim\-|t\-mo|to(pl|sh)|ts(70|m\-|m3|m5)|tx\-9|up(\.b|g1|si)|utst|v400|v750|veri|vi(rg|te)|vk(40|5[0-3]|\-v)|vm40|voda|vulc|vx(52|53|60|61|70|80|81|83|85|98)|w3c(\-| )|webc|whit|wi(g |nc|nw)|wmlb|wonu|x700|yas\-|your|zeto|zte\-/i.test(
                    a.substr(0, 4),
                )
            )
                check = true;
        })(navigator.userAgent || navigator.vendor || window.opera);
        return check;
    }
}

export default Voyager;
/**
 * voyager 打点
 */

import Voyager from './voyager/index';

let voyagerInstance = null;

function track(trackName, trackParams) {
    if (!voyagerInstance) {
        voyagerInstance = new Voyager({});
    }

    return voyagerInstance.trackEvent(trackName, '', trackParams);
}

export { track as default };
// 使用
import voyagerTrack from '@/utils/voyager_track';
voyagerTrack('crm_my_customer_ff_click', {
    ffkey: ffItem?.fastFilterKey,
});
```

### 权限控制

```
const authView = useCallback(
          (name: string) => {
              const isShow =
                  name === '全国'
                      ? data?.viewOrgType === '6'
                      : !!data?.dataBoardOrgList?.find(org => {
                            return name.split(',').includes(org.orgName || '');
                        });
              return isShow;
          },
          [data],
    );

const authViewWithComponent = (name: string) => {
	const isShow = authView(name);
	return (component: React.ReactNode) => {
		return isShow ? component : null;
	};
};
```



## 浏览器

### 获取页面打开方式

```TypeScript
performance.getEntriesByType('navigation')
// reload 点击刷新 window.performance.navigation.type=1
// navigate 通过点击链接或输入链接 window.performance.navigation.type=0
// back_forword 后退
//
document.referrer  // 当前一页的上一页
```

## CSS 样式

### 问题：当屏幕过小，页面高度过高，无法查看页面以下的部分

```CSS
{
    /* 最小高度为500px 父元素比500小就出现滚动条 */
    overflow: scroll;
    min-height: 500px;
    /*或*/
    /* 最大高度为500px 子元素总高度比500大就出现滚动条 */
    overflow: scroll;
    max-height: 500px;
}
```

### 合并边框

```TypeScript
border-collapse: collapse;/* 合并相邻边框*/
```

### 在宽度未知的情况下，实现一个正方形

```html
<div class="square3">
  <img src="images/square.jpg" class="squareimg2"/>
</div>
```

```css
.square3{
  	width:33%
    padding-bottom:33%;
  	/* width 是百分之几，padding-bottom 就写百分之几 */
    height: 0;
    position: relative;
} 
.squareimg2{
    position: absolute;
    width: 100%;
    height: 100%; 
  	/* 图片居中显示 */
    -o-object-fit: cover;
    object-fit: cover;
}
```

## React

### 问题：在字符串中输入连续的多个空格，页面只显示一个

解决办法：使用` <pre></pre>` 标签

```TypeScript
const str="12 34   123"
<pre>{str}</pre>
```

### 问题：useEffect 在组件挂载时不执行

解决方法：使用 ahooks 中的 useUpdateEffect

```javascript
import { useUpdateEffect } from 'ahooks';
const fn=()=>{
  useUpdateEffect(() => {
      getHomeNewPerformanceOptData();
  }, [homeNewSelectedData, homeNewFilterData]);
  return <div>APP</div>
}
```

## TypeScript 语法

[TypeScript 学习](https://maimai.feishu.cn/docx/Dm88drJZOoK0fexe1RfcktGQnKg)

当参数已限定类型，而传参需要多加一个字段时，可使用 & 操作符，解决兼容问题，且不污染原来组件

```TypeScript
function OpportunityTable<T>({
    dataSource,
    loading,
    // customerType字段本不在 TableCardProps 中，使用 & 进行兼容
    customerType = 0,
    ...rest
}: TableCardProps<T> & { customerType?: number }){

}
```

对于一个有可能为 undefined or null 的值，可能需要在后面使用 || 来给一个默认值

```TypeScript
// t 是一个字符串
const [num = '-', range = '', rate = ''] = t?.split('_') || []
```

自定义类型

```TypeScript
declare type Nullable<T> = null | T;
```

ReturnType

源码

```TypeScript
type ReturnType<T extends (...args: any) => any> = T extends (...args: any) => infer R ? R : any;
```

## Less 语法

```CSS
/*
等于
.p{
    xxx
}
类名为 p 的元素的后代中类名为 s 的元素
.p .s{
    yyy
}
*/
.p{
    xxx
    .s{
        yyy
    }
}

/*
等于
.p{
    xxx
}
类名中同时有 p 和 s 的元素
.p .s{
    yyy
}
*/
.p{
    xxx
    &.s{
        yyy
    }
}
```

## 第三方库

### lodash

[lodash 官网](https://www.lodashjs.com/)

[Lodash 常用函数记录](https://maimai.feishu.cn/docx/CH9edy0dTorp8kx5PnBcABVfnke)

### ahooks

[官网](https://ahooks.js.org/zh-CN/hooks/use-debounce)

[ahooks 学习记录](https://maimai.feishu.cn/docx/Tk8vdE0JnoJd1sxhV6scIGRdnSe)

### SWR

[SWR github 链接](https://github.com/vercel/swr)

[SWR 学习记录](https://maimai.feishu.cn/docx/S85ddodnmod7dZxwrincA7tRnWe)

### Antd

[Antd3.x 官网](https://3x.ant.design/docs/react/introduce-cn)

[Antd 学习记录](https://maimai.feishu.cn/docx/KmxddilPzozLhaxNUUdcScXWnRf)

### ECharts

https://echarts.apache.org/zh/option.html#title

[ECharts 学习记录](https://maimai.feishu.cn/docx/MN7ZdSZRhoWxFpxjIoIcuhWLnMc)

### Next.js

[官网](https://www.nextjs.cn/)

[Next.js 学习记录](https://maimai.feishu.cn/docx/UDsJdjXzjoM4YCxix8Mc0H25nAd)

### Mobx

[mobx 中文文档](https://cn.mobx.js.org/)

**Mobx 是简单、可扩展的状态管理库**。是经过战斗洗礼的库，通过透明的函数响应式编程使状态管理变得更加简单且可扩展。（类似 Redux ）

#### 问题：

1. 为什么使用 mobx 而不是 redux

答：

1. mobx 和 Redux 的区别

**编程思想不同**

Mobx 遵循面向对象编程思想，redux 遵循函数式编程思想

**store 的区别**

redux 的 store 集合了一整个应用的状态，mobx 按模块将应用状态划分，在多个独立的 store 中管理

**存储数据的形式不一样**

Redux 以 js 原生对象形式存储数据，mobx 可以自定义一个类创建一个对象

1. Redux 需要手动追踪所有状态对象的变更；
2. Mobx 中可以监听可观察对象，当其变更时将自动触发监听；

**操作对象的方式不一样**

redux 不能直接操作状态对象，而是需要在原有的对象基础上返回一个新的状态对象。mobx 可以直接使用新值在原来状态对象上进行更新

**代码对比**

在 Redux 应用中，我们首先需要配置，创建 store，并使用 redux-thunk 或 redux-saga 中间件以支持异步 action，然后使用 Provider 将 store 注入应用：

```JavaScript
// src/store.jsimport { applyMiddleware, createStore } from "redux";
import createSagaMiddleware from 'redux-saga'import React from 'react';
import { Provider } from 'react-redux';
import { BrowserRouter } from 'react-router-dom';
import { composeWithDevTools } from 'redux-devtools-extension';
import rootReducer from "./reducers";
import App from './containers/App/';

const sagaMiddleware = createSagaMiddleware()
const middleware = composeWithDevTools(applyMiddleware(sagaMiddleware));

export default createStore(rootReducer, middleware);

// src/index.js
…
ReactDOM.render(
  <BrowserRouter><Provider store={store}><App /></Provider></BrowserRouter>,
  document.getElementById('app')
);
```

Mobx 应用则可以直接将所有 store 注入应用：

```JavaScript
import React from 'react';
import { render } from 'react-dom';
import { Provider } from 'mobx-react';
import { BrowserRouter } from 'react-router-dom';
import { useStrict } from 'mobx';
import App from './containers/App/';
import * as stores from './flux/index';

// set strict mode for mobx// must change store through actionuseStrict(true);

render(
  <Provider {...stores}><BrowserRouter><App /></BrowserRouter></Provider>,
  document.getElementById('app')
);

复制代码
```

**Props 的注入不同**

- Redux

```JavaScript
// src/containers/Company.js
…
class CompanyContainer extends Component {
  componentDidMount () {
    this.props.loadData({});
  }
  render () {
    return <Companyinfos={this.props.infos}loading={this.props.loading}
    />
  }
}
…

// function for injecting state into propsconst mapStateToProps = (state) => {
  return {
    infos: state.companyStore.infos,
    loading: state.companyStore.loading
  }
}

const mapDispatchToProps = dispatch => {
  return bindActionCreators({
      loadData: loadData
  }, dispatch);
}

// injecting both state and actions into propsexport default connect(mapStateToProps, { loadData })(CompanyContainer);

复制代码
```

- Mobx

```JavaScript
@inject('companyStore')
@observer
class CompanyContainer extends Component {
  componentDidMount () {
    this.props.companyStore.loadData({});
  }
  render () {
    const { infos, loading } = this.props.companyStore;
    return <Companyinfos={infos}loading={loading}
    />
  }
}
```

### Sentry

sentry 是一个基于 Django 构建的现代化的实时事件日志监控、记录和聚合平台,主要用于如何快速的发现故障。支持几乎所有主流开发语言和平台,并提供了现代化 UI,它专门用于监视错误和提取执行适当的事后操作所需的所有信息,而无需使用标准用户反馈循环的任何麻烦。

## crm 项目文档

[CRM 项目有关于 Form 表单组件的文档](https://maimai.feishu.cn/wiki/wikcnjG6Uv8WigpV1KxYYKvjqqh)

## git 使用

### 同步远程代码

`git fetch && git reset --hard origin/远程分支名` 同步远程代码

### master 分支合并本地分支

`git checkout master` 切换到 master 分支

`git merge 分支名` 合并某一分支

`git push` 推送代码

### 本地某一分支同步远端分支

`git checkout master` 切换到 master 分支

`git fetch && git reset --hard origin/远程分支名` 同步远程代码

`git checkout feat-xxx` 切换到本地 feat-xxx 分支

`git rebase master` 同步 master 分支
