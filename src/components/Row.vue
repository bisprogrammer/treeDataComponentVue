<template>
  <tr>
    <td :style="{ 'padding-left': item.level ? `${15 * item.level}px` : '15px' }">
      <template v-if="item.children">
        <button v-if="item.expanded" @click="collapse(item)">-</button>
        <button v-else @click="expand(item)">+</button>
      </template>
      {{item.name}}
    </td>
    <td>{{item.phone}}</td>
  </tr>
</template>

<script>
export default {
  name: "row",
  props: ['item'],
  methods: {
    expand(treeData) {
      this.$emit("expand", treeData);
    },
    collapse(treeData) {
      this.$emit("collapse", treeData);
    }
  }
}
</script>

<style scoped>
td {
  padding: 10px 15px;
}

button {
  cursor: pointer;
}
</style>
