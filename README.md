<!DOCTYPE html>

<html>
<head>
  <title>Run!</title>
</head>
<body>
  <canvas id="canvas" width="640" height="480"></canvas>
  <script>
    var canvas = document.getElementById("canvas");
    var ctx = canvas.getContext("2d");
    function floor(x, height) {
      this.x = x;
      this.width = 700;
      this.height = height;
    }
    var world = {
      height: 480,
      width: 640,
      gravity: 10,
      highestFloor: 240,
      speed: 5,
      distanceTravelled: 0,
	  maxSpeed: 40,
	  tilesPassed: 0,
      autoScroll: true,
      floorTiles: [
        new floor(0, 140)
      ],
      stop: function() {
        this.autoScroll = false;
      },
      moveFloor: function() {
        for(index in this.floorTiles) {
          var tile = this.floorTiles[index];
          tile.x -= this.speed;
          this.distanceTravelled += this.speed;
        }
      },
      addFutureTiles: function() {
        if(this.floorTiles.length >= 3) {
          return;
        }
        var previousTile = this.floorTiles[this.floorTiles.length - 1];
        var biggestJumpableHeight = previousTile.height + player.height *4.5;
		if (biggestJumpableHeight > this.highestFloor) {
		   biggestJumpableHeight = this.highestFloor;
		}   
		var randomHeight = Math.floor(Math.random() * biggestJumpableHeight) + player.height;
        var leftValue = (previousTile.x + previousTile.width);
        var next = new floor(leftValue, randomHeight);
        this.floorTiles.push(next);
      },
      cleanOldTiles: function() {
        for(index in this.floorTiles) {
          if(this.floorTiles[index].x <= -this.floorTiles[index].width) {
            this.floorTiles.splice(index, 1);
			this.tilesPassed++;
			if(this.tilesPassed % 3 == 0 && this.speed < this.maxSpeed) {
			  this.speed++;
			}
          }
        }
      },
      getDistanceToFloor: function(playerX) {
        for(index in this.floorTiles) {
          var tile = this.floorTiles[index];
          if(tile.x <= playerX && tile.x + tile.width >= playerX) {
            return tile.height;
          }
        }
        return -1;
      },
      tick: function() {
        if(!this.autoScroll) {
          return;
        }
        this.cleanOldTiles();
        this.addFutureTiles();
        this.moveFloor();
      },
      draw: function() {
        ctx.fillStyle = "black";
        ctx.fillRect (0, 0, this.width, this.height);
        for(index in this.floorTiles) {
          var tile = this.floorTiles[index];
          var y = world.height - tile.height;
          ctx.fillStyle = "blue";
          ctx.fillRect(tile.x, y, tile.width, tile.height);
        }
		ctx.fillStyle = "white";
		ctx.font = "30px Monaco";
		ctx.fillText("Speed: " + this.speed, 10, 40);
		ctx.fillText("Travelled: " + this.distanceTravelled, 10, 75);
      }
    };
    var player = {
      x: 160,
      y: 340,
      height: 20,
      width: 20, 
      downwardForce: world.gravity,
      jumpHeight: 0,
      getDistanceFor: function(x) {
        var platformBelow = world.getDistanceToFloor(x);
        return world.height - this.y - platformBelow;
      },
      applyGravity: function() {
        this.currentDistanceAboveGround = this.getDistanceFor(this.x);
        var rightHandSideDistance = this.getDistanceFor(this.x + this.width);
        if(this.currentDistanceAboveGround < 0 || rightHandSideDistance < 0) {
          world.stop();
        }
      },
      processGravity: function() {
        this.y += this.downwardForce;
        var floorHeight = world.getDistanceToFloor(this.x, this.width);
        var topYofPlatform = world.height - floorHeight;
        if(this.y > topYofPlatform) {
          this.y = topYofPlatform;
        }
        if(this.downwardForce < 0) {
          this.jumpHeight += (this.downwardForce * -1);
          if(this.jumpHeight >= player.height * 6) {
            this.downwardForce = world.gravity;
            this.jumpHeight = 0;
          }
        }
      },
      keyPress: function(keyInfo) {
        var floorHeight = world.getDistanceToFloor(this.x, this.width);
        var onTheFloor = floorHeight == (world.height - this.y);
        if(onTheFloor) {
          this.downwardForce = -8;
        }
      },
      tick: function() {
        this.processGravity();
        this.applyGravity();
      },
      draw: function() {
        ctx.fillStyle = "Snow";
        ctx.fillRect(player.x, player.y - player.height, this.height, this.width);
      }
    };
    window.addEventListener("keypress", function(keyInfo) { player.keyPress(keyInfo); }, false);
    function tick() {
      player.tick();
      world.tick();
      world.draw();
      player.draw();
      window.setTimeout("tick()", 1000/60);
    }
    tick();
  </script>
</body>
</html>
