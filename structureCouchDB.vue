<script lang="ts">
export default {
  data() {
    return {
      datas: [],
      databaseReference: null
    }
  },

  methods: {
    inc() {},

    initDatabase() {
      // lancer une initialisation de la base de données
      // Si la connexion est ok, alors je fais un fetchdata
      const db = new PouchDB('http://localhost:5984')
      if (db) {
        console.log('valid')
        return
      }
      console.log('invalid')
    },

    fetchData() {
      // récupérer tous les documents de ma collection
    }
  },

  mounted() {
    this.initDatabase()
  }
}
</script>

<template>
  <h1>InfraDon2</h1>
  <p>Counter: {{ datas }}</p>
  <button type="button" role="button" @click="inc">+1</button>
</template>
