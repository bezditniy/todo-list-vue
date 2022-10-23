<template>
    <div class="container" >
      <ul 
        v-for="(items, key) in this.items"
        class="drop-zone"
        :key="key"
        draggable="true"
        @dragstart="startDragArr($event, items, key)"
        @drop="dropArray(items, key)"
        @dragover.prevent
        @dragenter.prevent
      >
        {{ getTitle(key) }}
        <li 
          class="drag-el"
          v-for="(item, indexItem) in items"
          :key="item.title"
          draggable="true"
          @dragstart="startDrag($event, item, indexItem, key)"
          @drop="dropItem(indexItem, key)"
          @dragover.prevent
          @dragenter.prevent
        >
          <input type="checkbox" v-model="item.done"/>
          <span :class="{done: item.done}" >{{item.title}}</span>
          <div class="ev-container">
            <div class="input-container">
              <input placeholder="Type new task and press enter" type="text" v-model="item.newItem" @keyup.enter="changeTitle(indexItem, key, item.newItem)">
            </div>  
            <span class="ev-item" @click="removeItem(item, key)">X</span> 
          </div>
          
        </li>
      </ul>
    </div>
  </template>
  
  <script>
  
  const DB_NAME = 'storage';
  const DB_VERSION = 1;
  
  export default {
    data() {
      return {
        index: -1,
        indexArray: -1,
  
        array : [],
  
        items: [
          [
            {
              id: 1,
              title: 'Item A',
              list: 'Category1',
            },
            {
              id: 2,
              title: 'Item B',
              list: 'Category1',
            },
            {
              id: 3,
              title: 'Item C',
              list: 'Category1',
            },
          ],
          [
            {
              id: 4,
              title: 'Item D',
              list: 'Category2',
            },
            {
              id: 5,
              title: 'Item E',
              list: 'Category2',
            },
            {
              id: 6,
              title: 'Item F',
              list: 'Category2',
            },
          ], 
        ],
      }
    },
  
    async created() {
      this.db = await this.getDb(); 
  
      this.array = await this.getItemsFromDb()
      if (this.array.length === 0) {
        return this.item
      } else {
        this.items = this.array[0]
      }
      
      this.ready = true;
    },
  
    methods: {
  
      async changeTitle(index, key, newItem = '') {
        if (newItem === '') {
          console.log(newItem)
          alert("Please typing something")
          return
          } else if (newItem !== 'nothing') {
            this.items[key][index].title = newItem
            this.changeArr()
        }
        
      },
  
      getTitle(index) {
        if (index === index) {
          let title = "Category"
          return title
        } 
      },
  
      startDrag(evt, item, indexItem, key ) {
        evt.dataTransfer.dropEffect = 'move'
        evt.dataTransfer.effectAllowed = 'move'
        evt.dataTransfer.setData('itemID', item.id)
        this.index = indexItem
        console.log(this.index, key)
      },
      
      dropItem(index, key) {
        this.moveItem(this.index, index, key);
        console.log(this.index, index, key)
      },
      
      async moveItem(from, to, key) {
        if (to === -1 || key !==  key) {
          console.log("Error")
        } else{
          
         console.log(from, to, key)
        
        this.items[key].splice(to, 0, this.items[key].splice(from, 1)[0])
          
  
        this.changeArr()
          
        }  
      },
  
      async changeArr () {
  
        let item = this.items;
          console.log('about to add ' + JSON.stringify(item), this.array);   
  
          await this.clearStorage();
          await this.addItemToDb(item);
          this.array = await this.getItemsFromDb();
  
      },
      
      async addItemToDb(item) {
        return new Promise((resolve) => {
  
          let trans = this.db.transaction(['items'],'readwrite');
          trans.oncomplete = () => {
            resolve();
          };
  
          let store = trans.objectStore('items');
          store.add(item);
  
        });
      },
  
      
      removeItem(item, key) {
        if(this.items[key].indexOf(item) === -1) {
          console.log("Error")
          console.log(this.items.indexOf(item), key, this.items)
          return
        } else {
          this.items[key].splice(this.items.indexOf(item), 1);
          this.changeArr()
        }
      },
  
      startDragArr(evt, item ) {
        evt.dataTransfer.dropEffect = 'move'
        evt.dataTransfer.effectAllowed = 'move'
        this.index = this.items.indexOf(item)
      },
  
      dropArray(index) {
        this.moveArray(this.index, this.items.indexOf(index));
        console.log(this.index, this.items.indexOf(index))
      },
  
      moveArray(from,to) {
        if (to === -1 || from === -1) {
          console.log("Error")
        } else{
          this.items.splice(to, 0, this.items.splice(from, 1)[0]);
  
          this.changeArr()
        }
      },
  
      async clearStorage() {
        let trans = this.db.transaction("items", "readwrite")
  
        await trans.objectStore("items").clear()
      },
  
      async getItemsFromDb() {
        return new Promise((resolve) => {
  
          let trans = this.db.transaction(['items'],'readonly');
          trans.oncomplete = () => {
            resolve(items);
          };
          
          let store = trans.objectStore('items');
          
          let items = [];
          
          store.openCursor().onsuccess = e => {
            let cursor = e.target.result;
            if (cursor) {
              items.push(cursor.value)
              console.log(items)
              cursor.continue();
            }
          };
  
        });
      },
      
      async getDb() {
        return new Promise((resolve, reject) => {
  
          let request = window.indexedDB.open(DB_NAME, DB_VERSION);
          
          request.onerror = e => {
            console.log('Error opening db', e);
            reject('Error');
            
          };
    
          request.onsuccess = e => {
            resolve(e.target.result);
          };
          
          request.onupgradeneeded = e => {
            console.log('onupgradeneeded');
            let db = e.target.result;
            let objectStore = db.createObjectStore("items", { autoIncrement: true, keyPath:'id' });
            objectStore.createIndex("items", {keyPath: "id"}, {multientry: true} )
          };
        });
      }
    },
  }
  </script>
  
  <!-- Add "scoped" attribute to limit CSS to this component only -->
  <style scoped>
  
  .container {
    display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: column;
  }
  .drop-zone {
    background-color: #eee;
    margin-bottom: 10px;
    padding: 10px;
    width: 500px;
    margin-left: 0;
    margin-right: 0;
    text-align: center;
  }
  
  .drag-el {
    display: flex;
    justify-content: space-around;
    align-items: center;
    background-color: #fff;
    list-style-type: none;
    margin-top: 10px;
    margin-bottom: 10px;
    padding: 5px;
  }
  
  .ev-item {
    padding: 10px;
  }
  
  .done {
    text-decoration: line-through;
    opacity: 0.2;
  }
  
  .active {
      visibility: hidden;
    }
  
  .ev-container {
    display: flex;
    align-items: center;
  }
  
  </style>