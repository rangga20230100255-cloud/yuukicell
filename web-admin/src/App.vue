<script setup>
import { ref, onMounted, computed } from 'vue'
import { createClient } from '@supabase/supabase-js'

// Kunci dari Supabase kamu tetap dipertahankan
const supabaseUrl = 'https://shcgienrxphrfhjcpauh.supabase.co'
const supabaseKey = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InNoY2dpZW5yeHBocmZoamNwYXVoIiwicm9sZSI6ImFub24iLCJpYXQiOjE3ODE3OTMxOTcsImV4cCI6MjA5NzM2OTE5N30.i9XwEv1i3zjifrRRTcz1ONY2ABnXyWJstwgrAfyygiM'
const supabase = createClient(supabaseUrl, supabaseKey)

// STATE TAB NAVIGASI
const tabAktif = ref('fisik') // Bisa 'fisik' atau 'digital'

// ==========================================
// KODE UNTUK BARANG FISIK
// ==========================================
const produkList = ref([])
const inputTipe = ref(''); const inputNama = ref(''); const inputStokGudang = ref(''); const inputBeli = ref(''); const inputJual = ref('')

async function ambilDataFisik() {
  const { data } = await supabase.from('Produk').select('*').order('id', { ascending: true })
  if (data) produkList.value = data
}

async function tambahProduk() {
  if (!inputNama.value || !inputTipe.value) return alert('Isi data dengan lengkap!')
  await supabase.from('Produk').insert([{ tipe_produk: inputTipe.value, nama_produk: inputNama.value, stok: inputStokGudang.value, stok_per_kasir: {}, harga_beli: inputBeli.value, harga_jual: inputJual.value }])
  inputTipe.value = ''; inputNama.value = ''; inputStokGudang.value = ''; inputBeli.value = ''; inputJual.value = ''
  ambilDataFisik()
}

async function hapusProduk(id) {
  if (confirm('Yakin hapus?')) { await supabase.from('Produk').delete().eq('id', id); ambilDataFisik() }
}

async function transferFisik(id, stokGudangSekarang, stokKasirObj) {
  const namaKasir = prompt('Transfer ke Kasir/Kode Unik mana? (Cth: Kasir-01)')
  if (!namaKasir) return
  const jumlah = parseInt(prompt(`Stok Gudang: ${stokGudangSekarang}\nMau transfer berapa ke ${namaKasir}?`))
  if (!jumlah || isNaN(jumlah) || jumlah <= 0 || jumlah > stokGudangSekarang) return alert('Stok tidak cukup / tidak valid!')
  
  const stokSaatIni = stokKasirObj || {}
  const stokBaruObj = { ...stokSaatIni, [namaKasir]: (stokSaatIni[namaKasir] || 0) + jumlah }
  await supabase.from('Produk').update({ stok: stokGudangSekarang - jumlah, stok_per_kasir: stokBaruObj }).eq('id', id)
  ambilDataFisik()
}

async function tarikFisik(id, stokGudangSekarang, stokKasirObj) {
  const stokSaatIni = stokKasirObj || {}
  if (Object.keys(stokSaatIni).length === 0) return alert('Belum ada stok di kasir manapun!')
  const namaKasir = prompt(`Tarik dari Kasir/Kode Unik mana?\nPilihan: ${Object.keys(stokSaatIni).join(', ')}`)
  if (!namaKasir || !stokSaatIni[namaKasir]) return alert('Tidak ditemukan!')
  
  const jumlah = parseInt(prompt(`Stok ${namaKasir}: ${stokSaatIni[namaKasir]}\nMau ditarik berapa?`))
  if (!jumlah || isNaN(jumlah) || jumlah <= 0 || jumlah > stokSaatIni[namaKasir]) return alert('Tidak valid!')
  
  const stokBaruObj = { ...stokSaatIni, [namaKasir]: stokSaatIni[namaKasir] - jumlah }
  if (stokBaruObj[namaKasir] === 0) delete stokBaruObj[namaKasir]
  await supabase.from('Produk').update({ stok: stokGudangSekarang + jumlah, stok_per_kasir: stokBaruObj }).eq('id', id)
  ambilDataFisik()
}

