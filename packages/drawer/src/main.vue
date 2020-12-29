<template>
  <transition
    name="el-drawer-fade"
    @after-enter="afterEnter"
    @after-leave="afterLeave">
    <div
      class="el-drawer__wrapper"
      v-show="visible">
      <div
        class="el-drawer__container"
        :class="visible && 'el-drawer__open'"
        @click.self="handleWrapperClick"
        role="document">
        <div
          aria-modal="true"
          aria-labelledby="el-drawer__title"
          :aria-label="title"
          class="el-drawer"
          :class="[direction, customClass]"
          :style="drawerStyle"
          ref="drawer"
          role="dialog"
          >
          <header class="el-drawer__header" id="el-drawer__title" v-if="withHeader">
            <slot name="title">
              <span role="heading" :title="title">{{ title }}</span>
            </slot>
            <button
              :aria-label="`close ${title || 'drawer'}`"
              class="el-drawer__close-btn"
              type="button"
              v-if="showClose"
              @click="closeDrawer">
              <i class="el-dialog__close el-icon el-icon-close"></i>
            </button>
          </header>
          <section class="el-drawer__body" ref="body" v-if="rendered">
            <slot></slot>
          </section>
          <div
            class="el-drawer-drag"
            :class="direction"
            @mousedown="handleMousedown"
            v-if="resizable"
            ref="resizable"
            >
            <slot name="resizable">
              <div class="el-drawer-drag-move-trigger">
                <i v-for="i in 5" :key="i"></i>
              </div>
            </slot>
          </div>
        </div>
      </div>
    </div>
  </transition>
</template>

<script>
import Popup from 'element-ui/src/utils/popup';
import emitter from 'element-ui/src/mixins/emitter';
import Utils from 'element-ui/src/utils/aria-utils';

export default {
  name: 'ElDrawer',
  mixins: [Popup, emitter],
  props: {
    appendToBody: {
      type: Boolean,
      default: false
    },
    beforeClose: {
      type: Function
    },
    customClass: {
      type: String,
      default: ''
    },
    closeOnPressEscape: {
      type: Boolean,
      default: true
    },
    destroyOnClose: {
      type: Boolean,
      default: false
    },
    modal: {
      type: Boolean,
      default: true
    },
    direction: {
      type: String,
      default: 'rtl',
      validator(val) {
        return ['ltr', 'rtl', 'ttb', 'btt'].indexOf(val) !== -1;
      }
    },
    modalAppendToBody: {
      type: Boolean,
      default: true
    },
    showClose: {
      type: Boolean,
      default: true
    },
    size: {
      type: String,
      default: '30%'
    },
    title: {
      type: String,
      default: ''
    },
    visible: {
      type: Boolean
    },
    wrapperClosable: {
      type: Boolean,
      default: true
    },
    withHeader: {
      type: Boolean,
      default: true
    },
    focusFirst: {
      type: Boolean,
      default: true
    },
    resizable: {
      type: Boolean,
      default: false
    },
    maxSize: {
      type: String,
      default: 'calc(100% - 8px)'
    },
    minSize: {
      type: String
    }
  },
  computed: {
    isHorizontal() {
      return this.direction === 'rtl' || this.direction === 'ltr';
    },
    drawerStyle() {
      const { isHorizontal, maxSize, minSize, drawerSize } = this;
      return {
        [isHorizontal ? 'width' : 'height']: drawerSize,
        [isHorizontal ? 'maxWidth' : 'maxHeight']: maxSize,
        [isHorizontal ? 'minWidth' : 'minHeight']: minSize
      };
    }
  },
  data() {
    return {
      closed: false,
      drawerSize: ''
    };
  },
  watch: {
    visible(val) {
      if (val) {
        this.closed = false;
        this.$emit('open');
        if (this.appendToBody) {
          document.body.appendChild(this.$el);
        }
        this.prevActiveElement = document.activeElement;
        if (this.focusFirst) {
          this.$nextTick(() => {
            Utils.focusFirstDescendant(this.$refs.body);
          });
        }
      } else {
        if (!this.closed) this.$emit('close');
        if (this.prevActiveElement) {
          this.$nextTick(() => {
            this.prevActiveElement.focus();
            this.prevActiveElement = null;
          });
        }
      }
    }
  },
  methods: {
    afterEnter() {
      this.$emit('opened');
    },
    afterLeave() {
      this.$emit('closed');
    },
    init() {
      this.drawerSize = this.size;
      if (this.resizable) {
        window.addEventListener('mousemove', this.handleMouseMove);
        window.addEventListener('mouseup', this.handleMouseUp);
      }
    },
    hide(cancel) {
      if (cancel !== false) {
        this.$emit('update:visible', false);
        this.$emit('close');
        if (this.destroyOnClose === true) {
          this.rendered = false;
        }
        this.closed = true;
      }
    },
    handleWrapperClick() {
      if (this.wrapperClosable) {
        this.closeDrawer();
      }
    },
    closeDrawer() {
      if (typeof this.beforeClose === 'function') {
        this.beforeClose(this.hide);
      } else {
        this.hide();
      }
    },
    handleClose() {
      // This method here will be called by PopupManger, when the `closeOnPressEscape` was set to true
      // pressing `ESC` will call this method, and also close the drawer.
      // This method also calls `beforeClose` if there was one.
      this.closeDrawer();
    },
    handleMousedown(event) {
      if (event.button) return;
      this.dragging = true;
    },
    handleMouseMove(event) {
      if (!this.dragging) return;
      let drawerSize;
      if (this.isHorizontal) {
        drawerSize = this.direction === 'ltr' ? event.clientX : windown.innerWidth - event.clientX;
      } else {
        drawerSize = this.direction === 'ttb' ? event.clientY : windown.innerHeight - event.clientY;
      }
      this.drawerSize = `${drawerSize}px`;
      this.$emit('resize', drawerSize);
    },
    handleMouseUp() {
      this.dragging = false;
    }
  },
  mounted() {
    this.init();
    if (this.visible) {
      this.rendered = true;
      this.open();
    }
  },
  destroyed() {
    // if appendToBody is true, remove DOM node after destroy
    if (this.appendToBody && this.$el && this.$el.parentNode) {
      this.$el.parentNode.removeChild(this.$el);
    }
    if (this.resizable) {
      window.removeEventListener('mousemove', this.handleMouseMove);
      window.removeEventListener('mouseup', this.handleMouseUp);
    }
  }
};
</script>
