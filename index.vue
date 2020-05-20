<template>
    <div class="containers">
        <div class="canvas" ref="canvas"></div>
    </div>
</template>

<script>
    import BpmnModeler from 'bpmn-js/lib/Modeler'
    import BaseRenderer from 'diagram-js/lib/draw/BaseRenderer'
    import {
        attr as svgAttr,
        select,
        remove,
        create as svgCreate
    } from 'tiny-svg';
    const HIGH_PRIORITY = 1500;
    // 编辑节点的样式
    import 'bpmn-js/dist/assets/diagram-js.css'
    export default {
        name: "index",
        data(){
            return {
                bpmnModeler:undefined,
            }
        },
        props:{
            bpmnData:{
               require:true
            },
            dataType:{
                type:String
            },
            hasFinishedNodes:{
                type:Array,
                require:true
            },
            noteFinishedNodes:{
                type:Array,
                require:true
            },
            customConnecteNodes:{
                type:Array
            }
        },
        mounted() {
            this.ini();
        },
        methods:{
            ini(){
                this.iniParamas();
                const canvas = this.$refs.canvas;
                // 定义建模器
                let customModule = this.iniCustomModule();
                this.bpmnModeler = new BpmnModeler({
                    container: canvas,
                    additionalModules: [
                        customModule
                    ]
                });
                // 建模
                this.createNewDiagram();
            },
            iniParamas(){

            },
            /**
             * 根据bpmn数据创建图像
             */
            createNewDiagram(){
                let bpmnData = this.getDataByType();
                this.bpmnModeler.importXML(bpmnData, err => {
                    if (err) {
                        console.error(err)
                    } else {
                        // 成功之后回调逻辑
                        this.success();

                    }
                })
            },

            /**
             * 自定义节点连线
             * */
            createCustomConnect(sourceID,targetID,attrs){
                try {
                    let source = undefined,target = undefined,elementRegistry = undefined,elementFactory = undefined,modeling=undefined ;
                    elementRegistry = this.bpmnModeler.get('elementRegistry');
                    elementFactory = this.bpmnModeler.get('elementFactory');
                    modeling = this.bpmnModeler.get('modeling');
                    source = elementRegistry.get(sourceID);
                    target =  elementRegistry.get(targetID);
                   /* *
                    创建连接之前需要判断当前两个节点的连接线是否已存在，
                    如果存在则不创建,反之则创建连线
                   **/
                    let existConnects = elementRegistry.getAll().filter(e=>e.type == "bpmn:SequenceFlow");
                    let isConnectExist = existConnects.find(e=>e.source.id == sourceID && e.target.id == targetID);
                    if(isConnectExist){
                       return;
                    }else {
                        let cusConnect = modeling.connect(source, target, attrs, null);
                        // cusConnect.labels.add('test')

                    }

                }catch (e) {
                    console.log(e);
                }

            },
            /**
             * 构建自定义流程加载器
             * */
            iniCustomModule(){
               let me = this;
                let hasFinishedNodes = me.hasFinishedNodes||[];
                let noteFinishedNodes =  me.noteFinishedNodes||[];
                let customConnectNodes = me.customConnecteNodes||[];
               let CustomRenderer = me.iniCustomRenderer(hasFinishedNodes,noteFinishedNodes,customConnectNodes);
               return {
                   __init__: ['customRenderer'],
                   customRenderer: ['type', CustomRenderer]
               }
            },
            /**
             * 构建自定义渲染器
             * */
            iniCustomRenderer(hasFinishedNodes,noteFinishedNodes,customConnectNodes){
                let me = this;
                class CustomRenderer extends BaseRenderer {
                    constructor(eventBus, bpmnRenderer, modeling) {
                        super(eventBus, HIGH_PRIORITY);
                        this.bpmnRenderer = bpmnRenderer;
                        this.modeling = modeling;
                    }
                    canRender(element) {
                        // ignore labels
                        return !element.labelTarget;
                    }
                    drawShape(parentNode, element) {
                        const type = element.type // 获取到类型

                        let id  = element.id;
                        const shape = this.bpmnRenderer.drawShape(parentNode, element);
                        if(id && hasFinishedNodes.includes(id)){
                            this.customRenderShape(parentNode,type,'blue',shape)
                        }
                        return shape
                    }
                    drawConnection(parentGfx,element){
                        let id  = element.id;
                        const connection = this.bpmnRenderer.drawConnection(parentGfx,element);
                        if(id && hasFinishedNodes.includes(id)){
                            this.customRenderConnection(parentGfx,connection,'blue','redTriangle');
                        }
                        if(id && customConnectNodes.map(e=>e.id).includes(id)){
                            this.drawDashesLine(parentGfx,connection,'blue','blueTriangle');
                        }
                        return connection;
                    }

                    getShapePath(shape) {
                        return this.bpmnRenderer.getShapePath(shape);
                    }

                    /**
                     * 自定义流程模块样式
                     * @param parentNode
                     * @param type
                     * @param color
                     * @param shape
                     */
                    customRenderShape(parentNode,type,color,shape){
                        if(type == 'bpmn:InclusiveGateway'){
                            let oriCircle = select(parentNode,'circle');
                            let circle = svgCreate('circle',{
                                cx:"20",
                                cy:"20",
                                r:"10" ,
                                style:`stroke: ${color}; stroke-width: 2.5px; fill: white;`
                            });
                            remove(oriCircle);
                            parentNode.append(circle);
                        }
                        svgAttr(shape, 'stroke',color);
                    }

                    /**
                     * 自定义连接线样式
                     * @param parentGfx
                     * @param connection
                     * @param color
                     * @param markerID
                     */
                    customRenderConnection(parentGfx,connection,color,markerID){
                        let defs = svgCreate('defs');
                        let marker = svgCreate('marker',{
                            id:markerID,
                            viewBox:"0 0 20 20",
                            refX:"11",
                            refY:"10",
                            markerWidth:"10",
                            markerHeight:"10",
                            orient:"auto",
                        });
                        let path = svgCreate('path');

                        marker.append(path);
                        defs.append(marker);
                        parentGfx.append(defs);
                        svgAttr(connection, 'stroke',color);
                        svgAttr(connection, 'marker-end',`url("#${markerID}")`);
                    }

                    drawDashesLine(parentGfx,connection,color,markerID){
                        let defs = svgCreate('defs');
                        let marker = svgCreate('marker',{
                            id:markerID,
                            viewBox:"0 0 20 20",
                            refX:"11",
                            refY:"10",
                            markerWidth:"10",
                            markerHeight:"10",
                            orient:"auto",
                        });
                        let path = svgCreate('path');

                        marker.append(path);
                        defs.append(marker);
                        parentGfx.append(defs);

                        svgAttr(connection, 'stroke',color);
                        svgAttr(connection, 'stroke-dasharray','5,5');
                        svgAttr(connection, 'marker-end',`url("#${markerID}")`);
                    }
                }

                CustomRenderer.$inject = ['eventBus', 'bpmnRenderer', 'modeling'];
                return CustomRenderer;
            },
            getDataByType(){
                let result = undefined;
                switch (this.dataType) {
                    case 'bpmn':
                        result = this.bpmnData.default;
                        break;
                    case 'str':
                        result = this.bpmnData;
                        break;
                    default:
                        result = '';
                        break;
                }
                return result;
            },
            success(){
                // 让图能自适应屏幕
                var canvas = this.bpmnModeler.get('canvas');
                canvas.zoom('fit-viewport');
                // 垂直居中标签<g>
                let elementDom = document.getElementsByClassName('viewport')[0];
                let bBox = elementDom.getBBox();
                let width = bBox.width/2+bBox.x;
                let height = bBox.height/2+bBox.y;
                elementDom.style.transform =  `translate(calc(50% - ${width}px), calc(50% - ${height}px))`;

                //添加自定义连线
                if(this.customConnecteNodes.length>0){
                    this.customConnecteNodes.forEach(e=>{
                        let attrs= {
                            id:e.id,
                            type: "bpmn:SequenceFlow",
                            // label:'test'
                        };
                        this.createCustomConnect(e.sourceID,e.targetID,attrs)
                    })
                }
            }
        }
    }
</script>

<style scoped>
    .containers {
        background-color: #ffffff;
        width: 100%;
        height: calc(100vh - 52px);
    }
    .canvas {
        width: 100%;
        height: 100%;
    }
    /deep/ .djs-palette,/deep/ .bjs-powered-by,/deep/ .djs-context-pad{
        display: none;
    }
    /deep/ .viewport{
        /*transform: translate(calc(50% - 484px), calc(50% - 211px));*/
    }
</style>