async function editStokGudang(id, stokLama) {
  const stokBaru = prompt('Ubah/Tambah Stok Gudang menjadi:', stokLama)
  if (stokBaru !== null && stokBaru !== '') {
    await supabase.from('Produk').update({ stok: parseInt(stokBaru) }).eq('id', id)
    ambilDataFisik()
  }
}

async function editHargaBeli(id, hargaLama) {
  const hargaBaru = prompt('Masukkan Harga Beli baru:', hargaLama)
  if (hargaBaru !== null && hargaBaru !== '') {
    await supabase.from('Produk').update({ harga_beli: parseInt(hargaBaru) }).eq('id', id)
    ambilDataFisik()
  }
}

async function editHargaJual(id, hargaLama) {
  const hargaBaru = prompt('Masukkan Harga Jual baru:', hargaLama)
  if (hargaBaru !== null && hargaBaru !== '') {
    await supabase.from('Produk').update({ harga_jual: parseInt(hargaBaru) }).eq('id', id)
    ambilDataFisik()
  }
}

function hitungTotalStokKasir(stokKasirObj) {
  return stokKasirObj ? Object.values(stokKasirObj).reduce((a, b) => a + b, 0) : 0
}
const totalModal = computed(() => produkList.value.reduce((tot, item) => tot + ((item.stok + hitungTotalStokKasir(item.stok_per_kasir)) * item.harga_beli), 0))
const totalPotensi = computed(() => produkList.value.reduce((tot, item) => tot + ((item.stok + hitungTotalStokKasir(item.stok_per_kasir)) * item.harga_jual), 0))
const totalKeuntungan = computed(() => totalPotensi.value - totalModal.value)

// ==========================================
// KODE UNTUK SALDO DIGITAL
// ==========================================
const saldoList = ref([])
const inputNamaSaldo = ref('')
const inputSaldoAwal = ref('')

async function ambilDataDigital() {
  const { data } = await supabase.from('SaldoDigital').select('*').order('id', { ascending: true })
  if (data) saldoList.value = data
}

async function tambahSaldoDigital() {
  if (!inputNamaSaldo.value) return alert('Isi Nama Saldo!')
  await supabase.from('SaldoDigital').insert([{ nama_saldo: inputNamaSaldo.value, saldo_gudang: inputSaldoAwal.value, saldo_per_kasir: {}, saldo_kembalian: 0 }])
  inputNamaSaldo.value = ''; inputSaldoAwal.value = ''
  ambilDataDigital()
}

async function hapusSaldo(id) {
  if (confirm('Yakin hapus saldo ini?')) { await supabase.from('SaldoDigital').delete().eq('id', id); ambilDataDigital() }
}

// === FUNGSI TRANSFER SALDO TANPA UPLOAD FOTO ===
async function transferDigitalRingkas(item) {
  const namaKasir = prompt('Transfer ke Kasir/Kode Unik mana?\n(Cth: Kasir-01)')
  if (!namaKasir) return
  
  const nominal = parseInt(prompt(`Sisa Saldo ${item.nama_saldo}: Rp ${item.saldo_gudang}\nMau transfer berapa ke ${namaKasir}?`))
  if (!nominal || isNaN(nominal) || nominal <= 0 || nominal > item.saldo_gudang) return alert('Nominal tidak valid atau saldo kurang!')

  // Catat riwayat tanpa foto (Keterangan: via WA)
  await supabase.from('TransaksiSaldo').insert([{
    nama_kasir: namaKasir,
    jenis_transaksi: `Isi Saldo ${item.nama_saldo}`,
    nominal: nominal,
    bukti_foto: 'Dikirim via WA' // Otomatis tulis keterangan ini
  }])

  // Update jumlah saldo di gudang dan kasir
  const stokSaatIni = item.saldo_per_kasir || {}
  const stokBaruObj = { ...stokSaatIni, [namaKasir]: (stokSaatIni[namaKasir] || 0) + nominal }
  await supabase.from('SaldoDigital').update({ saldo_gudang: item.saldo_gudang - nominal, saldo_per_kasir: stokBaruObj }).eq('id', item.id)

  alert(`Sukses! Saldo terkirim ke ${namaKasir}. Jangan lupa kirim bukti *screenshot*-nya lewat WA ya!`)
  ambilDataDigital()
}

