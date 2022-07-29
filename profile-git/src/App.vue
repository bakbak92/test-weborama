<template>
  <div>
    <h1>Recherche un profil sur github</h1>
    <form @submit.prevent="search">
      <input type="text" v-model="valueSearch">
        <button type="submit">test</button>
    </form>
    <div v-if="profil">
      <h2>{{profil.name}}</h2>
      <p>Créé le {{profil.created_at}}</p>
      <img :src="profil.avatar_url">
      <div>
        <div v-for="repo in repos" :key="repo.id"
             class="repo"
        >
         <h3><a :href="repo.url">lien repo : </a></h3>
         <p>description: {{repo.description}}</p>
         <p>créé le {{repo.created_at}}</p>
         <p v-if="repo.last_assignee">Dernier ticket assigné à {{repo.last_assignee.login}}</p>
        </div>
      </div>
    </div>
    <div class="error" v-if="error">
      Ce profil n'existe pas
    </div>
  </div>
</template>
<script>
  import axios from "axios";

  export default {
    data(){
      return {
        valueSearch: null,
        profil: null,
        loading: false,
        repos: [],
        error: null
      }
    },
    methods: {
      /* 
       liste des dépôts Git du profil, avec pour chaque dépôt
■ un lien vers le dépôt sur GitHub
■ la description du dépôt
■ la date de création du dépôt
■ le titre du ticket le plus récent et le nom de l'utilisateur assigné
 (Bonus) Les cas d’erreur sont affichés proprement (pas de profil correspondant,
erreur lors de l'accès à l'API GitHub, ...).

      */
      search(){
        this.loading = true;
        axios.get(`https://api.github.com/users/${this.valueSearch}`).then((response) => {
          this.profil = response.data;
          this.error = false;
          axios.get(this.profil.repos_url).then((response) => {
            this.repos = response.data;
            // Promise.all(
            //   this.repos.map((repo) => {
            //     return this.getAssignee(repo.full_name)
            //   })
            // )
            this.repos.forEach(repo => {
              axios.get(`https://api.github.com/repos/${repo.full_name}/assignees`).then((response) => {
                console.log(response.data);
                let assignees = response.data;
                repo.last_assignee = assignees.length ? assignees[assignees.length - 1] : null;
                console.log(repo);
              })
            });
          })
        }).catch((error) => {
          console.log(error)
          this.error = true;
        })
      },
      // getAssignee(repo){
      //   return new Promise((resolve) => {
      //     axios.get(`https://api.github.com/repos/${repo.full_name}/assignees`).then((response) => {
      //       console.log(response.data);
      //       let assignees = response.data;
      //       repo.fi
      //       repo.last_assignee = assignees.length ? assignees[assignees.length - 1] : null;
      //       console.log(repo);
      //       resolve();
      //     })
      //   })
      // }
    }
  }
</script>
<style scoped>
.repo{
  border-bottom: 1px solid #eeee;
  padding: 16px 0;
}
header {
  line-height: 1.5;
  max-height: 100vh;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

nav {
  width: 100%;
  font-size: 12px;
  text-align: center;
  margin-top: 2rem;
}

nav a.router-link-exact-active {
  color: var(--color-text);
}

nav a.router-link-exact-active:hover {
  background-color: transparent;
}

nav a {
  display: inline-block;
  padding: 0 1rem;
  border-left: 1px solid var(--color-border);
}

nav a:first-of-type {
  border: 0;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }

  nav {
    text-align: left;
    margin-left: -1rem;
    font-size: 1rem;

    padding: 1rem 0;
    margin-top: 1rem;
  }
}
</style>
