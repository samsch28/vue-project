<script lang="ts">
import { ref } from 'vue'
import PouchDB from 'pouchdb'

declare interface Post {
  _id: string
  _rev?: string
  doc: {
    post_name: string
    post_content: string
    attributes: {
      creation_date: string
    }
  }
}

export default {
  data() {
    return {
      total: 0,
      postsData: [] as Post[],
      document: null as Post | null,
      storage: null as PouchDB.Database | null,
      editMode: false, // Nouveau: pour savoir si on est en mode édition
      editedPost: null as Post | null // Nouveau: post en cours d'édition
    }
  },

  mounted() {
    this.initDatabase()
    this.fetchData()
  },

  methods: {
    fetchData() {
      const storage = ref(this.storage)
      if (storage.value) {
        storage.value
          .allDocs({
            include_docs: true,
            attachments: true
          })
          .then((result) => {
            console.log('fetchData success', result)
            this.postsData = result.rows.map((row: any) => ({
              _id: row.id,
              _rev: row.doc?._rev,
              doc: row.doc || {}
            })) as Post[]
          })
          .catch((error) => console.error('fetchData error:', error))
      }
    },

    initDatabase() {
      const db = new PouchDB('http://admin:aBcD1234!!@127.0.0.1:5984/demo')
      if (db) {
        console.log("Connected to collection 'demo'")
      } else {
        console.warn('Something went wrong')
      }
      this.storage = db
    },

    generateFakePost(): Post {
      return {
        _id: new Date().toISOString(),
        doc: {
          post_name: `Post_${Math.random().toString(36).substring(7)}`,
          post_content: 'Ceci est un contenu fictif.',
          attributes: {
            creation_date: new Date().toISOString()
          }
        }
      }
    },

    addFakePost() {
      const fakePost = this.generateFakePost()
      this.putDocument(fakePost)
    },

    putDocument(document: Post) {
      if (this.storage) {
        this.storage
          .put(document)
          .then(() => console.log('Document ajouté/mis à jour avec succès'))
          .catch((error) => console.error('Erreur d’ajout/mise à jour du document:', error))
      }
    },

    editPost(post: Post) {
      this.editMode = true
      this.editedPost = { ...post } // Cloner le post pour l'édition
    },

    savePost() {
      if (this.editedPost && this.storage) {
        this.storage
          .put(this.editedPost)
          .then(() => {
            console.log('Mise à jour réussie')
            this.editMode = false
            this.editedPost = null
            this.fetchData() // Rafraîchir les données pour refléter la modification
          })
          .catch((error) => console.error('Erreur de mise à jour:', error))
      }
    },

    cancelEdit() {
      this.editMode = false
      this.editedPost = null
    },

    deletePost(post: Post) {
      if (this.storage && post._rev) {
        this.storage
          .remove(post._id, post._rev)
          .then(() => {
            console.log('Suppression réussie')
            this.fetchData()
          })
          .catch((error) => console.error('Erreur de suppression:', error))
      } else {
        console.warn('Impossible de supprimer : _rev manquant.')
      }
    }
  }
}
</script>

<template>
  <h1>Nombre de posts: {{ postsData.length }}</h1>
  <button @click="addFakePost">Ajouter un Post</button>

  <!-- Formulaire d'édition -->
  <div v-if="editMode && editedPost" class="edit-form">
    <h2>Modifier le Post</h2>
    <label>Nom du post :</label>
    <input v-model="editedPost.doc.post_name" />
    <label>Contenu du post :</label>
    <textarea v-model="editedPost.doc.post_content"></textarea>
    <button @click="savePost">Sauvegarder</button>
    <button @click="cancelEdit">Annuler</button>
  </div>

  <!-- Liste des posts -->
  <ul>
    <li v-for="post in postsData" :key="post._id">
      <div class="ucfirst">
        {{ post.doc.post_name }}
        <em style="font-size: x-small" v-if="post.doc.attributes?.creation_date">
          - {{ post.doc.attributes?.creation_date }}
        </em>
      </div>
      <button @click="editPost(post)">Modifier</button>
      <button @click="deletePost(post)">Supprimer</button>
    </li>
  </ul>
</template>

<style scoped>
li {
  display: flex;
  align-items: center;
  gap: 10px;
}

.ucfirst {
  flex-grow: 1;
}

.edit-form {
  margin-bottom: 20px;
  padding: 15px;
  border: 1px solid #ddd;
  border-radius: 5px;
}

button {
  padding: 5px 10px;
  font-size: 0.9rem;
}
</style>
