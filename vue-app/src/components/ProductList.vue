<script setup lang="ts">
import { ref, computed } from 'vue'
import type { Product } from '../types/Product'

// Mock Data - ข้อมูลทดสอบ
const products = ref<Product[]>([
  {
    id: 1,
    name: 'ข้าวผัด',
    sku: 'FOOD-001',
    price: 45,
    stock: 20,
    category: 'อาหาร',
    createdAt: new Date('2024-01-15')
  },
  {
    id: 2,
    name: 'น้ำส้ม',
    sku: 'DRINK-001',
    price: 25,
    stock: 50,
    category: 'เครื่องดื่ม',
    createdAt: new Date('2024-01-16')
  },
  {
    id: 3,
    name: 'สบู่',
    sku: 'SUPPLY-001',
    price: 35,
    stock: 0,
    category: 'ของใช้',
    createdAt: new Date('2024-01-17')
  },
  {
    id: 4,
    name: 'เสื้อยืด',
    sku: 'CLOTH-001',
    price: 299,
    stock: 5,
    category: 'เสื้อผ้า',
    createdAt: new Date('2024-01-18')
  },
  {
    id: 5,
    name: 'กาแฟ',
    sku: 'DRINK-002',
    price: 55,
    stock: 8,
    category: 'เครื่องดื่ม',
    createdAt: new Date('2024-01-19')
  }
])

// Category Filter
const selectedCategory = ref('ทั้งหมด')

// Form Data
const newProduct = ref({
  name: '',
  sku: '',
  price: 0,
  stock: 0,
  category: 'อาหาร'
})

const formErrors = ref({
  name: '',
  sku: '',
  price: '',
  stock: ''
})

// ฟังก์ชันกรองสินค้าตามหมวดหมู่
const getFilteredProducts = computed(() => {
  if (selectedCategory.value === 'ทั้งหมด') {
    return products.value
  }
  return products.value.filter(product => product.category === selectedCategory.value)
})

const totalProducts = computed(() => {
  return getFilteredProducts.value.length
})

const totalValue = computed(() => {
  return getFilteredProducts.value.reduce((sum, product) => {
    return sum + (product.price * product.stock)
  }, 0)
})

const isFormValid = computed(() => {
  return newProduct.value.name.length >= 3 &&
         newProduct.value.sku.length > 0 &&
         !products.value.some(p => p.sku === newProduct.value.sku) &&
         newProduct.value.price > 0 &&  
         newProduct.value.stock >= 0
})

const validateForm = () => {
  formErrors.value.name = newProduct.value.name.length < 3 
    ? 'ชื่อสินค้าต้องมีอย่างน้อย 3 ตัวอักษร' 
    : ''
  
  formErrors.value.sku = newProduct.value.sku.length === 0 
    ? 'กรุณากรอก SKU' 
    : products.value.some(p => p.sku === newProduct.value.sku)
    ? 'SKU นี้มีอยู่ในระบบแล้ว'
    : ''
  
  formErrors.value.price = newProduct.value.price <= 0 
    ? 'ราคาต้องมากกว่า 0' 
    : ''
  
  formErrors.value.stock = newProduct.value.stock < 0 
    ? 'สต็อกต้องมากกว่าหรือเท่ากับ 0' 
    : ''
}

const sellProduct = (productId: number) => {
  const product = products.value.find(p => p.id === productId)
  if (!product) return

  if (product.stock === 0) {
    alert(' ไม่สามารถขายได้ สินค้าหมด!')
    return
  }

  product.stock -= 1
  alert(` ขายสำเร็จ! ${product.name} เหลือสต็อก ${product.stock} ชิ้น`)
}

const addProduct = () => {
  validateForm()
  
  if (!isFormValid.value) {
    alert(' กรุณากรอกข้อมูลให้ถูกต้อง')
    return
  }

  const product: Product = {
    id: Date.now(),
    name: newProduct.value.name,
    sku: newProduct.value.sku,
    price: newProduct.value.price,
    stock: newProduct.value.stock,
    category: newProduct.value.category,
    createdAt: new Date()
  }

  products.value.push(product)
  
  // Clear form
  newProduct.value = {
    name: '',
    sku: '',
    price: 0,
    stock: 0,
    category: 'อาหาร'
  }
  
  formErrors.value = {
    name: '',
    sku: '',
    price: '',
    stock: ''
  }

  alert(' เพิ่มสินค้าสำเร็จ!')
}

// ฟังก์ชันจัดรูปแบบราคา
const formatPrice = (price: number): string => {
  return price.toFixed(2)
}

// เช็คว่าสินค้าหมดหรือไม่
const isOutOfStock = (stock: number): boolean => {
  return stock === 0
}

