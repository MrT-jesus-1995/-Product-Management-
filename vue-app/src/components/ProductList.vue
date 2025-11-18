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
  const name = newProduct.value.name.trim()
  const sku = newProduct.value.sku.trim()
  const price = newProduct.value.price
  const stock = newProduct.value.stock
  
  return name.length >= 3 &&
         sku.length > 0 &&
         !products.value.some(p => p.sku.toLowerCase() === sku.toLowerCase()) &&
         price > 0 &&
         !isNaN(price) &&
         stock >= 0 &&
         !isNaN(stock)
})

const validateForm = () => {
  const name = newProduct.value.name.trim()
  const sku = newProduct.value.sku.trim()
  const price = newProduct.value.price
  const stock = newProduct.value.stock
  
  // Validate name
  if (!name) {
    formErrors.value.name = 'กรุณากรอกชื่อสินค้า'
  } else if (name.length < 3) {
    formErrors.value.name = 'ชื่อสินค้าต้องมีอย่างน้อย 3 ตัวอักษร'
  } else if (name.length > 100) {
    formErrors.value.name = 'ชื่อสินค้าต้องไม่เกิน 100 ตัวอักษร'
  } else {
    formErrors.value.name = ''
  }
  
  // Validate SKU
  if (!sku) {
    formErrors.value.sku = 'กรุณากรอก SKU'
  } else if (!/^[A-Z0-9-]+$/i.test(sku)) {
    formErrors.value.sku = 'SKU ต้องเป็นตัวอักษรและตัวเลขเท่านั้น'
  } else if (products.value.some(p => p.sku.toLowerCase() === sku.toLowerCase())) {
    formErrors.value.sku = 'SKU นี้มีอยู่ในระบบแล้ว'
  } else {
    formErrors.value.sku = ''
  }
  
  // Validate price
  if (!price && price !== 0) {
    formErrors.value.price = 'กรุณากรอกราคา'
  } else if (isNaN(price)) {
    formErrors.value.price = 'กรุณากรอกตัวเลขเท่านั้น'
  } else if (price <= 0) {
    formErrors.value.price = 'ราคาต้องมากกว่า 0'
  } else if (price > 1000000) {
    formErrors.value.price = 'ราคาต้องไม่เกิน 1,000,000 บาท'
  } else {
    formErrors.value.price = ''
  }
  
  // Validate stock
  if (!stock && stock !== 0) {
    formErrors.value.stock = 'กรุณากรอกจำนวนสต็อก'
  } else if (isNaN(stock)) {
    formErrors.value.stock = 'กรุณากรอกตัวเลขเท่านั้น'
  } else if (stock < 0) {
    formErrors.value.stock = 'สต็อกต้องมากกว่าหรือเท่ากับ 0'
  } else if (stock > 100000) {
    formErrors.value.stock = 'สต็อกต้องไม่เกิน 100,000'
  } else {
    formErrors.value.stock = ''
  }
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
    name: newProduct.value.name.trim(),
    sku: newProduct.value.sku.trim().toUpperCase(),
    price: Number(newProduct.value.price),
    stock: Number(newProduct.value.stock),
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
              @blur="validateForm"
              type="text" 
              placeholder="ชื่อสินค้า (อย่างน้อย 3 ตัวอักษร)"
              :class="{ invalid: formErrors.name }"
            />
            <span v-if="formErrors.name" class="error">{{ formErrors.name }}</span>
          </div>

          <div class="form-group">
            <label>SKU</label>
            <input 
              v-model="newProduct.sku" 
              @input="validateForm"
              @blur="validateForm"
              type="text" 
              placeholder="รหัสสินค้า (ไม่ซ้ำ)"
              :class="{ invalid: formErrors.sku }"
            />
            <span v-if="formErrors.sku" class="error">{{ formErrors.sku }}</span>
          </div>

          <div class="form-group">
            <label>ราคา (บาท)</label>
            <input 
              v-model.number="newProduct.price" 
              @input="validateForm"
              @blur="validateForm"
              type="number" 
              placeholder="ราคา > 0"
              step="0.01"
              :class="{ invalid: formErrors.price }"
            />
            <span v-if="formErrors.price" class="error">{{ formErrors.price }}</span>
          </div>

          <div class="form-group">
            <label>สต็อก</label>
            <input 
              v-model.number="newProduct.stock" 
              @input="validateForm"
              @blur="validateForm"
              type="number" 
              placeholder="สต็อก >= 0"
              :class="{ invalid: formErrors.stock }"
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
          <td :class="{ 'low-stock': product.stock > 0 && product.stock < 10 }">
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
  padding: 0;
  max-width: 100%;
  width: 100vw;
  height: 100vh;
  margin: 0;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  box-sizing: border-box;
}

h1 {
  color: #ffffff;
  margin-bottom: 15px;
  padding: 20px 20px 0 20px;
  font-size: 24px;
  font-weight: 700;
  text-align: center;
  letter-spacing: -0.5px;
  flex-shrink: 0;
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

/* Form Section */
.add-product-form {
  margin: 0 20px 15px 20px;
  padding: 15px;
  background: rgba(255, 255, 255, 0.95);
  border-radius: 10px;
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.15);
  flex-shrink: 0;
  border: 1px solid rgba(255, 255, 255, 0.3);
}

.add-product-form h2 {
  color: #667eea;
  margin-bottom: 12px;
  font-size: 16px;
  font-weight: 600;
}

.form-row {
  display: grid;
  grid-template-columns: repeat(6, 1fr);
  gap: 10px;
  background-color: transparent;
  padding: 12px;
  border-radius: 8px;
}

