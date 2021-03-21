<template>
  <div id="app">
    <button class="button bg-primary add-button" @click="showDialog = true">Добавить</button>
    <TreeData :treeData="sortedTreeData" :expanded="true" @sort="sort" />
    <Dialog v-if="showDialog" @close="closeDialog" title="Добавление пользователя">
      <form @submit.prevent="addUser" class="form">
        <div class="form__field">
          <label>Имя</label>
          <input type="text" placeholder="Имя" v-model="name">
        </div>
        <div class="form__field">
          <label>Телефон</label>
          <input type="text" placeholder="Телефон" v-model="phone">
        </div>
        <div class="form__field">
          <label>Начальник</label>
          <select v-model="selectedChief">
            <option v-for="chief in chiefList" :key="chief.id" :value="chief.id">{{chief.name}}</option>
          </select>
        </div>
        <div class="form__field">
          <button class="button bg-primary" type="submit" :disabled="!validForm">Сохранить</button>
        </div>
      </form>
    </Dialog>
  </div>
</template>

<script>
import TreeData from "./components/TreeData";
import Dialog from "./components/Dialog";

export default {
  name: 'App',
  data() {
    return {
      showDialog: false,
      name: "",
      phone: "",
      selectedChief: null,
      treeData: [
        {
          name: "Марина",
          phone: "+79999999999",
          id: 1
        },
        {
          name: "Петр",
          phone: "+79999999999",
          id: 2
        },
        {
          name: "Алексей",
          phone: "+79999999999",
          id: 3,
          expanded: false,
          children: [
            {
              name: "Иван",
              phone: "+79999999999",
              id: 4,
              parentId: 3,
              expanded: false,
              children: [
                {
                  name: "Кирилл",
                  phone: "+79999999999",
                  id: 5,
                  parentId: 4
                },
                {
                  name: "Елена",
                  phone: "+79999999999",
                  id: 6,
                  parentId: 4
                }
              ]
            },
            {
              name: "Наталья" ,
              phone: "+79999999999",
              id: 7,
              parentId: 3,
            }
          ]
        },
        {
          name: "Борис",
          phone: "+79999999999",
          id: 8
        },
      ],
      sortValue: null
    }
  },
  components: {
    TreeData,
    Dialog
  },
  computed: {
    chiefList() {
      return this.flattenTreeData(this.treeData, []);
    },
    validForm() {
      return this.name && this.phone;
    },
    sortedTreeData() {
      let sortedTreeData = this.treeData.map(item => item);
      if (!this.sortValue) {
        return sortedTreeData;
      }
      switch(this.sortValue.type) {
        case "ASC": {
          sortedTreeData = this.sortOrderByAsc(sortedTreeData);
          break;
        }
        case "DESC": {
          sortedTreeData = this.sortOrderByDesc(sortedTreeData);
          break;
        }
      }
      return sortedTreeData;
    }
  },
  methods: {
    closeDialog() {
      this.showDialog = false;
    },
    flattenTreeData(treeData, flattenedTreeData) {
      let _treeData = flattenedTreeData;
      treeData.forEach(data => {
        _treeData.push(data);
        if (data.children) {
          this.flattenTreeData(data.children, _treeData);
        }
      });
      return _treeData;
    },
    uniqueID() {
      return Math.floor(Math.random() * Date.now());
    },
    findTreeDataItem(treeData) {
      let user = treeData.find(item => item.id == this.selectedChief);
      if (user) {
        return user;
      }
      for (let i = 0; i < treeData.length; i++) {
        if (treeData[i].children) {
          user = this.findTreeDataItem(treeData[i].children);
          if (user) {
            return user;
          }
        }
      }
    },
    addUser() {
      const id = this.uniqueID();
      let newUser = {
        id,
        name: this.name,
        phone: this.phone
      }
      if (this.selectedChief) {
        let user = this.findTreeDataItem(this.treeData);
        if (!user.children) {
          this.$set(user, "children", []);
          this.$set(user, "expanded", false);
        }
        newUser.parentId = user.id;
        user.children.push(newUser);
      }
      else {
        this.treeData.push(newUser);
      }
      this.name = "";
      this.phone = "";
      this.selectedChief = null;
      this.showDialog = false;
    },
    collapseAll(treeData) {
      treeData.forEach(data => {
        if (data.children) {
          data.expanded = false;
          this.collapseAll(data.children);
        }
      });
    },
    sort(sortValue) {
      this.sortValue = sortValue;
    },
    sortOrderByAsc(treeData) {
      treeData = treeData.sort((valueA, valueB) => {
          let res = valueA.name == valueB.name ? 0 : valueA.name > valueB.name ? 1 : -1;
          return res;
      });
      treeData.forEach((item) => {
        if (item.children) {
          this.sortOrderByAsc(item.children);
        }
      });
      return treeData;
    },
    sortOrderByDesc(treeData) {
      treeData = treeData.sort((valueA, valueB) => {
          let res = valueA.name == valueB.name ? 0 : valueA.name < valueB.name ? 1 : -1;
          return res;
      });
      treeData.forEach((item) => {
        if (item.children) {
          this.sortOrderByDesc(item.children);
        }
      });
      return treeData;
    },
  },
  created() {
    if (localStorage.getItem("treeData")) {
      this.treeData = JSON.parse(localStorage.getItem("treeData"));
    }
  },
  beforeMount() {
    window.addEventListener("beforeunload", () => {
      this.collapseAll(this.treeData);
      localStorage.setItem("treeData", JSON.stringify(this.treeData));
    });
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
input, select {
  display: block;
  box-sizing: border-box;
}
input {
  width: 100%;
  padding: 15px 25px;
  border-radius: 5px;
  width: 300px;
  border: 1px solid #ccc;
}
select {
  width: 100%;
  padding: 15px 25px;
  border-radius: 5px;
  width: 300px;
  border: 1px solid #ccc;
}
.button {
  width: 250px;
  padding: 15px 25px;
  color: #fff;
  border-radius: 5px;
  border: none;
  display: block;
  cursor: pointer;
}
.bg-primary {
  background: #3700b3;
}
.form__field {
  margin-bottom: 15px;
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.add-button {
  margin-bottom: 20px;
}
</style>
