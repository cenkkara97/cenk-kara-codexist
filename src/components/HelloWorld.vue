<template>
  <div class="hello">
    <div class="bookstore-banner-section">
      <div class="wrapper">
        <div class="flex-area">
          <div class="text-block">
            <h1>CODEXIST Kitapcım</h1>
            <p>Kitaplarla dolu dünyamızda, her sayfa bir serüven. Keşfetmeye hazır mısınız?</p>
          </div>
        </div>
      </div>
    </div>
    <div class="main-container">
      <div class="search-bar">
        <div class="search-wrapper">
          <input v-model="searchQuery" type="text" placeholder="Kitap Ara..." />
          <label>Kitap Ara:</label>
        </div>
      </div>
      <div class="books">
        <div v-for="book in filteredBooks" :key="book.isbn13" class="book">
          <img :src="book.image" :alt="book.title">
          <h3>{{ book.title }}</h3>
          <p>{{ book.subtitle }}</p>
          <p>Fiyat: <span>{{ book.price }}</span></p>
          <button class="add-btn" @click="addToCart(book)">Sepete Ekle</button>
        </div>
      </div>
      <div class="cart">
        <div class="headling">
          <h2>Sepetim</h2>
          <i class="fa fa-cart-arrow-down s-basket" aria-hidden="true"></i>
        </div>
        <ul>
          <li v-for="item in cart" :key="item.isbn13">
            <div class="basket-info"> 
              <img :src="item.image" :alt="item.title">
              <p>{{ item.title }} - ${{ item.price }} x {{ item.quantity }}</p>
            </div>
            <div class="basket-buttons">
              <span @click="addOneFromCart(item)"><i class="fa fa-plus-circle s-plus" aria-hidden="true"></i></span>
              <span @click="removeOneFromCart(item)"><i class="fa fa-minus-circle s-minus" aria-hidden="true"></i></span>
              <span @click="removeFromCart(item.isbn13)"><i class="fa fa-trash s-trash" aria-hidden="true"></i></span>
            </div>
          </li>
        </ul>
        <p class="total-price">Toplam Fiyat: ${{ getTotalPrice().toFixed(2) }}</p>
        <div class="flex-modal-area">
        <button class="modal-btn" @click="showModal">Sepeti Onayla</button>
        <div v-if="formSubmittedSuccessfully" class="success">
          <div>Üyeliğiniz ve Siparişiniz Başarılı bir şekilde oluşturulmuştur.</div>
        </div>
        <div v-if="formSubmissionFailed" class="error">
          <div>Lütfen tüm alanları doldurun.</div>
        </div>
        <div v-if="isModalVisible" class="modal">
          <div class="modal-content">
            <div class="modal-header">
              <span class="close" @click="closeModal">&times;</span>
            </div>
            <div class="modal-title-info">
              <h1>Codexist Hesabınız Yok Mu?</h1>
              <p>Satın alma işlemine devam edebilmeniz için lütfen üyelik oluşturun.</p>
            </div>
            <div class="modal-info">
              <form @submit.prevent="submitForm">
                <label for="firstName">İsim:</label>
                <input type="text" id="firstName" v-model="formData.firstName" @input="validateInput" required />
                <label for="lastName">Soyisim:</label>
                <input type="text" id="lastName" v-model="formData.lastName" @input="validateInput" required />
                <label for="email">E-posta:</label>
                <input type="email" id="email" v-model="formData.email" required />
                <label for="password">Şifre:</label>
                <input type="password" id="password" v-model="formData.password" required />
                <button type="submit">Üye Ol</button>
              </form>
            </div>
          </div>
        </div>
        </div>
      </div>
     </div>
  </div>
</template>

