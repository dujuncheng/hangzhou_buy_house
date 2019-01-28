<template>
    <div class="map-container">
        <el-input class="search-input" v-model="searchInput" placeholder="请输入内容" id="tipinput"></el-input>

        <div class="btn-container">
            <div class="btn dudu-work"
                 @click="showDuduWork = !showDuduWork"
                 :style="!showDuduWork ? {'background': '#32a30d'}: {'color': '#32a30d'}"
            >
                {{showDuduWork?'隐藏':'显示'}}肚肚工作地点
            </div>
            <div class="btn baobei-work"
                 @click="showBaobeiWork = !showBaobeiWork"
                 :style="!showBaobeiWork ? {'background': '#ff66b6', 'color': 'white'}: {'color': '#ff66b6'}"
            >
                {{showBaobeiWork?'隐藏':'显示'}}宝贝工作地点
            </div>
        </div>
        <div class="markdown-container" v-if="markdownShow" >
            <div class="title">
                从这儿到【丁香园】的车程 <span class="num">{{markDistance}}米</span>, 需要 <span class="num">{{markTime}}分钟</span>,
            </div>
            <div class="content" v-html="markdownHtml"></div>
        </div>
        <div class="markdown-close" v-if="markdownShow" @click="closeMarkdown">x</div>
        <div class="map-container" id="container"></div>
        <div id="panel"></div>
    </div>
</template>

