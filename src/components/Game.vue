<template>
  <div id="Game">
    <h1>2048小游戏</h1>
    <h2>当前分数：{{ game.score }} </h2>
    <div id="content">
      <div id="veil" v-show="game.over">
        <p><b>游戏结束!</b><br/>
        你的分数：{{ game.score }}</p>
        <p class="btn" @click="init()">🔁</p>
      </div>
      <div id="cheat" v-show="game.cheat_mode">
        <div id="cheat-content">
          <label>X坐标：</label><input type="text" v-model="game.cheat_X"/><br/>
          <label>Y坐标：</label><input type="text" v-model="game.cheat_Y"/><br/>
          <label>变更值：</label><input type="text" v-model="game.cheat_value"/><br/>
          <button @click="cheat()">变更</button>
        </div>
      </div>
      <div class="row" v-for="row in game.list" :key="row.index">
        <div class="col cell" :class="'cell-'+col " v-for="col in row" :key="col.index">{{ col?col:"" }}</div>
      </div>
    </div>
    
  </div>
</template>

<script>
export default {

  data() {
    return {
      SIZE: 4,
      game:{
        score: 0,
        over: true,
        list: null,
        // list: [
        //   [1,3,1,3],
        //   [3,1,3,1],
        //   [1,3,1,3],
        //   [8,4,0,0],
        // ],
        cheat_mode: false,
        cheat_value: -1,
        cheat_X: -1,
        cheat_Y: -1,
      }
    }
  },

  created(){
    this.init();
  },

  methods:{
    init(){
      this.$set(this.game, "list",  Array.from(Array(this.SIZE)).map(() => Array(this.SIZE).fill(0)))
      document.addEventListener('keydown', this.keyDown)
      this.$set(this.game, "over", false);
      this.$set(this.game, "score", 0);
      this.randomFill();
      this.randomFill();
    },

    keyDown(e) {
      let _list = this.game.list;

      switch (e.keyCode){
        case 82:
          this.init();
          return;
        case 67:
          this.$set(this.game, "cheat_mode", !this.game.cheat_mode);
          break;
        case 37:  //  左移
          _list = this.rotateAnticlockwise(_list, 4);
          _list = this.moveLeft(_list);
          break;
        case 38:  //  上移
          _list = this.rotateAnticlockwise(_list, 1);
          _list = this.moveLeft(_list);
          _list = this.rotateAnticlockwise(_list, 3);
          break;
        case 39:  //  右移
          _list = this.rotateAnticlockwise(_list, 2);
          _list = this.moveLeft(_list);
          _list = this.rotateAnticlockwise(_list, 2);
          break;
        case 40:  //  下移
          _list = this.rotateAnticlockwise(_list, 3);
          _list = this.moveLeft(_list);
          _list = this.rotateAnticlockwise(_list, 1);
          break;
        default:
          break;
      }

      if( this.ifListChange(_list, this.game.list) ){
        this.$set(this.game, "list",  _list)
        this.randomFill();
      }
      if( this.ifGameOver() ) {
        this.$set(this.game, "over", true);
        document.removeEventListener('keydown', this.keyDown);
      }
    },

    ifGameOver(){
      let _list = this.game.list;

      for( let i=0; i<_list.length; i++){
        for( let j=0; j<_list.length; j++){
          if ( _list[i][j]==0 ){
            return false;
          }
        }
      }

      for( let i=0; i<_list.length; i++){
        for( let j=0; j<_list.length-1; j++){
          if ( _list[i][j]==_list[i][j+1] ||  _list[j][i]==_list[j+1][i] ){
            return false;
          }
        }
      }

      return true;
    },

    ifListChange(list1, list2){
      for( let i=0; i<list1.length; i++ ){
        for( let j=0; j<list1.length; j++ ){
          if( list1[i][j]!=list2[i][j] ){
            return true;
          }
        }
      }
      return false;
    },

    rotateAnticlockwise(list, n){
      let list_rotate = Array.from(Array(this.SIZE)).map(() => Array(this.SIZE).fill(0));

      for(let i=0; i<list.length; i++){
        for(let j=0; j<list.length; j++){
          list_rotate[i][j] = list[j][list.length-i-1]
        }
      }
      if( n>1 ){
        list_rotate = this.rotateAnticlockwise(list_rotate, n-1);
      }
      return list_rotate;
    },

    moveLeft(list){
      list.map( row => this.mergeRow(row, true) )
      return list;
    },

    mergeRow(row, flag){
      if( !flag ){
        return row;
      }
      flag = false;
      // 都移动到左边
      for( let i=1; i<row.length ; i++ ){
        if( row[i] ){
          for( let j=0; j<=i ; j++ ){
            if( row[j]==0 ){
              [row[i], row[j]] = [row[j], row[i]]
            }
          }
        }
      }
      // 有相同相邻的方块则合并
      for( let i=0; i<row.length ; i++ ){
        if( row[i] && row[i]==row[i+1]  ){
          flag = true;
          row[i] = 2*row[i];
          row[i+1] = 0;
          this.addScore(row[i]);
        }
      }
      return this.mergeRow(row, flag);
    },

    // 随机生成新方块y
    randomFill(){
      let _list = this.game.list;
      let random_cell_x, random_cell_y 
     
     [ random_cell_x, random_cell_y ] = this.randomChooseCell(_list);
     if (random_cell_x == -1) {
       return
     }
      _list[random_cell_x][random_cell_y] = this.random2or4();
      this.$set(this.game, "list",  _list)
    },

    // 随机获取一个空单元格的坐标返回
    randomChooseCell(list){
      let empty_cell = [];

      for(let i=0; i<list.length; i++){
        for(let j=0; j<list.length; j++){
          if( !list[i][j] ){
            empty_cell.push([i, j]);
          }
        }
      }
      if (empty_cell.length == 0) {
        return [-1, -1]
      }
      return empty_cell[ Math.floor( Math.random() * empty_cell.length ) ];
    },

    // 按一定概率比返回2或4
    random2or4(){
      return (Math.random()<0.7)?2:4;
    },

    addScore(score_add){
      this.$set(this.game, "score",  this.game.score+=score_add)
    },

    canCheat(X, Y, value){
      if( value%2 || value<0 ){
        return false;
      }
      if( X > this.SIZE || X < 0 ){
        return false;
      }
      if( Y > this.SIZE || Y < 0 ){
        return false;
      }
      return true;
    },

    cheat(){
      let _list = this.game.list;
      let _X = this.game.cheat_X-1;
      let _Y = this.game.cheat_X-1;
      let _value = this.game.cheat_value;

      if( !this.canCheat(_X, _Y, _value) ){
        alert("非法参数！");
        return
      }
      _list[_X][_Y] = _value;
      _list = this.rotateAnticlockwise(_list, 4);
      this.$set(this.game, "list",  _list)
    },

    pass(something){
      console.log(something)
    }
  }

};
</script>

