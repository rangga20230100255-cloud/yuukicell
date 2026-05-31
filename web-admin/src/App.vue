<script setup>
import { ref, onMounted, computed } from 'vue'
import { createClient } from '@supabase/supabase-js'

// Kunci dari Supabase kamu tetap dipertahankan
const supabaseUrl = 'https://shcgienrxphrfhjcpauh.supabase.co'
const supabaseKey = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InNoY2dpZW5yeHBocmZoamNwYXVoIiwicm9sZSI6ImFub24iLCJpYXQiOjE3ODE3OTMxOTcsImV4cCI6MjA5NzM2OTE5N30.i9XwEv1i3zjifrRRTcz1ONY2ABnXyWJstwgrAfyygiM'

const supabase = createClient(supabaseUrl, supabaseKey)
const produkList = ref([])

const inputTipe = ref('')
const inputNama = ref('')
const inputStok = ref('')
const inputBeli = ref('')
const inputJual = ref('')

// FUNGSI UTAMA
async function ambilData() {
  const { data } = await supabase.from('Produk').select('*').order('id', { ascending: true })
  if (data) produkList.value = data
}

async function tambahProduk() {
  if (!inputNama.value || !inputTipe.value) {
    alert('Tipe dan Nama Produk harus diisi!')
    return
  }
  const { error } = await supabase.from('Produk').insert([
    { 
      tipe_produk: inputTipe.value, 
      nama_produk: inputNama.value, 
      stok: Number(inputStok.value) || 0, 
      harga_beli: Number(inputBeli.value) || 0, 
      harga_jual: Number(inputJual.value) || 0 
    }
  ])
  if (!error) {
    inputTipe.value = ''
    inputNama.value = ''
    inputStok.value = ''
    inputBeli.value = ''
    inputJual.value = ''
    ambilData()
  }
}

async function hapusProduk(id) {
  const yakin = confirm('Apakah kamu yakin ingin menghapus produk ini?')
  if (yakin) {
    await supabase.from('Produk').delete().eq('id', id)
    ambilData()
  }
}

async function editStok(id, stokLama) {
  const stokBaru = prompt('Masukkan angka stok baru:', stokLama)
  if (stokBaru !== null && stokBaru !== '') {
    await supabase.from('Produk').update({ stok: Number(stokBaru) }).eq('id', id)
    ambilData()
  }
}

async function editHargaBeli(id, hargaLama) {
  const hargaBaru = prompt('Masukkan harga beli baru:', hargaLama)
  if (hargaBaru !== null && hargaBaru !== '') {
    await supabase.from('Produk').update({ harga_beli: Number(hargaBaru) }).eq('id', id)
    ambilData()
  }
}

async function editHargaJual(id, hargaLama) {
  const hargaBaru = prompt('Masukkan harga jual baru:', hargaLama)
  if (hargaBaru !== null && hargaBaru !== '') {
    await supabase.from('Produk').update({ harga_jual: Number(hargaBaru) }).eq('id', id)
    ambilData()
  }
}

// FORMAT RUPIAH AGAR LEBIH RAPI
function formatRupiah(angka) {
  return 'Rp ' + Number(angka).toLocaleString('id-ID')
}

// FITUR BARU: HITUNG TOTAL KESELURUHAN (BERDASARKAN STOK YANG ADA)
const totalHargaBeliSemua = computed(() => {
  return produkList.value.reduce((total, item) => total + (Number(item.harga_beli) * Number(item.stok || 0)), 0)
})

const totalHargaJualSemua = computed(() => {
  return produkList.value.reduce((total, item) => total + (Number(item.harga_jual) * Number(item.stok || 0)), 0)
})

const totalEstimasiKeuntungan = computed(() => {
  return totalHargaJualSemua.value - totalHargaBeliSemua.value
})

onMounted(() => {
  ambilData()
})
</script>

