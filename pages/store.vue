<template>
  <a-layout class="layout">
    <a-layout-header class="header">
      <div class="logo">🍕 Pizza-Pampano</div>
      <a-menu theme="dark" mode="horizontal" default-selected-keys="['1']" class="nav-menu">
        <a-menu-item key="1" class="menu-item" @click="scrollToHome">Home</a-menu-item>
        <a-menu-item key="2" class="menu-item" @click="scrollToMenu">Menu</a-menu-item>
        <a-menu-item key="3" class="menu-item" @click="viewCart">Cart ({{ cart.length }})</a-menu-item>
        <a-menu-item key="4" class="menu-item" @click="goToIndex">Login</a-menu-item>
      </a-menu>
    </a-layout-header>

    <a-carousel autoplay :infinite="true" style="margin-top: 40px;">
      <div>
        <img src="@/images/BannerOne.gif" alt="Pepperoni Pizza" style="width: 100%; height: 500px; object-fit: fill;" />
      </div>
      <div>
        <img src="@/images/BannerTwo.png" alt="Hawaiian Pizza" style="width: 100%; height: 500px; object-fit: fill;" />
      </div>
    </a-carousel>
    
    <a-row justify="center" ref="homeSection" id="home" style="margin-top: 20px; text-align: center;">
      <a-col :xs="24" :md="20">
        <a-typography-title :level="1" style="color: #002776;">Welcome to Pizza-Pampano 🍕</a-typography-title>
        <p style="font-size: 23px; max-width: 700px; margin: 0 auto; color: #444;">
          Serving the cheesiest, meatiest, and most delicious pizzas in town! Whether you're craving a classic
          Pepperoni or our overloaded Meat Lovers, we've got something fresh and hot for everyone.
        </p>
        <a-button type="primary" size="large" style="margin-top: 25px; background-color: #e31837; border: none;">
          Order Now
        </a-button>
      </a-col>
    </a-row>

    <a-layout-content class="content">
      <a-typography-title ref="menuSection" :level="2" style="text-align:center; margin-bottom: 15px;">
        Explore Our Menu
      </a-typography-title>

      <div style="text-align:center; margin-bottom: 20px;">
        <a-radio-group v-model:value="selectedCategory" button-style="solid">
          <a-radio-button value="All">All</a-radio-button>
          <a-radio-button value="Pizza">Pizza</a-radio-button>
          <a-radio-button value="Sides">Sides</a-radio-button>
          <a-radio-button value="Drinks">Drinks</a-radio-button>
        </a-radio-group>
      </div>

      <a-row :gutter="[16, 24]" justify="center">
        <a-col v-for="item in filteredItems" :key="item.id" :md="8" :lg="9.5">
          <a-card :hoverable="true" class="menu-card">
            <template #cover>
              <div class="image-wrapper">
                <img :src="item.image" :alt="item.name" class="menu-image" />
                <span v-if="item.name.includes('Pepperoni')" class="badge">Best Seller</span>
              </div>
            </template>
            <div class="menu-info">
              <h3 class="menu-title">{{ item.name }}</h3>
              <p class="menu-description">{{ item.description }}</p>
              <p class="menu-price">₱{{ item.price }}</p>
              <a-button type="primary" block class="add-to-cart-btn" @click="addToCart(item)">
                Add to Cart
              </a-button>
            </div>
          </a-card>
        </a-col>
      </a-row>
    </a-layout-content>

    <a-layout-footer style="text-align: center; background-color: #e31837; color: #000;">
      Pizza-Pampano ©2025 | Crafted with Heart and Cheese
    </a-layout-footer>

    <a-modal
      v-model:visible="cartModalVisible"
      title="Your Cart"
      @ok="checkout"
      @cancel="cartModalVisible = false"
      width="60%"
      style="max-width: 900px; border-radius: 15px; background-color: #fff;"
    >
      <a-row>
        <a-col span="24">
          <a-list
            item-layout="horizontal"
            bordered
            v-if="cart.length > 0"
            style="padding: 20px 0; max-height: 400px; overflow-y: auto; border: none;"
          >
            <a-list-item
              v-for="(item, index) in cart"
              :key="index"
              class="cart-item"
              style="border-bottom: 1px solid #ddd; padding: 15px 20px; display: flex; align-items: center;"
            >
              <a-list-item-meta
                :title="item.name"
                :description="`${item.description} - ₱${item.price}`"  
              >
                <template #avatar>
                  <img :src="item.image" alt="Item Image" class="cart-item-img" />
                </template>
              </a-list-item-meta>
              <div class="cart-item-actions" style="display: flex; align-items: center; gap: 10px;">
                <a-button
                  type="link"
                  size="small"
                  @click="removeItem(index)"
                  style="color: #e31837; padding: 0;"
                >
                  Remove
                </a-button>
                <a-input-number
                  v-model:value="item.quantity"
                  :min="1"
                  :max="99"
                  style="width: 80px;"
                  @change="updateQuantity(index, item)"
                />
              </div>
            </a-list-item>
          </a-list>
          <a-divider style="margin-top: 10px;" />
          <div class="cart-total" style="display: flex; justify-content: space-between; padding: 10px 20px; font-weight: bold; font-size: 16px;">
            <span>Total:</span>
            <span style="color: #e31837;">₱{{ cartTotal }}</span>
          </div>
        </a-col>
      </a-row>

      <p v-if="cart.length === 0" style="text-align: center; color: #777;">Your cart is empty.</p>
    </a-modal>


  </a-layout>

  <a-alert
    v-if="alert.visible"
    :message="alert.message"
    :type="alert.type as 'success' | 'warning' | 'error' | 'info'"
    show-icon
    closable
    @close="alert.visible = false"
    style="position: fixed; top: 80px; left: 50%; transform: translateX(-50%); width: 19%; z-index: 2000;"
  />
  </template>