<style lang="stylus" scoped>
#Game{
  width: 500px;
  margin: 0 auto;
}

h1, h2{
  color: white;
}

#content{
  position: relative;

  width: 400px;
  height: 400px;
  padding: 10px 10px 10px 10px;

  background-color: #756e64;
  border-radius: 10px;
}

#veil{
    position: absolute;
    left: 0px;
    top: 0px;
    
    width: 400px;
    height: 360px;
    padding: 50px 10px 10px 10px;

    background-color: rgba(157, 147, 137, 0.7);
    background-color: rgba(255, 220, 0, 0.7);
    background-color: rgba(147, 137, 157, 0.7);

    z-index: 2;
    text-align: center;

    p{
      color: white;
      font-size: 30px;
      b{
        font-size: 50px;
      }
    }
    .btn{
      font-size: 50px;
      cursor: pointer;
      // width: auto;
    }
}

#cheat{
    position: absolute;
    left: 0px;
    top: 0px;
    
    width: 400px;
    height: 360px;
    padding: 50px 10px 10px 10px;

    background-color: rgba(84, 175, 249, 0.8);

    z-index: 3;
    text-align: center;
  #cheat-content{
    position: absolute;

    width:200px;
    height: 200px;

    margin-left:-(@width/2)px;
    margin-top:-(@height/2)px;
    left:50%;
    top:50%;

    input{
      float: right;
      font-size: 20px;
      width:100px;
      margin-bottom: 30px;
    }
    label{
      float: left;
      color: white;
      font-size: 20px;
      margin-bottom: 30px;
    }
    button{
      position: absolute;
      bottom:0px;
      left:0px;
      width: 200px;
      height: 30px;
    }
  }
}

cell-color(_color, _bgColor){
  color: _color;
  background-color: _bgColor;
}

.cell{
    width: 80px;
    height: 80px;
    margin: 10px 10px 10px 10px;

    float: left;
    text-align: center;
    background-color: white;
    
    font-size: 30px;
    font-weight: bold;
    line-height: 80px;
    border-radius: 10px;

    cell-color(#ffffff, #5dd991)
    &-0{
      color: white;
      background-color: white;
    }
    &-2{
      cell-color(#736e62, #eee4d6)
    }
    &-4{
      cell-color(#736e62, #ece1c5)
    }
    &-8{
      cell-color(#ffffff, #faae70)
    }
    &-16{
      cell-color(#ffffff, #f78546)
    }
    &-32{
      cell-color(#ffffff, #fe6d52)
    }
    &-64{
      cell-color(#ffffff, #f26549)
    }
    &-128{
      cell-color(#ffffff, #f2d869)
    }
    &-256{
      cell-color(#ffffff, #f3ce46)
    }
    &-512{
      cell-color(#ffffff, #e3c028)
    }
    &-1048{
      cell-color(#ffffff, #ebc301)
    }
}
</style>

