<template>
  <div class="Components" :style="`height:${realHeight}px`">
    <div class="main-category-components" v-if="showCateName === 'main'">
      <div class="main-item" v-for="item in allScreenComponents" :key="item.id" @click="handleChooseMainCate(item)">
        {{ item.name }}
      </div>
    </div>

    <el-container v-if="showCateName === 'sub-main'">
      <el-header @click="showCateName = 'main'" height="35px">
        <i class="el-icon-arrow-left"></i>
        <span> {{ subMainDate.name }}</span>
      </el-header>
      <el-main :style="`height:${realHeightMain}px`">
        <div class="sub-main-item" @click="handleChooseSubCate(item)" v-for="item in subMainDate.subSet" :key="item.id">
          {{ item.name }}
        </div>
      </el-main>
    </el-container>

    <el-container v-if="showCateName === 'sub-sub'">
      <el-header @click="showCateName = 'sub-main'" height="35px">
        <i class="el-icon-arrow-left"></i>
        <span>{{ subSubData.name }}</span>
      </el-header>
      <el-main :style="`height:${realHeightMain}px`" class="components-container">
        <div
          class="component-item"
          @mousedown="handleAddComponent($event, item)"
          v-for="item in subSubData.subSet"
          :key="item.id"
          ondragstart="return false;"
        >
          <img :src="item.icon" alt="" />
          <img
            class="img-drag"
            v-if="readyAddComponentName === item.componentName"
            :style="readyAddComponetIconStyle"
            :src="item.icon"
          />
          <span class="component-name"> {{ item.name }}</span>
        </div>
      </el-main>
    </el-container>
  </div>
</template>

<script>
import { mapState, mapMutations } from 'vuex';
import lodash from 'lodash';
export default {
  name: 'Components',
  components: {},
  setup() {},
  data() {
    return {
      readyAddComponetIconStyle: {
        position: 'fixed',
        zIndex: 99999,
        // display: 'none',
        opacity: 0.4,
        left: '0px',
        top: '0px',
      },
      readyAddComponentName: '',
      showCateName: 'main',
      subMainDate: null,
      subSubData: null,
      realHeight: 0,
      realHeightMain: 0,
    };
  },
  created() {},
  beforeMount() {
    console.log(this.allScreenComponents);
  },
  mounted() {
    this.getRealHeight();
    window.addEventListener('resize', this.onResize);
  },
  unmounted() {
    window.removeEventListener('resize', this.onResize);
  },
  watch: {},
  computed: {
    ...mapState({
      allScreenComponents: state => state.screenComponents,
      screenPosition: state => state.workbench.screen.position,
    }),
  },
  methods: {
    ...mapMutations(['workbench/addComponentToCurrentScene']),
    getRealHeight() {
      this.realHeight = document.body.clientHeight - 110;
      this.realHeightMain = document.body.clientHeight - 150;
    },
    handleChooseMainCate(data) {
      console.log(data);
      this.showCateName = 'sub-main';
      this.subMainDate = data;
    },
    handleChooseSubCate(data) {
      this.showCateName = 'sub-sub';
      this.subSubData = data;
    },
    handleAddComponent(event, component) {
      event.stopPropagation();
      this.readyAddComponentName = component.componentName;
      const addComponent = lodash.cloneDeep(component);
      const startY = event.clientY;
      const startX = event.clientX;
      this.readyAddComponetIconStyle.top = startY - 50 + 'px';
      this.readyAddComponetIconStyle.left = startX + -50 + 'px';
      const startTop = parseInt(this.readyAddComponetIconStyle.top);
      const startLeft = parseInt(this.readyAddComponetIconStyle.left);
      let move = moveEvent => {
        const currX = moveEvent.clientX;
        const currY = moveEvent.clientY;
        this.readyAddComponetIconStyle.top = currY - startY + startTop + 'px';
        this.readyAddComponetIconStyle.left = currX - startX + startLeft + 'px';
      };
      const up = upEvent => {
        this.readyAddComponentName = '';
        const { top: screenTop, left: screenLeft } = this.screenPosition;
        addComponent.componentStyle.containerStyle.left = upEvent.clientX - 270 - screenLeft;
        addComponent.componentStyle.containerStyle.top = upEvent.clientY - 70 - screenTop;
        setTimeout(() => {
          this['workbench/addComponentToCurrentScene'](addComponent);
        }, 100);

        document.removeEventListener('mousemove', move);
        document.removeEventListener('mouseup', up);
      };
      document.addEventListener('mousemove', move);
      document.addEventListener('mouseup', up);
    },
  },
};
</script>

<style lang="scss" scoped>
.Components {
  width: 100%;
  height: 100%;
  .el-container {
    width: 100%;
    height: 100%;
    .el-header {
      user-select: none;
      color: #ccc;
      background-color: rgb(37, 37, 37);
      line-height: 35px;
      text-align: center;
      font-size: 14px;
      position: relative;
      .el-icon-arrow-left {
        position: absolute;
        cursor: pointer;
        left: 10px;
        top: 50%;
        transform: translateY(-50%);
        &:hover {
          color: #ccc;
        }
      }
    }
    .el-main {
      display: flex;
      flex-wrap: wrap;
      padding: 20px 0 0;
      .sub-main-item {
        font-size: 14px;
        letter-spacing: 1px;
        color: #c2c7c7;
        box-sizing: border-box;
        cursor: pointer;
        user-select: none;
        width: 100px;
        height: 80px;
        line-height: 80px;
        text-align: center;
        background-color: rgba(35, 35, 35, 0.89);
        margin-left: 22px;
        margin-bottom: 20px;
        box-shadow: 2px 2px 8px rgba(23, 23, 23, 0);
        border-radius: 6px;
        border: 1.5px solid rgba(0, 0, 0, 0.2);
        transition: all 0.2s linear;
        &:hover {
          box-shadow: 1px 1px 10px #151515;
          border: 1.5px solid #8063ff;
          color: #fff;
        }
      }
    }
    .components-container {
      display: flex;
      flex-direction: column;
      align-items: center;
      padding: 25px 0 0;
      .component-item {
        user-select: none;
        width: 220px;
        height: 120px;
        margin-bottom: 25px;
        position: relative;
        background: rgb(29, 29, 29);
        border-radius: 5px;
        .component-name {
          position: absolute;
          top: 10px;
          left: 10px;
          color: #ccc;
        }
        img {
          width: 100%;
          height: 100%;
          position: absolute;
          top: 0;
        }
        .img-drag {
          overflow: hidden;
          position: absolute;
          top: 0;
          width: 100px;
          height: 60px;
        }
      }
    }
  }
  .main-category-components {
    width: 100%;
    display: flex;
    flex-wrap: wrap;
    padding-top: 20px;
    .main-item {
      font-size: 14px;
      letter-spacing: 1px;
      color: #c2c7c7;
      box-sizing: border-box;
      cursor: pointer;
      user-select: none;
      width: 100px;
      height: 62px;
      line-height: 62px;
      text-align: center;
      background-color: rgba(35, 35, 35, 0.89);
      margin-left: 22px;
      margin-bottom: 20px;
      box-shadow: 2px 2px 8px rgba(23, 23, 23, 0);
      border-radius: 6px;
      border: 1.5px solid rgba(0, 0, 0, 0.2);
      transition: all 0.2s linear;
      &:hover {
        box-shadow: 1px 1px 10px #151515;
        border: 1.5px solid #8063ff;
        color: #fff;
      }
    }
  }
}
</style>
