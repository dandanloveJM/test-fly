<template>
  <div class="container1">
    <Breadcrumb :items="['排列布设推演']" />
    <div>已激发炮点</div>
    <div v-for="(shot, index) in state.shots1" :key="index" class="shot-info">
      <p>{{ shot.sline }}, {{ shot.spoint }}</p>
    </div>
    <div id="container"></div>
  </div>
</template>

<script lang="ts" setup>
  import {
    onMounted,
    onUnmounted,
    onBeforeMount,
    reactive,
    ref,
    set,
    toRefs,
    nextTick,
  } from 'vue';
  import AMapLoader from '@amap/amap-jsapi-loader';
  import { getRecvs, getShots, getShots2, getRecvs2 } from '@/api/recv';

  onBeforeMount(() => {
    // eslint-disable-next-line no-underscore-dangle
    window._AMapSecurityConfig = {
      securityJsCode: '2ce1b3ef16f0241078486e48aa480eaa',
    };
  });
  let socket;
  let loca;

  // 使用ref来定义地图和图层的引用
  let map = null;
  const recvLayer = ref(null);
  const shotLayer = ref(null);

  const state = ref({
    recvs: { type: 'FeatureCollection', features: [] },
    shots: { type: 'FeatureCollection', features: [] },
    shots1: {},
  });
  async function updateMapLayers(updatedData) {
    if (
      !recvLayer.value ||
      !state.value.recvs ||
      !Array.isArray(state.value.recvs.features)
    ) {
      console.error('RecvLayer is not yet initialized.');
      return;
    }
    console.log('Pre-merge data:', state.value.recvs);
    // Merge new data with existing data
    state.value.recvs = mergeUpdates(state.value.recvs, updatedData);

    // const newdata = {
    //   type: 'FeatureCollection',
    //   features: [
    //     {
    //       type: 'Feature',
    //       properties: {
    //         name: '2758.0,1450.0',
    //         rstatus: 0,
    //       },
    //       geometry: {
    //         type: 'Point',
    //         coordinates: [106.439559, 30.905203],
    //       },
    //     },
    //     {
    //       type: 'Feature',
    //       properties: {
    //         name: '2758.0,1451.0',
    //         rstatus: 0,
    //       },
    //       geometry: {
    //         type: 'Point',
    //         coordinates: [106.439812, 30.90548],
    //       },
    //     },
    //     {
    //       type: 'Feature',
    //       properties: {
    //         name: '2758.0,1452.0',
    //         rstatus: 0,
    //       },
    //       geometry: {
    //         type: 'Point',
    //         coordinates: [106.440099, 30.90575],
    //       },
    //     },
    //     {
    //       type: 'Feature',
    //       properties: {
    //         name: '2758.0,1453.0',
    //         rstatus: 0,
    //       },
    //       geometry: {
    //         type: 'Point',
    //         coordinates: [106.440354, 30.906037],
    //       },
    //     },
    //     {
    //       type: 'Feature',
    //       properties: {
    //         name: '2758.0,1454.0',
    //         rstatus: 0,
    //       },
    //       geometry: {
    //         type: 'Point',
    //         coordinates: [106.440602, 30.906328],
    //       },
    //     },
    //     {
    //       type: 'Feature',
    //       properties: {
    //         name: '2758.0,1455.0',
    //         rstatus: 0,
    //       },
    //       geometry: {
    //         type: 'Point',
    //         coordinates: [106.440893, 30.906584],
    //       },
    //     },
    //     {
    //       type: 'Feature',
    //       properties: {
    //         name: '2758.0,1456.0',
    //         rstatus: 0,
    //       },
    //       geometry: {
    //         type: 'Point',
    //         coordinates: [106.441133, 30.906877],
    //       },
    //     },
    //     {
    //       type: 'Feature',
    //       properties: {
    //         name: '2758.0,1457.0',
    //         rstatus: 0,
    //       },
    //       geometry: {
    //         type: 'Point',
    //         coordinates: [106.441407, 30.907155],
    //       },
    //     },
    //     {
    //       type: 'Feature',
    //       properties: {
    //         name: '2758.0,1458.0',
    //         rstatus: 0,
    //       },
    //       geometry: {
    //         type: 'Point',
    //         coordinates: [106.441657, 30.907445],
    //       },
    //     },
    //     {
    //       type: 'Feature',
    //       properties: {
    //         name: '2758.0,1459.0',
    //         rstatus: 0,
    //       },
    //       geometry: {
    //         type: 'Point',
    //         coordinates: [106.441903, 30.907741],
    //       },
    //     },
    //     {
    //       type: 'Feature',
    //       properties: {
    //         name: '2758.0,1460.0',
    //         rstatus: 0,
    //       },
    //       geometry: {
    //         type: 'Point',
    //         coordinates: [106.44214, 30.908034],
    //       },
    //     },
    //     {
    //       type: 'Feature',
    //       properties: {
    //         name: '2758.0,1461.0',
    //         rstatus: 0,
    //       },
    //       geometry: {
    //         type: 'Point',
    //         coordinates: [106.442469, 30.908285],
    //       },
    //     },
    //     {
    //       type: 'Feature',
    //       properties: {
    //         name: '2758.0,1462.0',
    //         rstatus: 0,
    //       },
    //       geometry: {
    //         type: 'Point',
    //         coordinates: [106.442673, 30.908592],
    //       },
    //     },
    //     {
    //       type: 'Feature',
    //       properties: {
    //         name: '2758.0,1463.0',
    //         rstatus: 0,
    //       },
    //       geometry: {
    //         type: 'Point',
    //         coordinates: [106.442914, 30.908881],
    //       },
    //     },
    //     {
    //       type: 'Feature',
    //       properties: {
    //         name: '2758.0,1464.0',
    //         rstatus: 0,
    //       },
    //       geometry: {
    //         type: 'Point',
    //         coordinates: [106.4432, 30.909153],
    //       },
    //     },
    //     {
    //       type: 'Feature',
    //       properties: {
    //         name: '2758.0,1465.0',
    //         rstatus: 0,
    //       },
    //       geometry: {
    //         type: 'Point',
    //         coordinates: [106.443442, 30.909449],
    //       },
    //     },
    //     {
    //       type: 'Feature',
    //       properties: {
    //         name: '2758.0,1466.0',
    //         rstatus: 0,
    //       },
    //       geometry: {
    //         type: 'Point',
    //         coordinates: [106.443701, 30.909731],
    //       },
    //     },
    //     {
    //       type: 'Feature',
    //       properties: {
    //         name: '2758.0,1467.0',
    //         rstatus: 0,
    //       },
    //       geometry: {
    //         type: 'Point',
    //         coordinates: [106.443996, 30.909989],
    //       },
    //     },
    //     {
    //       type: 'Feature',
    //       properties: {
    //         name: '2758.0,1468.0',
    //         rstatus: 0,
    //       },
    //       geometry: {
    //         type: 'Point',
    //         coordinates: [106.444251, 30.910279],
    //       },
    //     },
    //     {
    //       type: 'Feature',
    //       properties: {
    //         name: '2758.0,1469.0',
    //         rstatus: 0,
    //       },
    //       geometry: {
    //         type: 'Point',
    //         coordinates: [106.444082, 30.910855],
    //       },
    //     },
    //     {
    //       type: 'Feature',
    //       properties: {
    //         name: '2758.0,1470.0',
    //         rstatus: 0,
    //       },
    //       geometry: {
    //         type: 'Point',
    //         coordinates: [106.444466, 30.911048],
    //       },
    //     },
    //     {
    //       type: 'Feature',
    //       properties: {
    //         name: '2758.0,1471.0',
    //         rstatus: 0,
    //       },
    //       geometry: {
    //         type: 'Point',
    //         coordinates: [106.445037, 30.911124],
    //       },
    //     },
    //     {
    //       type: 'Feature',
    //       properties: {
    //         name: '2758.0,1472.0',
    //         rstatus: 0,
    //       },
    //       geometry: {
    //         type: 'Point',
    //         coordinates: [106.445284, 30.911402],
    //       },
    //     },
    //     {
    //       type: 'Feature',
    //       properties: {
    //         name: '2758.0,1473.0',
    //         rstatus: 0,
    //       },
    //       geometry: {
    //         type: 'Point',
    //         coordinates: [106.445568, 30.911683],
    //       },
    //     },
    //     {
    //       type: 'Feature',
    //       properties: {
    //         name: '2758.0,1474.0',
    //         rstatus: 0,
    //       },
    //       geometry: {
    //         type: 'Point',
    //         coordinates: [106.445868, 30.911937],
    //       },
    //     },
    //     {
    //       type: 'Feature',
    //       properties: {
    //         name: '2758.0,1475.0',
    //         rstatus: 0,
    //       },
    //       geometry: {
    //         type: 'Point',
    //         coordinates: [106.446085, 30.912243],
    //       },
    //     },
    //     {
    //       type: 'Feature',
    //       properties: {
    //         name: '2758.0,1476.0',
    //         rstatus: 0,
    //       },
    //       geometry: {
    //         type: 'Point',
    //         coordinates: [106.446353, 30.912519],
    //       },
    //     },
    //     {
    //       type: 'Feature',
    //       properties: {
    //         name: '2758.0,1477.0',
    //         rstatus: 0,
    //       },
    //       geometry: {
    //         type: 'Point',
    //         coordinates: [106.446643, 30.91279],
    //       },
    //     },
    //     {
    //       type: 'Feature',
    //       properties: {
    //         name: '2758.0,1478.0',
    //         rstatus: 0,
    //       },
    //       geometry: {
    //         type: 'Point',
    //         coordinates: [106.446796, 30.91314],
    //       },
    //     },
    //     {
    //       type: 'Feature',
    //       properties: {
    //         name: '2758.0,1479.0',
    //         rstatus: 0,
    //       },
    //       geometry: {
    //         type: 'Point',
    //         coordinates: [106.447112, 30.913385],
    //       },
    //     },
    //     {
    //       type: 'Feature',
    //       properties: {
    //         name: '2758.0,1480.0',
    //         rstatus: 0,
    //       },
    //       geometry: {
    //         type: 'Point',
    //         coordinates: [106.44739, 30.913657],
    //       },
    //     },
    //     {
    //       type: 'Feature',
    //       properties: {
    //         name: '2758.0,1481.0',
    //         rstatus: 0,
    //       },
    //       geometry: {
    //         type: 'Point',
    //         coordinates: [106.447691, 30.913908],
    //       },
    //     },
    //     {
    //       type: 'Feature',
    //       properties: {
    //         name: '2758.0,1482.0',
    //         rstatus: 0,
    //       },
    //       geometry: {
    //         type: 'Point',
    //         coordinates: [106.447911, 30.914212],
    //       },
    //     },
    //     {
    //       type: 'Feature',
    //       properties: {
    //         name: '2758.0,1483.0',
    //         rstatus: 0,
    //       },
    //       geometry: {
    //         type: 'Point',
    //         coordinates: [106.448203, 30.914459],
    //       },
    //     },
    //     {
    //       type: 'Feature',
    //       properties: {
    //         name: '2758.0,1484.0',
    //         rstatus: 0,
    //       },
    //       geometry: {
    //         type: 'Point',
    //         coordinates: [106.448431, 30.914778],
    //       },
    //     },
    //     {
    //       type: 'Feature',
    //       properties: {
    //         name: '2758.0,1485.0',
    //         rstatus: 0,
    //       },
    //       geometry: {
    //         type: 'Point',
    //         coordinates: [106.448706, 30.915052],
    //       },
    //     },
    //     {
    //       type: 'Feature',
    //       properties: {
    //         name: '2758.0,1486.0',
    //         rstatus: 0,
    //       },
    //       geometry: {
    //         type: 'Point',
    //         coordinates: [106.448967, 30.915338],
    //       },
    //     },
    //     {
    //       type: 'Feature',
    //       properties: {
    //         name: '2758.0,1487.0',
    //         rstatus: 0,
    //       },
    //       geometry: {
    //         type: 'Point',
    //         coordinates: [106.449211, 30.915634],
    //       },
    //     },
    //   ],
    // };
    console.log('updateMapLayers');
    console.log('Post-merge data:', state.value.recvs);

    await nextTick();

    // Update the layer's source with new data
    const geoJSONSource = new Loca.GeoJSONSource({
      data: state.value.recvs,
    });

    recvLayer.value.setSource(geoJSONSource);

    // Update style to reflect changes in 'rstatus'
    recvLayer.value.setStyle({
      radius: 3.5,
      unit: 'px',
      color: (index, feature) => {
        console.log(index, feature);
        return feature.properties?.rstatus === 2 ? '3c3c3c' : '#0d79ca';
      },

      borderWidth: 0,
      blurWidth: 3.5,
    });

    recvLayer.value.show(); // Re-render the layer to apply the new style
  }

  function updateShotsLayer(data) {
    // Update shots layer with new data
    console.log('Updating shots layer:', data);
    state.value.shots1 = data.shots_data;
  }

  function setupWebSocket() {
    socket = new WebSocket('ws://localhost:8000/ws');

    socket.onopen = function () {
      console.log('WebSocket connection established');
    };

    socket.onmessage = function (event) {
      console.log('Data received from WebSocket');
      const updatedData = JSON.parse(event.data);
      if (updatedData.type === 'shots') {
        updateShotsLayer(updatedData.data);
      } else if (updatedData.type === 'receivers') {
        console.log('recevier');
        console.log(updatedData);
        updateMapLayers(updatedData.data);
      }
    };

    socket.onerror = function (error) {
      console.error('WebSocket error:', error);
    };
  }

  function mergeUpdates(original, updates) {
    if (!updates || !updates.features || !original || !original.features) {
      console.error('Invalid data provided for merging.');
      return original; // Return original or an empty template if preferred
    }
    console.log('original');
    console.log(original);
    console.log('updates');
    console.log(updates);
    // const updatedIndex = new Map(
    //   updates.features.map((f) => [f.properties.name, f])
    // );
    // console.log(updatedIndex);
    // 创建一个映射，以便快速访问更新数据
    const updateMap = new Map(
      updates.features.map((feature) => [feature.properties.name, feature])
    );

    // 遍历原始特征并尝试合并更新
    const newFeatures = original.features.map((feature) => {
      const updateFeature = updateMap.get(feature.properties.name);
      if (updateFeature) {
        // 更新已有特征的属性
        return updateFeature;
      }
      return feature;
    });

    // 将所有新的未存在于原始数据中的特征添加到数组中
    updates.features.forEach((feature) => {
      if (
        !newFeatures.some((f) => f.properties.name === feature.properties.name)
      ) {
        newFeatures.push(feature);
      }
    });

    // 更新原始数据的 features
    original.features = newFeatures;
    return original;
  }

  const newrecvdata = {
    type: 'FeatureCollection',
    features: [
      {
        type: 'Feature',
        properties: {
          name: '2758.0,1145.0',
          rstatus: 2,
        },
        geometry: {
          type: 'Point',
          coordinates: [106.359893, 30.819269],
        },
      },
    ],
  };

  const fetchData = async () => {
    const result = await getRecvs2();
    const shots = await getShots2();
    console.log(result);
    if (result) {
      state.value.recvs = result;
    }
    if (shots) {
      state.value.shots = shots;
    }
  };

  onMounted(async () => {
    await AMapLoader.load({
      key: 'ww', // 申请好的Web端开发者Key，首次调用 load 时必填
      version: '2.0', // 指定要加载的 JSAPI 的版本，缺省时默认为 1.4.15
      plugins: [], // 需要使用的的插件列表，如比例尺'AMap.Scale'等
      Loca: {
        // 是否加载 Loca， 缺省不加载
        version: '2.0.0', // Loca 版本，缺省 1.3.2
      },
    })
      .then(async (AMap) => {
        map = new AMap.Map('container', {
          // 设置地图容器id
          viewMode: '2D', // 是否为3D地图模式
          zoom: 13, // 初始化地图级别
          center: [106.449021, 30.828544], // 初始化地图中心点位置
          mapStyle: 'amap://styles/whitesmoke', // 设置地图的显示样式
        });
        // 创建 Loca 实例
        // eslint-disable-next-line no-undef
        loca = new Loca.Container({
          map,
        });
        await fetchData();

        console.log(state.value.recvs);
        // 创建圆点图层
        // eslint-disable-next-line no-undef
        recvLayer.value = new Loca.PointLayer({
          zIndex: 10,
          opacity: 1,
          visible: true,
        });
        // eslint-disable-next-line no-undef
        shotLayer.value = new Loca.PointLayer({
          zIndex: 11,
          opacity: 1,
          visible: true,
        });
        // eslint-disable-next-line no-undef
        const geoRecv = new Loca.GeoJSONSource({
          data: state.value.recvs,
        });
        // eslint-disable-next-line no-undef
        const geoShots = new Loca.GeoJSONSource({
          data: state.value.shots,
        });

        console.log(state.value.recvs);

        // 图层添加数据
        recvLayer.value.setSource(geoRecv);
        shotLayer.value.setSource(geoShots);

        // 设置样式
        // layer.setStyle({
        //   unit: 'px',
        //   icon: 'https://a.amap.com/Loca/static/loca-v2/demos/images/accident.png',
        //   iconSize: [40, 40],
        //   offset: [0, -40],
        //   rotation: 0,
        // });
        recvLayer.value.setStyle({
          radius: (index, feature) => {
            return feature.properties?.rstatus === 2 ? 4 : 3.5;
          },
          unit: 'px',
          color: (index, feature) => {
            return feature.properties?.rstatus === 2 ? '#3c3c3c' : '#0d79ca';
          },
          borderWidth: 0,
          blurWidth: 3.5,
        });

        shotLayer.value.setStyle({
          radius: 3.5,
          unit: 'px',
          color: '#f14124',
          borderWidth: 0,
          blurWidth: 3.5,
        });

        setupWebSocket(); // Setup WebSocket after the map is ready

        // 添加到地图上
        loca.add(recvLayer.value);
        loca.add(shotLayer.value);

        recvLayer.value.show();
        shotLayer.value.show();

        // 样式调试工具，仅用于开发阶段方便调试样式
        // 可以将可视化图层添加到调试工具中使用
        // eslint-disable-next-line no-undef
        const dat = new Loca.Dat();
        dat.addLayer(recvLayer.value, '示例的点图层');
      })
      .catch((e) => {
        console.log(e);
      });
  });

  onUnmounted(() => {
    map?.destroy();
    if (socket) {
      socket.close();
    }
  });
</script>

<style scoped lang="less">
  #container {
    padding: 0px;
    margin: 0px;
    width: 100%;
    height: 1000px;
  }
  .container1 {
    padding: 0 20px 20px;
  }

  :deep(.arco-table-th) {
    &:last-child {
      .arco-table-th-item-title {
        margin-left: 16px;
      }
    }
  }

  .action-icon {
    margin-left: 12px;
    cursor: pointer;
  }

  .active {
    color: #0960bd;
    background-color: #e3f4fc;
  }

  .setting {
    display: flex;
    align-items: center;
    width: 200px;

    .title {
      margin-left: 12px;
      cursor: pointer;
    }
  }
</style>
