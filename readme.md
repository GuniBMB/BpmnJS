# a-bpmn自定义封装

利用vue封装Bpmn，用以实现加载Bpmn数据，绘制流程图。

## 功能点

 1、重写了drawShape和drawConnect函数，简单实现了自定义连接线和节点样式；

 2、封装实现了任意个节点间连接线


## 程序安装
### 配置和依赖：
``` javascript
/**
依赖包：
*/

 {
    "bpmn-js": "^6.0.4",
    "core-js": "^3.6.4",
    "raw-loader": "^4.0.1",
    "xml-loader": "^1.2.1"
}
/**
webpack配置：
*/
  config.module
    .rule('raw-loader')
    .test(/.(bpmn|xml)$/)
    .use('raw-loader')
    .loader('raw-loader')
    .end()
```
### 安装
```
yarn install
```

### 用法
``` javascript
// 1、组件注册和使用
import ABpmn from "@/components/ABpmn"
<a-bpmn
      :bpmnData="bpmnData"
      :dataType="dataType"
      :hasFinishedNodes="hasFinishedNodes"
      :noteFinishedNodes = 'noteFinishedNodes'
      :customConnecteNodes = 'customConnecteNodes'
    >
</a-bpmn>
/**
bpmnData：Any,流程图数据，可以是bpmn文件和字符串
dataType：String,数据格式('bpmn'|'str'),分别对应bpmn文件和字符串
hasFinishedNodes：Array,标识已经走完的流程 
noteFinishedNodes：Array,标识没有走完的流程
customConnecteNodes:Array,自定义连接点,
格式：
    [
      {
        sourceID:'usertask_1253234441636155392',
        targetID:'usertask_1253312880087011335',
        id:'sequenceflow_1253312880087011391'
      },
    ]
**/


### 交友连接
点击 [GitHub](https://github.com/GuniBMB/BpmnJS).