<script lang="ts" setup>
import { ref, computed, reactive } from 'vue';
import { useRouter } from 'vue-router';

const router = useRouter();

const goToIndex = () => {
 
  router.push({ name: 'index' }); 
};

const menuSection = ref<HTMLElement | null>(null);
const homeSection = ref<HTMLElement | null>(null);

const scrollToMenu = () => {
  menuSection.value?.scrollIntoView({ behavior: 'smooth' });
};

const scrollToHome = () => {
  window.scrollTo({ top: 0, behavior: 'smooth' });
  if (homeSection.value) {
    homeSection.value.scrollIntoView({
      behavior: 'smooth',
      block: 'start',
    });
  }
};


import bbqChicken from '@/images/Bbq Chicken.jpg';
import cheesy from '@/images/Cheesy.jpg';
import hawaiian from '@/images/Hawaiian.jpg';
import meatLovers from '@/images/Meat lovers.jpg';
import pepperoni from '@/images/Pepperoni.jpg';
import veggieLovers from '@/images/Veggie lovers.jpg';
import spinach from '@/images/Spinach.jpg';
import baconMushroom from '@/images/Bacon & Mushroom.jpg';
import ultPepperoni from '@/images/Ultimate Pepperoni.jpg';
import carbonara from '@/images/Pizza Carbonara.jpg';
import cheesyBreadsticks from '@/images/Cheesy Breadsticks.jpg';
import garlicParmesanWings from '@/images/Garlic Parmesan Wings.jpg';
import chickenPoppers from '@/images/Chicken Poppers.jpg';
import chickenKickers from '@/images/Chicken Kickers.jpg';
import bakedWings from '@/images/Baked Wings.jpg';
import icedCola from '@/images/Iced cola.jpg';
import rootbeerFloat from '@/images/Rootbeer float.jpg';
import spriteCola from '@/images/Sprite Cola.jpg';
import pepsiCola from '@/images/Pepsi.jpg';
import minuteMaid from '@/images/Minute Maid.jpg';

const selectedCategory = ref('All');