// เช็คว่าสินค้าใกล้หมดหรือไม่
const isLowStock = (stock: number): boolean => {
  return stock > 0 && stock < 10
}
</script>

<template>
  <div class="product-list-container">
    <h1>รายการสินค้า</h1>
    
    <div class="add-product-form">
      <h2>เพิ่มสินค้าใหม่</h2>
      <form @submit.prevent="addProduct">
        <div class="form-row">
          <div class="form-group">
            <label>ชื่อสินค้า</label>
            <input 
              v-model="newProduct.name" 
              @input="validateForm"
              type="text" 
              placeholder="ชื่อสินค้า (อย่างน้อย 3 ตัวอักษร)"
            />
            <span v-if="formErrors.name" class="error">{{ formErrors.name }}</span>
          </div>

          <div class="form-group">
            <label>SKU</label>
            <input 
              v-model="newProduct.sku" 
              @input="validateForm"
              type="text" 
              placeholder="รหัสสินค้า (ไม่ซ้ำ)"
            />
            <span v-if="formErrors.sku" class="error">{{ formErrors.sku }}</span>
          </div>

          <div class="form-group">
            <label>ราคา (บาท)</label>
            <input 
              v-model.number="newProduct.price" 
              @input="validateForm"
              type="number" 
              placeholder="ราคา > 0"
              step="0.01"
            />
            <span v-if="formErrors.price" class="error">{{ formErrors.price }}</span>
          </div>

          <div class="form-group">
            <label>สต็อก</label>
            <input 
              v-model.number="newProduct.stock" 
              @input="validateForm"
              type="number" 
              placeholder="สต็อก >= 0"
            />
            <span v-if="formErrors.stock" class="error">{{ formErrors.stock }}</span>
          </div>

          <div class="form-group">
            <label>หมวดหมู่</label>
            <select v-model="newProduct.category">
              <option value="อาหาร">อาหาร</option>
              <option value="เครื่องดื่ม">เครื่องดื่ม</option>
              <option value="ของใช้">ของใช้</option>
              <option value="เสื้อผ้า">เสื้อผ้า</option>
            </select>
          </div>

          <div class="form-group">
            <button 
              type="submit" 
              class="btn-submit"
              :disabled="!isFormValid"
            >
              เพิ่มสินค้า
            </button>
          </div>
        </div>
      </form>
    </div>
    
    <!-- Filter Section -->
    <div class="filter-section">
      <label for="category-filter">กรองตามหมวดหมู่:</label>
      <select id="category-filter" v-model="selectedCategory" class="category-select">
        <option value="ทั้งหมด">ทั้งหมด</option>
        <option value="อาหาร">อาหาร</option>
        <option value="เครื่องดื่ม">เครื่องดื่ม</option>
        <option value="ของใช้">ของใช้</option>
        <option value="เสื้อผ้า">เสื้อผ้า</option>
      </select>
    </div>
    
    <table class="product-table">
      <thead>
        <tr>
          <th>ลำดับ</th>
          <th>ชื่อสินค้า</th>
          <th>SKU</th>
          <th>ราคา (บาท)</th>
          <th>สต็อก</th>
          <th>หมวดหมู่</th>
          <th>วันที่สร้าง</th>
          <th>การขาย</th>
        </tr>
      </thead>
      <tbody>
        <tr 
          v-for="(product, index) in getFilteredProducts" 
          :key="product.id"
          :class="{ 'out-of-stock': product.stock === 0 }"
        >
          <td>{{ index + 1 }}</td>
          <td>{{ product.name }}</td>
          <td>{{ product.sku }}</td>
          <td>{{ product.price.toFixed(2) }}</td>
          <td :class="{ 'low-stock': product.stock > 0 && product.stock <= 5 }">
            {{ product.stock }}
          </td>
          <td>{{ product.category }}</td>
          <td>{{ new Date(product.createdAt).toLocaleDateString('th-TH') }}</td>
          <td>
            <button 
              @click="sellProduct(product.id)"
              :disabled="product.stock === 0"
              class="btn-sell"
            >
              ขาย
            </button>
          </td>
        </tr>
      </tbody>
    </table>

    <div class="summary-section">
      <h2>สรุปข้อมูล</h2>
      <div class="summary-content">
        <div class="summary-item">
          <span class="summary-label">จำนวนสินค้าทั้งหมด:</span>
          <span class="summary-value">{{ totalProducts }} รายการ</span>
        </div>
        <div class="summary-item">
          <span class="summary-label">มูลค่ารวม:</span>
          <span class="summary-value">{{ totalValue.toFixed(2) }} บาท</span>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.product-list-container {
  padding: 20px;
  max-width: 1200px;
  margin: 0 auto;
}

