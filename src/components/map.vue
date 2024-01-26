<template>
  <div>
    <div class="button" @click="mapOpen">开启</div>
    <div ref="mapRef" style="width: auto; height: 450px"></div>
  </div>
</template>

<script>
import {
  DrawScene,
  PolygonEdit,
  PolygonDraw,
} from "bmap-draw";


export default {
  data() {
    return {
      BMap: null,
      polygon: null,
      polyLayer: null,
      editPolygon: null,
      scene: null,
    };
  },
  mounted() {
    this.loadMap();
  },
  methods: {
    mapOpen() {
      this.polygon.open();
    },
    loadMap() {
      this.BMap = new BMapGL.Map(this.$refs.mapRef);
      // map.setMapStyleV2({ styleJson: customStyleJSON });
      this.BMap.enableScrollWheelZoom();
      this.BMap.enableInertialDragging();
      this.BMap.enableContinuousZoom();
      this.BMap.centerAndZoom("北京", 12);
      const scene = new DrawScene(this.BMap, { noLimit: true });
      const polygon = new PolygonDraw(scene, {
        isOpen: false,
        enableSnap: true, // 开启吸附绘制
        matchOverlay: {
          // 自定义吸附点样式
          type: "Marker",
          icon: new BMapGL.Icon(
            "http://maponline0.bdimg.com/sty/map_icons2x/MapRes/shenghui_1.png",
            new BMapGL.Size(10, 10),
            { offset: new BMapGL.Size(5, 5) }
          ),
        },
      });
      this.polygon = polygon;
      this.editPolygon = new PolygonEdit(scene);

      this.polyLayer = new BMapGL.GeoJSONLayer('poly');
        this.BMap.addGeoJSONLayer(this.polyLayer);
        scene.attachSnapSource(this.polyLayer.overlayData);


      scene.addEventListener("operate-ok", (e) => {
          e.target.overlay.addContextMenu(this.getMenu(e.target.overlay)); //覆盖物绑定菜单
        this.polyLayer.overlayData.push(e.target.overlay);
        this.polygon.close();
        if (this.polyLayer.overlayData.length > 1) {
          this.formatterOverlay();
        }
      });

      this.scene = scene;
      
    },
    getMenu(rectangle) {
      var markerMenu = new BMapGL.ContextMenu();
      markerMenu.addItem(
        new BMapGL.MenuItem("删除区域", this.removeMarker.bind(rectangle))
      ); //为菜单绑定事件
      markerMenu.addItem(
        new BMapGL.MenuItem("开始编辑", this.beginEdit.bind(rectangle))
      ); //为菜单绑定事件
      return markerMenu;
    },
    removeMarker(e, ee, marker) {
        this.polyLayer.removeOverlay(marker)
        this.scene.removeOverlay(marker)
    },
    beginEdit(e, ee, marker) {
      this.editPolygon.open(marker);
    },
    overlaysFn(ele, arr, eleI) {
      for (let i = 0; i < arr.length; i++) {
        if (this.isPolygonsIntersectant(ele.getPath(), arr[i].getPath())) {
          const difference = turf.difference(
            ele.toGeoJSON(),
            arr[i].toGeoJSON()
          );
          this.polyLayer.overlayData[eleI].updateByGeoJSON(difference);
        }
      }
    },
    formatterOverlay() {
      for (let i = 0; i < this.polyLayer.overlayData.length; i++) {
        let overlays2 = [...this.polyLayer.overlayData];
        overlays2.splice(i, 1);
        this.overlaysFn(this.polyLayer.overlayData[i], overlays2, i);
      }
    },
    /**
     * 线段是否相交
     * seg: [{ lat: xxx, lng: xxx }, { lat: xxx, lng: xxx }]
     * */
    isSegmentsIntersectant(segA, segB) {
      var abc =
        (segA[0].lat - segB[0].lat) * (segA[1].lng - segB[0].lng) -
        (segA[0].lng - segB[0].lng) * (segA[1].lat - segB[0].lat);
      var abd =
        (segA[0].lat - segB[1].lat) * (segA[1].lng - segB[1].lng) -
        (segA[0].lng - segB[1].lng) * (segA[1].lat - segB[1].lat);
      if (abc * abd >= 0) {
        return false;
      }

      var cda =
        (segB[0].lat - segA[0].lat) * (segB[1].lng - segA[0].lng) -
        (segB[0].lng - segA[0].lng) * (segB[1].lat - segA[0].lat);
      var cdb = cda + abc - abd;
      return !(cda * cdb >= 0);
    },

    /**
     * 判断两多边形边界是否相交
     */
    isPolygonsIntersectant(plyA, plyB) {
      for (var i = 0, il = plyA.length; i < il; i++) {
        for (var j = 0, jl = plyB.length; j < jl; j++) {
          var segA = [plyA[i], plyA[i === il - 1 ? 0 : i + 1]];
          var segB = [plyB[j], plyB[j === jl - 1 ? 0 : j + 1]];
          if (this.isSegmentsIntersectant(segA, segB)) {
            return true;
          }
        }
      }
      return false;
    },
  },
};
</script>

<style lang="less">
.button {
  width: 80px;
  height: 30px;
  border: 1px solid #ccc;
  line-height: 30px;
  text-align: center;
  cursor: pointer;
  margin-bottom: 10px;
}
</style>
