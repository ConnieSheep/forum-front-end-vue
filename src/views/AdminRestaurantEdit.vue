<template>
  <div class="container py-5">
    <AdminRestaurantForm
      :initial-restaurant="restaurant"
      @after-submit="handleAfterSubmit"
      :is-processing="isProcessing"
    />
  </div>
</template>

<script>
import AdminRestaurantForm from './../components/AdminRestaurantForm'
import adminAPI from './../apis/admin'
import { Toast } from './../utils/helpers'

export default {
  components: {
    AdminRestaurantForm
  },
  data() {
    return {
      restaurant: {
        id: -1,
        name: '',
        categoryId: '',
        tel: '',
        address: '',
        description: '',
        image: '',
        openingHours: ''
      },
      isProcessing: false
    }
  },
  created() {
    const { id } = this.$route.params
    this.fetchRestaurant(id)
  },
  beforeRouteUpdate(to, from, next) {
    const { id } = to.params
    this.fetchRestaurant(id)
    next()
  },
  methods: {
    async fetchRestaurant(restaurantId) {
      try {
        const { data } = await adminAPI.restaurants.getDetail({ restaurantId })

        if (data.status === 'error') {
          throw new Error(data.message)
        }

        const {
          id,
          name,
          tel,
          address,
          opening_hours: openingHours,
          description,
          image,
          CategoryId: categoryId
        } = data.restaurant

        this.restaurant = {
          ...this.restaurant,
          id,
          name,
          categoryId,
          tel,
          address,
          description,
          image,
          openingHours
        }
      } catch (error) {
        console.log(error)

        Toast.fire({
          icon: 'error',
          title: '無法取得餐廳資料，請稍後再試'
        })
      }
    },
    async handleAfterSubmit(formData) {
      try {
        this.isProcessing = true
        const { data } = await adminAPI.restaurants.update({
          restaurantId: this.restaurant.id,
          formData
        })

        if (data.status === 'error') {
          throw new Error(data.message)
        }
        this.$router.push({ name: 'admin-restaurants' })
      } catch (error) {
        this.isProcessing = false
        console.log(error)

        Toast.fire({
          icon: 'error',
          title: '無法更新餐廳資料，請稍後再試'
        })
      }
    }
  }
}
</script>