<script>
    var markdown = require( "markdown" ).markdown;
    let hangzhouPos = [120.2, 30.266667];
    let DUDU_AVATER = 'https://ws2.sinaimg.cn/large/006tNbRwgy1fygn4skahmj309p09pdgf.jpg';
    let BAOBEI_AVATER = 'https://ws1.sinaimg.cn/large/006tNbRwgy1fygn6ko8ydj309p09paah.jpg';

    import {DUDU_WORK, BAOBEI_WORK, YU_JIANG_NAN, YUE_HONG_WAN} from "../config";
    import {YU_JIANG_NAN_M} from "../markdown/YU_JIANG_NAN.js";
    import {YUE_HONG_WAN_M} from "../markdown/YUE_HONG_WAN.js";


    export default {
        data() {
            return {
                // 输入框里面的内容
                searchInput: '',
                map: '',
                // 肚肚工作地点的mark
                duduWorkMark: {},
                // 宝贝工作地点mark
                baobeiWorkMark: {},
                // 显示肚肚的工作地点
                showDuduWork: false,
                // 显示宝贝的工作地点
                showBaobeiWork: false,
                // 是否显示markdown的介绍
                markdownShow: false,
                markdownHtml: '',
                markTime: 0,
                markDistance: 0,
                driving: '',
            }
        },
        watch: {
            showDuduWork(newValue) {
                if (newValue) {
                    this.addDuduWork();
                } else {
                    this.removeDuduWork();
                }
            },
            showBaobeiWork(newValue) {
                if (newValue) {
                    this.addBaoBeiWork();
                } else {
                    this.removeBaobeiWork();
                }
            }
        },
        methods: {
            // 添加杭州西站的位置
            addHangzhouXiZhan() {
                var circle = new AMap.Circle({
                    center: new AMap.LngLat("119.988262", "30.308642"), // 圆心位置
                    radius: 1000,  //半径
                    borderWeight: 3,
                    strokeColor: "#FF33FF",
                    strokeOpacity: 1,
                    strokeWeight: 6,
                    strokeOpacity: 0.2,
                    fillOpacity: 0.4,
                    strokeStyle: 'dashed',
                    strokeDasharray: [10, 10],
                    // 线样式还支持 'dashed'
                    fillColor: '#1791fc',
                    zIndex: 50,
                });
                // 创建纯文本标记
                var text = new AMap.Text({
                    text:'杭州西站',
                    textAlign:'center', // 'left' 'right', 'center',
                    verticalAlign:'middle', //middle 、bottom
                    draggable:false,
                    cursor:'pointer',
                    style:{
                        'padding': '.75rem 1.25rem',
                        'border-width': 0,
                        'text-align': 'center',
                        'font-size': '10px',
                        'color': 'blue',
                        'background': 'transparent'
                    },
                    position: [119.988262, 30.308642]
                });

                text.setMap(this.map);
                this.map.add(circle);
            },
            /**
             * 添加御江南的房子位置
             */
            addYuJiangNan() {
                var path = [];
                for (let i = 0; i < YU_JIANG_NAN.length; i++) {
                    let obj = new AMap.LngLat(YU_JIANG_NAN[i].lng,YU_JIANG_NAN[i].lat);
                    path.push(obj)
                }
                var polygon = new AMap.Polygon({
                    path: path,
                    strokeColor: "#FF33FF",
                    strokeWeight: 6,
                    strokeOpacity: 0.2,
                    fillOpacity: 0.4,
                    fillColor: '#1791fc',
                    zIndex: 50,
                    click: function (e) {
                        console.log(e)
                    }
                });

                // 创建御江南纯文本标记
                var text = new AMap.Text({
                    text:'御江南',
                    textAlign:'center', // 'left' 'right', 'center',
                    verticalAlign:'middle', //middle 、bottom
                    draggable:false,
                    cursor:'pointer',
                    style:{
                        'padding': '.75rem 1.25rem',
                        'border-width': 0,
                        'text-align': 'center',
                        'font-size': '10px',
                        'color': 'blue',
                        'background': 'transparent'
                    },
                    position: [120.252062, 30.175328]
                });
                text.setMap(this.map);

                // 绑定事件
                polygon.on('click', (e) => {
                    let html_code = markdown.toHTML(YU_JIANG_NAN_M )
                    this.showMarkdown(html_code)
                    this.addDrive({
                        a: 120.252062,
                        b: 30.175328,
                    });
                })
                // 绑定事件
                text.on('click', (e) => {
                    let html_code = markdown.toHTML(YU_JIANG_NAN_M )
                    this.showMarkdown(html_code)
                    this.addDrive({
                        a: 120.252062,
                        b: 30.175328,
                    });
                })
                this.map.add(polygon);
            },
            /**
             * 添加悦虹湾的房子位置
             */
            addYueHongWan() {
                var path = [];
                for (let i = 0; i < YUE_HONG_WAN.length; i++) {
                    let obj = new AMap.LngLat(YUE_HONG_WAN[i].lng,YUE_HONG_WAN[i].lat);
                    path.push(obj)
                }
                var polygon = new AMap.Polygon({
                    path: path,
                    strokeColor: "#FF33FF",
                    strokeWeight: 6,
                    strokeOpacity: 0.2,
                    fillOpacity: 0.4,
                    fillColor: '#1791fc',
                    zIndex: 50,
                    click: function (e) {
                        console.log(e)
                    }
                });

                // 创建御江南纯文本标记
                var text = new AMap.Text({
                    text:'悦虹湾',
                    textAlign:'center', // 'left' 'right', 'center',
                    verticalAlign:'middle', //middle 、bottom
                    draggable:false,
                    cursor:'pointer',
                    style:{
                        'padding': '.75rem 1.25rem',
                        'border-width': 0,
                        'text-align': 'center',
                        'font-size': '10px',
                        'color': 'blue',
                        'background': 'transparent'
                    },
                    position: [120.228326, 30.16891]
                });
                text.setMap(this.map);

                // 绑定事件
                polygon.on('click', (e) => {
                    this.addDrive({
                        a: 120.228326,
                        b: 30.16891,
                    });
                    let html_code = markdown.toHTML(YUE_HONG_WAN_M )
                    this.showMarkdown(html_code)
                })
                // 绑定事件
                text.on('click', (e) => {
                    this.addDrive({
                        a: 120.228326,
                        b: 30.16891,
                    });
                    let html_code = markdown.toHTML(YUE_HONG_WAN_M )
                    this.showMarkdown(html_code)
                })
                this.map.add(polygon);
            },
            showMarkdown(html) {
                this.markdownShow = true;
                this.markdownHtml = html;
            },
            closeMarkdown() {
                this.markdownShow = false;
                this.markdownHtml = '';
                this.driving.clear()
            },
            addDuduWork() {
                for (let key in DUDU_WORK) {
                    var marker = new AMap.Marker({
                        position: new AMap.LngLat(DUDU_WORK[key].lng, DUDU_WORK[key].lat),
                        icon: DUDU_AVATER,
                        offset: new AMap.Pixel(-13, -30)
                    });
                    this.duduWorkMark[key] = marker;
                    this.map.add(marker);
                }
            },
            removeDuduWork() {
                for(let key in this.duduWorkMark) {
                    this.map.remove(this.duduWorkMark[key]);
                }
            },
            addBaoBeiWork() {
                for (let key in BAOBEI_WORK) {
                    var marker = new AMap.Marker({
                        position: new AMap.LngLat(BAOBEI_WORK[key].lng, BAOBEI_WORK[key].lat),
                        icon: BAOBEI_AVATER,
                        offset: new AMap.Pixel(-13, -30)
                    });
                    this.baobeiWorkMark[key] = marker;
                    this.map.add(marker);
                }
            },
            // 添加行驶路线
            addDrive({a, b}) {
                if (this.driving) {
                    this.driving.clear()
                }
                this.driving = new AMap.Driving({
                    map: this.map,
                    panel: "panel"
                });
                // 根据起终点经纬度规划驾车导航路线
                this.driving.search(new AMap.LngLat(a, b), new AMap.LngLat(120.20226, 30.188784), (status, result) =>{
                    // result 即是对应的驾车导航信息，相关数据结构文档请参考
                    console.log(result)

                    if (status === 'complete') {
                        let route = result.routes[0];
                        this.markTime = Math.round( Number(route.time) / 60)
                        this.markDistance = Number(route.distance)
                    } else {
                        console.log('失败')
                    }
                });
            },
            removeBaobeiWork() {
                for(let key in this.baobeiWorkMark) {
                    this.map.remove(this.baobeiWorkMark[key]);
                }
            },
            setCenter(lng,lat ) {
                if (!lng || !lat) {
                    return;
                }
                this.map.setCenter([lng, lat]);
            },
            createMap() {
                this.map = new AMap.Map('container', {
                    resizeEnable: true,
                    zoom:12,//级别
                    center: hangzhouPos,//中心点坐标
                    viewMode:'3D'//使用3D视图
                });
            },
            addToolBar() {
                AMap.plugin('AMap.ToolBar', () => {
                    var toolbar = new AMap.ToolBar();
                    this.map.addControl(toolbar);
                });
            },
            addElasticMarker() {
                AMap.plugin('AMap.ElasticMarker', () => {
                    var toolbar = new AMap.ElasticMarker();
                    this.map.addControl(toolbar);
                });
            },
            addPlaceSearch() {
                var autoOptions = {
                    input: "tipinput"
                };
                var auto = new AMap.Autocomplete(autoOptions);
                var placeSearch = new AMap.PlaceSearch({
                    map: this.map
                });  //构造地点查询类
                AMap.event.addListener(auto, "select", (e) => {
                    placeSearch.setCity(e.poi.adcode);
                    placeSearch.search(e.poi.name);  //关键字查询查询
                });
            },
            addConsoleEvent() {
                this.map.on('click', function(ev) {
                    // 触发事件的对象
                    var target = ev.target;
                    // 触发事件的地理坐标，AMap.LngLat 类型
                    var lnglat = ev.lnglat;
                    console.log(lnglat);
                    // 触发事件的像素坐标，AMap.Pixel 类型
                    var pixel = ev.pixel;
                    // 触发事件类型
                    var type = ev.type;
                });
            }
        },
        mounted() {
            this.createMap();
            // 添加插件
            // this.addToolBar();
            this.addElasticMarker();
            this.addPlaceSearch();

            // 添加事件
            this.addConsoleEvent();

            this.addHangzhouXiZhan();

            this.addYuJiangNan();
            this.addYueHongWan();
        }
    }