const items = ref([
  { id: 1, name: 'Pepperoni Pizza', description: 'Classic pepperoni pizza with mozzarella and tomato sauce.', price: 399, image: pepperoni, category: 'Pizza' },
  { id: 2, name: 'Hawaiian Pizza', description: 'Ham and pineapple goodness.', price: 429, image: hawaiian, category: 'Pizza' },
  { id: 3, name: 'Cheesy Overload', description: 'Four-cheese blend melted to perfection.', price: 459, image: cheesy, category: 'Pizza' },
  { id: 4, name: 'BBQ Chicken Pizza', description: 'Smoky BBQ sauce topped with grilled chicken.', price: 469, image: bbqChicken, category: 'Pizza' },
  { id: 5, name: 'Meat Lovers Pizza', description: 'Packed with pepperoni, ham, sausage and more.', price: 499, image: meatLovers, category: 'Pizza' },
  { id: 6, name: 'Veggie Lovers Pizza', description: 'Loaded with fresh vegetables and cheese.', price: 389, image: veggieLovers, category: 'Pizza' },
  { id: 7, name: 'Spinach & Feta Pizza', description: 'Cream Cheese, Mozzarella, Spinach, Feta Cheese', price: 469, image: spinach, category: 'Pizza' },
  { id: 8, name: 'Bacon & Mushroom Pizza', description: 'Tomato Ketchup, Mozzarella Cheese, Mayonnaise, Mushroom', price: 169, image: baconMushroom, category: 'Pizza' },
  { id: 9, name: 'Ultimate Pepperoni Pizza', description: 'Pizza Sauce, Mozzarella Cheese, Pepperoni, Havarti Cheese', price: 469, image: ultPepperoni, category: 'Pizza' },
  { id: 10, name: 'Carbonara Pizza', description: 'White Sauce, Mozzarella Cheese, Diced Bacon, Bacon Strips, Parmesan', price: 469, image: carbonara, category: 'Pizza' },
  { id: 11, name: 'Garlic Parmesan Wings', description: 'Crispy wings with savory garlic parmesan glaze.', price: 299, image: garlicParmesanWings, category: 'Sides' },
  { id: 12, name: 'Cheesy Breadsticks', description: 'Mozzarella-filled golden breadsticks.', price: 189, image: cheesyBreadsticks, category: 'Sides' },
  { id: 13, name: 'Chicken Poppers', description: 'Chicken Poppers.', price: 129, image: chickenPoppers, category: 'Sides' },
  { id: 14, name: 'Chicken Kickers', description: 'Oven-baked, battered chicken breast meat with herbs and spices.', price: 299, image: chickenKickers, category: 'Sides' },
  { id: 15, name: 'Baked Wings', description: 'Oven-baked chicken wings made with herbs and special ingredients that you will love.', price: 299, image: bakedWings, category: 'Sides' },
  { id: 16, name: 'Iced Cola', description: 'Chilled and fizzy. Pairs well with any pizza.', price: 49, image: icedCola, category: 'Drinks' },
  { id: 17, name: 'Rootbeer Float', description: 'Classic rootbeer topped with creamy vanilla ice cream.', price: 69, image: rootbeerFloat, category: 'Drinks' },
  { id: 18, name: 'Sprite Cola', description: 'Crisp, lemon-lime refreshment to quench your thirst.', price: 49, image: spriteCola, category: 'Drinks' },
  { id: 19, name: 'Pepsi Cola', description: 'Bold cola flavor that hits the spot every time.', price: 49, image: pepsiCola, category: 'Drinks' },
  { id: 20, name: 'Minute Maid', description: 'Refreshing fruit goodness in every sip, bursting with natural flavor.', price: 39, image: minuteMaid, category: 'Drinks' },
]);

interface CartItem {
  id: number;
  name: string;
  description: string;
  price: number;
  image: string;
  category: string;
  quantity: number; 
}

const cart = ref<CartItem[]>([]);

const removeItem = (index: number) => {
  cart.value.splice(index, 1);
};

const updateQuantity = (index: number, item: CartItem) => {
  cart.value[index] = item;  
};

const cartTotal = computed(() => {
  return cart.value.reduce((total, item) => total + item.price * item.quantity, 0);  
});

const cartModalVisible = ref(false);

const filteredItems = computed(() => {
  if (selectedCategory.value === 'All') return items.value;
  return items.value.filter(item => item.category === selectedCategory.value);
});

