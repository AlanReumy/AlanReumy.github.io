# 前端监控SDK简单实现

## 前端监控目标

### 稳定性

对错误进行监控：

1. js 错误
2. 资源异常
3. 接口错误
4. 白屏

### 用户体验

提高用户体验：

1. 加载时间：各个阶段的加载时间
2. TTFB(time to first byte)：浏览器发起第一个请求到数据返回第一个字节的时间
3. FP(first print)：首次绘制时间，第一个像素点绘制到屏幕的时刻
4. FCP(first content print)：首次内容绘制，浏览器将第一个 DOM 渲染到屏幕的时间
5. FMP(first meaningful print)：首次有意义绘制，是页面可用性的量度标准
6. FID(first input delay)：首次输入延迟，用户首次与页面交互到页面响应的时间
7. 卡顿：超过 50ms 的长任务

### 业务

1. PV(page view)：页面浏览量
2. UV：访问某个站点的不同 IP 地址的人数
3. 页面停留时间

## 前端监控流程

- 数据埋点
- 数据上报
- 数据格式建模
- 数据传输
- 数据统计
- 数据可视化 | 报告和报警

## 如何实现一个简单的前端监控 SDK

```ts
import { DefaultOptions, Options, TrackerVersion } from "../types/index";
import { createHistoryEvent } from "../utils/pv";

// 鼠标事件
const MouseEventList = [
  "click",
  "dbClick",
  "contextmenu",
  "mousedown",
  "mouseup",
  "mouseenter",
  "mouseout",
  "mouseover",
];
class Tracker {
  public data: Options;
  constructor(options: Options) {
    this.data = Object.assign(this.initDef(), options);
    this.installTracker();
  }

  // 默认初始化
  private initDef(): DefaultOptions {
    window.history["pushState"] = createHistoryEvent("pushState");
    window.history["replaceState"] = createHistoryEvent("replaceState");
    // 默认配置
    return <DefaultOptions>{
      historyTracker: false,
      hashTracker: false,
      domTracker: false,
      jsError: false,
      sdkVersion: TrackerVersion.version,
    };
  }

  // 手动上报
  public sendTracker<T>(data: T) {
    this.reportTracker(data);
  }

  private captureEvents<T>(
    mouseEventList: string[],
    targetKey: string,
    data?: T
  ) {
    mouseEventList.forEach((event) => {
      window.addEventListener(event, () => {
        console.log("got it");
        this.reportTracker({
          event,
          targetKey,
          data,
        });
      });
    });
  }

  private installTracker() {
    // 对history事件的监控
    if (this.data.historyTracker) {
      this.captureEvents(
        ["pushState", "popState", "replaceState"],
        "history-pv"
      );
    }
    // 对hashChange的监控
    if (this.data.hashTracker) {
      this.captureEvents(["hashChange"], "hash-pv");
    }
    // 对dom元素的监控
    if (this.data.domTracker) {
      this.tragetKeyReport();
    }
    // 对js错误的监控
    if (this.data.jsError) {
      this.jsError();
    }
  }

  // 将登录用户的id进行携带, 如果没有登录，可以在本地生成一个uuid，存储到本地缓存中
  public setUserId<T extends DefaultOptions["uuid"]>(uuid: T) {
    this.data.uuid = uuid;
  }

  public setExtra<T extends DefaultOptions["extra"]>(extra: T) {
    this.data.extra = extra;
  }

  // 使用blob,利用navigator.sendBeacon将统计数据发送到 Web 服务器
  private reportTracker<T>(data: T) {
    const params = Object.assign(this.data, data, {
      time: new Date().getTime(),
    });
    let headers = {
      type: "application/x-www-form-unlencoded",
    };
    let blob = new Blob([JSON.stringify(params)], headers);
    navigator.sendBeacon(this.data.requestUrl, blob);
  }

  // 对dom元素监控
  private tragetKeyReport() {
    MouseEventList.forEach((ev) => {
      window.addEventListener(ev, (e) => {
        const target = e.target as HTMLElement;
        const targetKey = target.getAttribute("target-key");
        if (targetKey) {
          this.reportTracker({
            event: ev,
            targetKey,
          });
        }
      });
    });
  }

  private jsError() {
    this.errorEvent();
    this.promiseReject();
  }

  private errorEvent() {
    window.addEventListener("error", (event) => {
      this.reportTracker({
        event: "error",
        targetKey: "message",
        message: event.message,
      });
    });
  }

  private promiseReject() {
    window.addEventListener("unhandledrejection", (event) => {
      event.promise.catch((error) => {
        this.reportTracker({
          event: "promise",
          targetKey: "message",
          message: error,
        });
      });
    });
  }
}

export default Tracker;
```
