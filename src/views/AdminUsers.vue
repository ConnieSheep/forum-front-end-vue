<template>
  <div class="container py-5">
    <!-- AdminNav Component -->
    <AdminNav />
    <Spinner v-if="isLoading" />
    <table v-else class="table">
      <thead class="thead-dark">
        <tr>
          <th scope="col">#</th>
          <th scope="col">Email</th>
          <th scope="col" width="140">Role</th>
          <th scope="col" width="140">Action</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="user in users" :key="user.id">
          <th scope="row">{{ user.id }}</th>
          <td>{{ user.email }}</td>
          <td>{{ user.isAdmin ? 'Admin' : 'User' }}</td>
          <td>
            <button
              v-if="user.id !== currentUser.id"
              type="button"
              class="btn btn-link"
              @click.stop.prevent="
                toggleUserRole({ userId: user.id, isAdmin: user.isAdmin })
              "
              :disabled="isProcessing"
            >
              {{ user.isAdmin ? 'set as user' : 'set as admin' }}
            </button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
import AdminNav from './../components/AdminNav'
import adminAPI from './../apis/admin'
import { Toast } from './../utils/helpers'
import { mapState } from 'vuex'
import Spinner from './../components/Spinner'

export default {
  components: {
    AdminNav,
    Spinner
  },
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
    this.fetchUsers()
  },
  methods: {
    async fetchUsers() {
      try {
        this.isLoading = true
        const { data } = await adminAPI.users.get()

        if (data.status === 'error') {
          throw new Error(data.message)
        }

        this.users = data.users
        this.isLoading = false
      } catch (error) {
        this.isLoading = false
        console.log(error)

        Toast.fire({
          icon: 'error',
          title: '無法取得使用者資料，請稍後再試'
        })
      }
    },
    async toggleUserRole({ userId, isAdmin }) {
      try {
        this.isProcessing = true
        const { data } = await adminAPI.users.update({
          userId,
          isAdmin: (!isAdmin).toString()
        })

        if (data.status === 'error') {
          throw new Error(data.message)
        }

        this.users = this.users.map(user => {
          if (user.id === userId) {
            return {
              ...user,
              isAdmin: !isAdmin
            }
          }
          return user
        })
        this.isProcessing = false
      } catch (error) {
        this.isProcessing = false
        console.log(error)

        Toast.fire({
          icon: 'error',
          title: '無法更改使用者角色，請稍後再試'
        })
      }
    }
  }
}
</script>