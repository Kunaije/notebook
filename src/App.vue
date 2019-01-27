
    <template>
      <div id="app">
        <!-- notebook is handling pages changes and creating new pages. It also requires the list of pages and the active page, which we pass in with :pages="pages" and :activePage="index" -->
          <Notebook @change-page="changePage" @new-page="newPage" :pages="pages" :activePage="index" />
          <Page @save-page="savePage" @delete-page="deletePage" :page="pages[index]" />
      </div>
    </template>

    <script>
    // first lets import our components, Notebook and Page. and Firebase
    import Notebook from './components/Notebook'
    import Page from './components/Page'
    import Firebase,{ firestore } from 'firebase'

    // now we initialize a new Firebase database instance
    var database =  Firebase.initializeApp({
      apiKey: 'AIzaSyAvQQORDmzUNg4_iOmZG3xtr-7nN3v6XD0',
      authDomain: 'notebook-3b03f.firebaseapp.com',
      databaseURL: 'https://notebook-3b03f.firebaseio.com',
      projectId: 'notebook-3b03f',
      storageBucket: 'notebook-3b03f.appspot.com',
      messagingSenderId: '972626205241'
    }).database().ref();

    export default {
      name: 'app',
      components: {
        Notebook,
        Page
      },
      // Second, we define the data() function which represents the pieces of data this component needs. We only need an array of pages and a number to hold the current page's index in the array.
      data: () => ({
        pages: [],
        index: 0
      }),
      mounted(){
        database.once('value', (pages) => {
          pages.forEach((page) => {
            this.pages.push({
              ref:page.ref,
              title: page.child('title').val(),
              content: page.child('content').val()
            })
          })
        })
      },
      methods: {
        // Creates a new object and pushes it onto our array and then sets the active page to the last element in the array (to the page we just pushed)
        newPage () {
          this.pages.push({
            title: '',
            content: ''
          })
          this.index = this.pages.length - 1
        },
        // takes the desired index as a parameter and updates this.index to that new value.
        changePage (index) {
          this.index = index
        },
        savePage () {
           var page = this.pages[this.index]
           if (page.ref){
             this.updateExistingPage(page)
           }else{
             this.insertNewPage(page)
           }
        },
        updateExistingPage (page) {
          page.ref.set({
            title: page.title,
            content: page.content
          })
        },
        insertNewPage (page) {
          page.ref = database.push(page)
        },
        // removes the currently active page from the array and sets the index to the next valid index
        deletePage () {
          var ref = this.pages[this.index].ref
          ref && ref.remove()
          this.pages.splice(this.index, 1)
          this.index = Math.max(this.index - 1, 0)
        }
      }
    }
    </script>

    <style>
    html, body, #app {
        height: 100%;
    }

    body {
        margin: 0;
    }

    #app {
        font-family: 'Avenir', Helvetica, Arial, sans-serif;
        display: flex;
        flex-direction: row;
    }
    </style>
	