<template>
  <div class="dialog">
    <div class="dialog__wrap" :style="style">
      <div class="dialog__header">
        <h2>{{title}}</h2>
        <button class="dialog__close" @click="close" title="Закрыть">X</button>
      </div>
      <div class="dialog__content">
        <slot></slot>
      </div>
    </div>
  </div>
</template>

<script>

export default {
  props: ["width", "height", "title"],
  computed: {
    style() {
      return {
        width: this.width || "500px",
        height: this.height
      }
    }
  },
  methods: {
    close() {
      this.$emit("close");
    }
  }
}
</script>


<style scoped>
button {
  cursor: pointer;
  outline: none;
}
.dialog__close {
  background: none;
  border: none;
  font-size: 24px;
}
.dialog {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  display: flex;
  background: rgba(188,188,188,0.6);
}
.dialog__wrap {
  margin: auto;
  background: #fff;
  position: relative;
}
.dialog__header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 5px 15px;
  margin-bottom: 20px;
}
.dialog__content {
  padding: 5px 15px;
  margin-bottom: 20px;
}
</style>
