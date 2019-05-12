<template>
  <div
    class="test-range"
    ref="container"
    @mousedown.stop="handleMouseDown"
    @mousemove.stop="handleMouseMove"
  >
    <div class="test-range__select" ref="select"></div>
    <div class="test-range__content">
      <div class="title-wrapper">
        <span class="range">时间段</span>
      </div>
      <div class="date-wrapper">
        <div class="time-wrapper">
          <div class="test-range__item" v-for="noon in splitHelperArr" :key="noon.key">
            <div class="inner-wrapper" v-for="range in noon.timeRange" :key="range.label">
              <label>{{range.label}}</label>
              <div class="six-time-wrapper">
                <div
                  class="test-range__item-inner"
                  :class="tick.class"
                  v-for="tick in range.range"
                  :key="tick.time"
                  :data-time="tick.time"
                >{{tick.time}}</div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "Demo",
  data() {
    return {
      timeRange: [],
      mouseStopId: undefined,
      mouseOn: false
    };
  },
  created() {
    this.timeRange = this.getDefaultTimeRange();
  },
  mounted() {
    document.addEventListener("mouseup", this.handleMouseUp);
  },
  destroyed() {
    document.removeEventListener("mouseup", this.handleMouseUp);
  },
  computed: {
    splitHelperArr() {
      const splitHelperArr = [
        {
          key: "morning",
          timeRange: this.timeRange.slice(0, 2)
        },
        {
          key: "afternoon",
          timeRange: this.timeRange.slice(2, 4)
        }
      ];

      return splitHelperArr;
    },
    flattenTimeRange() {
      return this.timeRange.reduce((acc, curr) => {
        return acc.concat(curr.range);
      }, []);
    }
  },
  methods: {
    getDefaultTimeRange() {
      const labels = ["0:00—05:59", "6:00—11:59", "12:00—17:59", "18:00—23:59"];
      const timeRange = labels.map((p, index) => {
        return {
          label: p,
          range: new Array(6).fill().map((_, n) => {
            return {
              time: n + 6 * index,
              class: "is-deactive"
            };
          })
        };
      });
      return timeRange;
    },
    clearEditing() {
      Array.from(
        document.getElementsByClassName("test-range__item-inner")
      ).forEach(p => p.classList.remove("is-editing"));
    },
    handleEdit(todoList) {
      alert(JSON.stringify(todoList));
    },
    handleMouseDown(e) {
      // 判断是否为鼠标左键被按下
      if (e.buttons !== 1 || e.which !== 1) return;
      this.mouseStopId = window.setTimeout(() => {
        this.mouseOn = true;
        // 设置选框的初始位置
        this.startX = e.clientX;
        this.startY = e.clientY;
      }, 300);

      console.log(this.startX, "startX");
      console.log(this.startY, "startY");

      this.$refs.select.style.cssText = `display:block;
                                         left:${this.startX}px;
                                         top:${this.startY}px
                                         width:0;
                                         height:0;`;
    },
    handleMouseMove(e) {
      if (!this.mouseOn) return;
      const $select = this.$refs.select;
      const _w = e.clientX - this.startX;
      const _h = e.clientY - this.startY;
      this.top = _h > 0 ? this.startY : e.clientY;
      this.left = _w > 0 ? this.startX : e.clientX;
      this.width = Math.abs(_w);
      this.height = Math.abs(_h);
      $select.style.left = `${this.left}px`;
      $select.style.top = `${this.top}px`;
      $select.style.width = `${this.width}px`;
      $select.style.height = `${this.height}px`;
      const selList = document.getElementsByClassName(
        "test-range__item-inner"
      );

      const { bottom, left, right, top } = $select.getBoundingClientRect();

      for (let i = 0; i < selList.length; i++) {
        const rect = selList[i].getBoundingClientRect();

        const isIntersect = !(
          rect.top > bottom ||
          rect.bottom < top ||
          rect.right < left ||
          rect.left > right
        );
        selList[i].classList[isIntersect ? "add" : "remove"]("is-editing");
      }
    },
    handleMouseUp(e) {
      this.mouseStopId && window.clearTimeout(this.mouseStopId);
      if (!this.mouseOn) return;
      this.mouseOn = false;
      const $select = this.$refs.select;
      const selList = document.getElementsByClassName(
        "test-range__item-inner"
      );

      const { bottom, left, right, top } = $select.getBoundingClientRect();

      const todoList = [];
      for (let i = 0; i < selList.length; i++) {
        const rect = selList[i].getBoundingClientRect();

        const isIntersect = !(
          rect.top > bottom ||
          rect.bottom < top ||
          rect.right < left ||
          rect.left > right
        );
        if (isIntersect) {
          const curTime = this.flattenTimeRange.find(
            p => p.time == selList[i].dataset.time
          );
          todoList.push(curTime);
        }
      }
      $select.style.display = "none";
      //批量操作
      if (todoList.length) {
        this.handleEdit(todoList);
      }
    }
  }
};
</script>

<style lang="scss">
.test-range {
  width: 700px;
  font-size: 12px;
  position: relative;
  user-select: none;
  .legend-wrapper {
    font-size: 0;

    .legend {
      display: inline-block;
      height: 14px;
      width: 14px;
      border-radius: 2px;
      margin-right: 10px;
      vertical-align: -2px;
    }

    .txt {
      display: inline-block;
      font-size: 12px;
      margin-right: 20px;

      &:last-of-type {
        margin-right: 0;
      }
    }

    .active-legend {
      background: #0590ff;
    }

    .deactive-legend {
      background: #e9ebee;
    }
  }
}

.test-range__title {
  display: flex;
  justify-content: space-between;
  margin-bottom: 15px;
}

.test-range__content {
  border: 1px solid #dddee1;
  border-radius: 4px 0;

  .title-wrapper {
    display: flex;

    .allpick {
      height: 40px;
      width: 48px;
      text-align: center;
      line-height: 40px;
      border: 1px solid #dddee1;
      border-top-width: 0;
      border-left-width: 0;
    }

    .range {
      width: calc(100% - 48px);
      text-align: center;
      height: 40px;
      line-height: 40px;
      border-bottom: 1px solid #dddee1;
    }
  }

  .checkbox-wrapper {
    width: 48px;
    height: 148px;
    text-align: center;
    line-height: 148px;
    border-right: 1px solid #dddee1;
  }

  .date-wrapper {
    display: flex;
  }

  .test-range__item {
    display: flex;
    padding: 10px 15px;
    border-bottom: 1px solid #dddee1;

    &:last-of-type {
      border-bottom: 0;
    }

    .inner-wrapper {
      display: flex;
      flex-direction: column;
      align-items: center;
      margin-right: 10px;

      &:last-of-type {
        margin-right: 0;
      }

      label {
        display: block;
        margin-bottom: 8px;
      }

      .six-time-wrapper {
        display: flex;
      }
    }
  }

  .test-range__item-inner {
    width: 50px;
    height: 32px;
    line-height: 32px;
    text-align: center;
    font-size: 14px;
    margin-right: 1px;
    cursor: pointer;

    &:last-child {
      margin-right: 0;
    }

    &.is-deactive {
      background: #e9ebee;
      color: #495060;
    }

    &.is-active {
      background: #0590ff;
      color: #fff;
    }

    &.is-deactive.is-editing {
      border: 1px dashed #0590ff;
    }

    &.is-active.is-editing {
      border: 1px dashed #fff;
    }
  }
}

.test-range__select {
  background: #598fe6;
  position: fixed;
  width: 0;
  height: 0;
  display: none;
  top: 0;
  left: 0;
  opacity: 0.6;
  pointer-events: none;
}
</style>