h1 {
  color: #2c3e50;
  margin-bottom: 20px;
  font-size: 28px;
}

.filter-section {
  margin-bottom: 20px;
  display: flex;
  align-items: center;
  gap: 12px;
}

.filter-section label {
  font-size: 16px;
  font-weight: 500;
  color: #2c3e50;
}

.category-select {
  padding: 8px 16px;
  font-size: 16px;
  border: 2px solid #27ae60;
  border-radius: 6px;
  background-color: white;
  color: #2c3e50;
  cursor: pointer;
  outline: none;
  transition: all 0.3s;
}

.category-select:hover {
  border-color: #229954;
  background-color: #f0f8f4;
}

.category-select:focus {
  border-color: #1e8449;
  box-shadow: 0 0 0 3px rgba(39, 174, 96, 0.1);
}

.product-table {
  width: 100%;
  border-collapse: collapse;
  border: 2px solid #34495e;
  background: white;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.product-table thead {
  background-color: #27ae60;
  color: white;
}

.product-table th {
  padding: 12px;
  text-align: left;
  font-weight: 600;
  border: 1px solid #2c3e50;
}

.product-table td {
  padding: 10px 12px;
  border: 1px solid #bdc3c7;
  color: #000;
}

.product-table tbody tr {
  transition: background-color 0.2s;
}

.product-table tbody tr:hover {
  background-color: #ecf0f1;
}

/* แถวที่สินค้าหมด - พื้นหลังสีแดงอ่อน */
.product-table tbody tr.out-of-stock {
  background-color: #ffebee;
}

.product-table tbody tr.out-of-stock:hover {
  background-color: #ffcdd2;
}

/* ราคา - จัดชิดขวา */
.price {
  text-align: right;
  font-family: 'Courier New', monospace;
}

/* สต็อก - จัดกึ่งกลาง */
.stock {
  text-align: center;
}

/* สต็อกใกล้หมด - สีแดงและตัวหนา */
.low-stock {
  color: #e74c3c;
  font-weight: bold;
}

/* ปุ่มขาย */
.btn-sell {
  padding: 6px 16px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
  background-color: #27ae60;
  color: white;
  font-weight: 500;
  transition: all 0.2s;
}

.btn-sell:hover:not(:disabled) {
  background-color: #229954;
}

.btn-sell:disabled {
  background-color: #95a5a6;
  cursor: not-allowed;
  opacity: 0.6;
}

/* Summary Section */
.summary-section {
  margin-top: 30px;
  padding: 20px;
  background-color: #ecf0f1;
  border-radius: 8px;
  border: 2px solid #27ae60;
}

.summary-section h2 {
  color: #2c3e50;
  margin-bottom: 15px;
  font-size: 20px;
}

.summary-content {
  display: flex;
  gap: 30px;
}

.summary-item {
  display: flex;
  flex-direction: column;
  gap: 5px;
}

.summary-label {
  font-size: 14px;
  color: #7f8c8d;
  font-weight: 500;
}

.summary-value {
  font-size: 24px;
  color: #27ae60;
  font-weight: bold;
}

/* Form Section */
.add-product-form {
  margin-bottom: 30px;
  padding: 20px;
  background-color: #f8f9fa;
  border-radius: 8px;
  border: 2px solid #3498db;
}

.add-product-form h2 {
  color: #000;
  margin-bottom: 15px;
  font-size: 20px;
}

.form-row {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 15px;
}

.form-group {
  display: flex;
  flex-direction: column;
  gap: 5px;
}

.form-group label {
  font-size: 14px;
  font-weight: 500;
  color: #000;
}

.form-group input,
.form-group select {
  padding: 8px 12px;
  border: 1px solid #bdc3c7;
  border-radius: 4px;
  font-size: 14px;
  color: #000;
  outline: none;
  transition: border-color 0.2s;
}

.form-group input:focus,
.form-group select:focus {
  border-color: #3498db;
  box-shadow: 0 0 0 2px rgba(52, 152, 219, 0.1);
}

.form-group .error {
  color: #e74c3c;
  font-size: 12px;
  font-weight: 500;
}

.btn-submit {
  padding: 10px 24px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
  font-weight: 600;
  background-color: #3498db;
  color: white;
  transition: all 0.2s;
  align-self: flex-end;
  margin-top: 20px;
}

.btn-submit:hover:not(:disabled) {
  background-color: #2980b9;
}

.btn-submit:disabled {
  background-color: #95a5a6;
  cursor: not-allowed;
  opacity: 0.6;
}
</style>
