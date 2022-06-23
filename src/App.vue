<template>
  <div id="app" :class="(theme === 'dark-theme') ? 'dark-theme' : 'light-theme'">
    <main>
      <div class="theme-toggle">
      <input
        @change="toggleTheme"
        id="checkbox"
        type="checkbox"
        class="switch-checkbox"
      />
      <label for="checkbox" class="switch-label">Dark Mode</label>
      <div
        class="switch-toggle"
        :class="{ 'switch-toggle-checked': userTheme === 'dark-theme' }"
      ></div>
      </div>
      <div class="search-box">
        <input type="text" 
        class="search-bar" 
        placeholder="Search..."
        v-model="query"
        @keypress="fetchProfile">
      </div>
      <div class="noprofile" v-if="notFound == true">
        <h2>User not found</h2>
      </div>
       <div class="avatar">
        <img :src="profile.avatar_url" alt="">
      </div>
      <div class="username">{{profile.login}}</div>
      <div class="profile-wrap" v-if="result == true">
        <div class="followers">Followers: {{profile.followers}}</div>
        <div class="repositories">Public Repositories: {{profile.public_repos}}</div>
        <div class="recentrepos">Recent Repos: </div>
        <li v-for="repos in reposarray" :key="repos.id">
          {{repos.name}}
        </li>
      </div>
    </main>
  </div>
</template>

<script>

export default {

  name: 'App',
  data(){
    return {
      theme: "",
      baseURL: 'https://api.github.com/',
      query: '',
      profile: {},
      result: false,
      notFound: false,
      reposarray: []
    }
  },
  methods: {

    async fetchProfile(e){
      if (e.key == "Enter"){
        this.notFound = false;
        this.result = false;
        let response = await fetch(`${this.baseURL}users/${this.query}`);
        let reposData = await fetch(`${this.baseURL}users/${this.query}/repos`)
        let json = await response.json();
        reposData.json().then( (response) => {
        try {
          response.sort((a, b) => {
            if (a.created_at < b.created_at) {
              return 1;
            }
            if (a.created_at > b.created_at) {
              return -1;
            }
            return 0;
          });
          this.reposarray = response.slice(0,4);
          
        } catch (error) {
            console.log(error)
        }
      });
        this.profile = await json;
        console.log(this.repos);
        
        if(json.message == "Not Found"){
          this.notFound = true;
        }else{
          this.result = true;
        }
      }
    },

    setTheme(theme){
      localStorage.setItem("user-theme", theme);
      this.userTheme = theme;
      document.documentElement.className = theme;
    },

    toggleTheme() {
      const activeTheme = localStorage.getItem("user-theme");
      if (activeTheme === "light-theme") {
        this.setTheme("dark-theme");
      } else {
        this.setTheme("light-theme");
      }
    }
  }
}
</script>

<style>

:root {
  --background-color-primary: #ebebeb;
  --text-primary-color: #222;
  --element-size: 4rem;
}

:root.dark-theme {
  --background-color-primary: #212529;
  --text-primary-color: #899095;
}

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'montserrat', sans-serif;
  background-color: var(--background-color-primary);
  color: var(--text-primary-color);
}

#app {
  background-size: cover;
  background-position: bottom;
  transition: 0.4s;
}

main {
  min-height: 100vh;
  padding: 25px;
}

.search-box {
  width: 100%;
  margin-bottom: 30px;
}

.search-box .search-bar {
  display: block;
  width: 100%;
  padding: 15px;

  color: #313131;
  font-size: 20px;

  appearance: none;
  border: none;
  outline: none;
  background: none;

  box-shadow: 0px 0px 16px rgba(0, 0, 0, 0.25);
  background-color: rgba(255, 255, 255, 0.5);
  border-radius: 0px 16px 0px 16px;
  transition: 00.4s;
}

.search-box .search-bar:focus {
  box-shadow: 0px 0px 16px rgba(0, 0, 0, 0.25);
  background-color: rgba(255, 255, 255, 0.75);
  border-radius: 16px 0px 16px 0px;
}

.avatar {
  display: flex;
  justify-content: center;
}

.username {
  display: flex;
  justify-content: center;
  padding: 20px;
  font-size: 25px;
}

.profile-wrap {
  display: grid;
  grid-template-columns: 250px 250px;
  grid-row-gap: 20px;
  justify-content: center;
}

.theme-toggle {
  margin: 10px;
}

.switch-label {
  margin: 10px;
}

</style>
