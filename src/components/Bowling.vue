<template>
    <div id="Bowling">
        <div v-if="errored">Unable to gather scores</div>
        <div v-else>
          <div 
          class="scoreboard">
            <div class="frame" v-for="(points, index) in info.points" v-bind:key="index">
                <div class="points">
                  <div class="roll">{{points[0]}}</div>
                  <div class="roll">{{points[1]}}</div>
                </div>
                <div class="tallyFrame"></div>
                <div class="tally">
                {{pointTally[index]}}
                </div>
            </div>
          </div>
         
          <div>Token: {{info.token}}</div>
          <div>Response: Success = {{response}}</div> 
      </div> 
    </div>
</template>

<script>
import axios from 'axios'

export default {
  props: {
    msg: String
  },
  data(){
      return{
          info: [],
          pointTally: [],
          loading: true,
          errored: false,
          response: null
      }
  },
  created(){
  },
  mounted() {
    
    axios
    .get("http://13.74.31.101/api/points")
    .then(response => {this.info = response.data, this.cleanData(this.info.points)})
    .catch(error => {
        console.log(error)
        this.errored = true})
    .finally(() => this.loading = false);  
    
    

  },
  methods: {
    cleanData(points){
      console.log(this.info);
      var result = new Array();
      console.log(points[0]);

      var total = 0;

      for (let i = 0; i < points.length; i++) {
        var roll1 = points[i][0];
        var roll2 = points[i][1];

        if(roll1 === 10 && i < 9 && points[i+1] !== undefined){
          total += 10 + this.getNextFramePoint(i, 2);
          result.push(total);
        } else if(roll1 + roll2 === 10 && i < 9 && points[i+1] !== undefined){
          total += 10 + this.getNextFramePoint(i, 1);
          result.push(total);
        } else if(i === 9 && points.length > 9){
          total += this.getThreeRolls(i);
          result.push(total);
        } else  if (i < 9 || points.length < 9){
          total += roll1+roll2;
          result.push(total);
        }
        //result.push(total);

        //console.log(this.info.points[0]);
      }
      
      console.log(result);
      let data = {
        "token": this.info.token,
        "points": result,
      }
      this.sendData(result);
      this.pointTally = result;
      console.log(data);
      //console.log(JSON.parse(jsonArr));
    },
    getNextFramePoint(index, bonus){
      var roll1 = this.info.points[index+1][0];
      var roll2 = this.info.points[index+1][1];
      
      if(roll1 === 10 ){
        return bonus === 1 ? roll1 : roll1 + this.info.points[index+2][0];
      } else if (roll1 === 10){
        return roll1;
      }
      return bonus === 1 ? roll1 : (roll2 === 10 && roll1 !== 10 ? roll2 : roll2 + roll1);
    },
    getThreeRolls(i){
      var roll1 = this.info.points[i][0];
      var roll2 = 0;
      var roll3 = 0;
      if(roll1 === 10){
        roll2 = this.info.points[i+1][0];
        roll3 = this.info.points[i+1][1];
      } else {
        roll2 = this.info.points[i][1];
        roll3 = this.info.points[i+1][0];
      }

      
      return roll3 === 10 && roll2 !== 10 ? roll3 + roll1 : (roll2 === 10 && roll1 !== 10 ? roll3 + roll2 : roll1 + roll2 + roll3);
    },
    async sendData(result){
      
      await axios.post("http://13.74.31.101/api/points", {points:result, token: this.info.token})
      .then(response => {this.response = response.data.success, console.log(response.data)})
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.scoreboard{
  background-color:aliceblue;
  padding: 50px 0;
}
.frame{
  display:inline-block;
  margin: 0 10px;
  vertical-align: top;
}
.points{
  float:right;
}

.roll{
  border: 1px solid black;
  width:20px;
  padding: 2px;
  display: inline-block;
  vertical-align: top;
}
.tally {
  display: inline-block;
}
</style>
