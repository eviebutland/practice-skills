<template>
  <form v-show="visible">
    <label>First Name</label>
    <input id="fname" type="text" v-model="contact.firstname">
    <label>Last Name</label>
    <input id="lname" type="text" v-model="contact.lastname">
    <div id="email">
      <label>Email</label>
      <input type="text" v-for="(addr, index) in contact.email" v-model="contact.email[index]" :key="index">
      <button id="btnAddEMail" @click.prevent="(index) => addEmail(contact.email[index])">Add email</button>
    </div>
    <div id="phone">
      <label>Phone</label>
      <input type="text" @keyup="purgeNumber(index)" v-for="(number, index) in contact.phoneNumber" v-model="contact.phoneNumber[index]" :key="index">
      <button id="btnAddNumber" @click.prevent="addPhone">Add Phone</button>
    </div>
    <div>
      <button id="btnSubmit" @click.self="submit">Submit</button>
      <button id="btnDelete" v-show="!newCon" @click="(e) => removeContact(e, contact.id)">Delete Contact</button>
      <button id="btnCancel" @click="cancel">Cancel</button>
    </div>
  </form>
</template>

<script>
/**
 * Event bus for contact
 */
import { ContactBus } from '../bus/ContactBus'

export default {
  name: 'ContactForm',
  props: {
    contact: {
      type: Object,
      required: true
    },
    visible: {
      type: Boolean
    },
    newCon: {
      type: Boolean
    },
    index: {
      type: [Number, String]
    }
  },
  computed: {
    validatedEmail: function () {
      // VERY simple validation
      // normally I would use a library to handle this

      return this.contact.email.every(email => email.includes('@'))
    },
    validatedPhone: function () {
      // VERY simple validation
      // normally I would use a library to handle this
      return this.contact.phoneNumber.every(number => !(/^[a-z]*$/).test(number))
    }
  },
  methods: {
    submit: function (e) {
      e.preventDefault()
      if (this.contact.firstname && this.contact.lastname) {
        if (this.validatedEmail || this.validatedPhone) {
          ContactBus.$emit('addCon', this.contact)
        } else {
          ContactBus.$emit('noAddCon', [])
        }
      } else {
        ContactBus.$emit('noAddCon', [])
      }
    },
    purgeNumber: function (index) {
      this.contact.phoneNumber[index] = this.contact.phoneNumber[index].replace(/\D/g, '')
    },
    cancel: function () {
      ContactBus.$emit('close', true)
    },
    removeContact: function (e, index) {
      e.preventDefault()
      ContactBus.$emit('delCon', index)
    },
    addEmail: function () {
      this.contact.email.push('')
    },
    addPhone: function () {
      this.contact.phoneNumber.push('')
    }
  }
}
</script>

<style scoped>
form {
  margin: 30px;
}
input {
  margin: 20px 10px;
}
</style>