.form-group {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.form-group label {
  font-size: 13px;
  font-weight: 600;
  color: #2d3748;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.form-group input,
.form-group select {
  padding: 6px 10px;
  border: 2px solid #e2e8f0;
  border-radius: 6px;
  font-size: 14px;
  color: #2d3748;
  outline: none;
  transition: all 0.3s ease;
  background-color: #fff;
}

.form-group input.invalid,
.form-group select.invalid {
  border-color: #e53e3e;
  background-color: #fff5f5;
}

.form-group input:focus,
.form-group select:focus {
  border-color: #667eea;
  box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
  transform: translateY(-1px);
}

.form-group input.invalid:focus,
.form-group select.invalid:focus {
  border-color: #e53e3e;
  box-shadow: 0 0 0 3px rgba(229, 62, 62, 0.1);
}

.form-group .error {
  color: #e53e3e;
  font-size: 12px;
  font-weight: 600;
  margin-top: 2px;
  background-color: #fff5f5;
  padding: 4px 8px;
  border-radius: 4px;
  border-left: 3px solid #e53e3e;
}

.btn-submit {
  padding: 8px 20px;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 13px;
  font-weight: 700;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  transition: all 0.3s ease;
  align-self: flex-end;
  margin-top: 5px;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  box-shadow: 0 4px 12px rgba(102, 126, 234, 0.3);
}

.btn-submit:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(102, 126, 234, 0.4);
}

.btn-submit:disabled {
  background: #cbd5e0;
  cursor: not-allowed;
  opacity: 0.6;
  box-shadow: none;
}

/* Filter Section */
.filter-section {
  margin: 0 20px 12px 20px;
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 10px 15px;
  background-color: rgba(255, 255, 255, 0.9);
  border-radius: 8px;
  border: 1px solid rgba(255, 255, 255, 0.3);
  flex-shrink: 0;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.filter-section label {
  font-size: 15px;
  font-weight: 600;
  color: #667eea;
}

.category-select {
  padding: 10px 18px;
  font-size: 15px;
  border: 2px solid #48bb78;
  border-radius: 8px;
  background-color: white;
  color: #2d3748;
  cursor: pointer;
  outline: none;
  transition: all 0.3s ease;
  font-weight: 500;
}

.category-select:hover {
  border-color: #38a169;
  background-color: #f0fff4;
  transform: translateY(-1px);
}

.category-select:focus {
  border-color: #2f855a;
  box-shadow: 0 0 0 3px rgba(72, 187, 120, 0.15);
}

/* Table */
.product-table {
  width: calc(100% - 40px);
  margin: 0 20px;
  border-collapse: collapse;
  border-radius: 10px;
  overflow: hidden;
  background: rgba(255, 255, 255, 0.95);
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.15);
  display: block;
  max-height: 35vh;
  overflow-y: auto;
  border: 1px solid rgba(255, 255, 255, 0.3);
  flex: 1;
  min-height: 0;
}

.product-table thead,
.product-table tbody {
  display: table;
  width: 100%;
  table-layout: fixed;
}

.product-table thead {
  background: linear-gradient(135deg, #48bb78 0%, #38a169 100%);
  color: white;
}

.product-table th {
  padding: 10px 8px;
  text-align: left;
  font-weight: 600;
  font-size: 12px;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  border: none;
}

.product-table td {
  padding: 8px;
  border-bottom: 1px solid #e2e8f0;
  color: #2d3748;
  font-size: 13px;
}

.product-table tbody tr {
  transition: all 0.2s ease;
}

.product-table tbody tr:hover {
  background-color: #f7fafc;
  transform: scale(1.01);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
}

.product-table tbody tr:last-child td {
  border-bottom: none;
}

/* แถวที่สินค้าหมด - พื้นหลังสีแดงอ่อน */
.product-table tbody tr.out-of-stock {
  background-color: #fff5f5;
}

.product-table tbody tr.out-of-stock:hover {
  background-color: #fed7d7;
}

/* สต็อกใกล้หมด - สีแดงและตัวหนา */
.low-stock {
  color: #e53e3e !important;
  font-weight: 700;
  background-color: #fff5f5;
  padding: 4px 8px !important;
  border-radius: 4px;
}

/* ปุ่มขาย */
.btn-sell {
  padding: 8px 20px;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 14px;
  background: linear-gradient(135deg, #48bb78 0%, #38a169 100%);
  color: white;
  font-weight: 600;
  transition: all 0.3s ease;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  box-shadow: 0 2px 8px rgba(72, 187, 120, 0.3);
}

.btn-sell:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(72, 187, 120, 0.4);
}

.btn-sell:disabled {
  background: linear-gradient(135deg, #a0aec0 0%, #718096 100%);
  cursor: not-allowed;
  opacity: 0.5;
  box-shadow: none;
  transform: none;
}

/* Summary Section */
.summary-section {
  margin: 12px 20px 20px 20px;
  padding: 15px;
  background: rgba(255, 255, 255, 0.9);
  border-radius: 10px;
  border: 2px solid rgba(255, 255, 255, 0.5);
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.12);
  flex-shrink: 0;
}

.summary-section h2 {
  color: #667eea;
  margin-bottom: 12px;
  font-size: 16px;
  font-weight: 700;
}

.summary-content {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 20px;
}

.summary-item {
  display: flex;
  flex-direction: column;
  gap: 5px;
  padding: 12px 15px;
  background-color: white;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
  border-left: 4px solid #48bb78;
}

.summary-label {
  font-size: 11px;
  color: #718096;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.summary-value {
  font-size: 20px;
  color: #48bb78;
  font-weight: 700;
}
</style>
