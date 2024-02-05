<script setup>
import $ from "jquery";
import axios from "axios"

import SvgIcon from '@jamescoyle/vue-icon'
import { mdiTreasureChest } from '@mdi/js'

</script>

<template>
  <div>
    <div>Current facing: {{ facing }}</div>
    <div>Curr coord: {{ curr_coord }}</div>
    <div>Next coord: {{ next_coord }}</div>
    <div>Moves Left: {{ movement_point }}</div>
    <div>Money Available: {{ money_remaining }}</div>
    <div>Money Collected: {{ money_collected }}</div>
    <div>Total Earnings: {{ total_earning }}</div>
  </div>
  <br>
  <div>
    <button @click="place" type="button"> Place</button>
  </div>
  <div>
    <button v-if="movement_point == 0 && movement_log.length > 1" @click="save_game_result"> Save game result into json</button>
  </div>
  <br>
  <div>
    <v-table >
        <thead></thead>
        <tbody>
            <tr v-for="row in map_size" row="" col="" >
                <td v-for="col in map_size" 
                  v-bind:id="`cell_${row}_${col}`" 
                  :row=row
                  :col=col
                  :money=money_map[row-1][col-1]
                >
                  <!--  -->
                  <svg-icon v-show="is_placed" v-if="money_map[row-1][col-1] > 0" type="mdi" :path="path_treasure"></svg-icon>
                  <!-- <span class="mdi mdi-treasure-chest"></span>

                  <span v-if="money_map[row-1][col-1] > 0" class="mdi mdi-treasure-chest"></span> -->
                  <!-- {{ money_map[row-1][col-1]}} -->
                </td>
            </tr>
        </tbody>
    </v-table>
  </div>


</template>
<style>
table, th, td {
  border: 1px solid;
}
.selected{
  background-color: limegreen;
  
}
.east{
  background-image: url('../assets/east.svg')
}
.west{
  background-image: url('../assets/west.svg')
}
.north{
  background-image: url('../assets/north.svg')
}
.south{
  background-image: url('../assets/south.svg')
}

table {
  table-layout: fixed;
}
td {
  height: 10vh;
  width: 10vh;
}
</style>
<script>
let url = "http://localhost:3000/report";
let money_map = []

function getRndInteger(min, max) {
  return Math.floor(Math.random() * (max - min) ) + min;
}

