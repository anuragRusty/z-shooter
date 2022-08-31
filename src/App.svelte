<script>
const bg = 'https://i.postimg.cc/3RWm5318/background.png';
const zombie1 = 'https://i.postimg.cc/9MBCFvM9/zombie1.gif';
const zombie2 = 'https://iili.io/4kxCFe.gif';
const zombie3 = 'https://i.postimg.cc/FsVTyLmL/zombie3.gif';
const zombie1_dead = 'https://i.postimg.cc/X7NG0SLK/zombie-dead.gif';
const zombie2_dead = 'https://i.postimg.cc/BvLDrcxy/zombie2-dead.gif';
const zombie3_dead = 'https://i.postimg.cc/pLBLxqLc/zombie3-dead.gif';
const human_run = 'https://i.postimg.cc/zX4g2rBK/human-run.gif';
const human_dead = 'https://i.postimg.cc/gJBJwVL8/human-dead.gif';
const sniper = 'https://i.postimg.cc/PJtsVzkj/sniper.gif';
const sniper_shoot = 'https://i.postimg.cc/vmkS67mz/sniper-shoot.gif';
const heart = 'https://i.postimg.cc/qqsj2JLJ/heart.png';

const screen_height = 1152;
const z_speed = 4;
const z_height = 64;

const x_cordinates = [32,96,160,224,288,352,416,480,544,608,672];
const degree_stack = [-30,-25,-20,-15,-5,0,5,15,20,25,30];
const level_stack = [20,27,37,51,67,80,95,120,130,155,170];
const human_prob_stack = new Array(5).fill(false);
const zombie_stack = [zombie1,zombie2,zombie3];
const zombie_dead_stack = [zombie1_dead,zombie2_dead,zombie3_dead];

human_prob_stack[Math.floor(Math.random()*human_prob_stack.length)] = true;

let gameOver = false;
let gameState = "Pause";
let gamePause = false;
let score = 0;
let life = 3;
let level = 0;
let life_stack = [heart,heart,heart];
let gun_load = true;
let current_snip = sniper;
let rotate = 0;

function z_row(max){
 let arr = [];
  for(let i = 0; i < max; i++){
  let zombie_rand = Math.floor(Math.random()*zombie_stack.length);
    arr.push(
    {
    zombie:true,
    x:x_cordinates[Math.floor(Math.random()*x_cordinates.length)],
    y:-32,
    move:true,
    health:1,
    speed:z_speed,
    img:zombie_stack[zombie_rand],
    reached:false,
    img_dead:zombie_dead_stack[zombie_rand],
    opac:0,
    }
    );

    if(human_prob_stack[Math.floor(Math.random() * human_prob_stack.length)]){
      arr.push(
          {
           zombie:false,
           x:x_cordinates[Math.floor(Math.random()*x_cordinates.length)],
           y:-128,
           move:true,
           health:1,
           speed: z_speed,
           img:human_run,
           img_dead:human_dead,
           reached:false,
           opac:0,
          }
      );
    }
  }
  for(let i = 0; i<arr.length; i++){
    for(let j = i + 1; j < arr.length; j++){
      if(arr[i].x === arr[i].x && arr[i].y - arr[j].y <= z_height/2){
        arr[j].y -= z_height;
      }
    }
  }
  return arr;
}

let zombies = z_row(level_stack[0]);

function move_down(){
for(let i = 0; i<zombies.length; i++){
  if(zombies[i].y <= screen_height + z_height/2 && zombies[i].move && !gameOver && !gamePause){
    zombies[i].y += zombies[i].speed;
    zombies[i].move = true;
  }
  if(zombies[i].y >= screen_height + z_height/2 && zombies[i].move){
    zombies[i].move = false;
    zombies[i].reached = true;
    if(zombies[i].zombie){
      life -= 1;
      life_stack.length -= 1;
    }else{
       score += 10;
    }
  }
 }
}

