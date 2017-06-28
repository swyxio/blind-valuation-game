<template>
  <div id="app" class="container">
    <div class="page-header">
      <h1>Sentieo Blink <br/> <small>High Frequency Value Investing</small></h1>
    </div>
    <div class="panel panel-default">
      <div class="panel-heading">
        <h3 class="panel-title">Blind Valuation Financials (Stage: {{ currentstage + 1 }} of 3)</h3>
      </div>
      <div class="panel-body">
          Sector: Consumer Discretionary. Absolute numbers in millions.
         <VueTable :tableticker="currentstock" :showanswer="showanswer" :currentguess="currentguess"></VueTable>
      </div>
    </div>
    <div class="panel panel-default">
      <div class="panel-heading">
        <h3 class="panel-title">Guess the current per-share price (not what <em>you</em> would pay, but what the market pays)</h3>
      </div>
      <div class="panel-body" v-if="!showanswer">
         <form id="form" class="form-inline" v-on:submit.prevent="showAnswer">
          <div class="form-group">
            <label for="SPlabel">Current Price per Share (min. 0.01):</label>
            <input type="number" placeholder="0.00" step="0.01" min="0.01" id="SharePrice" class="form-control" v-model="currentguess">
          </div>
          <input type="submit" class="btn btn-primary" value="Submit">
        </form>
      </div>
      <div class="panel-body" v-if="showanswer">
         <form id="form" class="form-inline" v-on:submit.prevent="nextStock">
          The stock was: <a :href="`https://www.google.com/finance?q=${currentstock}`" target="_blank">{{ currentstock.toUpperCase() }}</a>! Are you surprised?
          <input type="submit" class="btn btn-secondary" value="Show the Next Stock!">
        </form>
      </div>
      <!--
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
      -->
    </div>
    <!--
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
    -->
  </div>
</template>
<script>
import Firebase from 'firebase';
import toastr from 'toastr';
import Hello from './components/Hello';
import VueTable from './components/VueTable';

const stocks = ['lkq', 'gpc', 'azo'];
// const finalstock = 'aap';
console.log('yes, we know you can see this, this game is meant for less technical folk');

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
      currentstock: stocks[0],
      currentstage: 0,
      showanswer: false,
      currentguess: 10.00,
      allguesses: [],
    };
  },
  methods: {
    addBook: function abc() {
    //   booksRef.push(this.newBook);
      this.newBook.title = '';
      this.newBook.author = '';
    },
    removeBook: function def(book) {
      booksRef.child(book['.key']).remove();
      toastr.success('Book removed successfully');
    },
    showAnswer: function defa() {
      this.showanswer = true;
    },
    nextStock: function defb() {
    //   this.allguesses.push(this.currentguess);
      this.currentstage += 1;
      this.currentstock = stocks[this.currentstage];
      this.currentguess = 0.01;
      this.showanswer = false;
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
