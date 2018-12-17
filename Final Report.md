#Final Report

Course| ART 3504 TS: Augmented and Virtual Reality
-|:-:
Instructor| Dr. Wallace S. Lages
Project name | ***Ninja Quest***  ~~*(Redirected Walking Algorithm)*~~
Reporters/Members| Nanlin Sun



---
###Overview
This VR game generates random paths connected by teleportation doors and elevators in the air. The player will need to find their way through those paths and collect certain mount of coins existing on the way in order to unlock the final destination.

Notice:
This project is originally about the generation algorithm that can redirect the players walking inside the limited room. The algorithm will using built-in game elements to generate dynamic game level that the player will never be out of reach in real world. 

The original project is now part of the research: ***Redirected Walking Algorithm***. And the current project is now about the development of the VR game: ***Ninja Quest*** which is a game built with the ***Redirected Walking Algorithm***. The ***Redirected Walking Algorithm*** is designed to improve the players' locomotion VR game experience but it is not necessary for the current project.

---
####Similar Project

* ***Ninja Quest***
The Super Mario Class project is similar to current project but it does not has dynamic redirected walking algorithm to generate dynamic level. 

* ***Redirected Walking Algorithm***
A Japanese firm has designed a unlimited corridor which redirect player walking along a curved wall. The project is designed to redirect players through physical objects which is different from current project.

---
###Design

The idea came from the class project that each classmate has to design a game element. This class project had not yet completed and I found it quite interesting to continue. The most interesting part of this design is that the game is of different game objects such as elevators, teleportation doors and more that the player can interact with to complete the game. And each game object has different functions, the teleportation door, that can transport player from one spot to another in one frame, the elevator that is pretty much like the real word elevator that can lift player up vertically. The possibility of the game is endless as more game objects can be added into the game.

* ##### Elevator
Elevator is pretty much like the real word elevator. It is designed to lift player up vertically. The reason behind this design is because we have multiple paths vertically aligned to each other and in order to let player to get from the lower path to the higher path, the player need to take the elevator to get to the higher path. 

The player needs to step into the elevator station, and a progress bar will appear on the floor and the value of the progress bar will increase. If player leaves the elevator station, the progress will cleared and the progress bar will disappear. If the progress bar value has reached its maximum value, a platform will be generated on the players' foot's location and will start moving upward and lift the player up vertically, and will eventually stop at its destination. If the player leave the elevator, the elevator will be disappear.

The older version of the elevator does not require player to stand on the station and wait the progress bar to charge. It generate the elevator instantly when the player step on it and a cool down will be applied till next time of using it. The problem of this design is that the if the player happen to fall from the higher paths to it, it will instantly elevate player up which is not the original purpose of the player's action. The player wants to fall down to the lower path rather than taking the elevator up. The idea of comedown after triggered was then changed into the charging to trigger as it is now.

The elevator station is consist of a charing station, which has a trigger on it. It handles the charging task through OnTriggerEnter(), OnTriggerStay(), OnTriggerExit(). A new elevator gameobject will be instantiated after the the progress bar on the elevator station has been charged. The elevator is a object with only a trigger, and it makes the VR camera as its child when the trigger collides with the VR camera, and it removes the VR camera as the VR camera exits the trigger. The elevator also destroys itself when the VR camera leaves the trigger. 

* ##### Falling Spot
Falling Spot is the place on one path that the player can fall off the current path to another path. Every Falling spot occurs on the higher paths between two vertically aligned paths. You can also consider it as the reverse travel method of the elevator. 

The player needs to step one step out of the falling spot in order to fall off the current path to another path.

Notice: The player can always step out the current path and fall down on the ground, but it is not consider to be a valid falling. A valid falling is considered to falling from the falling spot to another path only.

The falling spot exists when two path vertically aligned have different path pattern, and it exists at the location where it connects the sub path that has same path pattern and the sub path that has different path pattern.

