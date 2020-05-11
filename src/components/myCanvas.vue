<template>
	<div class="fuck">
		<div>
			
			<button @click="startGame">开始游戏</button>
			<button @click="stopGame">停止游戏</button>
			<button @click="changeSpeed">当前速度{{speed}}</button>
		</div>
		<canvas id="canvas" :width="col*(L+1)+1" :height="row*(L+1)+1" ref="canvas" @keydown="keyDown(e)"></canvas>
		<!-- <div class="rows" v-for="(rows,y) in grids" :key="y">
			<div class="grid" :style="'background-color:'+returnColor(x,y)" v-for="(grid,x) in rows" :key="x+y" @click="colour(grid)"></div>
		</div> -->
	</div>
</template>

<script>
	export default {
		name:'myCanvas',
		data(){
			return {
				L:20,
				context:null,
				snake:null,
				food:null,
				T:null,
				speed:1000,
			}
		},
		props:{
			//多少排
			row:{
				type:Number,
				default:32
			},
			//多少列
			col:{
				type:Number,
				default:32 
			},
			pen:{
				type:String,
				default:'#000000'
			}
		},
		computed:{
			grids(){
				let cols=new Array()
				for(let y=0;y<this.col;y++){
					let rows=new Array()
					for(let x=0;x<this.row;x++){
						rows.push({
							x:x,
							y:y,
							color:'#ffffff',
						})
					}
					cols.push(rows)
				}
				return cols
			},
			W(){
				return this.col*(this.L+1)+1
			},
			H(){
				return this.row*(this.L+1)+1
			}
		},
		mounted(){
			// console.log(this.grids)
			this.initCanvas()
			// this.initGame()
			this.initGrids()
		},
		watch:{
			pen(){
				console.log(this.pen)
			}
		},
		methods:{
			//初始化画布
			initCanvas(){
				const canvas=this.$refs.canvas
				this.context=canvas.getContext('2d')
				
			},
			initGrids(){
				this.context.strokeStyle='#ccc'
				for(let i=0;i<=this.row;i++){
					this.drawLine(0,i*(this.L+1),this.col*(this.L+1),i*(this.L+1))
				}
				for(let i=0;i<=this.col;i++){
					this.drawLine(i*(this.L+1),0,i*(this.L+1),this.row*(this.L+1))
				}
			},
			drawLine(x1,y1,x2,y2){
				this.context.beginPath()
				this.context.moveTo(x1,y1)
				this.context.lineTo(x2,y2)
				this.context.stroke()
			},
			
			
			drawSnake:function(){
				// initGrids()
				this.snake.body.forEach(function(e,i){
					this.context.fillStyle=this.snake.color
					this.context.fillRect(e.x*(this.L+1)+1,e.y*(this.L+1)+1,this.L,this.L)
				},this)
			},
			drawFood:function(){
				this.food.foods.forEach(function(e,i){
					this.context.fillStyle=this.food.color
					this.context.fillRect(e.x*(this.L+1)+1,e.y*(this.L+1)+1,this.L,this.L)
				},this)
			},
			drawAll(){
				this.context.clearRect(0,0,this.W,this.H)
				this.initGrids()
				this.drawSnake()
				this.drawFood()
			},
			initGame(){
				this.snake=new Snake()
				this.food=new Food({snake:this.snake})
				// console.log(this.food)
				var that=this
				window.onkeydown=function(ev){
					switch(ev.keyCode){
						case 37:
						if(that.snake.direction=='up'||that.snake.direction=='down'){
							that.snake.direction='left';
						}
						break;
						case 38:
						if(that.snake.direction=='left'||that.snake.direction=='right'){
							that.snake.direction='up';
						}
						break;
						case 39:
						if(that.snake.direction=='up'||that.snake.direction=='down'){
							that.snake.direction='right';
						}
						break;
						case 40:
						if(that.snake.direction=='left'||that.snake.direction=='right'){
							that.snake.direction='down';
						}
						break;
						
					}
				}
				// console.log(this.snake.body)
				this.drawAll()
				clearTimeout(this.T)
				this.T=setTimeout(this.move,this.speed)
			},
			move(){
				// var eat=this.food.foods.some(e=>this.snake.body[0].x==e.x&&this.snake.body[0].y==e.y,this)
				// this.snake.move(!this.food.foods.some(e=>this.snake.body[0].x==e.x&&this.snake.body[0].y==e.y,this))
				var that=this
				function afterEat(){
					that.food.supFood(that.snake.body[0])
				}
				this.snake.move(this.food.foods,afterEat)
				if(this.snake.isDead()){
					console.log(this.snake.body[0])
					this.gameOver()
					return
				}
				this.drawAll()
				this.T=setTimeout(this.move,this.speed)
			},
			gameOver(){
				console.log('game voer!')
				// this.initGame()
			},
			keyDown(e){
				console.log(e)
			},
			startGame(){
				this.initGame()
			},
			stopGame(){
				
				clearTimeout(this.T)
			},
			changeSpeed(){
				this.speed=this.speed==1000?100:1000
			}
			
		}
	}
	function Coord(x,y){
		this.x=x;
		this.y=y
	}
	function Snake({initLength=4,
		initHead=new Coord(0,0),
		gameOver=function(){
			alert('game over')
		},
		w=32,
		h=32,
	}={}){
		this.L=20
		this.color='#000'
		this.direction='right'
		this.body=[];
		for(let i = 0;i<initLength;i++){
			this.body.unshift(new Coord(initHead.x+i,initHead.y))
		}
		// this.body.reverse()
		this.move=function(foods,fn){
			const oldHead=this.body[0]
			var dx=0,dy=0;
			switch (this.direction){
				case 'up':
				dx=0,dy=-1
				break;
				case 'down':
				dx=0,dy=1;
				break;
				case 'left':
				dx=-1,dy=0;
				break;
				case 'right':
				dx=1,dy=0;
				break;
			}
			this.body.unshift(new Coord(this.body[0].x+dx,this.body[0].y+dy))
			if(! foods.some(e=>this.body[0].x==e.x&&this.body[0].y==e.y,this)){
				this.body.pop()
				
			}else{
				fn()
			}
			// if(this.isDead()){
			// 	gameOver()
			// 	return
			// }
			// this.draw()
			// setTimeout(this.move,1000)
		}
		this.isDead=function(){
			const head=this.body[0];
			//碰到墙壁
			if(head.x<0||head.x>=w||head.y<0||head.y>=h){
				return true
			}
			//碰到自己
			var flag=this.body.some(function(e,i){
				if(i!==0&&head.x==e.x&&head.y==e.y){
					return true
				}
			})
			return flag
		}
		this.isEat=function(){
			return this.food.some(e=>this.body[0].x==e.x&&this.body[0].y==e.y,this)
		}
		// this.draw=function(){
		// 	// initGrids()
		// 	this.body.forEach(function(e,i){
		// 		ctx.fillStyle=this.color
		// 		ctx.fillRect(e.x*(this.L+1)+1,e.y*(this.L+1)+1,this.L,this.L)
		// 	},this)
		// }
		// this.draw()
	}
	
	function Food({
		num=1,
		w=32,
		h=32,
		snake
	}={}){
		this.foods=[]
		function genFood(){
			var food=new Coord(Math.floor(Math.random()*w),Math.floor(Math.random()*h))
			var flag=snake.body.some(e=>e.x==food.x&&e.y==food.y)&&arr.some(e=>e.x==food.x&&e.y==food.y)
			if(flag){
				return genFood()
			}else{
				return food
			}
		}
		this.initFood=function(){
			for(let i=0;i<num;i++){
				this.foods.push(genFood())
			}
		}
		this.initFood()
		this.supFood=function(){
			this.foods.push(genFood())
			this.foods.shift()
		}
		this.color='#ff0000'
	}
</script>

<style scoped="scoped">
	.rows{
		margin:0;
		padding:0;
		font-size: 0;
		white-space: nowrap;
	}
	.grid{
		display: inline-block;
		width:20px;
		height:20px;
		border: 1px solid #ccc;
	}
	#canvas{
		position: relative;
		/* background: #00e; */
	}
</style>
