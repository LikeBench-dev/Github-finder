<template>
  <div class="wrapper-content wrapper-content--fixed">
    <section>
      <div class="container">
  
        <!--        errors-->
        <div class="error" v-if="error">
          <p>{{ error }}</p>
        </div>
        
<!--        search-->
        <search
                :value="search"
                placeholder="Type user name..."
                @search="search = $event"
        />
        
        <!--      buttons-->
        <button v-if="!repos"
                class="btn btnPrimary"
                @click="getRepos">
          Search!</button>
        <button v-else
                class="btn btnPrimary"
                @click="getRepos">
          Search again!</button>
  
  
        
        <!--          info-->
        <div class="person-info repos__wrapper" v-if="person">
          <img :src="person.avatar_url" :alt="person.name">
          <div class="person-item">
            <h3>{{ person.name }}</h3>
            <p>Repositories: <span>  {{ person.public_repos }}</span></p>
          </div>
  
        </div>
        
        
        <!--        wrapper-->
        <div class="repos__wrapper" v-if="repos">
          
          <div class="repos-info-head">
            <p @click="sort('name')">Name &#8595;</p>
            <p @click="sort('stargazers_count')">Star &#8595;</p>
          </div>
<!--          item-->
          <div class="repos-item" v-for="repo in reposSort" :key="repo.id">
            <div class="repos-info">
              <a class="link" target="_blank" :href="repo.html_url">{{
                repo.name
                }}</a>
              <span>{{ repo.stargazers_count }} ⭐</span>
            </div>
          </div>
          
          
<!--          pagination-->
          <section>
            <div class="container">
              <div class="button-list">
        
                <!--          left-->
                <div class="btn btnPrimary" @click="prevPage">
                  &#8592;
                </div>
  
                <p> Page {{ this.page.current }} / {{ numberOfPages }}</p>
                
                <!--          right-->
                <div class="btn btnPrimary" @click="nextPage">
                  &#8594;
                </div>
              </div>
            </div>
          </section>
          
        </div>
        
      </div>
      
    </section>
  </div>
</template>


<script>
import axios from 'axios'

import search from '@/components/Search.vue'
export default {
    components: {
        search
    },
    data () {
        return {
            search: '',
            error: null,
            repos: null,
            person: null,
            page: {
                current: 1,
                length: 4
            },
            currentSort: 'name',
            currentSortDir: 'asc'
        }
    },
    computed: {
        reposSort () {
            return this.repos.sort((a, b) => {
                let mod = 1
                if (this.currentSortDir === 'desc') mod = -1
                if (a[this.currentSort] < b[this.currentSort]) return -1 * mod
                if (a[this.currentSort] > b[this.currentSort]) return 1 * mod
                return 0
            }).filter((row, index) => {
                let start = (this.page.current-1)*this.page.length
                let end = this.page.current * this.page.length
                if (index >= start && index < end) return true
            })
        },
        numberOfPages () { return Math.ceil(this.repos.length /
            this.page.length) }
    },
    methods: {
        getRepos () {
            axios
            .get(`https://api.github.com/users/${this.search}/repos`)
            .then((res) => {
                this.page.current = 1
                this.repos = res.data
                this.error = null
            })
            .catch(err => {
                console.log(err)
                this.page.current = 1
                this.repos = null
                this.error = `Can't find this user`
            })
            axios
                .get(`https://api.github.com/users/${this.search}`)
                .then((res) => {
                    this.person = res.data
                })
        },
        sort (e) {
            if (e === this.currentSort) {
                this.currentSortDir = this.currentSortDir === 'asc' ?
                    'desc' : 'asc'
            }
            this.currentSort = e
        },
        // pagination
        prevPage () {
            if (this.page.current > 1) this.page.current -= 1
        },
        nextPage () {
            if ((this.page.current * this.page.length) < this.repos.length)
                this.page.current += 1
        }
    }

}
</script>

<style lang="scss" scoped>
.container {
  display: flex;
  align-items: center;
  flex-direction: column;
}
button {
  margin-top: 40px;
}
.repos__wrapper {
  width: 400px;
  margin: 30px 0;
}
.repos-info {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 10px 0;
  margin-bottom: 10px;
  border-bottom: 1px solid #dbdbdb;
}
.repos-info-head {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 10px 0;
  margin-bottom: 10px;
  p {
    font-weight: 500;
  }
}
.error {
  margin-bottom: 20px;
}
.person-info {
  display: flex;
  flex-direction: column;
  align-content: center;
  img {
    margin: 0 auto;
    width: 100%;
    max-width: 250px;
    border-radius: 50%;
  }
  .person-item {
    display: flex;
    align-content: center;
    justify-content: space-between;
    margin: 20px 0 50px;
    h3 {
      font-size: 21px;
      font-weight: 500;
    }
    span {
      font-size: 21px;
      font-weight: 500;
    }
  }
}
.button-list {
  display: flex;
  justify-content: space-between;
  align-content: center;
  text-align: center;
  width: 100%;
  p {
    align-self: center;
  }
  .btn {
    border-right: 60px;
    margin: 0 20px;
  }
}
</style>