</script>

<style lang="less">
    .info {
        padding: .75rem 1.25rem;
        margin-bottom: 1rem;
        border-radius: .25rem;
        position: fixed;
        top: 1rem;
        background-color: white;
        width: auto;
        min-width: 22rem;
        border-width: 0;
        right: 1rem;
        box-shadow: 0 2px 6px 0 rgba(114, 124, 245, .5);
    }
    .context_menu{
        position: relative;
        min-width: 12rem;
        padding: 0;
    }
    .context_menu p{
        cursor: pointer;
        padding: 0.25rem 1.25rem;
    }

    .context_menu p:hover{
        background: #ccc;
    }
    .map-container {
        width: 100vw;
        height: 100vh;
        position: relative;
        .map {
            width: 100%;
            height: 100%;
        }
        .search-input {
            width: 360px;
            height: 45px;
            position: fixed;
            left: 20px;
            top: 20px;
            z-index: 98;
        }
        .markdown-close {
            width: 20px;
            height: 20px;
            border-radius: 50%;
            border: 3px solid grey;
            text-align: center;
            line-height: 20px;
            justify-content: center;
            position: absolute;
            right: 20px;
            top: 20px;
            color: grey;
            font-size: 30px;
            font-weight: 800;
            z-index: 100;
            cursor: pointer;
            padding: 10px;
        }
        .markdown-container {
            padding: 10px;
            width: 40%;
            height: 100%;
            background-color: white;
            position: absolute;
            right: 0;
            top: 0;
            z-index: 99;
            overflow-y: scroll;
            cursor: cell;
            border-left: 3px grey solid;
            text-align: left;
            img {
                width: 100%;
            }
            .title {
                margin-top: 70px;
                padding-bottom: 10px;
                padding-top: 10px;
                padding-left: 5px;
                border-radius: 10px;
                background-color: rgba(0,0,0,0.1);
                .num {
                    font-weight: 800;
                }
            }
        }
        .btn-container {
            width: 400px;
            position: absolute;
            bottom: 20px;
            right: 20px;
            z-index: 100;
            .btn {
                padding-left: 10px;
                padding-right: 10px;
                color: white;
                font-size: 14px;
                border-radius: 16px;
                cursor: pointer;
                user-select: none;

            }
            .baobei-work {
                position: absolute;
                right: 0;
                bottom: 0;
                border: 1px solid #ff66b6;
            }
            .dudu-work {
                position: absolute;
                right: 0;
                bottom: 30px;
                border: 1px solid #32a30d;
            }
        }
    }

</style>