async function tarikSaldoDigital(item) {
  const stokSaatIni = item.saldo_per_kasir || {}
  if (Object.keys(stokSaatIni).length === 0) return alert('Belum ada saldo di kasir manapun!')
  const namaKasir = prompt(`Tarik saldo dari kasir mana?\nKasir aktif: ${Object.keys(stokSaatIni).join(', ')}`)
  if (!namaKasir || !stokSaatIni[namaKasir]) return alert('Tidak valid!')
  
  const nominal = parseInt(prompt(`Saldo ${namaKasir}: Rp ${stokSaatIni[namaKasir]}\nMau ditarik berapa ke Saldo Kembalian?`))
  if (!nominal || isNaN(nominal) || nominal <= 0 || nominal > stokSaatIni[namaKasir]) return alert('Gagal ditarik!')
  
  const stokBaruObj = { ...stokSaatIni, [namaKasir]: stokSaatIni[namaKasir] - nominal }
  if (stokBaruObj[namaKasir] === 0) delete stokBaruObj[namaKasir]
  
  await supabase.from('SaldoDigital').update({ 
    saldo_kembalian: item.saldo_kembalian + nominal, 
    saldo_per_kasir: stokBaruObj 
  }).eq('id', item.id)

  await supabase.from('TransaksiSaldo').insert([{
    nama_kasir: namaKasir,
    jenis_transaksi: `Penarikan/Kembalian ${item.nama_saldo}`,
    nominal: nominal,
    bukti_foto: '-' 
  }])

  alert('Saldo berhasil ditarik menjadi Saldo Kembalian!')
  ambilDataDigital()
}

onMounted(() => {
  ambilDataFisik()
  ambilDataDigital()
})
</script>

