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
        <p>Fiyat: <span>{{ book.price }}</span>
        </p>
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
  data: function() {
  return {
    books: [],
    cart: [],
    searchQuery: ""
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
          price: Math.random() * 100,
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
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
body {
  font-family: Arial, sans-serif;
  margin-top: 0px !important;
  margin-right: 0px !important;
  margin-left: 0px !important;
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
      /* top: 8px; */
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
  justify-content: space-around;
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
    gap: 15px;
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

.modal-vue .overlay {
  position: fixed;
  z-index: 9998;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, .5);
}

.modal-vue .modal {
  position: relative;
  width: 300px;
  z-index: 9999;
  margin: 0 auto;
  padding: 20px 30px;
  background-color: #fff;
}

.modal-vue .close{
  position: absolute;
  top: 10px;
  right: 10px;
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
