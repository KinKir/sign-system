<template>
    <center>
        <form>
            <div style=" margin:auto;width:95%;height:500px;border:2px solid gray; margin-bottom:10px;" id="container"></div>
            <h3 style="color: red;">介绍：①可点击地图确定签到地点；②可通过下方经度，纬度，半径查询确定签到地点</h3>
            经度：
            <input id="lonlat" name="lonlat" type="number" value="114.339972" maxlength="10">
            纬度：
            <input id="lonlat2" name="lonlat2" type="number" value="30.517935" maxlength="9">
            半径：
            <input id="dis" name="dis" type="number" value="50" maxlength="9">
            <input type="button" value="查找" id="search" /><br>
            <div id="detail"></div>
            
        </form>
    </center>
</template>

<script>
import GeoUtils from '../../../static/js/GeoUtils.js'
import BMap from 'BMap'
export default {
    data () {
        return {
        }
    },
    methods: {
        ready: function startmap() {
            var longitude=114.339972;
            var latitude=30.517935;
            var dis=50;
            var map = new BMap.Map("container",{enableMapClick:false});
            map.setDefaultCursor("crosshair");
            map.enableScrollWheelZoom();
            var point = new BMap.Point(longitude,latitude);
            var circle = new BMap.Circle(point,50,{fillColor:"blue", strokeWeight: 1 ,fillOpacity: 0.3, strokeOpacity: 0.3});
            map.centerAndZoom(point, 17);
            var gc = new BMap.Geocoder();
            map.disableDoubleClickZoom();//禁用双击事件
            var marker = new BMap.Marker(point);
            map.addOverlay(circle);
            map.addOverlay(marker);
            
            //初始化地址
            gc.getLocation(point,
            function(rs) {
                showdetail(point, rs);
            });
            //地图点击事件
            map.addEventListener("click",
            function mapdraw(e,rs) {
                map.clearOverlays();
                document.getElementById("lonlat").value = e.point.lng;
                document.getElementById("lonlat2").value = e.point.lat;
                longitude = e.point.lng;
                latitude = e.point.lat;
                point = new BMap.Point(longitude,latitude);
                circle = new BMap.Circle(point,dis,{fillColor:"blue", strokeWeight: 1 ,fillOpacity: 0.3, strokeOpacity: 0.3});
                marker = new BMap.Marker(point);
                map.addOverlay(circle);
                map.addOverlay(marker);
                //动态画圆
                
                
                //动态获取地址
                gc.getLocation(e.point,
                function(rs) {
                    showdetail(e.point, rs);
                });
            });
            
            //精准查询
            var lonlat = document.getElementById("lonlat");
            var lonlat2 = document.getElementById("lonlat2");
            var inputdis = document.getElementById("dis") 
            document.getElementById('search').onclick = function(point,rs){
                longitude = lonlat.value;
                latitude = lonlat2.value;
                dis = inputdis.value;
                mapdraw(point,rs);
                function mapdraw(point,rs) {
                    map.clearOverlays();
                    circle = new BMap.Circle(new BMap.Point(longitude,latitude),dis,{fillColor:"blue", strokeWeight: 1 ,fillOpacity: 0.3, strokeOpacity: 0.3});
                    point = new BMap.Point(longitude,latitude);
                    marker = new BMap.Marker(point);
                    map.addOverlay(marker);
                    map.addOverlay(circle);
                   //动态获取地址
                    gc.getLocation(point,
                    function(rs) {
                        showdetail(point, rs);
                    });
                }
            }
            //获取地址的函数
            function showdetail(pt, rs) {
                var addComp = rs.addressComponents;
                var addr = '地址：'+ addComp.province + addComp.city + addComp.district  + addComp.street  + addComp.streetNumber
                document.getElementById("detail").innerHTML = addr;
            }
        },
    },
    mounted(){
        this.ready();
        
    }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
    
</style>