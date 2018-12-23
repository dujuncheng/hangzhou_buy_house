<template>
    <div>
        <el-input class="search-input" v-model="searchInput" placeholder="请输入内容" id="tipinput"></el-input>
        <div class="map-container" id="container"></div>
    </div>
</template>

<script>
    let hangzhouPos = [120.2, 30.266667];
    let dudu_avater = 'https://ws4.sinaimg.cn/large/006tNbRwly1fyftnkuhszj30d80damyh.jpg';
    let baobei_avater = 'https://ws2.sinaimg.cn/large/006tNbRwly1fyftobwjm1j30im0is40g.jpg';

    import {DUDU_WORK, BAOBEI_WORK} from "../config";

    export default {
        data() {
            return {
                // 输入框里面的内容
                searchInput: '',
                map: '',
            }
        },
        methods: {
            addDuduWork() {
                for (let key in DUDU_WORK) {
                    var marker = new AMap.Marker({
                        position: new AMap.LngLat(DUDU_WORK[key].lng, DUDU_WORK[key].lat),
                        icon: dudu_avater,
                        offset: new AMap.Pixel(-13, -30)
                    });
                    this.map.add(marker);
                }
            },
            addBaoBeiWork() {
                for (let key in BAOBEI_WORK) {
                    var marker = new AMap.Marker({
                        position: new AMap.LngLat(BAOBEI_WORK[key].lng, BAOBEI_WORK[key].lat),
                        icon: baobei_avater,
                        offset: new AMap.Pixel(-13, -30)
                    });
                    this.map.add(marker);
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

            // 添加宝贝的工作地点
            this.addDuduWork();
            // 添加dudu的工作地点
            this.addBaoBeiWork();
        }
    }
</script>

<style scoped>
    .map-container {
        width: 100vw;
        height: 100vh;
    }
    .search-input {
        width: 360px;
        height: 45px;
        position: fixed;
        left: 20px;
        top: 20px;
        z-index: 100;
    }
</style>
