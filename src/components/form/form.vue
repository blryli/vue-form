<template>
  <form ref="vueForm" v-if="reload" class="vue-form" :class="formClass" :style="{ margin: `0 -${itemGutter / 2}px` }">
    <slot></slot>
    <vue-form-mark v-if="mark" :id="clickItemId" :markEffect="markEffect" :markContent="markContent"></vue-form-mark>
  </form>
</template>

<script>
import vueFormMark from 'components/form-mark'

export default {
  name: "VueForm",
  componentName: "VueForm",
  props: {
    model: [Object, Array],
    layer: {
      type: Array,
      default: () => []
    },
    labelWidth: String,
    labelPosition: String,
    lineHeight: {
      type: String,
      default: "32px"
    },
    itemGutter: {
      type: Number,
      default: 0
    },
    response: {
      type: Boolean,
      default: true
    },
    rowledge: {
      type: String,
      default: "24px"
    },
    isTable: Boolean,
    mark: Boolean,
    markEffect: String,
    markContent: Function
  },
  components: { vueFormMark },
  data() {
    return {
      initModel: Object.freeze(null),
      isResponse: false,
      initLayer: Object.freeze([]),
      reload: true,
      clickItemId: null
    };
  },
  computed: {
    formClass() {
      let formClass = "";
      this.labelPosition &&
        (formClass += `vue-form--label-${this.labelPosition}`);
      this.response && this.isResponse && (formClass += " vue-form-response");
      return formClass;
    }
  },
  created() {
    this.layerComponents = [];
    this.$on("layer.add", obj => {
      this.layerComponents.push(obj);
    });
    this.init();
  },
  watch: {
    layer: {
      handler() {
        this.reload = false;
        this.initLayer = Object.freeze([]);
        this.$nextTick(() => {
          this.reload = true;
          this.init();
        });
      },
      deep: false
    }
  },
  methods: {
    formationLayer() {
      return (this.layer || []).reduce((acc, cur) => {
        const show = cur.show === undefined ? true : cur.show;
        (cur.data || []).forEach(da => {
          da.id = cur.id;
          const layer = { ...cur.view, ...da, ...{show: show} };
          const findIndex = acc.findIndex(l => l.prop === da.prop);
          if (findIndex === -1) {
            acc.push({
              prop: da.prop,
              layer: [layer]
            });
          } else {
            acc[findIndex].layer.push(layer);
          }
        });
        return acc;
      }, [])
    },
    init() {
      this.initLayer = Object.freeze(this.formationLayer());
      this.initModelFn();
    },
    initModelFn() {
      this.model &&
        (this.initModel = Object.freeze(
          JSON.parse(JSON.stringify(this.model))
        ));
    },
    changeShow(id) {
      !id && console.error(`changeShow 方法必须传入 layer id`);
      if (!this.layer.find(d => d.id && d.id === id)) console.error(`not find ${id} in layer`)
      this.layerComponents.forEach(d => {
        d.layer.changeShow(id)
      })
    },
    recalculate(id, callback) {
      if (typeof callback !== "function") {
        console.error("recalculate方法 回调参数必须是 函数");
        return;
      }
      if (!id) {
        console.error("recalculate方法 必须传入 layer ID");
        return;
      }
      let valid = true;
      this.layerComponents.forEach(d => {
        d.layer.recalculateField(id, null, fieldCb => {
          fieldCb.message && (valid = false)
          this.$emit('recalculate', fieldCb)
        })
      })
      callback(valid);
    },
    recalculateField(id, prop) {
      !id && console.error(`recalculateField 方法必须传入 layer id`);
      !this.model && console.error(`model is not define`);
      this.layerComponents.forEach(d => {
        d.layer.recalculateField(id, prop, fieldCb => {
          this.$emit('recalculate', fieldCb)
        })
      })
    },
    clearCalculate(id, props = [], resetModel) {
      this.layerComponents.forEach(d => {
        d.layer.clearCalculate(id, props = [], resetModel)
      })
    },
    resetData(prop) {
      // 初始化值
      if (!this.initModel) return;
      !this.model && console.error(`model is not define`);

      if (Array.isArray(this.model)) {
        (this.model || []).forEach((d, i) => {
          this.$set(d, prop, this.initModel[i][prop] || "");
        });
      } else {
        this.$set(this.model, prop, this.initModel[prop] || "");
      }
    },
    resetFields(id, props = []) {
      this.clearCalculate(id, props, true);
    }
  },
  mounted() {
    if (this.response) {
      let width;
      if (window.innerWidth) {
        width = window.innerWidth;
      } else if (document.body && document.body.clientWidth) {
        width = document.body.clientWidth;
      }
      if (width <= 768) {
        this.isResponse = true;
      }
    }
  }
};
</script>
