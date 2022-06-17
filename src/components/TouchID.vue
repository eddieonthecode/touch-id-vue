<template>
  <div
    class="touch-id d-flex"
    :class="[
      {
        moving: isMoving,
        pin: isPin,
        'flex-row': GetSide == 'top' || GetSide == 'bottom',
        'flex-column': GetSide == 'right' || GetSide == 'left',
        'align-items-start': GetSide == 'top' || GetSide == 'left',
        'align-items-end': GetSide == 'bottom' || GetSide == 'right',
      },
      GetSide,
    ]"
    ref="dragable"
    @mousedown="mousedown($event)"
  >
    <div
      class="touch-id__collapse flex-center"
      :class="[
        {
          'm-r-4': GetSide == 'top' || GetSide == 'bottom',
          'm-b-4': GetSide == 'right' || GetSide == 'left',
        },
        GetSide,
      ]"
      :style="{
        visibility: isPin ? 'visible' : 'hidden',
      }"
      @mousedown="mousedownCollapse()"
      @mouseup="mouseupCollapse()"
    >
      <i class="fa-solid fa-chevron-right icon--collapse"></i>
    </div>
    <div class="touch-id__main">
      <div
        ref="extend"
        class="touch-id__extend flex-center"
        :class="{
          horizontal: GetSide == 'top' || GetSide == 'bottom',
          vertical: GetSide == 'left' || GetSide == 'right',
        }"
      >
        <i class="fa-solid fa-chevron-left icon--extend"></i>
      </div>
      <div
        class="touch-id__function flex-center"
        title="Chat với nhân viên tư vấn"
        :class="{
          horizontal: GetSide == 'top' || GetSide == 'bottom',
          vertical: GetSide == 'left' || GetSide == 'right',
        }"
        @mousedown="mousedownFunction()"
        @mouseup="mouseupFunction()"
      >
        <i class="fa-solid fa-comment-dots"></i>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'TouchID',
  props: {
    Side: String,
    Percent: Number,
  },
  data() {
    return {
      SideCurr: '',
      PercentCurr: 0,
      isPin: false,
      startX: 0,
      startY: 0,
      offsetVertical: 50,
      modeCollapse: '',
      collapseTimeout: null,
      modeFunction: '',
      functionTimeout: null,
      element: null,
      extendElement: null,
      isMoving: false,
      isShowLiveChat: false,
    };
  },
  mounted() {
    this.SideCurr = this.Side;
    this.PercentCurr = this.Percent;

    // Sự kiện di chuột khắp màn hình
    document.addEventListener('mousemove', (e) => {
      if (this.isMoving) {
        this.clearSelection();
        var outWindow = this.edgeWindow(e);
        if (!outWindow.X) {
          this.element.style.left = e.clientX - this.startX + 'px';
          this.element.style.right = 'unset';
        }
        if (!outWindow.Y) {
          this.element.style.top = e.clientY - this.startY + 'px';
          this.element.style.bottom = 'unset';
        }
      }
    });

    // Sự kiện thay đổi độ rộng màn hình
    window.addEventListener('resize', () => {
      if (this.element) {
        this.setPosition();
      }
    });

    // Sự kiện nhấc chuột lên
    document.addEventListener('mouseup', (e) => {
      if (this.isMoving) {
        this.isMoving = false;
        this.onStopMove();
      }
    });

    setTimeout(() => {
      this.element = this.$refs.dragable;
      this.extendElement = this.$refs.extend;
      // Đặt vị trí khởi tạo
      this.setPosition();
    }, 300);

    let interval = setInterval(() => {
      if (window.LiveChatWidget) {
        // window.LiveChatWidget.call('hide');
        [
          'ready',
          'new_event',
          'visibility_changed',
          'availability_changed',
        ].forEach((eventName) => {
          window.LiveChatWidget.on(eventName, (payload) => {
            console.log(eventName);
            if (eventName == 'ready') {
              document.querySelector('#chat-widget-container').style.display =
                'block';
              // window.LiveChatWidget.call('hide');
            }
            if (payload.visibility == 'minimized') {
              setTimeout(() => {
                window.LiveChatWidget.call('hide');
              }, 200);
            }
          });
        });
        clearInterval(interval);
      }
    });
  },
  computed: {
    GetSide() {
      return this.SideCurr ? this.SideCurr.toLowerCase() : 'right';
    },
  },
  methods: {
    /**
     * Đặt vị trí
     * createdby ntdung5 13.06.2022
     */
    setPosition() {
      this.removePosition(this.element);
      this.removePosition(this.extendElement);

      switch (this.SideCurr) {
        case 'Top':
          this.element.style.top = 0;
          this.extendElement.style.top = 0;

          break;
        case 'Bottom':
          this.element.style.bottom = 0;
          this.extendElement.style.bottom = 0;
          break;
        case 'Right':
          this.element.style.right = 0;
          this.extendElement.style.right = 0;

          break;
        case 'Left':
          this.element.style.left = 0;
          this.extendElement.style.left = 0;
          break;
        default:
          break;
      }
      var rectElement = this.element.getBoundingClientRect();

      if (this.SideCurr == 'Top' || this.SideCurr == 'Bottom') {
        var left = (window.innerWidth * this.PercentCurr) / 100;
        if (left + rectElement.width >= window.innerWidth) {
          this.element.style.right = 0;
          this.extendElement.style.right = 0;
        } else {
          this.element.style.left = left + 'px';
          this.extendElement.style.left = left + 'px';
        }
      } else {
        this.element.style.top =
          (window.innerHeight * this.PercentCurr) / 100 + 'px';
        this.extendElement.style.top =
          (window.innerHeight * this.PercentCurr) / 100 + 'px';
      }
    },
    /**
     * Sự kiện ấn chuột xuống
     * createdby ntdung5 13.06.2022
     */
    mousedown(e) {
      this.element = this.$refs.dragable;
      this.extendElement = this.$refs.extend;
      this.isMoving = true;
      this.startX = e.clientX - parseInt(getComputedStyle(this.element).left);
      this.startY = e.clientY - parseInt(getComputedStyle(this.element).top);
    },

    /**
     * Xử lý sự kiện bấm nút phân biệt với sự kiện kéo thả
     * created by ntdung5 13.06.2022
     */
    mousedownCollapse() {
      this.modeCollapse = 'click';
      clearTimeout(this.collapseTimeout);
      this.collapseTimeout = setTimeout(() => {
        this.modeCollapse = 'move';
      }, 200);
    },

    mouseupCollapse() {
      if (this.modeCollapse == 'click') {
        this.isPin = false;
      }
    },
    mousedownFunction() {
      this.modeFunction = 'click';
      clearTimeout(this.functionTimeout);
      this.functionTimeout = setTimeout(() => {
        this.modeFunction = 'move';
      }, 200);
    },

    mouseupFunction() {
      console.log(this.modeFunction);
      if (this.modeFunction == 'click') {
        this.isPin = true;
        if (window.LiveChatWidget) {
          if (window.LiveChatWidget.get('state').visibility == 'hidden') {
            window.LiveChatWidget.call('maximize');
          } else {
            window.LiveChatWidget.call('hide');
          }
        } else {
          window.LiveChatWidget.call('maximize');
        }
        if (this.isShowLiveChat) {
          window.LiveChatWidget.call('hide');
          this.isShowLiveChat = false;
        } else {
          window.LiveChatWidget.call('maximize');
          this.isShowLiveChat = true;
        }
      }
    },

    /**
     * Dừng di chuyển
     * createdby ntdung5 13.06.2022
     */
    onStopMove() {
      var rectElement = this.element.getBoundingClientRect();
      var PercentTop = Number.parseFloat(
        rectElement.top / window.innerHeight + ''
      ).toFixed(5);
      var PercentLeft = Number.parseFloat(
        rectElement.left / window.innerWidth + ''
      ).toFixed(5);

      if (rectElement.top <= this.offsetVertical) {
        this.SideCurr = 'Top';
        this.PercentCurr = Number(PercentLeft) * 100;
      } else if (
        window.innerHeight - rectElement.bottom <=
        this.offsetVertical
      ) {
        this.SideCurr = 'Bottom';
        this.PercentCurr = Number(PercentLeft) * 100;
      } else {
        if (Number(PercentLeft) < 0.5) {
          this.SideCurr = 'Left';
          this.PercentCurr = Number(PercentTop) * 100;
        } else {
          this.SideCurr = 'Right';
          this.PercentCurr = Number(PercentTop) * 100;
        }
      }

      // Đặt lại vị trí
      this.setPosition();
    },
    /**
     * Con trỏ chuột nằm ngoài màn hình window
     * createdby ntdung5 13.06.2022
     */
    edgeWindow(e) {
      var result = { X: true, Y: true };
      if (this.element) {
        var rectElement = this.element.getBoundingClientRect();
        var positionX = e.clientX - this.startX;
        var positionY = e.clientY - this.startY;
        if (
          positionX >= 0 &&
          positionX + rectElement.width <= window.innerWidth
        ) {
          result.X = false;
        }
        if (
          positionY >= 0 &&
          positionY + rectElement.height <= window.innerHeight
        ) {
          result.Y = false;
        }
        return result;
      }
    },

    /**
     * Xóa vùng đang chọn
     * createdby ntdung5 13.06.2022
     */
    clearSelection() {
      if (window.getSelection()) {
        if (window.getSelection().empty) {
          window.getSelection().empty();
        } else if (window.getSelection().removeAllRanges) {
          window.getSelection().removeAllRanges();
        }
      }
      // else if (document.selection) {
      //   document.selection.empty();
      // }
    },

    /**
     * Loại bỏ vị trí của element
     * createdby ntdung5 13.06.2022
     */
    removePosition(element) {
      element.style.inset = 'unset';
    },
  },
};
</script>