<template>
  <div style="padding: 20px; font-family: sans-serif; background-color: #f9fafb; min-height: 100vh;">
    <h1 style="color: #1f2937;">Dashboard Admin (Gudang Utama)</h1>

    <div style="margin-bottom: 20px; display: flex; gap: 10px;">
      <button @click="tabAktif = 'fisik'" :style="{ backgroundColor: tabAktif === 'fisik' ? '#1f2937' : '#e5e7eb', color: tabAktif === 'fisik' ? 'white' : 'black', padding: '10px 20px', border: 'none', borderRadius: '8px', cursor: 'pointer', fontWeight: 'bold' }">
        📦 Stok Barang Fisik
      </button>
      <button @click="tabAktif = 'digital'" :style="{ backgroundColor: tabAktif === 'digital' ? '#3b82f6' : '#e5e7eb', color: tabAktif === 'digital' ? 'white' : 'black', padding: '10px 20px', border: 'none', borderRadius: '8px', cursor: 'pointer', fontWeight: 'bold' }">
        📱 Deposit & Saldo Digital
      </button>
    </div>

    <!-- ============================================== -->
    <!-- HALAMAN BARANG FISIK -->
    <!-- ============================================== -->
    <div v-if="tabAktif === 'fisik'">
      <div style="display: flex; gap: 20px; margin-bottom: 20px;">
        <div style="flex: 1; background-color: #fff; padding: 20px; border-radius: 8px; border-left: 5px solid #ef4444; box-shadow: 0 2px 4px rgba(0,0,0,0.1);">
          <p style="margin: 0; color: #6b7280; font-size: 14px;">Total Modal Barang Fisik</p>
          <h2 style="margin: 5px 0 0 0;">Rp {{ totalModal }}</h2>
        </div>
        <div style="flex: 1; background-color: #fff; padding: 20px; border-radius: 8px; border-left: 5px solid #3b82f6; box-shadow: 0 2px 4px rgba(0,0,0,0.1);">
          <p style="margin: 0; color: #6b7280; font-size: 14px;">Potensi Penjualan</p>
          <h2 style="margin: 5px 0 0 0;">Rp {{ totalPotensi }}</h2>
        </div>
        <div style="flex: 1; background-color: #fff; padding: 20px; border-radius: 8px; border-left: 5px solid #10b981; box-shadow: 0 2px 4px rgba(0,0,0,0.1);">
          <p style="margin: 0; color: #6b7280; font-size: 14px;">Potensi Keuntungan Bersih</p>
          <h2 style="margin: 5px 0 0 0; color: #10b981;">Rp {{ totalKeuntungan }}</h2>
        </div>
      </div>

      <div style="background-color: #fff; padding: 20px; margin-bottom: 20px; border-radius: 8px; box-shadow: 0 2px 4px rgba(0,0,0,0.1);">
        <h3 style="margin-top: 0;">+ Tambah Produk Fisik</h3>
        <input v-model="inputTipe" placeholder="Tipe" style="margin-right: 10px; padding: 8px; border-radius: 4px; border: 1px solid #d1d5db;">
        <input v-model="inputNama" placeholder="Nama Produk" style="margin-right: 10px; padding: 8px; border-radius: 4px; border: 1px solid #d1d5db;">
        <input v-model="inputStokGudang" type="number" placeholder="Stok Awal" style="margin-right: 10px; padding: 8px; width: 100px; border-radius: 4px; border: 1px solid #d1d5db;">
        <input v-model="inputBeli" type="number" placeholder="H. Beli" style="margin-right: 10px; padding: 8px; width: 100px; border-radius: 4px; border: 1px solid #d1d5db;">
        <input v-model="inputJual" type="number" placeholder="H. Jual" style="margin-right: 10px; padding: 8px; width: 100px; border-radius: 4px; border: 1px solid #d1d5db;">
        <button @click="tambahProduk" style="padding: 8px 15px; background-color: #10b981; color: white; border: none; border-radius: 4px; cursor: pointer; font-weight: bold;">Simpan Fisik</button>
      </div>

      <div style="background-color: #fff; border-radius: 8px; overflow: hidden; box-shadow: 0 2px 4px rgba(0,0,0,0.1);">
        <table border="0" cellpadding="12" style="border-collapse: collapse; width: 100%; text-align: left;">
          <thead style="background-color: #f3f4f6; border-bottom: 2px solid #e5e7eb;">
            <tr>
              <th>Nama Produk</th>
              <th>Stok Gudang</th>
              <th style="color: #3b82f6;">Stok Kasir (List)</th>
              <th>Harga Beli</th>
              <th>Harga Jual</th>
              <th style="color: #10b981;">Untung/Unit</th>
              <th>Aksi Transfer & Edit</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="item in produkList" :key="item.id" style="border-bottom: 1px solid #e5e7eb;">
              <td>{{ item.tipe_produk }} - <strong>{{ item.nama_produk }}</strong></td>
              <td><span style="background-color: #fde68a; padding: 4px 8px; border-radius: 4px; font-weight: bold;">{{ item.stok }} pcs</span></td>
              <td style="color: #3b82f6;">
                <div v-if="!item.stok_per_kasir || Object.keys(item.stok_per_kasir).length === 0">
                  <span style="background-color: #e5e7eb; padding: 4px 8px; border-radius: 4px; font-size: 12px; color: #6b7280;">Kosong</span>
                </div>
                <div v-else style="display: flex; flex-direction: column; gap: 4px;">
                  <span v-for="(jumlah, nama) in item.stok_per_kasir" :key="nama" style="background-color: #bfdbfe; padding: 4px 8px; border-radius: 4px; font-weight: bold; font-size: 12px; display: inline-block;">
                    {{ nama }} : {{ jumlah }} pcs
                  </span>
                </div>
              </td>
              <td>Rp {{ item.harga_beli }}</td>
              <td>Rp {{ item.harga_jual }}</td>
              <td style="color: #10b981; font-weight: bold;">Rp {{ item.harga_jual - item.harga_beli }}</td>
              <td>
                <div style="display: flex; gap: 5px; flex-wrap: wrap;">
                  <button @click="transferFisik(item.id, item.stok, item.stok_per_kasir)" style="background-color: #8b5cf6; color: white; border: none; padding: 6px 10px; border-radius: 4px; cursor: pointer; font-size: 12px;">Kirim ➜</button>
                  <button @click="tarikFisik(item.id, item.stok, item.stok_per_kasir)" style="background-color: #475569; color: white; border: none; padding: 6px 10px; border-radius: 4px; cursor: pointer; font-size: 12px;">Tarik ↩</button>
                  <button @click="editStokGudang(item.id, item.stok)" style="background-color: #eab308; color: white; border: none; padding: 6px 10px; border-radius: 4px; cursor: pointer; font-size: 12px;">Edit Stok</button>
                  <button @click="editHargaBeli(item.id, item.harga_beli)" style="background-color: #f97316; color: white; border: none; padding: 6px 10px; border-radius: 4px; cursor: pointer; font-size: 12px;">H.Beli</button>
                  <button @click="editHargaJual(item.id, item.harga_jual)" style="background-color: #10b981; color: white; border: none; padding: 6px 10px; border-radius: 4px; cursor: pointer; font-size: 12px;">H.Jual</button>
                  <button @click="hapusProduk(item.id)" style="background-color: #ef4444; color: white; border: none; padding: 6px 10px; border-radius: 4px; cursor: pointer; font-size: 12px;">Hapus</button>
                </div>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <!-- ============================================== -->
    <!-- HALAMAN SALDO DIGITAL -->
    <!-- ============================================== -->
    <div v-if="tabAktif === 'digital'">
      <div style="background-color: #eff6ff; padding: 20px; margin-bottom: 20px; border-radius: 8px; box-shadow: 0 2px 4px rgba(0,0,0,0.1); border: 1px solid #bfdbfe;">
        <h3 style="margin-top: 0; color: #1e3a8a;">+ Tambah Dompet / Saldo Digital Baru</h3>
        <input v-model="inputNamaSaldo" placeholder="Cth: Saldo DANA, Saldo Telkomsel" style="margin-right: 10px; padding: 8px; width: 250px; border-radius: 4px; border: 1px solid #d1d5db;">
        <input v-model="inputSaldoAwal" type="number" placeholder="Modal Saldo Awal (Rp)" style="margin-right: 10px; padding: 8px; width: 200px; border-radius: 4px; border: 1px solid #d1d5db;">
        <button @click="tambahSaldoDigital" style="padding: 8px 15px; background-color: #3b82f6; color: white; border: none; border-radius: 4px; cursor: pointer; font-weight: bold;">Simpan Dompet</button>
      </div>

      <div style="background-color: #fff; border-radius: 8px; overflow: hidden; box-shadow: 0 2px 4px rgba(0,0,0,0.1);">
        <table border="0" cellpadding="12" style="border-collapse: collapse; width: 100%; text-align: left;">
          <thead style="background-color: #f3f4f6; border-bottom: 2px solid #e5e7eb;">
            <tr>
              <th>Nama Dompet</th>
              <th>Saldo Gudang (Sisa)</th>
              <th style="color: #3b82f6;">Saldo di Kasir (List)</th>
              <th style="color: #10b981;">Saldo Kembalian</th>
              <th>Aksi Transfer</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="item in saldoList" :key="item.id" style="border-bottom: 1px solid #e5e7eb;">
              <td><strong>{{ item.nama_saldo }}</strong></td>
              <td><span style="background-color: #fde68a; padding: 6px 10px; border-radius: 4px; font-weight: bold; font-size: 14px;">Rp {{ item.saldo_gudang }}</span></td>
              <td style="color: #3b82f6;">
                <div v-if="!item.saldo_per_kasir || Object.keys(item.saldo_per_kasir).length === 0">
                  <span style="background-color: #e5e7eb; padding: 4px 8px; border-radius: 4px; font-size: 12px; color: #6b7280;">Kosong</span>
                </div>
                <div v-else style="display: flex; flex-direction: column; gap: 4px;">
                  <span v-for="(jumlah, nama) in item.saldo_per_kasir" :key="nama" style="background-color: #bfdbfe; padding: 4px 8px; border-radius: 4px; font-weight: bold; font-size: 12px; display: inline-block;">
                    {{ nama }} : Rp {{ jumlah }}
                  </span>
                </div>
              </td>
              <td style="color: #10b981;"><strong>Rp {{ item.saldo_kembalian || 0 }}</strong></td>
              <td>
                <button @click="transferDigitalRingkas(item)" style="background-color: #2563eb; color: white; border: none; padding: 6px 10px; border-radius: 4px; cursor: pointer; margin-right: 5px; font-size: 12px;">Transfer ➜</button>
                <button @click="tarikSaldoDigital(item)" style="background-color: #10b981; color: white; border: none; padding: 6px 10px; border-radius: 4px; cursor: pointer; margin-right: 5px; font-size: 12px;">Tarik Kembalian ↩</button>
                <button @click="hapusSaldo(item.id)" style="background-color: #ef4444; color: white; border: none; padding: 6px 10px; border-radius: 4px; cursor: pointer; font-size: 12px;">Hapus</button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>