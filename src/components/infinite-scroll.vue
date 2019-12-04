<template>
  <div class="infinite-scroll" :style="{ paddingTop: paddingTop + 'px', paddingBottom: paddingBottom + 'px' }">
    <div class="wrap" v-for="panel in scrollList" :key="panel.page" :class="panel.ref" :ref="panel.ref">
      <div class="list">
        <div v-for="(item, index) in panel.list" :key="item[uniqueKey] || index" :ref="item.ref" :id="item.id">
          <slot :item="item"></slot>
        </div>
      </div>
    </div>
    <div class="loading" v-show="loading">
      <slot name="loading">加载中...</slot>
    </div>
    <div class="ended" v-if="ended">
      <slot name="ended">没有更多了</slot>
    </div>
  </div>
</template>

<script>
  const FIRST_ITEM_ID = '__infinite-scroll-first-item__';
  const LAST_ITEM_ID = '__infinite-scroll-last-item__';
  export default {
    name: "infinite-scroll",
    props: {
      uniqueKey: String,
      loadMore: {
        required: true,
        type: Function,
        default: () => Promise.resolve()
      },
      pageSize: {
        type: Number,
        required: true
      },
    },
    data() {
      return {
        list: [],
        loading: false,
        ended: false,
        scrollList: [],
        intersectionObserver: null,
        firstItem: '',
        lastItem: '',
        page: 0,
        paddingTop: 0,
        paddingBottom: 0
      }
    },
    created() {
      this._loadMore()

      this.intersectionObserver = new IntersectionObserver(entries => {
        for (let entry of entries) {
          if (!entry.isIntersecting) continue;
          if (entry.target.id === FIRST_ITEM_ID) {
            this._scrollTop();
          }
          if (entry.target.id === LAST_ITEM_ID) {
            this._loadMore();
          }
          this.intersectionObserver.unobserve(entry.target);
        }
      });
    },
    mounted() {
    },
    methods: {
      setScrollList(scrollList) {
        scrollList.forEach(panel => {
          panel.list[0].id = ''
          panel.list[0].ref = ''
          panel.list[panel.list.length - 1].id = ''
          panel.list[panel.list.length - 1].ref = ''
        })

        const scrollListLen = scrollList.length - 1
        const len = scrollList[scrollListLen].list.length - 1
        scrollList[0].list[0].id = FIRST_ITEM_ID
        scrollList[0].list[0].ref = 'firstItem'
        scrollList[scrollListLen].list[len].id = LAST_ITEM_ID;
        scrollList[scrollListLen].list[len].ref = 'lastItem';

        this.scrollList = scrollList
      },
      _loadMore() {
        if (this.list.length > this.page * this.pageSize) {
          return this._scrollBottom()
        }
        this.loading = true
        this.loadMore().then(more => {
          this.loading = false
          this.page++
          if (more.length < this.pageSize) this.ended = true;

          const list = JSON.parse(JSON.stringify(more));

          if (this.scrollList.length >= 3) {
            const ref = this.scrollList[0].ref
            const padding = this.$refs[ref][0].scrollHeight;
            this.paddingTop += padding
            const [_, ...other] = this.scrollList
            this.scrollList = [...other]
          }

          const scrollList = [...this.scrollList, {
            page: this.page,
            ref: 'panel-' + this.page,
            list
          }]

          this.setScrollList(scrollList)

          this.list = [...this.list, ...list];
          this.$nextTick(() => {
            this.setIntersectionObserver();
          });
        });
      },
      _scrollBottom() {
        const [first, second, third] = this.scrollList;
        const padding = this.$refs[first.ref][0].scrollHeight
        this.paddingTop += padding;

        const bottomList = this.list.slice(this.page * this.pageSize, (this.page + 1) * this.pageSize)

        this.page++
        const scrollList = [
          second,
          third,
          {
            page: this.page,
            ref: 'panel-' + this.page,
            list: bottomList
          }
        ]
        this.setScrollList(scrollList)

        this.$nextTick(() => {
          this.setIntersectionObserver()

          const scrollList = this.scrollList;
          const ref = scrollList[scrollList.length - 1].ref
          const padding = this.$refs[ref][0].scrollHeight
          this.paddingBottom -= padding;
        })
      },
      _scrollTop() {
        if (this.page <= this.scrollList.length) return
        const [first, second, third] = this.scrollList;

        const padding = this.$refs[third.ref][0].scrollHeight
        this.paddingBottom += padding;

        const topList = this.list.slice((this.page - 4) * this.pageSize, (this.page - 3) * this.pageSize)

        const scrollList = [
          {
            page: this.page - 3,
            ref: 'panel-' + (this.page - 3),
            list: topList
          },
          first,
          second
        ]
        this.setScrollList(scrollList)
        this.page--;

        this.$nextTick(() => {
          this.setIntersectionObserver()

          const scrollList = this.scrollList
          const ref = scrollList[0].ref
          const padding = this.$refs[ref][0].scrollHeight
          this.paddingTop -= padding;
        })
      },
      setIntersectionObserver() {
        const first = this.$refs.firstItem
        if (first) {
          this.firstItem = first[0]
          this.intersectionObserver.observe(this.firstItem)
        }
        this.lastItem = this.$refs.lastItem[0];
        this.intersectionObserver.observe(this.lastItem);
      }
    }
  }
</script>

<style scoped>
  .infinite-scroll {
    width: 100%;
  }
</style>
