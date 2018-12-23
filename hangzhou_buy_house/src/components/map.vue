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
        <div class="map-container" id="container"></div>
    </div>
</template>

<script>
    let hangzhouPos = [120.2, 30.266667];
    let DUDU_AVATER = 'https://ws2.sinaimg.cn/large/006tNbRwgy1fygn4skahmj309p09pdgf.jpg';
    let BAOBEI_AVATER = 'https://ws1.sinaimg.cn/large/006tNbRwgy1fygn6ko8ydj309p09paah.jpg';

    import {DUDU_WORK, BAOBEI_WORK} from "../config";

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
                this.map.setFitView();
            },
            removeDuduWork() {
                for(let key in this.duduWorkMark) {
                    this.map.remove(this.duduWorkMark[key]);
                }
                this.map.setFitView();
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
                this.map.setFitView();
            },
            removeBaobeiWork() {
                for(let key in this.baobeiWorkMark) {
                    this.map.remove(this.baobeiWorkMark[key]);
                }
                this.map.setFitView();
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
        }
    }
</script>

<style scoped lang="less">
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
            z-index: 100;
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
