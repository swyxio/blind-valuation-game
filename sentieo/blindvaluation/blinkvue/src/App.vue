<template>
  <div id="app" class="container">
    <div class="page-header">
      <h1>Sentieo Blink <br/> <small>High Frequency Value Investing</small></h1>
    </div>
    <div class="panel panel-default">
      <div class="panel-heading">
        <h3 class="panel-title">Blind Valuation Financials</h3>
      </div>
      <div class="panel-body">
         <VueTable></VueTable>
      </div>
    </div>
    <div class="panel panel-default">
      <div class="panel-heading">
        <h3 class="panel-title">Guess the current per-share price</h3>
      </div>
      <div class="panel-body">
         <form id="form" class="form-inline" v-on:submit.prevent="addBook">
          <div class="form-group">
            <label for="bookTitle">High:</label>
            <input type="text" id="bookTitle" class="form-control" v-model="newBook.title">
          </div>
          <div class="form-group">
            <label for="bookAuthor">Low:</label>
            <input type="text" id="bookAuthor" class="form-control" v-model="newBook.author">
          </div>
          <input type="submit" class="btn btn-primary" value="Submit">
        </form>
      </div>
    </div>
    <div class="panel panel-default">
      <div class="panel-heading">
        <h3 class="panel-title">Book List</h3>
      </div>
      <div class="panel-body">
        <table class="table table-striped">
          <thead>
            <tr>
              <th>Title</th>
              <th>Author</th>
              <th></th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="book in books">
              <td>{{book.title}}</td>
              <td>{{book.author}}</td>
              <td><span class="glyphicon glyphicon-trash" aria-hidden="true" v-on:click="removeBook(book)"></span></td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>
<script>
import Firebase from 'firebase';
import toastr from 'toastr';
import Hello from './components/Hello';
import VueTable from './components/VueTable';

const config = {
  apiKey: 'AIzaSyCnuHrfCLTmriFw9t3iWPDnpe2gt7CJR1M',
  authDomain: 'sentieoblinkvue.firebaseapp.com',
  databaseURL: 'https://sentieoblinkvue.firebaseio.com',
  projectId: 'sentieoblinkvue',
  storageBucket: 'sentieoblinkvue.appspot.com',
  messagingSenderId: '751495605204',
};

const app = Firebase.initializeApp(config);
const db = app.database();
const booksRef = db.ref('items');

export default {
  name: 'app',
  firebase: {
    books: booksRef,
  },
  data() {
    return {
      newBook: {
        title: '',
        author: '',
      },
    };
  },
  methods: {
    addBook: function abc() {
      booksRef.push(this.newBook);
      this.newBook.title = '';
      this.newBook.author = '';
    },
    removeBook: function def(book) {
      booksRef.child(book['.key']).remove();
      toastr.success('Book removed successfully');
    },
  },

  components: {
    Hello,
    VueTable,
  },
};
</script>
<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin-top: 20px;
}
</style>
