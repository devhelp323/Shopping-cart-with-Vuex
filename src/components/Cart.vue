<template>
  <div class="cart container">
    <h2 @click="checkItems"> {{user.id}}'s Cart </h2>
    <p class="item_quantity">total <span>{{cartitems.length}}</span>product</p>

    <table class="cart_table">
        <tr class="table title">
            <th v-for="column in columns" :key="column.index">{{column}}</th>
            <th>　　</th>
        </tr>

        <tr v-for="cartitem in cartitems" :key="cartitem.id" class="table_value">
            <td>{{cartitem.id}}</td>
            <td>{{cartitem.user}}</td>
            <td>{{cartitem.product_id}}</td>
            <td>{{cartitem.title}}</td>
            <td>{{cartitem.color}}</td>
            <td>{{cartitem.size}}</td>
            <td>{{addComma(cartitem.price)}}</td>
            <td>{{cartitem.quantity}}</td>
            <td>{{addComma(cartitem.total)}}</td>
            <td><i class="material-icons delete_btn"  @click="deleteCartitem(cartitem.id)">clear</i></td>
        </tr>

    </table>
    <div class="result">
      <table>
        <tr>
          <th>total amount</th>
          <td>{{quantitySum()}}</td>
        </tr>
        <tr>
          <th>total price</th>
          <td>{{addComma(priceSum())}}</td>
        </tr>
      </table>
      
    </div>
  </div>
</template>

<script>
import db from '@/firebase/init'
import firebase from 'firebase'
import moment from 'moment'


export default {
  name: 'cart',
  data () {
    return {
      moment: moment,
      columns:['No','User', 'Product ID', 'Title', 'Color', 'Size', 'Price', 'Quantity' ,'Total'],
      cartitems: [],
      user: null,
      matchitems:[]
    }
  },
  created(){
    //fetch data from the firestore
    db.collection('cartitems').get()
    .then(snapshot =>{//
      snapshot.forEach(doc =>{//
        //console.log(doc.data(), doc.id);
        let cartitem = doc.data()
        cartitem.id =  doc.id
        this.cartitems.push(cartitem) 
      })
      console.table(this.cartitems)
    })

    //get current user
    console.log("current uid", firebase.auth().currentUser.uid);
    db.collection('users').where('user_id','==',firebase.auth().currentUser.uid).get()
    .then(snapshot=>{
      snapshot.forEach(doc => {
        this.user =doc.data(),
        this.user.id = doc.id//
      })
    })
  },
  methods:{
    priceSum(){
        return this.cartitems.reduce((prev,cur) => prev + cur.total,0)
    },
    quantitySum(){
        return this.cartitems.reduce((prev,cur) => prev + cur.quantity,0)
    },
    checkItems(){
      for(var i=0; i<=this.cartitems.length; i++){
        if(this.cartitem.user == this.user.alias){
          this.matchitems.push(this.cartitem)
          console.log(this.matchitems)
        }
      }
    },
    addComma(num) {
        var regexp = /\B(?=(\d{3})+(?!\d))/g
        return num.toString().replace(regexp, ',')
    },
    deleteCartitem(id){ 
        var result = confirm('Delete from cart?')
        if(result){
        db.collection('cartitems').doc(id).delete() //delete form db. 
        .then(() =>{
          this.$swal({
            position: 'center',
            type: 'success',
            title: 'Delete complete',
            showConfirmButton: false,
            timer: 1000
          })
          this.cartitems = this.cartitems.filter(cartitem =>{
            return cartitem.id !=id //
          })
        })
       }      
    }
  }
}
</script>

<style scoped>
table th, td{
    text-align: center;
}

.delete_btn{
    cursor: pointer;
}

.result{
  margin-top: 40px;
  width: 50%;
  float: right;
}
</style>
