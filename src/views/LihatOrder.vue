<template>
    <div class="lihat-order">
      <Navbar />
      <div class="container">
        <div class="row justify-content-center mt-5">
          <div class="col text-center">
            <h2><strong>Pesanan Anda</strong></h2>
            <!-- Periksa apakah ada data pesanan -->
            <ul v-if="orders.length > 0">
              <div v-for="(order, index) in orders" :key="index" class="mb-4">
                <p><strong>Nama:</strong> {{ order.nama }}</p>
                <p><strong>No Meja:</strong> {{ order.noMeja }}</p>
                <div v-for="(item, itemIndex) in order.keranjang" :key="itemIndex" class="keranjang-item">
                  <img :src="item.product.gambar" alt="Gambar Produk" class="product-image" />
                  <p><strong>Nama Makanan:</strong> {{ item.product.nama }}</p>
                  <p><strong>Jumlah:</strong> {{ parseInt(item.jumlah_pemesanan) || 'Tidak ada jumlah' }}</p>
                  <p><strong>Harga:</strong> {{ formatCurrency(item.product.harga) }}</p>
                  <p><strong>Keterangan:</strong> {{ item.keterangan }}</p>
                  <p><strong>Total Harga:</strong> 
                    {{
                      (item.product.harga && item.jumlah_pemesanan && !isNaN(parseInt(item.product.harga)) && !isNaN(parseInt(item.jumlah_pemesanan)))
                        ? formatCurrency(parseInt(item.product.harga) * parseInt(item.jumlah_pemesanan))
                        : 'Harga atau jumlah tidak valid'
                    }}
                  </p>
                </div>
              </div>
            </ul>
  
            <p v-else>Belum ada pesanan...</p>
          </div>
        </div>
      </div>
    </div>
  </template>
  
  <script>
  import { db } from "@/firebase"; // Pastikan file Firebase Anda sudah diatur
  import Navbar from "@/components/Navbar.vue";
  import { collection, getDocs, orderBy } from "firebase/firestore";
  
  export default {
    name: "LihatOrder",
    components: {
      Navbar,
    },
    data() {
      return {
        orders: [], // Menyimpan data pesanan
      };
    },
    async created() {
      await this.fetchOrders();
    },
    methods: {
      async fetchOrders() {
        try {
          // Mengambil data pesanan dan mengurutkan berdasarkan timestamp
          const querySnapshot = await getDocs(
            collection(db, "pesanan"),
            orderBy("timestamp", "desc") // Mengurutkan berdasarkan timestamp (terbaru)
          );
  
          // Map data pesanan dan ubah jumlah_pemesanan serta harga menjadi angka
          this.orders = querySnapshot.docs
            .map((doc) => {
              const orderData = doc.data();
              // Pastikan jumlah_pemesanan dan harga dalam bentuk angka
              orderData.keranjang.forEach(item => {
                item.jumlah_pemesanan = parseInt(item.jumlah_pemesanan); // Konversi jumlah_pemesanan ke angka
                item.product.harga = parseInt(item.product.harga); // Konversi harga ke angka
              });
              return {
                id: doc.id,
                ...orderData,
              };
            });
  
          // Tampilkan hanya 1 pesanan terbaru (pertama dalam urutan)
          this.orders = this.orders.slice(0, 1); // Ambil hanya pesanan terbaru
  
          console.log("Pesanan terbaru:", this.orders); // Debugging
        } catch (error) {
          console.error("Error fetching orders:", error);
        }
      },
      formatCurrency(value) {
        // Format harga ke dalam format rupiah
        return new Intl.NumberFormat("id-ID", {
          style: "currency",
          currency: "IDR",
        }).format(value);
      },
    },
  };
  </script>
  
  <style>
  .product-image {
    width: 150px;
    height: auto;
    margin-bottom: 10px;
    border-radius: 8px;
  }
  .keranjang-item {
    margin-bottom: 20px;
  }
  </style>
  