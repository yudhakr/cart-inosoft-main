<template>
  <Navigation :totalCartItems="totalItemCart" />
  <main class="py-3 px-5 position-relative">
    <Transition name="fade">
      <AddProductForm
        @showAddProductForm="showAddProductForm"
        v-if="addForm"
        v-model:name="addFormData.name"
        v-model:description="addFormData.description"
        v-model:price="addFormData.price"
        v-model:stock="addFormData.stock"
        @addProduct="addProduct"
      />
    </Transition>
    <Products 
      :data="availableProducts" 
      @addItem="addOneItem" 
      @addAll="addAllItem"
      @showAddProductForm="showAddProductForm"
    />
    <Cart 
      :data="availableCart" 
      @deleteItem="deleteItem" 
      @deleteAll="deleteAllItem"
      @checkout="checkout"
      :totalPrice="totalPriceCart"
    />
  </main>
</template>

<script setup>
import Navigation from './components/Navigation.vue'
import Products from './components/Products.vue';
import Cart from './components/Cart.vue';
import AddProductForm from './components/AddProductForm.vue';

import { ref,computed,reactive, Transition } from 'vue';
import Swal from 'sweetalert2'

const products = ref([
  {     
    id: 1, 
    name: 'Indomie Aceh', 
    description: 'Mie instan produk indonesia',
    stock: 10,
    price: 3900 
  },
  {     
    id: 2, 
    name: 'Mie Gelas Cup', 
    description: 'Mie dengan Cup dengan diseduh air panas',
    stock: 3,
    price: 1500 
  },
  {     
    id: 3, 
    name: 'Indomie Kari Ayam', 
    description: 'Dengan rasa ayam kari yang kental dan enak',
    stock: 20,
    price: 2000 
  },
  {     
    id: 4, 
    name: 'Mie Sedap goreng', 
    description: 'Dengan porsi banyak dan mie tebal',
    stock: 80,
    price: 10000 
  },
])

const availableProducts = computed(() => products.value.filter(product => product.stock > 0));

const cart = ref([])
const availableCart = computed(() => cart.value.filter(product => product.stock > 0));

const addForm = ref(false)
const showAddProductForm = () => {
  addForm.value = !addForm.value
}

const addFormData = reactive({
  id: Math.floor(Math.random() * 1000),
  name: '',
  description: '',
  stock: 0,
  price: 0
})

const addProduct = () => {
  products.value.push(addFormData)
  addForm.value = false
}

const addOneItem = (id) => {
  const product = products.value.find(product => product.id === id)
  if (product.stock > 0) {
    product.stock -= 1
    const item = cart.value.find(item => item.id === id)
    if (item) {
      item.stock += 1
      item.price = item.stock * product.price
    } else {
      cart.value.push({
        id: product.id,
        name: product.name,
        stock: 1,
        price: product.price
      })
    }
  }
}

const addAllItem = (id) => {
  const product = products.value.find(product => product.id === id)
  if (product.stock > 0) {
    const item = cart.value.find(item => item.id === id)
    if (item) {
      item.stock += product.stock
      item.price = item.stock * product.price
    } else {
      cart.value.push({
        id: product.id,
        name: product.name,
        stock: product.stock,
        price: product.price
      })
    }

    product.stock = 0;
  }
}

const deleteItem = (id) => {
  const product = products.value.find(product => product.id === id)
  const item = cart.value.find(item => item.id === id)
  if (item) {
    product.stock += 1
    item.stock -= 1;
    item.price = item.stock * product.price
  }
}

const deleteAllItem = (id) => {
  const product = products.value.find(product => product.id === id)
  const item = cart.value.find(item => item.id === id)
  if (item) {
    product.stock += item.stock
    item.stock = 0;
    item.price = item.stock * product.price
  }
}

const totalItemCart = computed(() => {
  let total = 0;
  cart.value.forEach(item => {
    total += item.stock
  })
  return total
})

const totalPriceCart = computed(() => {
  let total = 0;
  cart.value.forEach(item => {
    total += item.price
  })
  return total
})

const checkout = () => {
  Swal.fire({
    title: 'Are you sure?',
    text: `You will checkout your cart with total price Rp. ${totalPriceCart.value}`,
    icon: 'warning',
    showCancelButton: true,
    confirmButtonColor: '#3085d6',
    cancelButtonColor: '#d33',
    confirmButtonText: 'Yes, checkout!'
  }).then((result) => {
    if (result.isConfirmed) {
      Swal.fire(
        'Checkout!',
        'Your cart has been checkout. Click OK to reload page.',
        'success'
        ).then((result) => {
          if (result.isConfirmed) {
            location.reload()
          }
        }) 
    }
  })
}

</script>


<style scoped>
.fade-enter-active, .fade-leave-active {
  transition: opacity .5s;
}
.fade-enter, .fade-leave-to {
  opacity: 0;
}

</style>