export default {
  data () {
    return {
        results:[],
        map_size: 5,
        facing:"EAST",
        curr_coord: [1,1],
        next_coord: [1,2],
        movement_point: 0,
        movement_log: [[1,1]],
        interest_rate: getRndInteger(5, 25),
        money_collected: 0,
        money_remaining: 0,
        total_money: 0,
        total_earning:0,
        my_ip: "",
        path_treasure: mdiTreasureChest,
        is_placed:false,
        degree:90,
        degree_arr: {0: "NORTH", 90: "EAST", 180: "SOUTH", 270: "WEST"}
    }
  },
  methods: {
    keyPressedManager(event){
      console.log(event.key, event.code)
      if(event.key == " "){
        this.move(event)
      }
      else if(event.key == "ArrowLeft"){
        this.degree -= 90
        if (this.degree < 0) this.degree = 270
      }
      else if(event.key == "ArrowRight"){
        this.degree += 90
        if (this.degree >= 360) this.degree = 0
      }
      this.facing = this.degree_arr[this.degree]

      if(this.facing == "NORTH"){
        this.next_coord = [this.curr_coord[0]-1, this.curr_coord[1]]
      }
      else if(this.facing == "SOUTH"){
        this.next_coord = [this.curr_coord[0]+1, this.curr_coord[1]]
      }
      else if(this.facing == "WEST"){
        this.next_coord = [this.curr_coord[0], this.curr_coord[1]-1]
      }
      else if(this.facing == "EAST"){
        this.next_coord = [this.curr_coord[0], this.curr_coord[1]+1]
      }

      // else if(event.key == "ArrowUp"){
      //   this.facing = "NORTH"
      //   this.next_coord = [this.curr_coord[0]-1, this.curr_coord[1]]
      // }
      // else if(event.key == "ArrowDown"){
      //   this.facing = "SOUTH"
      //   this.next_coord = [this.curr_coord[0]+1, this.curr_coord[1]]
      // }
      // else if(event.key == "ArrowLeft"){
      //   this.facing = "WEST"
      //   this.next_coord = [this.curr_coord[0], this.curr_coord[1]-1]
      // }
      // else if(event.key == "ArrowRight"){
      //   this.facing = "EAST"
      //   this.next_coord = [this.curr_coord[0], this.curr_coord[1]+1]
      // }

      $('.selected').removeClass("north south east west").addClass(this.facing.toLowerCase());
    },
    move(event) {  
      if (this.movement_point <= 0) {
        alert("Game over: out of movement points. You can save the game result to the server")
        return 
      } else if(this.money_remaining == 0){
        alert("Game over: All available money have been collected, Congratz! You can save the game result to the server")
        return 
      }
      else if(
          this.next_coord[0]>this.map_size || 
          this.next_coord[1]>this.map_size ||
          this.next_coord[0] < 1 || 
          this.next_coord[1] < 1
        ) {
        alert("Oops.. stay away from the cliff side please!")
        return 
      }
      $(`#cell_${this.curr_coord[0]}_${this.curr_coord[1]}`).removeClass('selected north south east west')

      $(`#cell_${this.next_coord[0]}_${this.next_coord[1]}`).addClass('selected')


      this.curr_coord = this.next_coord
      
      if(this.facing == "NORTH"){
        this.next_coord = [this.curr_coord[0]-1, this.curr_coord[1]]
      }
      else if(this.facing == "SOUTH"){
        this.next_coord = [this.curr_coord[0]+1, this.curr_coord[1]]
      }
      else if(this.facing == "WEST"){
        this.next_coord = [this.curr_coord[0], this.curr_coord[1]-1]
      }
      else if(this.facing == "EAST"){
        this.next_coord = [this.curr_coord[0], this.curr_coord[1]+1]
      }
      this.movement_point--
      this.movement_log.push(this.curr_coord)

      let selected_money = parseInt($('.selected').attr("money"))
      this.money_collected += selected_money
      this.money_remaining -= selected_money
      $('.selected').attr("money", 0).empty()
      
      if (this.total_earning == 0) this.total_earning = this.money_collected
      this.total_earning += this.total_earning*(this.interest_rate/100)

      if (this.movement_point == 0) {
        alert("Game over: out of movement points. You can save the game result to the server")
        return 
      } else if(this.money_remaining == 0){
        alert("Game over: All available money have been collected, Congratz. You can save the game result to the server")
        return 
      }
    },
    save_game_result() {
      axios.post(url, {
        UserID:this.my_ip,
        MovementHistories:this.movement_log,
        TotalMoneyAvailable:this.total_money,
        TotalMoneyFound:this.money_collected,
        InterestRate: this.interest_rate,
        TotalMoneyEarn: this.total_earning
      })
      .then(function (response) {
        console.log(response);
        alert("Result saved")
      })
      .catch(function (error) {
        console.log(error);
        alert(error.message)
      });
    },
    get_ip_address(){
      axios.get("https://api.ipify.org?format=json").then(response => {
        console.log(response.data.ip)
        this.my_ip = response.data.ip
      })
    },
    place(){
      this.is_placed = true
      if ($('.selected').length == 0){
        // this.generate_new_map()
        document.addEventListener( "keydown", this.keyPressedManager );
        $("#cell_1_1").addClass('selected')
        $('.selected').addClass('east')
        this.movement_point = 15
      }
    },
    generate_new_map(){
      money_map = []
      for(let row=0;row<this.map_size;row++){
        let temp=[]
        for(let col=0;col<this.map_size;col++){
          
          let zonk_or_not = Math.round(Math.random())
          let money = 0
          if (row == 0 && col == 0) {}
          else if (zonk_or_not > 0) money = getRndInteger(500,20000)
          temp.push(money)
          this.total_money += money
        }
        money_map.push(temp)
        this.money_remaining = this.total_money
      }
    }
  },
  mounted() {
    // document.addEventListener( "keydown", this.keyPressedManager );
    
    this.get_ip_address()
    // $('.selected').addClass('east')
  },

  created(){
    this.generate_new_map()
    // console.log(money_map)
  },
  components: {
		SvgIcon
	},
}
</script>