<style lang="scss" scoped>
$color: #2566e9;

[aria-label*='widget'] {
  display: none !important;
}

.flex-center {
  display: flex;
  align-items: center;
  justify-content: center;
}

.touch-id {
  position: fixed;
  z-index: 2147483640;
  &:active {
    cursor: all-scroll;
  }
  $r: &;
  &__function,
  &__extend {
    width: fit-content;
    background-color: $color;
    color: #fff;
    cursor: pointer;

    &:active {
      cursor: all-scroll;
    }
  }
  #{$r}__extend {
    position: fixed;
    height: 70px;
    padding: 4px;
    &.vertical {
      margin-top: 28px;
    }
    &.horizontal {
      width: 70px;
      height: fit-content;
      margin-left: 24px;
    }
  }
  #{$r}__function {
    transition: transform 0.3s ease;
    font-size: 30px;
    padding: 8px 8px;
    &.vertical {
      margin-bottom: 28px;
    }
    &.horizontal {
      margin-right: 24px;
    }
  }
  #{$r}__collapse {
    cursor: pointer;
    background-color: $color;
    padding: 6px 6px;
    width: fit-content;
    height: fit-content;
    color: #fff;
    font-size: 12px;
  }

  &.top {
    #{$r}__extend,
    #{$r}__function,
    #{$r}__collapse {
      border-bottom-right-radius: 4px;
      border-bottom-left-radius: 4px;
    }
    #{$r}__function {
      transform: translateY(-100%);
    }
    #{$r}__collapse {
      transform: translateY(-100%);
    }
    #{$r}__main:hover,
    &.pin {
      #{$r}__extend {
        display: none;
      }
      #{$r}__function {
        transform: translateY(0);
      }
      #{$r}__collapse {
        transform: translateY(0);
      }
    }
    .icon--extend {
      transform: rotate(270deg);
    }
    .icon--collapse {
      transform: rotate(270deg);
    }
  }
  &.bottom {
    #{$r}__extend,
    #{$r}__function,
    #{$r}__collapse {
      border-top-left-radius: 4px;
      border-top-right-radius: 4px;
    }
    #{$r}__function {
      transform: translateY(100%);
    }
    #{$r}__collapse {
      transform: translateY(100%);
    }
    #{$r}__main:hover,
    &.pin {
      #{$r}__extend {
        display: none;
      }
      #{$r}__function {
        transform: translateY(0);
      }
      #{$r}__collapse {
        transform: translateY(0);
      }
    }
    .icon--extend {
      transform: rotate(90deg);
    }
    .icon--collapse {
      transform: rotate(90deg);
    }
  }
  &.right {
    #{$r}__extend,
    #{$r}__function,
    #{$r}__collapse {
      border-bottom-left-radius: 4px;
      border-top-left-radius: 4px;
    }
    #{$r}__function {
      transform: translateX(100%);
    }
    #{$r}__collapse {
      transform: translateX(100%);
    }
    #{$r}__main:hover,
    &.pin {
      #{$r}__extend {
        display: none;
      }
      #{$r}__function {
        transform: translateX(0);
      }
      #{$r}__collapse {
        transform: translateX(0);
      }
    }
  }
  &.left {
    #{$r}__extend,
    #{$r}__function,
    #{$r}__collapse {
      border-bottom-right-radius: 4px;
      border-top-right-radius: 4px;
    }
    #{$r}__function {
      transform: translateX(-100%);
    }
    #{$r}__collapse {
      transform: translateX(-100%);
    }
    #{$r}__main:hover,
    &.pin {
      #{$r}__extend {
        display: none;
      }
      #{$r}__function {
        transform: translateX(0);
      }
      #{$r}__collapse {
        transform: translateX(0);
      }
    }
    .icon--extend {
      transform: rotate(180deg);
    }
    .icon--collapse {
      transform: rotate(180deg);
    }
  }
  &.moving {
    #{$r}__extend {
      display: none;
    }
    #{$r}__function,
    #{$r}__collapse {
      transform: none !important;
    }
  }
}

.m-t-4 {
  margin-top: 4px;
}
.m-b-4 {
  margin-bottom: 4px;
}
.m-r-4 {
  margin-right: 4px;
}
.m-l-4 {
  margin-left: 4px;
}
</style>