const addToCart = (item: any) => {
  const existingItem = cart.value.find(cartItem => cartItem.id === item.id);
  if (existingItem) {
    existingItem.quantity += 1;  
  } else {
    cart.value.push({ ...item, quantity: 1 });
  }
  
  alert.message = `${item.name} has been added to the cart!`;
  alert.type = 'success';
  alert.visible = true;

  setTimeout(() => {
    alert.visible = false;
  }, 1500);
};

const viewCart = () => {
  cartModalVisible.value = true; 
};

const checkout = () => {
  console.log('Proceeding to checkout...');
  cartModalVisible.value = false;
};

const alert = reactive({
  visible: false,
  message: '',
  type: '',
});

</script>



<style scoped>
html, body {
  margin: 0;
  padding: 0;
  background-color: #f8f8f8;
  overflow-x: hidden;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

.header {
  position: sticky;
  top: 0;
  z-index: 1000;
  background:  #0cc0df;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.2); 
  padding: 0 24px;
}

.logo {
  float: left;
  color: #fff;
  font-size: 26px;
  font-weight: bold;
  line-height: 64px;
  font-family: 'Pacifico', cursive; 
  letter-spacing: 1px;
  animation: logoPulse 2s infinite ease-in-out;
}

@keyframes logoPulse {
  0%, 100% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.06);
  }
}


.nav-menu {
  background: transparent !important;
  display: flex;
  justify-content: flex-end;
  align-items: center;
  height: 64px;
  border-bottom: none;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

::v-deep(.menu-item) {
  position: relative;
  font-weight: 1000;
  color: #fff !important;
  padding: 0 16px;
  font-size: 16px;
  transition: color 0.3s;
}



.menu-item::after {
  content: '';
  position: absolute;
  bottom: 8px;
  left: 50%;
  transform: translateX(-50%);
  width: 0%;
  height: 2px;
  background-color: #e31837;
  transition: width 0.3s;
}

.menu-item:hover::after {
  width: 60%;
}

.menu-item:hover {
  color: #e31837 !important;
}



.layout {
  min-height: 100vh;
  background-color: #f8f8f8;
}

.content {
  padding: 120px 20px 50px 20px; 
  max-width: 1200px;
  margin: 0 auto;
}

.menu-card {
  border: 1px solid #ddd;
  background-color: #ffffff;
  color: #333;
  border-radius: 10px;
  overflow: hidden;
  transition: transform 0.3s ease, box-shadow 0.3s ease;
  min-height: 450px;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}

.menu-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 10px 20px rgba(0, 39, 118, 0.2);
}

.image-wrapper {
  position: relative;
  aspect-ratio: 4 / 3;
  overflow: hidden;
}

.menu-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
  border-bottom: 3px solid #e31837;
  transition: transform 0.3s ease;
}

.menu-image:hover {
  transform: scale(1.05);
}

.menu-info {
  padding: 16px;
}

.menu-title {
  font-size: 18px;
  font-weight: 700;
  color: #002776;
  margin-bottom: 5px;
}

.menu-description {
  font-size: 14px;
  color: #666;
  margin-bottom: 10px;
  min-height: 45px;
}

.menu-price {
  font-size: 16px;
  font-weight: bold;
  color: #e31837;
  margin-bottom: 12px;
}

.add-to-cart-btn {
  background-color: #e31837;
  color: #fff;
  border: none;
  font-weight: bold;
  transition: all 0.3s ease;
}

.add-to-cart-btn:hover {
  background-color: #002776;
  color: #fff;
}

.badge {
  position: absolute;
  top: 10px;
  right: 10px;
  background: #e31837;
  color: white;
  padding: 4px 8px;
  font-size: 12px;
  border-radius: 5px;
  font-weight: bold;
}

a-layout-footer {
  background-color: #002776;
  color: #fff;
  padding: 20px 0;
  text-align: center;
}

.cart-item-img {
  width: 60px;
  height: 60px;
  object-fit: cover;
  border-radius: 8px;
}

.cart-item-actions a-button {
  font-weight: bold;
  color: #e31837;
  transition: color 0.3s ease;
}

.cart-item-actions a-button:hover {
  color: #002776;
}

.cart-total {
  background-color: #f8f8f8;
  border-top: 1px solid #ddd;
  padding-top: 10px;
  text-align: right;
}
</style>
