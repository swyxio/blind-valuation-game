<template>
  <div id="app" class="container">
    <div class="page-header">
      <h1>Sentieo Blink <small>

        <transition name="with-mode-fade" mode="out-in">
          <span v-if="subtitleText" key="a"><br />High Frequency Value Investing<br /></span>
          <span v-else key="b">/bliNGk/ <em>verb</em> <br /><small>"There can be as much value in the blink of an eye as in months of rational analysis." - Malcolm Gladwell, <a href="https://www.goodreads.com/work/quotes/1180927-blink-the-power-of-thinking-without-thinking" target="_blank">Blink: The Power of Thinking Without Thinking</a></small></span>
        </transition>
      </small></h1>
    </div>
    <div v-if="!emailSubmitted">
      <div class="alert alert-warning" v-if="currentstage < 3">
        <h4>This is a blind stock valuation game inspired by Ben Graham and Warren Buffet and popularized by <a href="https://www.gannononinvesting.com/blog/how-much-would-you-pay-for-this-stock-blind-stock-valuation.html" target="_blank">Geoff Gannon</a> and <a href="http://www.distressed-debt-investing.com/2012/06/value-investing-hidden-stock-price.html" target="_blank">others</a>. </h4>
        <p>Buffett believes that guessing the stock price keeps the emphasis on the fundamentals and avoids anchoring bias explored by the likes of <a href="http://www.amazon.com/gp/product/0374275637/ref=as_li_ss_tl?ie=UTF8&tag=amildolonthew-20&linkCode=as2&camp=1789&creative=390957&creativeASIN=0374275637" target="_blank">Daniel Kahneman</a>.
        <p>To help you warm up, we give you three test stocks from the same sector (Consumer Discretionary) to calibrate your "feel". Then we put your snap valuation skills to the test!</p>
        <p>Absolute numbers below are in USD millions. Good luck!</p>
      </div>
      <div v-else class="alert alert-danger">
        <p>Last stage! Take everything you have learned from the last 3 stages and put in your best guess!</p>
      </div>
    </div>
    <div class="panel panel-default" v-if="!emailSubmitted">
      <div class="panel-heading">
        <h3 class="panel-title" v-if="currentstage < 3">Blind Valuation Financials (Test Stage: {{ currentstage + 1 }} of 3)</h3>
        <h3 class="panel-title" v-else>Blind Valuation Financials: Final Stage!</h3>
      </div>
      <div class="panel-body">
         <VueTable :tableticker="currentstock" :showanswer="showanswer" :currentguess="currentguess" :currentstage="currentstage"></VueTable>
      </div>
    </div>
    <div class="panel panel-default" v-if="!emailSubmitted">
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
      <div class="panel-body" v-if="showanswer && currentstage < 3">
         <form id="form" class="form-inline" v-on:submit.prevent="nextStock">
          The stock was: <a :href="`https://www.google.com/finance?q=${currentstock}`" target="_blank">{{ currentstock.toUpperCase() }}</a>! Are you surprised?
          <input type="submit" class="btn btn-secondary" value="Show the Next Stock!">
        </form>
      </div>
      <div class="panel-body" v-if="showanswer && currentstage > 2">
         <form id="form" class="form-inline" v-on:submit.prevent="saveEmail">
          <div class="form-group">
            <label for="emaillabel">Enter your email to receive results (and find out the distribution of answers!):</label>
            <input type="email" placeholder="your@email.com" id="UserEmail" class="form-control" v-model="useremail">
          </div>
          <div class="form-group">
            <label for="feedbacklabel">(Optional) Any feedback for us? Justification for your valuation?</label>
            <textarea rows="4" cols="50" placeholder="We <3 your ideas" id="UserFeedback" class="form-control" v-model="userfeedback" />
          </div>
          <input type="submit" class="btn btn-primary" value="Submit">
        </form>
      </div>
    </div>
    <div class="panel panel-default" v-if="emailSubmitted">
      <div class="panel-heading">
        <h3 class="panel-title">Thank you!</h3>
      </div>
      <div class="panel-body">
        <p>Thanks for submitting your final answer! We will be in touch shortly once we tally the results, and hopefully launch the next iteration of this blind valuation contest!</p>
        <p>If you enjoyed this game, tell your friends on <a href="https://twitter.com/intent/tweet?url=https%3A%2F%2Fsentieoblinkvue.firebaseapp.com%2F%23%2F&via=sentieo&text=Just%20completed%20the%20@Sentieo%20Blink%20blind%20valuation%20game%21%20What%20was%20your%20final%20answer%3F" target="_blank">Twitter</a> or <a href="mailto:yourfriends@email.com?subject=Sentieo%20Blink&body=Just%20completed%20the%20@Sentieo%20Blink%20blind%20valuation%20game%21%20What%20was%20your%20final%20answer%3F%0Ahttps%3A%2F%2Fsentieoblinkvue.firebaseapp.com%3F%0ARegards%20">email</a> and compare your results!</p>
        <p>If you are a fundamental investor looking for more time-saving research tools, check us out at <a href="https://sentieo.com/?utm_source=sentieoblink&utm_medium=lead-gen&utm_term=&utm_content=sentieoblink&utm_campaign=sentieoblink&utm_code=blink0001&code=blink0001">Sentieo</a> and get a free trial today!</p>
      </div>
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

const stocks = ['lkq', 'gpc', 'azo', 'aap'];
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
      useremail: '',
      userfeedback: '',
      emailSubmitted: false,
      subtitleText: true,
    };
  },
  mounted() {
    const self = this;
    setInterval(() => { self.subtitleText = !self.subtitleText; }, 4000);
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
      this.allguesses.push(this.currentguess);
      this.currentstage += 1;
      this.currentstock = stocks[this.currentstage];
      this.currentguess = 10.00;
      this.showanswer = false;
    },
    saveEmail: function defc() {
      booksRef.push({ email: this.useremail,
        feedback: this.userfeedback,
        guesses: this.allguesses,
        dt: String(new Date()),
      });
      this.emailSubmitted = true;
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

/*.bk {
  transition: all 1s ease-out;
}

.blur {
  opacity: 0;
}*/
/*
.fade-enter-active, .fade-leave-active {
  transition: opacity .5s
}
.fade-enter, .fade-leave-to {
  opacity: 0
}*/

.with-mode-fade-enter-active, .with-mode-fade-leave-active {
  transition: opacity 1s
}
.with-mode-fade-enter, .with-mode-fade-leave-active {
  opacity: 0
}

/*.blur {
  filter: blur(2px);
  opacity: 0.4;
}*/
</style>