function shoot(i){
  current_snip = sniper_shoot;
  if(zombies[i].move && zombies[i].health >= 1 && gun_load && !gameOver && !gamePause){
    zombies[i].health -= 1;
    gun_load= false;
  }
  if(zombies[i].health <= 0 && zombies[i].move){
    zombies[i].move = false;
    zombies[i].img = zombies[i].img_dead;
    if(zombies[i].zombie){
    zombies[i].move = false;
    score += 1;
    }
    if(!zombies[i].zombie){
      life -= 1;
      life_stack.length -= 1;
     }else{
       score += 1;
     }
    }
    if(zombies[i].x < 352){
       rotate = -20;
    }
    if(zombies[i].x > 352){
      rotate = 20;
    }
  rotate = degree_stack[x_cordinates.indexOf(zombies[i].x)];
  setTimeout(() => decomp(i),1600);
  setTimeout(reload,2000);
}

function reload(){
  gun_load = true;
  current_snip = sniper;
}

function decomp(i){
  if(zombies[i].health <= 0){
   zombies = [...zombies.slice(0,i),...zombies.slice(i+1)];
 }
}

function decomp2(){
for(let j = 0; j < zombies.length; j++){
  if(zombies[j].reached){
  zombies = [...zombies.slice(0,j),...zombies.slice(j+1)];
   }
  }
}

function visible(){
  for(let i = 0; i<zombies.length; i++){
     if(zombies[i].y >= 2*z_height){
        zombies[i].opac = 1;
     }
  }
}

function level_up(){
  if(zombies.length <= 0){
     level += 1;
     zombies = z_row(level_stack[level]);
  }
  if(zombies.length <= 3){
  for(let i = 0; i<zombies.length; i++){
       decomp(i);
     }
    }
}

function state_up(){
  if(life <= 0){
     gameOver = true;
  }
}

function handle_state(){
  gamePause = !gamePause;
  if(gamePause){
     gameState = "Pause"
  }else{
    gameState = "Start";
  }
}

function restart(){
   gameOver = false;
   gamePause = false;
   level = 0;
   life = 3;
   score = 0;
   zombies = z_row(level_stack[level]);
   life_stack = [heart,heart,heart];
}

function update(){
  move_down();
  level_up();
  decomp2();
  visible();
  state_up();
}

setInterval(update,166);
</script>
<div class="gamestatus">
<div class="hearts">
{#each life_stack as hearts}
<div class="heart" style="height:128px; width:128px; background-image: url('{hearts}');"></div>
{/each}
</div>
<h1>Score {score}</h1>
</div>
<div class="grid" style="background-image: url('{bg}')">
  {#each zombies as zombie,indx}
  <div on:click={() => shoot(indx)} class="box" style="position:absolute; top:{zombie.y + "px"}; left:{zombie.x + "px"}; background-image:url('{zombie.img}'); opacity:{zombie.opac}"></div>
 {/each}
 {#if gameOver}
 <h1 class="state">GAME OVER</h1>
 {/if}
 <div class="sniper" style="background-image: url('{current_snip}'); transform: rotate({rotate + "deg"})"></div>
</div>
<div class="control">
<button class="restart" on:click={restart}>RESTART</button>
<button class="gamestate" on:click={handle_state}>{gameState}</button>
</div>
<h1>Level {level + 1}</h1>

<style>
  .grid{
    width: 768px;
    height: 1152px;
    background-color: blue;
  }
  .box{
    width: 64px;
    height: 64px;
  }
  .sniper{
    height: 128px;
    width:128px;
    position: absolute;
    left: 352px;
    top: 1180px;
  }
  .restart{
    background-color: green;
  }
  .state{
    position: absolute;
    top:650px;
    left:352px;
    color: red;
  }
  .control{
    width: 768px;
    display: flex;
    justify-content: space-between;
  }
  .gamestate{
    background-color: blue;
  }
  button{
  font-size: 22px;
  border: none;
  border-radius: 0%;
  }
  .gamestatus{
    display: flex;
    justify-content: space-between;
  }
  .hearts{
    display: flex;
  }
  </style>