<template>
  <div style="padding: 20px; font-family: sans-serif; background-color: #f9fafb; min-height: 100vh;">
    <h1 style="color: #1f2937; margin-bottom: 5px;">Dashboard Admin - Yuuki Cell</h1>
    <p style="color: #6b7280; margin-top: 0; margin-bottom: 25px;">Manajemen Stok & Analisis Keuntungan</p>
    
    <div style="display: flex; gap: 20px; margin-bottom: 25px;">
      <div style="flex: 1; background-color: #fff; padding: 20px; border-radius: 8px; box-shadow: 0 1px 3px rgba(0,0,0,0.1); border-left: 5px solid #ef4444;">
        <div style="font-size: 14px; color: #6b7280; font-weight: bold;">TOTAL ASSET (HARGA BELI)</div>
        <div style="font-size: 24px; font-weight: bold; color: #1f2937; margin-top: 5px;">{{ formatRupiah(totalHargaBeliSemua) }}</div>
      </div>
      
      <div style="flex: 1; background-color: #fff; padding: 20px; border-radius: 8px; box-shadow: 0 1px 3px rgba(0,0,0,0.1); border-left: 5px solid #3b82f6;">
        <div style="font-size: 14px; color: #6b7280; font-weight: bold;">TOTAL NILAI JUAL</div>
        <div style="font-size: 24px; font-weight: bold; color: #1f2937; margin-top: 5px;">{{ formatRupiah(totalHargaJualSemua) }}</div>
      </div>
      
      <div style="flex: 1; background-color: #fff; padding: 20px; border-radius: 8px; box-shadow: 0 1px 3px rgba(0,0,0,0.1); border-left: 5px solid #10b981;">
        <div style="font-size: 14px; color: #6b7280; font-weight: bold;">ESTIMASI TOTAL UNTUNG</div>
        <div style="font-size: 24px; font-weight: bold; color: #10b981; margin-top: 5px;">{{ formatRupiah(totalEstimasiKeuntungan) }}</div>
      </div>
    </div>

    <div style="background-color: #fff; padding: 20px; margin-bottom: 25px; border-radius: 8px; box-shadow: 0 1px 3px rgba(0,0,0,0.1);">
      <h3 style="margin-top: 0; margin-bottom: 15px; color: #374151;">+ Tambah Produk Baru</h3>
      <div style="display: flex; flex-wrap: wrap; gap: 10px;">
        <input v-model="inputTipe" placeholder="Tipe (contoh: Aksesoris)" style="padding: 8px; border: 1px solid #d1d5db; border-radius: 4px; flex: 1; min-width: 120px;">
        <input v-model="inputNama" placeholder="Nama Produk" style="padding: 8px; border: 1px solid #d1d5db; border-radius: 4px; flex: 2; min-width: 180px;">
        <input v-model="inputStok" type="number" placeholder="Stok" style="padding: 8px; border: 1px solid #d1d5db; border-radius: 4px; width: 80px;">
        <input v-model="inputBeli" type="number" placeholder="Harga Beli" style="padding: 8px; border: 1px solid #d1d5db; border-radius: 4px; width: 110px;">
        <input v-model="inputJual" type="number" placeholder="Harga Jual" style="padding: 8px; border: 1px solid #d1d5db; border-radius: 4px; width: 110px;">
        <button @click="tambahProduk" style="padding: 8px 20px; background-color: #10b981; color: white; border: none; border-radius: 4px; cursor: pointer; font-weight: bold;">Simpan</button>
      </div>
    </div>

    <div style="background-color: #fff; border-radius: 8px; box-shadow: 0 1px 3px rgba(0,0,0,0.1); overflow: hidden;">
      <table style="border-collapse: collapse; width: 100%; text-align: left;">
        <thead style="background-color: #f3f4f6; color: #374151; font-weight: bold;">
          <tr>
            <th style="padding: 12px 15px;">Tipe</th>
            <th style="padding: 12px 15px;">Nama Produk</th>
            <th style="padding: 12px 15px; text-align: center;">Stok</th>
            <th style="padding: 12px 15px;">Harga Beli</th>
            <th style="padding: 12px 15px;">Harga Jual</th>
            <th style="padding: 12px 15px; color: #10b981; background-color: #f0fdf4;">Untung / Pcs</th>
            <th style="padding: 12px 15px; text-align: center;">Aksi</th>
          </tr>
        </thead>
        <tbody style="color: #4b5563;">
          <tr v-for="item in produkList" :key="item.id" style="border-bottom: 1px solid #e5e7eb;">
            <td style="padding: 12px 15px;">{{ item.tipe_produk }}</td>
            <td style="padding: 12px 15px; font-weight: 500; color: #1f2937;">{{ item.nama_produk }}</td>
            <td style="padding: 12px 15px; text-align: center;">{{ item.stok }}</td>
            <td style="padding: 12px 15px;">{{ formatRupiah(item.harga_beli) }}</td>
            <td style="padding: 12px 15px;">{{ formatRupiah(item.harga_jual) }}</td>
            
            <td style="padding: 12px 15px; font-weight: bold; color: #16a34a; background-color: #f0fdf4;">
              {{ formatRupiah(item.harga_jual - item.harga_beli) }}
            </td>
            
            <td style="padding: 12px 15px; text-align: center;">
              <button @click="editStok(item.id, item.stok)" style="margin-right: 5px; background-color: #3b82f6; color: white; border: none; padding: 6px 10px; border-radius: 4px; cursor: pointer; font-size: 12px;">Stok</button>
              <button @click="editHargaBeli(item.id, item.harga_beli)" style="margin-right: 5px; background-color: #f59e0b; color: white; border: none; padding: 6px 10px; border-radius: 4px; cursor: pointer; font-size: 12px;">H.Beli</button>
              <button @click="editHargaJual(item.id, item.harga_jual)" style="margin-right: 5px; background-color: #10b981; color: white; border: none; padding: 6px 10px; border-radius: 4px; cursor: pointer; font-size: 12px;">H.Jual</button>
              <button @click="hapusProduk(item.id)" style="background-color: #ef4444; color: white; border: none; padding: 6px 10px; border-radius: 4px; cursor: pointer; font-size: 12px;">Hapus</button>
            </td>
          </tr>
          <tr v-if="produkList.length === 0">
            <td colspan="7" style="padding: 20px; text-align: center; color: #9ca3af;">Belum ada data produk.</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>