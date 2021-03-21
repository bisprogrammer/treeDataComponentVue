<template>
  <div>
    <table border="1">
      <thead>
        <tr>
          <th class="sortable" @click="sort">Имя
            <font-awesome-icon v-if="sortValue" :icon="sortValue.icon"></font-awesome-icon>
          </th>
          <th>Телефон</th>
        </tr>
      </thead>
      <tbody>
        <Row v-for="item in flattenedTreeData" :key="item.id" :item="item" @expand="expand" @collapse="collapse" />
      </tbody>
    </table>
  </div>
</template>

<script>
import Row from "./Row";

export default {
  props: ['treeData', 'expanded'],
  data() {
    return {
      flattenedTreeData: [],
      sortList: [
        {
          type: "ASC",
          icon: "arrow-up"
        },
        {
          type: "DESC",
          icon: "arrow-down"
        }
      ],
      sortValue: null,
      expandedRows: [],
      parentLevels: []
    }
  },
  components: { Row },
  methods: {
    setLevel(treeDataItem) {
      let level;
      if (treeDataItem.parentId) {
        level = this.parentLevels.find(level => level.id === treeDataItem.parentId).value + 1;
      }
      else {
        level = 1;
      }
      return level;
    },
    addParentLevel(treeDataItem) {
      let levelValue;
      if (treeDataItem.parentId) {
        let dataInLevels = this.parentLevels.find(level => level.id === treeDataItem.parentId);
        levelValue = dataInLevels.value + 1;
      }
      else {
        levelValue = 1;
      }
      this.parentLevels.push({
        id: treeDataItem.id,
        value: levelValue
      });
    },
    expandAll(treeData, flattenedTreeData, level, isRecoverExpandRowsAfterSort) {
      let _treeData = flattenedTreeData;
      let _level = level;
      treeData.forEach((data) => {
        _treeData.push(data);
        if (!isRecoverExpandRowsAfterSort) {
          data.level = this.setLevel(data);
        }
        if (data.children) {
          let dataInLevels = this.parentLevels.find(level => level.id === data.id);
          if (!dataInLevels) {
            this.addParentLevel(data);
          }
          if (isRecoverExpandRowsAfterSort) {
             if (this.expandedRows.includes(data.id)) {
              data.level = _level;
              data.expanded = true;
              _level++;
              this.expandAll(data.children, _treeData, _level, isRecoverExpandRowsAfterSort);
             }
          }
          else {
            data.expanded = true;
            this.expandedRows.push(data.id);
            _level++;
            this.expandAll(data.children, _treeData, _level, isRecoverExpandRowsAfterSort);
          }
        }
      });
      return _treeData;
    },
    expand(treeData) {
      let startIndex = this.flattenedTreeData.findIndex(item => item.id === treeData.id);
      let level = treeData.level ? treeData.level : 1;
      startIndex++;
      level++;
      treeData.children.forEach((child) => {
        child.level = level;
        this.flattenedTreeData.splice(startIndex, 0, child);
        startIndex++;
      });
      treeData.expanded = true;
      this.expandedRows.push(treeData.id);
    },
    collapse(treeData) {
      let children = this.flattenedTreeData.filter(item => item.parentId === treeData.id);
      children.forEach((child) => {
        if (child.children) {
          this.collapse(child);
        }
      });
      this.flattenedTreeData = this.flattenedTreeData.filter(item => item.parentId !== treeData.id);
      treeData.expanded = false;
      this.expandedRows = this.expandedRows.filter(expandedRow => expandedRow !== treeData.id);
    },
    sort() {
      let newSortIndex;
      newSortIndex = this.sortValue ? this.sortList.findIndex(item => item.type == this.sortValue.type) + 1 : 0;
      this.sortValue = this.sortList[newSortIndex] ? this.sortList[newSortIndex] : null;
      this.$emit("sort", this.sortValue);
    }
  },
  created() {
    this.flattenedTreeData = this.expanded ? this.expandAll(this.treeData, [], 1) : this.treeData.map(item => item);
  },
  watch: {
    treeData() {
      this.flattenedTreeData = this.expanded ? this.expandAll(this.treeData, [], 1) : this.expandAll(this.treeData, [], 1, true);
    }
  }
}
</script>

<style scoped>
table {
  border-collapse: collapse;
  text-align: left;
}
th {
  padding: 10px 15px;
}
.sortable {
  cursor: pointer;
}
</style>
