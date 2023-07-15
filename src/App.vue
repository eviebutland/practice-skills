<template>
  <!--
    Add the directive called contactBubble to
    this element
  -->
  <div id="app" v-contactBubble>
    <div class="section">
      <button id="btnAddContact" @click="openNew">Add Contact</button>
    </div>
    <div class="section">

      <input id="search" placeholder="search" type="text" v-model="query">
      {{ query }}
    </div>
    <app-contactform  v-bind="formSettings" v-bind:contact="contact"></app-contactform>

    <app-contacts :contacts="search"></app-contacts>
  </div>
</template>

<script>
import Contacts from './components/Contacts'
import ContactForm from './components/ContactForm'
import { ContactBus } from './bus/ContactBus'
import contacts from '../src/assets/contacts.json'

export default {
  name: 'App',
  components: {
    'app-contacts': Contacts,
    'app-contactform': ContactForm
  },
  data () {
    return {
      contacts,
      contact: {
        firstname: '',
        lastname: '',
        email: [''],
        phoneNumber: ['']
      },
      query: '',
      formSettings: {
        index: '',
        newCon: false,
        visible: false

      }
    }
  },
  computed: {
    search: {

      get: function () {
        return this.contacts.filter((contact) => {
          const query = this.query.toLowerCase()
          const matchFirstName = contact.firstname.toLowerCase().includes(query)
          const matchLastname = contact.lastname.toLowerCase().includes(query)
          const matchEmail = contact.email.some(email => email.toLowerCase().includes(query))
          const matchPhone = contact.phoneNumber.some((number) => number.includes(query))

          return matchFirstName || matchLastname || matchEmail || matchPhone
        })
      }
    }
  },
  directives: {
    contactBubble: {
      bind:
        function (el, binding, vnode) {
          el.style.backgroundColor = 'rgb(159, 159, 199)'
          el.style.borderRadius = '20px'
          el.style.fontSize = '20px'
          el.style.color = 'white'
          el.style.fontVariant = 'all-petite-caps'
        }
    }
  },
  mounted () {
    this.sortContacts()
  },
  methods: {
    sortContacts: function () {
      this.contacts = this.contacts.sort(function (a, b) {
        const A = a.lastname.toLowerCase()
        const B = b.lastname.toLowerCase()

        if (A > B) { return 1 }
        if (A < B) { return -1 }
        return 0
      })
    },
    openNew: function () {
      this.formSettings.visible = true
    },
    close: function (data) {
      this.formSettings.visible = !data
    },
    resetFormToDefault () {
      this.formSettings.visible = false
      this.formSettings.index = ''
      this.formSettings.newCon = false
    }
  },
  beforeCreate () {
    const mockContacts = require('./assets/contacts.json')
    const clone = JSON.parse(JSON.stringify(mockContacts))
    this.contacts = clone
  },
  created: function () {
    ContactBus.$on('addCon', (data) => {
      if (!data.id) {
        // Only add to array if its a new contact
        this.contacts.push({ ...data, index: this.contacts.length })
        this.formSettings.newCon = true
        this.formSettings.index = this.contacts.length
      }
      this.sortContacts()

      // Reset form data
      this.resetFormToDefault()
    })

    ContactBus.$on('noAddCon', (data) => {
      this.nullContact = data
    })

    ContactBus.$on('close', (data) => {
      this.close(data)
    })

    ContactBus.$on('delCon', (index) => {
      this.contacts = this.contacts.filter(contact => contact.id !== index)
      this.sortContacts()
      this.resetFormToDefault()
    })

    ContactBus.$on('editCon', (contactIndex) => {
      this.contact = this.contacts.find(contact => contact.id === contactIndex)
      this.formSettings.index = contactIndex
      this.formSettings.newCon = false
      this.formSettings.visible = true
    })
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

.section {
  margin: 30px;
}
</style>
