<template>
  <div id="app">
    <div class="scroll">
      <InfiniteScroll :loadMore="getList" unique-key="name" :page-size="pageSize">
        <template v-slot="{ item }">
          <div class="user-item">
            <img class="avatar" :src="item.avatar" alt="">
            <div class="user">
              <div class="username">{{item.name}}</div>
              <div class="age">年龄 {{item.age}}</div>
              <template v-if="item.gender === 1">
                <div class="age">yo~~~</div>
                <div class="age">yo~~~</div>
                <div class="age">yo~~~</div>
              </template>
            </div>
          </div>
        </template>
      </InfiniteScroll>
    </div>
  </div>
</template>

<script>
  import InfiniteScroll from './components/infinite-scroll'

  export default {
    name: 'app',
    components: {
      InfiniteScroll
    },
    data() {
      return {
        list: [],
        page: 0,
        pageSize: 20,
        lastItem: '',
        intersectionObserver: null,
        showLoading: ''
      }
    },
    created() {
    },
    mounted() {

    },
    methods: {
      async getList() {
        this.page++;

        const list = await this.requestList({ page: this.page, pageSize: this.pageSize })

        this.list.push(...list)
        return Promise.resolve(list)
      },
      requestList({ page, pageSize }) {
        return new Promise((resolve) => {
          const start = (page - 1) * pageSize
          const end = page * 20
          const list = []

          for (let i = start; i < end; i++) {
            list.push({
              name: '路人' + i,
              age: Math.floor(Math.random() * 100),
              gender: Math.floor(Math.random() * 2),
              avatar: 'https://user-gold-cdn.xitu.io/2019/5/15/16ab9a1cccad1ef3?imageView2/1/w/180/h/180/q/85/format/webp/interlace/1'
            })
          }

          setTimeout(() => {
            return resolve(list)
          }, 1000)
        })
      }
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
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .scroll {
    width: 400px;
    height: 600px;
    overflow: scroll;
    border: 1px solid #e5e6e7;
    border-radius: 4px;
  }

  .user-item {
    display: flex;
    align-items: center;
    padding: 16px;
    border-bottom: 1px solid #e5e6e7;
  }

  .user {
    padding-left: 16px;
    display: flex;
    flex-direction: column;
    align-items: flex-start;
  }

  .username {
    font-size: 18px;
  }

  .age {
    font-size: 14px;
    color: #999;
  }

  .avatar {
    width: 60px;
    height: 60px;
  }
</style>
