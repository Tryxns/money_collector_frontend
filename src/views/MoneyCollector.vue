<script setup>
import $ from "jquery";
import axios from "axios"
</script>

<template>
  <div>
    <div>Current facing: {{ facing }}</div>
    <div>Curr coord: {{ curr_coord }}</div>
    <div>Next coord: {{ next_coord }}</div>
    <div>Moves Left: {{ total_movement }}</div>
    <div>Money Available: {{ total_money }}</div>
    <div>Money Collected: {{ money_collected }}</div>
    <div>Total Earnings: {{ total_earning }}</div>
  </div>
  <br>
  <div>
    <button @click="save_game_result"> Save game result into json</button>
  </div>
  <br>

  <v-table>
      <thead></thead>
      <tbody>
          <tr v-for="row in map_size" row="" col="" >
              <td v-for="col in map_size" 
                v-bind:id="`cell_${row}_${col}`" 
                :class="row==1 && col==1 ? 'selected' : ''" 
                :row=row
                :col=col
                :money=money_map[row-1][col-1]
              >
                {{ money_map[row-1][col-1]}}
              </td>
          </tr>
      </tbody>
  </v-table>

</template>
<style>
table, th, td {
  border: 1px solid;
}
.selected{
  background-color: purple;
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
        total_movement: 15,
        movement_log: [1,1],
        interest_rate: getRndInteger(5, 25),
        money_collected: 0,
        total_money: 0,
        total_earning:0,
        my_ip: ""
    }
  },
  methods: {
    keyPressedManager(event){
      console.log(event.key, event.code)
      if(event.key == " "){
        this.move(event)
      }
      else if(event.key == "ArrowUp"){
        this.facing = "NORTH"
        this.next_coord = [this.curr_coord[0]-1, this.curr_coord[1]]
      }
      else if(event.key == "ArrowDown"){
        this.facing = "SOUTH"
        this.next_coord = [this.curr_coord[0]+1, this.curr_coord[1]]
      }
      else if(event.key == "ArrowLeft"){
        this.facing = "WEST"
        this.next_coord = [this.curr_coord[0], this.curr_coord[1]-1]
      }
      else if(event.key == "ArrowRight"){
        this.facing = "EAST"
        this.next_coord = [this.curr_coord[0], this.curr_coord[1]+1]
      }
    },
    move(event) {  
      if (this.total_movement <= 0) {
        alert("u r out of move")
        return 
      } else if(
          this.next_coord[0]>this.map_size || 
          this.next_coord[1]>this.map_size ||
          this.next_coord[0] < 1 || 
          this.next_coord[1] < 1
        ) {
        alert("ga bs maju gan! tar masuk jurang!")
        return 
      }
      $(`#cell_${this.curr_coord[0]}_${this.curr_coord[1]}`).removeClass('selected')

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
      this.total_movement--
      this.movement_log.push(this.curr_coord)
      this.money_collected += parseInt($('.selected').attr("money"))
      $('.selected').attr("money", 0).text(0)
      if (this.total_earning == 0) this.total_earning = this.money_collected
      this.total_earning += this.total_earning*(this.interest_rate/100)
      // console.log()
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
      });
    },
    get_ip_address(){
      axios.get("https://api.ipify.org?format=json").then(response => {
        console.log(response.data.ip)
        this.my_ip = response.data.ip
      })
    }
  },
  mounted() {
    document.addEventListener( "keydown", this.keyPressedManager );
    this.get_ip_address()
  },
  created(){
    for(let x=0;x<5;x++){
      let temp=[]
      for(let x=0;x<5;x++){
        let zonk_or_not = Math.round(Math.random())
        let money = 0
        if (zonk_or_not > 0) money = getRndInteger(500,20000)
        temp.push(money)
        this.total_money += money
      }
      money_map.push(temp)
    }
    // console.log(money_map)
  }
}
</script>