![image3](https://i.imgur.com/ZLL7pBS.png)

* ##### Teleportation Doors
The teleportation doors are always pairs. They can transfer players from one end to the other in one frame. The teleportation doors are two-way transportation tools. They are  designed to let players to travel through different paths that are not vertically aligned. They are also the major travel option in game.

The player needs to step into the teleportation circle, and a progress bar will appear on the floor and the value of the progress bar will increase. If player leaves the teleportation circle, the progress will cleared and the progress bar will disappear. If the progress bar value has reached its maximum value, the player will be transferred to another teleportation door location which is connected to current one. If the player leave the elevator, the elevator will be disappear.

![image11](https://i.imgur.com/CR1AM2E.gif)

There are two old versions of the teleportation doors. The first one has the shape of door, and the second one has the shape of flag. The door shape design was deprecated because the large number of doors existing in the world block too much vision of the player. The flag shape design was deprecated because it is hard for player to understand its meaning. (it has poor representation of the teleportation door).

The teleportation door is consist of a charing station, which has a trigger on it. It handles the charging task through OnTriggerEnter(), OnTriggerStay(), OnTriggerExit(). The VR camera will be relocated after the the progress bar on the teleportation door has been charged. The Parent of the VR camera has a function called move2(float), that will move the VR camera to any location.

```CS{class="line-numbers"}
    public void Move2(Vector3 destination)
    {
        Vector3 displacement = destination - VRcam.position;
        Move(displacement);
    }

    public void Move(Vector3 displacement)
    {
        transform.position += displacement;
    }
```

* ##### Teleportation Indicator

The teleportation indicator is the indicator that helps players to identify the other teleportation door that is connected to the current one. There are many pairs of teleportation doors in the game world, without this design the player has no idea where he is going to. The design of indicator is critical to the game object teleportation doors. 


When player approach one teleportation door, it will trigger. A big arrow indicator will show up in the world pointing to the other teleportation door. And when the player goes away, the indicator will disappear.

![image12](https://i.imgur.com/c0NmieH.gif)

The older version of the teleportation indicator is the flag instead of the arrow. The flag will appear when the player approach it, and disappear when goes away. It serves the same purpose of helping players identify the location of the other end of the teleportation doors but it blocks the vision of the player and thus had been deprecated.


* ##### Footstep Indicator

Footstep is the indicator that will help player to identify its current foot location and facing direction. Since the game has no real body representation in the game, the player can see nothing on the floor, and sometime it is easy for them to fall off the path by accident.

The player can always see the footstep indicator below him, it moves with player and rotate with player.

It shoots a ray from the VR camera position vertically down and hit the first object. The footstep indicator has its dynamic location from the position of the hit point. The ray is cast in only "Floor" layer that only contains object that can be walked by player. The layer is obtained through LayerMask by bitwise operation.

```CS{class="line-numbers"}
void Update () {
    RaycastHit hit;
    LayerMask layer = 1<<8;
    if (Physics.Raycast(transform.position, -Vector3.up, out hit, float.PositiveInfinity,layer) )
    {
        target = hit.collider.tag;
        offsetDistance = hit.distance;
        CamRig.instance.RaycastPosition = hit.point;
        Debug.DrawLine(transform.position, hit.point, Color.cyan);
    }
}
```


* ##### Coin

The coin is designed to increase players' gaming experience. It is a motivation that drives the player to go through as many paths as possible, collecting coins as they go through each path. The coin currently only servers one purpose, which is to unlock the final destination. The player has to collect certain mount of coins to unlock the last teleportation door which leads to the final island.

The coin has a loop animation of rotation.

---
###Evaluation

* Script:
1. ask the player to adjust the HMD and put it on.
2. ensure they are ready and start the game.
3. ask the player do not more and stand still, ask them to follow the UI instruction for the height recording purpose.
4. ask the player to turn around and walk to that door and start the exploring the game as they want. And let he know he only have one task which is to collect as many coins as they can.

5. (optional, if he get stuck and want some help) ask the player to get closer to the flag, the red and the blue flag. 

6. (optional, if he get stuck and want some help) ask the to look to the lower platform and ask them to try to fall to that lower platform.

7. (optional, if he get stuck and want some help) ask the player to get closer to elevator station.

8. ask the player to take off his HMD after 3 minutes. (time limit)

* Instruction Questions:
1. "Are you comfortable with the head set?"
2. "Can you see the game scene clearly?"
3. "Do you feel dizzy?"
4. "If you can see a UI panel showed in front of you, please follow its instruction and stand still."
5. "Please feel free to start exploring the game, if you need any help or instruction, please ask for help."
6. "Your thinkings are valuable to the development of the game. Please speak out your thoughts loud at anytime."


* Interview Questions:

1. "What is the most interesting part of the game so far?"
2. "Do you have any difficulty with figuring out the purpose of certain game objects or the ?"
3. "What other suggestion you would love to share to make it a better game?"


* Results of analysis of the data
1. It seems Participant have high rate of having problems of understanding what to do and what they can do. 
2. It also seems that there need more game elements to be added to the game to make the game more challenging and more fun. 
3. Some players also want the game to be easy rather than difficult. 
4. The most urgent part is to add the a meaningful purpose to the coin system, what the player can do with coins? give more motivation to drive them collecting coins.
5. The tutorial might be a good idea to solve the problem one. (important!)
6. Add more art, add more grass or environment to the game make it more friendly and immersive.


* Improvement
1. As requested by the user, the tutorial level is current under development and it is going to guide player through the whole game and each game object.
2. The game is current under development on *Mirage Solo with Daydream* with Google VR and will launched on google play store later. So the game will no longer running on HTC vive only, gives more portability to the VR headset.
3. The art of the game has been evolved a lot. The game has now has the Japanese Low poly style scene. And has a better story. 

---
###Limitation

The main limitation of the game is the room size. If the player only has a small room size, the generation of the path in the level will be very tedious and there does not seem to be a way to solve this issue.

---
### Future work

* ###### Google VR
It requires more time to make the game full functional with Google VR and later updates.

* ###### Monsters and traps
Add monsters and traps into the game. Make the game more difficult and interesting by adding some difficulty to the game.

* ###### Controllers
Let players to be able to play the game with the controllers as well. Add weapons to players that allow them to use and kill monsters

* ###### Story
Add more story and content to the background of the game, make the ninja theme more interesting.

* ###### More Gameobject
Add more objects like slides, to let player have more option to travel through paths.

* ###### Redirected Walk Algorithm

Work more on the algorithm to make the game more interesting and more dynamic.

---
###Reference work

* ###### Citation

“Walking in Virtual Reality Redirected Walking Unlimited Corridor Trailer,” IamVR, Aug. 2016. http://www.iamvr.co/walking-virtual-reality-redirected-walking/

* ###### Related Asset for this project:

[Pooly - Professional Pooling System](https://assetstore.unity.com/packages/tools/utilities/pooly-professional-pooling-system-82941)
[POLYGON MINI - Fantasy Character Pack](https://assetstore.unity.com/packages/3d/characters/humanoids/polygon-mini-fantasy-character-pack-122084)
[POLYGON - Samurai Pack](https://assetstore.unity.com/packages/3d/environments/polygon-samurai-pack-89551)
[Cube World Town](https://assetstore.unity.com/packages/3d/environments/fantasy/cube-world-town-121923)
[Low Poly Rocks Pack](https://assetstore.unity.com/packages/3d/environments/low-poly-rocks-pack-70164)
[Health and Progress Bars Pro](https://assetstore.unity.com/packages/tools/gui/health-and-progress-bars-pro-82405)
[Floating grass platforms](https://assetstore.unity.com/packages/3d/props/exterior/floating-grass-platforms-72701)
[Simple Sky - Cartoon assets](https://assetstore.unity.com/packages/3d/simple-sky-cartoon-assets-42373)

