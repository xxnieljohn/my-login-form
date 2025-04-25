<template>
  <a-layout class="admin-layout">
    <a-layout-header class="admin-header">
      <div class="admin-title">🍕 Pizza-Pampano Admin</div>
    </a-layout-header>

    <a-layout-content style="padding: 24px;">
      <a-button type="primary" @click="showAddModal" style="margin-bottom: 16px">
        + Add New Item
      </a-button>

      <a-table :columns="columns" :data-source="items" row-key="id" bordered>
        <template #bodyCell="{ column, record }">
          <template v-if="column.dataIndex === 'actions'">
            <a-button type="link" @click="() => editItem(record as MenuItem)">Edit</a-button>
            <a-button type="link" danger @click="() => deleteItem((record as MenuItem).id)">Delete</a-button>
          </template>
        </template>
      </a-table>

      <a-modal
        v-model:visible="modalVisible"
        :title="editingItem ? 'Edit Item' : 'Add Item'"
        @ok="saveItem"
        @cancel="resetForm"
      >
        <a-form :model="form" layout="vertical">
          <a-form-item label="Name">
            <a-input v-model="form.name" />
          </a-form-item>
          <a-form-item label="Description">
            <a-input v-model="form.description" />
          </a-form-item>
          <a-form-item label="Price">
            <a-input-number v-model="form.price" :min="1" style="width: 100%;" />
          </a-form-item>
          <a-form-item label="Image URL">
            <a-input v-model="form.image" />
          </a-form-item>
          <a-form-item label="Category">
            <a-select v-model="form.category" style="width: 100%;">
              <a-select-option value="Pizza">Pizza</a-select-option>
              <a-select-option value="Sides">Sides</a-select-option>
              <a-select-option value="Drinks">Drinks</a-select-option>
            </a-select>
          </a-form-item>
        </a-form>
      </a-modal>
    </a-layout-content>
  </a-layout>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import { message } from 'ant-design-vue';

interface MenuItem {
  id: number;
  name: string;
  description: string;
  price: number;
  image: string;
  category: string;
}

const items = ref<MenuItem[]>([
  {
    id: 1,
    name: 'Sample Pizza',
    description: 'Yummy pizza.',
    price: 399,
    image: '@/images/Bacon & Mushroom.jpg',
    category: 'Pizza',
  },
]);

const form = ref<Partial<MenuItem>>({});
const modalVisible = ref(false);
const editingItem = ref<MenuItem | null>(null);

const columns = [
  { title: 'Name', dataIndex: 'name' },
  { title: 'Category', dataIndex: 'category' },
  { title: 'Price', dataIndex: 'price' },
  { title: 'Actions', dataIndex: 'actions' },
];

const showAddModal = () => {
  form.value = {};
  editingItem.value = null;
  modalVisible.value = true;
};

const saveItem = () => {
  if (!form.value.name || !form.value.category) {
    message.warning('Please fill out all required fields');
    return;
  }

  if (editingItem.value) {
    Object.assign(editingItem.value, form.value);
    message.success('Item updated successfully');
  } else {
    const newItem = {
      ...form.value,
      id: Date.now(),
    } as MenuItem;
    items.value.push(newItem);
    message.success('Item added successfully');
  }

  resetForm();
};

const editItem = (item: MenuItem | undefined) => {
  if (!item) {
    message.error('Invalid item selected for editing');
    return;
  }
  form.value = { ...item };
  editingItem.value = item;
  modalVisible.value = true;
};

const deleteItem = (id: number) => {
  items.value = items.value.filter((item) => item.id !== id);
  message.success('Item deleted');
};

const resetForm = () => {
  form.value = {};
  editingItem.value = null;
  modalVisible.value = false;
};
</script>

<style scoped>
.admin-layout {
  min-height: 100vh;
  background: #fff;
}

.admin-header {
  background: #0cc0df;
  padding: 0 20px;
  color: #fff;
  font-size: 20px;
  font-weight: bold;
  line-height: 64px;
}

.admin-title {
  font-family: 'Pacifico', cursive;
  letter-spacing: 1px;
}
</style>
