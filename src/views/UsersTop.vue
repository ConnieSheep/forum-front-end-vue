<template>
  <div class="container py-5">
    <NavTabs />
    <Spinner v-if="isLoading" />
    <template v-else>
      <h1 class="mt-5">美食達人</h1>
      <hr />
      <div class="row text-center">
        <div v-for="user in users" :key="user.id" class="col-3">
          <router-link :to="{ name: 'user', params: { id: user.id } }">
            <img :src="user.image | emptyImage" width="140px" height="140px"
          /></router-link>
          <h2>{{ user.name }}</h2>
          <span class="badge badge-secondary"
            >追蹤人數 : {{ user.followerCount }}</span
          >
          <p class="mt-3" v-show="currentUser.id !== user.id">
            <button
              v-if="user.isFollowed"
              type="button"
              class="btn btn-danger"
              @click.stop.prevent="deleteFollowing(user.id)"
              :disabled="isProcessing"
            >
              取消追蹤
            </button>
            <button
              v-else
              type="button"
              class="btn btn-primary"
              @click.stop.prevent="addFollowing(user.id)"
              :disabled="isProcessing"
            >
              加入追蹤
            </button>
          </p>
        </div>
      </div>
    </template>
  </div>
</template>

<script>
import NavTabs from './../components/NavTabs'
import { emptyImageFilter } from './../utils/mixins'
import usersAPI from './../apis/users'
import { Toast } from './../utils/helpers'
import { mapState } from 'vuex'
import Spinner from './../components/Spinner'

export default {
  components: {
    NavTabs,
    Spinner
  },
  mixins: [emptyImageFilter],
  data() {
    return {
      users: [],
      isLoading: true,
      isProcessing: false
    }
  },
  computed: {
    ...mapState(['currentUser'])
  },
  created() {
    this.fetchTopUsers()
  },
  methods: {
    async fetchTopUsers() {
      try {
        this.isLoading = true
        const { data } = await usersAPI.getTopUsers()

        if (data.status === 'error') {
          throw new Error(data.message)
        }

        this.users = data.users.map(user => ({
          id: user.id,
          name: user.name,
          image: user.image,
          followerCount: user.FollowerCount,
          isFollowed: user.isFollowed
        }))
        this.isLoading = false
      } catch (error) {
        this.isLoading = false
        console.log(error)

        Toast.fire({
          icon: 'error',
          title: '無法取得美食達人資料，請稍後再試'
        })
      }
    },
    async addFollowing(userId) {
      try {
        this.isProcessing = true
        const { data } = await usersAPI.addFollowing({ userId })

        if (data.status === 'error') {
          throw new Error(data.message)
        }
        this.users = this.users.map(user => {
          if (user.id !== userId) {
            return user
          } else {
            return {
              ...user,
              followerCount: user.followerCount + 1,
              isFollowed: true
            }
          }
        })
        this.isProcessing = false
      } catch (error) {
        this.isProcessing = false
        console.log(error)

        Toast.fire({
          icon: 'error',
          title: '無法加入追蹤，請稍後再試'
        })
      }
    },
    async deleteFollowing(userId) {
      try {
        this.isProcessing = true
        const { data } = await usersAPI.deleteFollowing({ userId })

        if (data.status === 'error') {
          throw new Error(data.message)
        }
        this.users = this.users.map(user => {
          if (user.id !== userId) {
            return user
          } else {
            return {
              ...user,
              followerCount: user.followerCount - 1,
              isFollowed: false
            }
          }
        })
        this.isProcessing = false
      } catch (error) {
        this.isProcessing = false
        console.log(error)

        Toast.fire({
          icon: 'error',
          title: '無法取消追蹤，請稍後再試'
        })
      }
    }
  }
}
</script>