<script>
export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  data() {
  return {
    books: [],
    cart: [],
    searchQuery: "",
    isModalVisible: false,
    formData: {
      firstName: "",
      lastName: "",  
      email: "",
      password: ""
    },
    formSubmittedSuccessfully: false,
    formSubmissionFailed: false
  };
},

  mounted() {
    this.fetchBooks();
    
  },
  computed: {
    filteredBooks() {
      return this.books.filter((book) =>
        book.title.toLowerCase().includes(this.searchQuery.toLowerCase())
      );
    },
    totalPrice() {
      return this.cart.reduce(
        (total, item) => total + item.price * item.quantity,
        0
      );
    }
  },
  methods: {
    fetchBooks() {
      fetch("https://api.itbook.store/1.0/search/mongodb")
        .then((response) => response.json())
        .then((data) => {
          this.books = data.books;
        })
        .catch((error) => {
          console.error("Kitapları çekerken bir hata oluştu:", error);
        });
    },
    addToCart(book) {
      const itemIndex = this.cart.findIndex(
        (item) => item.isbn13 === book.isbn13
      );

      if (itemIndex !== -1) {
        this.cart[itemIndex].quantity++;
      } else {
        this.cart.push({
          isbn13: book.isbn13,
          title: book.title,
           price: parseFloat(book.price.replace(/[^\d.]/g, "")),
          image: book.image,
          quantity: 1
        });
      }
    },
    removeFromCart(isbn13) {
      this.cart = this.cart.filter((item) => item.isbn13 !== isbn13);
    },
    removeOneFromCart(item) {
      const itemIndex = this.cart.findIndex(
        (cartItem) => cartItem.isbn13 === item.isbn13
      );

      if (itemIndex !== -1 && this.cart[itemIndex].quantity > 1) {
        this.cart[itemIndex].quantity--;
      }
    },
        addOneFromCart(item) {
      const itemIndex = this.cart.findIndex(
        (cartItem) => cartItem.isbn13 === item.isbn13
      );

      if (itemIndex !== -1 && this.cart[itemIndex].quantity > 0) {
        this.cart[itemIndex].quantity++;
      }
    },
    getTotalPrice() {
      // Sepetteki kitapların toplam fiyatını hesapla
      return this.cart.reduce(
        (total, item) => total + item.price * item.quantity,
        0
      );
    },
	showModal() {
      this.isModalVisible = true;
    },
    closeModal() {
      this.isModalVisible = false;
    },
    submitForm() {
      if (
        this.formData.firstName &&
        this.formData.lastName &&
        this.formData.email &&
        this.formData.password
      ) {
        this.formSubmittedSuccessfully = true;
        this.formSubmissionFailed = false;
        this.hideMessagesAfterDelay();
        this.isModalVisible = false;
      } else {
        this.formSubmissionFailed = true;
        this.formSubmittedSuccessfully = false;
        this.hideMessagesAfterDelay();
      }
    },
    hideMessagesAfterDelay() {
      // Belirtilen süre (3 saniye) sonra başarılı ve başarısız mesajları gizle
      setTimeout(() => {
        this.formSubmittedSuccessfully = false;
        this.formSubmissionFailed = false;
      }, 3000);
    },
    validateInput() {
      // Sadece harfleri kabul eden input değerini kontrol et
      this.formData.firstName = this.formData.firstName.replace(
        /[^a-zA-ZğüşıöçĞÜŞİÖÇ]/g,
        ""
      );
      this.formData.lastName = this.formData.lastName.replace(
        /[^a-zA-ZğüşıöçĞÜŞİÖÇ]/g,
        ""
      );
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
body {
  font-family: Arial, sans-serif;
  margin: 0 !important;
  padding: 0;
  background-color: #f6f6f6;
}

h1{
  font-family: fangsong
}

p{
  font-family: monospace
}

.bookstore-banner-section {
    background-image: url('https://thomasmore.qc.ca/wp-content/uploads/2016/10/research-banner.jpg');
    background-size: cover;
    background-repeat: no-repeat;
    background-position: center;
  height: 360px;
  display: flex;
  align-items: center;
  filter: grayscale(70%);
}
.bookstore-banner-section .wrapper {
  max-width: 1092px;
  margin: 0 auto;
}
.bookstore-banner-section .wrapper .flex-area {
  display: flex;
  justify-content: center;
}
.bookstore-banner-section .wrapper .flex-area .text-block h1 {
  padding: 15px 0;
  font-weight: bold;
  color: #000;
  text-align: center;
}

.bookstore-banner-section .wrapper .flex-area .text-block p {
  font-size: 16px;
  color: #000;
  font-weight: bold
}

.main-container {
  max-width: 1092px;
  margin: 25px auto;
}
.search-bar {
  display: flex;
  justify-content: end;
}

.search-wrapper {
    position: relative;
    label {
      position: absolute;
      font-size: 12px;
      color: rgba(0,0,0,.50);
      top: 8px;
      left: 12px;
      z-index: -1;
      transition: .15s all ease-in-out;
    }
    input {
      padding: 8px 12px;
      width:260px;
      color: rgba(0,0,0,.70);
      border: 1px solid green;
      border-radius: 4px;
      transition: .15s all ease-in-out;
      background: white;
      &:focus {
        outline: none;
        transform: scale(1.05);
        & + label  {
          font-size: 10px;
          transform: translateY(-24px) translateX(-12px);
        }
      }
      &::-webkit-input-placeholder {
          font-size: 12px;
          color: rgba(0,0,0,.50);
          font-weight: 100;
      }
    }
  }

.books {
  display: flex;
  justify-content: space-between;
  flex-wrap: wrap;
  padding: 20px 0;
}

.book {
  background-color: #fff;
  padding: 10px 5px;
  border-radius: 8px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
  margin-bottom: 20px;
  text-align: center;
  width: 30%;
}

.book img {
  width: 300px;
  height: 300px;
  border-radius: 4px;
}

.book h3 {
  color: gray;
  text-transform: capitalize;
  font-weight: 600;
  text-align: center;
  font-family: math;
}

.book span {
  color: green;
  font-weight: bold;
}

.add-btn {
  transition: all 0.5s ease;
  color: #000;
  border: 1px solid green;
  font-family: "Montserrat", sans-serif;
  text-align: center;
  line-height: 1;
  font-size: 12px;
  background-color: transparent;
  padding: 10px 25px;
  outline: none;
  border-radius: 4px;
  cursor: pointer;
}

.add-btn:hover {
  background-color: green;
  color: #fff;
  animation: bubble 0.4s ease-out;
}

.cart {
  background-color: #fff;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
  text-align: center;
  margin-top: 20px;

  .total-price {
    margin-top: 20px;
    font-weight: bold;
  }
}

.cart h2 {
  color: #333;
}

.cart ul {
  list-style: none;
  padding: 0;
}

.cart li {
  margin: 10px 0;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.cart li img {
  max-width: 50px;
  height: auto;
  border-radius: 4px;
  margin-right: 10px;
}

.basket-info{
  display: flex;
  align-items: center;
}

.basket-buttons{
display: flex;
    width: 30%;
    gap: 15px;
    align-items: center;
  .s-plus{
    color: green;
    font-size:20px;
      cursor:pointer
  }
    .s-minus{
    color: red;
    font-size:20px;
      cursor:pointer
  }
      .s-trash{
    color: #4c2626;
    font-size:20px;
        cursor:pointer
  }
}

flex-modal-area{
  widht: 100%
}

.cart .headling{
  display: flex;
  justify-content: center;
  align-items: center;
  gap:15px;
    .s-basket{
      font-size: 35px;
      color: #333333
      }
}

/* Modal */
.modal {
  display: flex;
  align-items: center;
  justify-content: center;
  position: fixed;
  z-index: 1;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  overflow: auto;
  background-color: rgba(0, 0, 0, 0.4);
}

.modal-content {
  background-color: #fefefe;
  padding: 20px;
  border: 1px solid #888;
  max-width: 600px;
  /* veya istediğiniz genişlik */
  width: 80%;
  text-align: center;
  /* İçerik ortalanacak şekilde ayarlandı */
}
  
  .modal-btn{
  background-color: #4caf50;
  color: #fff;
  padding: 10px;
  border:none;
  border-radius: 4px;
  cursor: pointer;
  }

.modal-header {
  display: flex;
  justify-content: end;
  align-items: center;
}

.modal-title-info {
  display: block;
  justify-content: center;
} 
  
  .modal-title-info h1{
font-size: 20px
} 
  
    .modal-title-info p{
font-size:12px
} 
  
.modal-info {
  display: flex;
  justify-content: center;
}

.close {
  color: #aaa;
  font-size: 28px;
  font-weight: bold;
  cursor: pointer;
}

.close:hover,
.close:focus {
  color: black;
  text-decoration: none;
}

form {
  padding: 20px;
  max-width: 300px;
  width: 100%;
}

form label {
  display: flex;
  margin-bottom: 8px;
}

form input {
  width: 100%;
  border-radius: 15px;
  border: 1px solid #dee3e1;
  padding: 14px;
  font-size: 14px;
  outline: none;
  margin-bottom: 12px;
  box-sizing: border-box;
}
  
  form input:focus {
border-color:green
}

  
  form button {
  background-color: #fff;
  color: #000;
  padding: 10px;
  border: 1px solid #4caf50;
  border-radius: 4px;
  cursor: pointer;
  width: 100%;
}
    form button:hover {
  background-color: #4caf50;
  color: #fff;
}
.success,
.error {
  text-align: center;
  margin-top: 20px;
  transition: opacity 0.5s ease-in-out;
}

.success {
  color: green;
}

.error {
  color: red;
}

.success.hidden,
.error.hidden {
  opacity: 0;
}

/* Mobil Uyum */
@media only screen and (max-width: 600px) {
  .books {
    flex-direction: column;
    align-items: center;
  }

  .book {
    width: 100%;
    max-width: 300px;
  }
  .bookstore-banner-section .wrapper{
    margin: 0 25px
  }
  .bookstore-banner-section .wrapper .flex-area .text-block p {
    text-align: center;
}
  .search-bar {
    justify-content: center;
}
  .basket-buttons{
    justify-content: center
  }
  .cart{
    margin-bottom: 20rem;
  }
  .cart li{
    display: block;
  }
  .basket-buttons{
  .s-plus{
    font-size:17px;
  }
    .s-minus{
    font-size:17px;
  }
      .s-trash{
    font-size:17px;
  }
}
}


</style>
