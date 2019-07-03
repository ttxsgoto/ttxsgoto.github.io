---
title: vue-echarts
date: 2019-07-03 20:01:47
tags:
  - vue
  - echart
categories:
  - Frontend
---

封装定义可视化组件chart.vue
```bash
<template>
  <div :id="id" :style="style"></div>
</template>
<script>
    export default {
      name: "Chart",
      props: {
        id: {
          type: String
        },
        width: {
          type: String,
          default: "100%",
        },
        height: {
          type: String,
          default: "300px"
        },
        option: {
          type: Object,
          required: true,
        }
      },
      computed: {
        style() {
          return {
            height: this.height,
            width: this.width
          }
        }
      },
      data() {
          return {
            chart: ""
          };
      },
      methods: {
        init() {
          this.chart = this.echarts.init(document.getElementById(this.id));
          // this.chart.showLoading();
          this.chart.setOption(this.option);
          // this.chart.hideLoading();
          window.addEventListener("resize", this.chart.resize);
        }
      },
      mounted: function () {
        this.init();
      },
      updated: function () {
      },
      watch: {
        option: {
          handler(newVal, oldVal) {
            if (this.chart) {
              if (newVal) {
                this.chart.setOption(newVal);
              } else {
                this.chart.setOption(oldVal);
              }
            } else {
              this.init();
            }
          },
          deep: true
        }
      }
    }
</script>
```
