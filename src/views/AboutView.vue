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
      storage: null as PouchDB.Database | null
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
            // Crée un tableau avec uniquement les données nécessaires, en gérant les valeurs optionnelles
            this.postsData = result.rows.map((row: any) => ({
              _id: row.id,
              _rev: row.doc?._rev, // Utilisation de l'opérateur optionnel pour éviter les erreurs
              doc: row.doc || {} // Assurez-vous d'avoir un objet vide si `doc` est indéfini
            })) as Post[] // Casting en `Post[]` pour correspondre au type
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
      const newName = prompt('Modifier le nom du post:', post.doc.post_name)
      if (newName) {
        post.doc.post_name = newName
        if (this.storage) {
          this.storage
            .put(post)
            .then(() => console.log('Mise à jour réussie'))
            .catch((error) => console.error('Erreur de mise à jour:', error))
        }
      }
    },
    deletePost(post: Post) {
      if (this.storage && post._rev) {
        // Vérifier que _rev est présent
        this.storage
          .remove(post._id, post._rev)
          .then(() => {
            console.log('Suppression réussie')
            this.fetchData() // Rafraîchir les données après suppression
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
  <h1>Nombre de post: {{ postsData.length }}</h1>
  <button @click="addFakePost">Ajouter un Post</button>
  <ul>
    <li v-for="post in postsData" :key="post._id">
      <div class="ucfirst">
        {{ post.doc.post_name
        }}<em style="font-size: x-small" v-if="post.doc.attributes?.creation_date">
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

.buttons {
  display: flex;
  gap: 5px;
}

button {
  padding: 5px 10px;
  font-size: 0.9rem;
}
</style>
