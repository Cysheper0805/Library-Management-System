<template>
  <div>
    <h1>图书管理系统</h1>
    <div>
      <input type="text" v-model="bookId" class="name" placeholder="请输入书籍编号" />
      <button class="search" @click="searchBooks()"> 查询 </button>
      <button class="search" @click="add()"> 添加 </button>
    </div>
    
    <!-- 添加学生表单弹窗 -->
    <div v-if="show_add" class="modal">
      <div class="modal-content add">
        <h2>添加图书</h2>
        <label>名称：<input v-model="book.name" /></label>
        <label>编号：<input v-model="book.book_id" /></label>
        <label>单价：<input v-model="book.price" type="number" /></label>
        <label>作者：<input v-model="book.author" /></label>
        <div style="margin-top:10px;">
          <button class="search" @click="addBooks()">提交</button>
          <button class="search" @click="closeAdd()">取消</button>
        </div>
      </div>
    </div>
    <div v-if="edit" class="modal">
      <div class="modal-content add">
        <h2>编辑图书</h2>
        <label>名称：<input v-model="book.name" /></label>
        <label>编号：<input v-model="book.book_id" /></label>
        <label>单价：<input v-model="book.price" type="number" /></label>
        <label>作者：<input v-model="book.author" /></label>
        <label>存在状态：
          <select v-model="book.exist">
            <option value="未借出">未借出</option>
            <option value="已借出">已借出</option>
          </select>
        </label>
        <label>借书人姓名：<input v-model="reader.name" /></label>
        <label>借书人学号：<input v-model="reader.id" type="number" /></label>
        <label>借书人性别：
          <select v-model="reader.gender">
            <option value="男">男</option>
            <option value="女">女</option>
          </select>
        </label>
        <div>
          <button class="search" @click="updateBooks()">提交</button>
          <button class="search" @click="closeAdd()">取消</button>
        </div>
      </div>
    </div>
    <!-- 学生信息表格 -->
    <div>
      <table class="table">
        <tr>
          <th>名称</th>
          <th>编号</th>
          <th>单价</th>
          <th>作者</th>
          <th>存在状态</th>
          <th>借书人姓名</th>
          <th>学号</th>
          <th>性别</th>
          <th>操作</th>
        </tr>
        <tr v-for="i in books.length" :key="i">
          <td>{{ books[i-1].name }}</td>
          <td>{{ books[i-1].book_id }}</td>
          <td>{{ books[i-1].price }}</td>
          <td>{{ books[i-1].author }}</td>
          <td>{{ books[i-1].exist }}</td>
          <td>{{ readers[i-1]?.name ?? ""}}</td>
          <td>{{ readers[i-1]?.id ?? "" }}</td>
          <td>{{ readers[i-1]?.gender ?? "" }}</td>
          <div style="margin-top:10px;">
            <button @click="deleteBooks(books[i-1].book_id)">删除</button>
            <button @click="editBooks(books[i-1].book_id, books[i-1].reader)">编辑</button>
          </div>
        </tr>
      </table>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      edit: false,
      show_add: false,
      bookId: '',
      readerId: '',
      
      book: {
        id: '',
        book_id: '',
        name : '',
        price: '',
        author: '',
        exist: '未借出',
        reader: '0'
      },
      reader: {
        id: '',
        name: '',
        gender: ''
      },
      readers: [],
      books: [],
    }
  },
  methods: {
    async add() {
      this.show_add = true;
      this.book = {
        id: 0,
        book_id: '',
        name : '',
        price: '',
        author: '',
        exist: '未借出',
        reader: '0'
      }
    },
    async getReader(readerId) {
      try {
        const reader = await axios.get('http://localhost:8081/getReader/' + readerId);
        return reader;
      } catch (error) {
        console.error('Error fetching student data:', error);
      }
    },
    async addReader() {
      console.log(this.reader);
      const response = await axios.post('http://localhost:8081/postReader', this.reader);
      if(response.data.code !== "200") {
        alert('添加失败');
        throw new Error('添加失败' + response.msg);
      }
      alert('添加成功');
    },
    async updateBooks() {
      if(this.book.exist === '已借出' && this.reader.id === '') {
        alert('借书人学号不能为空');
        return;
      }
      if(this.book.exist === '已借出' && this.reader.name === '') {
        alert('借书人姓名不能为空');
        return;
      }
      const book = this.book;
      this.book.reader = this.reader.id;
      this.deleteBooks(this.bookId);
      this.book = book;
      this.addBooks();
      this.addReader();
      window.location.reload();
    },
    async editBooks(bookId, readerId) {
      this.bookId = bookId;
      this.readerId = readerId;
      this.edit = true;
      this.book = this.books.find(book => book.book_id === bookId);
      if(this.book.exist === '已借出') {
        this.reader = this.readers.find(reader => reader.id === readerId);
      } else {
        this.reader = {
          id: '',
          name: '',
          gender: ''
        }
      }
    },
    closeAdd() {
      this.show_add = false;
      this.edit = false;
    },
    async searchBooks() {
      try {
        console.log(this.bookId);
        this.book.reader = this.reader.id;
        const info = await axios.get('http://localhost:8081/get/' + this.bookId);
        if(info.data.code !== "200") {
          alert('查询失败');
          throw new Error('查询失败' + info.msg);
        }
        console.log(info);
        this.books = [];
        if(info.data.data === null) {
          alert('没有该书籍');
          window.location.reload();
          return;
        }
        this.books.push(info.data.data);
        if(this.books[0].exist === '已借出') {
          const reader = await this.getReader(this.books[0].reader);
          this.readers = [];
          this.readers.push(reader.data.data);
          console.log(this.reader);
        } else {
          this.readers = []
        }
      } catch (error) {
        console.error('Error fetching student data:', error);
      }
    },
    async addBooks() {
      try {
        const response = await axios.post('http://localhost:8081/add', this.book);
        if(response.data.code !== "200") {
          alert('添加失败, ' + response.data.msg);
          console.log(response);
          throw new Error('添加失败' + response.msg);
        }
        alert('添加成功');
        this.closeAdd();
        
        this.books.push(this.book);
        this.show_add = false;
        this.show_info = true;
      } catch (error) {
        console.error('Error adding student:', error);
      }
    },
    async deleteBooks(bookId) {
      try {
        const response = await axios.post('http://localhost:8081/delete/' + bookId);
        if(response.data.code !== "200") {
          alert('删除失败');
          throw new Error('删除失败' + response.msg);
        }
        alert('删除成功');
        window.location.reload();
      } catch (error) {
        console.error('Error deleting student:', error);
      }
    }
  },
  async mounted() {
    try {
      const info = await axios.get('http://localhost:8081/get');
      if(info.data.code !== "200") {
        alert('查询失败');
        throw new Error('查询失败' + info.msg);
      }
      this.books = info.data.data;
      console.log(this.books);
      console.log(this.readers);
      // 更具book_id从小到大排序
      this.books.sort((a, b) => a.book_id - b.book_id);
      for(let i = 0; i < this.books.length; i++) {
        if(this.books[i].exist === '已借出') {
          const reader = await this.getReader(this.books[i].reader);
          this.readers.push(reader.data.data);
          console.log(reader.data.data);
        } else {
          this.books[i].reader = '0';
          this.readers.push({
            id: '',
            name: '',
            gender: ''
          });
        }
      }
    } catch (error) {
      console.error('Error fetching student data:', error);
    }
  },
}
</script>

<style>
.add {
  display: flex;
  flex-direction: column;
  width: 300px;
  margin: 20px auto;
  padding: 20px;
  border: 1px solid #ccc;
  border-radius: 5px;
  background-color: #f9f9f9;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  
}
.name {
  width: 500px;
  height: 30px;
  border-radius: 5px;
  border: 1px solid #ccc;
  padding: 5px;
}
.search {
  width: 100px;
  height: 40px;
  border-radius: 10px;
  background-color: #007bff;
  color: white;
  border: none;
  margin-left: 10px;
  cursor: pointer;
} 
.search:hover {
  background-color: #0056b3;
}
.table {
  width: 75%;
  border-collapse: collapse;
  margin: 20px auto;
  border-radius: 5px;
  overflow: hidden;

}
.modal {
  position: fixed;
  top: 0; left: 0; right: 0; bottom: 0;
  background: rgba(0,0,0,0.3);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}
.modal-content {
  background: #fff;
  padding: 30px 40px;
  border-radius: 8px;
  min-width: 350px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.2);
}
.add label {
  margin-bottom: 10px;
  display: flex;
  justify-content: space-between;
}
.add input {
  flex: 1;
  margin-left: 10px;
}
</style>