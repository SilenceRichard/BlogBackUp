---
title: react项目的ts迁移
date: 2020-06-10 17:47:25
tags:
  - 工作
---

> 与 TypeScript 结缘有一年的时间了， 从刚接触时的陌生到现在用 ts 写前端代码的真香，踩过一些坑，走过一些弯路。这篇文章主要阐述我在顺丰项目组中对 js + react 项目做 ts 化迁移的过程

---

## 工程整体迁移

> 参考文章: [React 项目从 Javascript 到 Typescript 的迁移经验总结](https://segmentfault.com/a/1190000019075274?utm_source=tag-newest)

### 项目配置

详细配置过程及博主所遇的坑在文章中都有描述，而且挺详尽的，以下贴出我在项目中的配置

tsconfig.json

```json
{
  "compilerOptions": {
    "declaration": true, // 是否自动创建类型声明文件
    "baseUrl": ".", // 工作根目录
    "paths": {
      // 指定模块的路径，和baseUrl有关联，和webpack中resolve.alias配置一样
      "@/*": ["src/*"]
    },
    "rootDirs": ["include"],
    "outDir": "dist",
    "module": "esnext",
    "target": "es5",
    "lib": ["es6", "dom"],
    "sourceMap": true,
    "jsx": "react", // 提示不能使用jsx语法时的设置
    "moduleResolution": "node",
    "rootDir": ".",
    "forceConsistentCasingInFileNames": true,
    "noImplicitReturns": true,
    "noImplicitThis": true,
    "noImplicitAny": true,
    "importHelpers": true,
    "strictNullChecks": true,
    "esModuleInterop": true,
    "noUnusedLocals": true
  },
  "include": ["src/**/*"], // TS编译器会编译项目目录下所有的.ts、.tsx、.d.ts文件
  "exclude": [
    "node_modules",
    "build",
    "dist",
    "scripts",
    "acceptance-tests",
    "webpack",
    "jest",
    "src/setupTests.ts",
    "*.js"
  ]
}
```

目录

src/typings 

  用来存放全局类型
  1. 需要声明的模块文件，如.png, .scss等，放入index.d.ts 
  2. 未重构或难以重构的js文件，放入jslibs.d.ts
  3. 全局命名空间，一些需要但没有类型定义的模块，自定义文件名称，如在此项目中，定义高德地图声明文件，方便ts类型检查

```
├── typings
  ├── AMap.d.ts
  ├── index.d.ts
  └── jslibs.d.ts
```

services/schemas

  用来存放接口相关类型的纲要
```
├── services
│   ├── line
│   ├── pro
│   │   └── pro1.6
│   └── schemas
│       ├── line
│       └── pro
│           └── pro1.6
```

### 项目踩坑：

#### 1. AntD 样式文件丢失

当我们把项目启动起来之后，某些同学的页面可能会出现样式丢失的情况

原因： 用来按需加载组件样式文件的 babel 插件 **babel-plugin-import** 丢失

解决方案:

```
yarn add ts-import-plugin -D
```

webpack.config.js

```js
const tsImportPluginFactory = require("ts-import-plugin");
module.exports = {
  //省略部分代码...
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        loader: "ts-loader",
        options: {
          transpileOnly: true, //（可选） 在大型项目中设置为true,关闭ts语意检查，减少编译时间
          getCustomTransformers: () => ({
            before: [
              tsImportPluginFactory({
                libraryDirectory: "es",
                libraryName: "antd",
                style: true,
              }),
            ],
          }),
        },
      },
    ],
  },
  //省略部分代码...
};
```

#### 2. 引入依赖报错

> Could not find a declaration file for module '\*\*'

找不到依赖模块的声明文件，需要我们安装声明文件

```
yarn add @types/** -D
```

#### 3. 引入 js 文件飘红

针对在 ts 文件中引入的未重构的 js 模块，在项目中的 jslibs.d.ts（名称可自定义）中添加声明

示例：

```js
declare module '@/hooks';
declare module '@/utils/loginUtils';
declare module '@/utils/functions';
declare module '@/App.redux';
// pro1.6 未重构部分
declare module '@/pages/pro/utils/index';
declare module '@/pages/pro/PositionChoose/Result/components/visual/MapView';
declare module '@/pages/plan3/Modify/components/map/MapView';
declare module '*.MapView';
```

---

## 接口层重定义

### 1. 请求拦截层

如果你的项目使用axios, 可以参考以下代码进行ts重构改造
```js
import { message } from 'antd';
import axios, { AxiosResponse } from 'axios';
import { getLoginUrl } from '@/utils/loginUtils';
import qs from 'qs';
import { TemplateDataError } from '@/utils/errorUtils';

const transAxiosResponse = (
  { data: axiosData }: any,
  noMessage?: boolean | Array<any>,
  getError?: boolean,
) => {
  if (axiosData.errno === 0) {
    return Promise.resolve(axiosData.result);
  }

  if (getError) {
    if (!noMessage || (Array.isArray(noMessage) && !noMessage.includes(axiosData.errno)))
      message.error(axiosData.errmsg);
    return Promise.reject(axiosData);
  }
  // 未登录
  if (axiosData.errno === 30003) {
    window.location.href = getLoginUrl(axiosData.result.url);
  }
  if (axiosData.errno === 30007) {
    // 该账号暂时不能登录
  }

  // 模板内容格式错误
  if (axiosData.errno === 50309 || axiosData.errno === 50329) {
    return Promise.reject(new TemplateDataError(axiosData.result.err_url));
  }

  if (!noMessage) message.error(axiosData.errmsg);

  if (axiosData.errno === 50001) {
    return Promise.reject(axiosData.result);
  }

  // 模板数据有误
  if (axiosData.errno === 10105) {
    return Promise.reject(new TemplateDataError(axiosData.result.error_file));
  }

  return Promise.reject(new Error(axiosData.errmsg));
};

const instance = axios.create({
  baseURL: '',
  timeout: 0,
});

const handleRequest = (
  promise: Promise<AxiosResponse<any>>,
  noMessage?: boolean | Array<any>,
  getError?: boolean,
) =>
  promise
    .then(
      response => {
        if (response.status >= 200 && response.status < 300) {
          return response;
        }
        if (!noMessage || Array.isArray(noMessage)) message.error(response.statusText);
        return Promise.reject(new Error(response.statusText));
      },
      error => {
        if (!noMessage || Array.isArray(noMessage)) message.error(error.message);
        return Promise.reject(error);
      },
    )
    .then((res: any) => transAxiosResponse(res, noMessage, getError));

export const get = (
  url: string,
  params?: any,
  noMessage?: boolean | Array<any>,
  getError?: boolean,
) =>
  handleRequest(
    instance.get(url, { params: { ...params, t: new Date().getTime() } }),
    noMessage,
    getError,
  );

export const post = (
  url: string,
  params?: any,
  noMessage?: boolean | Array<any>,
  getError?: boolean,
) =>
  handleRequest(
    instance.post(url, qs.stringify(params), {
      headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
    }),
    noMessage,
    getError,
  );

export const postJson = (
  url: string,
  params: any,
  noMessage?: boolean | Array<any>,
  getError?: boolean,
) => handleRequest(instance.post(url, params), noMessage, getError);

export const upload = (
  url: string,
  params: any,
  noMessage?: boolean | Array<any>,
  getError?: boolean,
) =>
  handleRequest(
    instance.post(url, params, {
      headers: { 'Content-Type': 'multipart/form-data' },
    }),
    noMessage,
    getError,
  );

```

### 2. 接口请求层

js 接口定义

```js
import { get, post, upload } from "@/services/axios";

export const createDistance = (params) =>
  get("/snp/p1/ui/distance/create", { use_type: 1, ...params });

export const getDistanceList = (params, noMessage) =>
  post("/snp/p1/ui/distance/list", { use_type: 1, ...params }, noMessage);

export const createDistanceLine = (params) =>
  get("/snp/p1/ui/distance/create", { use_type: 2, ...params });

export const getDistanceListLine = (params, noMessage) =>
  post("/snp/p1/ui/distance/list", { use_type: 2, ...params }, noMessage);
```

ts 接口定义

```js
/**
 * 仓网规划pro1.6相关接口
 * by Fangge Zhao
 */
import { post } from "@/services/axios";
import { ModifyRequireList, ModifyWHList } from "../../schemas/pro/pro1.6/plan";

/**
 * 覆盖方案
 * @param params
 */
export const coverPlan: (params: {
  wnp_id: string,
  dc_number: number,
}) => Promise<any> = (params) => post("/snp/p1/modify/cover", params);

/**
 * 修改所属仓库列表
 * @param params
 */
export const modifyRequire: (params: {
  wnp_id: string,
  dc_number: number,
  require_id: string,
  sku?: string,
}) => Promise<ModifyRequireList> = (params) =>
  post("/snp/p1/modify/require", params);

/**
 * 重新计算
 * @param params
 */
export const reRun: (params: {
  wnp_id: string,
  dc_number: number,
  new_candidate: string, // 新仓库id
  old_candidate?: string, // 旧仓库id
  require_id?: string, // 需求点id
  sku?: string,
}) => Promise<any> = (params) => post("/snp/p1/modify/run", params);

/**
 * 修改仓库位置列表
 * @param params
 */
export const modifyWH: (params: {
  wnp_id: string,
  dc_number: number,
  candidate_id: string,
}) => Promise<ModifyWHList> = (params) =>
  post("/snp/p1/modify/warehouse", params);
```

### 3. 接口文档类型定义

> 以下是根据接口文档在services/schemas 目录下整合的部分ts代码

```js
export enum PointType {
  NEED = 1, // 需求点
  CDC = 2,
  RDC = 3,
  DC = 4,
  FACTORY = 5, // 工厂
}

export interface PointInfo {
  city: string;
  cover_num: number; // 覆盖客户数量
  id: string;
  province: string;
  sku: 1 | 2 | 3; // 是否指定SKU 1. 不限 2. 指定 3. 不支持
  sku_data: string[]; // sku列表
  type: PointType; // 仓库属性
}

/**
 * 修改所属仓库列表
 */
export interface ModifyRequireList {
  list: PointInfo[];
  sku: string; // sku 限定
  sku_list: string[]; // sku列表
}

export interface WHListInfo {
  city: string;
  has_sku: boolean; // 是否展示sku限定字段 1.限定 2. 不限定
  id: string;
  province: string;
  sku: 1 | 2 | 3; // 是否指定SKU 1. 不限 2. 指定 3. 不支持
  type: 1 | 2 | 3; // 仓库类型：1，顺丰仓；2，用户上传；3，虚拟候选点
  sku_data: string[] | null;
}
/**
 * 修改仓库位置列表
 */
export interface ModifyWHList {
  list: WHListInfo[];
}
```

常用定义方式：

1. interface
2. type
3. enum  (当一个字段有多个值，且每个值对应具体的业务逻辑时，可以用enum枚举进行扩展)

**建议定义接口方式: 根据接口文档手动定义+配合VsCode Paste JSON as Code插件**

Paste JSON as Code使用方式：

 1. 新建一个ts文件
 2. 复制接口文档定义的返回值示例
 3. 在vscode中 command + shift + p, 键入指令 Paste Json as Types

用该插件能够极大程度地解放生产力，但对于一些返回示例的定义，还需要我们进行部分调整，才能充分发挥ts的优势，让代码更为语意化

如以下接口示例：

```json
{
    "aging_percent": 90,
    "aging_time": 23,
    "ball_coefficient": 1,
    "ball_speed": 1,
    "better_list": [
      3
    ],
    "block_param": [
      {
        "car_name": "货车",
        "cross_point": 1,
        "dist_price": 12.1,
        "end_attributes": 1,
        "gap": 1,
        "shipping_method": 1,
        "shipping_product": "SF1",
        "type": 5,
        "unit": 1
      }
    ],
    "candidate_base_definition": {
      "area_rent": 1.1,
      "cdc_replenishment_days": 2,
      "cdc_service_level": 1.1,
      "dc_replenishment_days": 2,
      "dc_service_level": 70,
      "operating_cost": 80,
      "rdc_replenishment_days": 2,
      "rdc_service_level": 70
    },
    "candidate_data": [
      {
        "area_rent": "1.1",
        "cdc_replenishment_days": "2",
        "cdc_service_level": "80",
        "city": "承德",
        "city_code": "承德",
        "dc_replenishment_days": "2",
        "dc_service_level": "75",
        "id": "wh008",
        "lat": "22.1",
        "level": "CDC",
        "lng": "120.1",
        "operating_cost": "10",
        "province": "河北",
        "rdc_replenishment_days": "2",
        "rdc_service_level": "70",
        "sku": "不限",
        "sku_data": "\"\"",
        "status": "可选",
        "type": "备选点",
        "volume_cap": "1000000"
      }
    ],
    "candidate_file": "https://scmp-1254389369.cos.ap-guangzhou.myqcloud.com/000000/2020/扬子江候选仓库清单20200417_0025.xlsx",
    "capital_coefficient": 1.3,
    "capital_cost": 2,
    "car_info_file": "https://scmp-1254389369.cos.ap-guangzhou.myqcloud.com/000000/2020/pro1.4/备选点数据上传模板.xlsx",
    "cdc_cover_client": 1,
    "cdc_num": 1,
    "choose_list": [
      1,
      2
    ],
    "consider_area": 1,
    "consider_cost": 1,
    "consider_trans_cost": 1,
    "cost_attribute": 1,
    "dc_num": "1",
    "dist_cost": 99999999999,
    "distance_file": "https://scmp-1254389369.cos.ap-guangzhou.myqcloud.com/656969235441356612/2020/距离时间矩阵20200417_0079.xlsx",
    "distance_type": 1,
    "had_rent": 1,
    "had_sku": 1,
    "iteration": 1,
    "ltl_distance_file": "https://scmp-1254389369.cos.ap-guangzhou.myqcloud.com/000000/2020/test/运营操作成本.xlsx",
    "ltl_flow_file": "https://scmp-1254389369.cos.ap-guangzhou.myqcloud.com/000000/2020/test/运营操作成本.xlsx",
    "name": "方案A",
    "operating_cost": 1,
    "operating_file": "https://scmp-1254389369.cos.ap-guangzhou.myqcloud.com/000000/2020/test/运营操作成本.xlsx",
    "rdc_num": "1",
    "rend_area": 1,
    "require_cycle": 12,
    "require_file": "https://scmp-1254389369.cos.ap-guangzhou.myqcloud.com/15727873360500/2020/扬子江需求数据解析结果20200417_0065.xlsx",
    "reserve_safe": 2,
    "runtime": 1,
    "sku_deviation": 1.2,
    "sku_file": "https://scmp-1254389369.cos.ap-guangzhou.myqcloud.com/202004/656799077672356612/SKU信息20200417_0049.xlsx",
    "sku_price": 123.2,
    "sku_ratio": 1.2,
    "vehicle_distance_file": "https://scmp-1254389369.cos.ap-guangzhou.myqcloud.com/000000/2020/test/运营操作成本.xlsx",
    "vehicle_flow_file": "https://scmp-1254389369.cos.ap-guangzhou.myqcloud.com/000000/2020/test/运营操作成本.xlsx",
    "warehouse_level": 2,
    "wnp_id": "WNP202004091804564153802560532"
  }
```

以下是我们用插件生成的ts代码
```js
export interface Result {
  aging_percent:             number;
  aging_time:                number;
  ball_coefficient:          number;
  ball_speed:                number;
  better_list:               number[];
  block_param:               BlockParam[];
  candidate_base_definition: CandidateBaseDefinition;
  candidate_data:            { [key: string]: string }[];
  candidate_file:            string;
  capital_coefficient:       number;
  capital_cost:              number;
  car_info_file:             string;
  cdc_cover_client:          number;
  cdc_num:                   number;
  choose_list:               number[];
  consider_area:             number;
  consider_cost:             number;
  consider_trans_cost:       number;
  cost_attribute:            number;
  dc_num:                    string;
  dist_cost:                 number;
  distance_file:             string;
  distance_type:             number;
  had_rent:                  number;
  had_sku:                   number;
  iteration:                 number;
  ltl_distance_file:         string;
  ltl_flow_file:             string;
  name:                      string;
  operating_cost:            number;
  operating_file:            string;
  rdc_num:                   string;
  rend_area:                 number;
  require_cycle:             number;
  require_file:              string;
  reserve_safe:              number;
  runtime:                   number;
  sku_deviation:             number;
  sku_file:                  string;
  sku_price:                 number;
  sku_ratio:                 number;
  vehicle_distance_file:     string;
  vehicle_flow_file:         string;
  warehouse_level:           number;
  wnp_id:                    string;
}

export interface BlockParam {
  car_name:         string;
  cross_point:      number;
  dist_price:       number;
  end_attributes:   number;
  gap:              number;
  shipping_method:  number;
  shipping_product: string;
  type:             number;
  unit:             number;
}

export interface CandidateBaseDefinition {
  area_rent:              number;
  cdc_replenishment_days: number;
  cdc_service_level:      number;
  dc_replenishment_days:  number;
  dc_service_level:       number;
  operating_cost:         number;
  rdc_replenishment_days: number;
  rdc_service_level:      number;
}
```
我们可以看到对于 *candidate_data* 字段，插件未给我们准确定义，这时候需要我们手动对该字段进行详尽定义。