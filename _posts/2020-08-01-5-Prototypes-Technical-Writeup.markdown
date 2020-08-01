---
layout: post
title:  "5 Prototypes Technical Writeup"
date:   2020-08-01 15:58:20 -0500
categories: Unity VR Project
---

<!--You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. You can rebuild the site in many different ways, but the most common way is to run `jekyll serve`, which launches a web server and auto-regenerates your site when a file is updated.

Jekyll requires blog post files to be named according to the following format:

`YEAR-MONTH-DAY-title.MARKUP`

Where `YEAR` is a four-digit number, `MONTH` and `DAY` are both two-digit numbers, and `MARKUP` is the file extension representing the format used in the file. After that, include the necessary front matter. Take a look at the source for this post to get an idea about how it works.

Jekyll also offers powerful support for code snippets:

{% highlight ruby %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
-->

In my last update I did a few primitive prototypes.  First a car-simulation; this is where I learned the Unity layout, what were components and gameobjects, and how to implement behaviors using C#.  I added two scripts, one to a camera and a vehicle. 

The camera script:
<a href="https://imgur.com/hs6iPob"><img src="https://i.imgur.com/hs6iPob.png" title="source: imgur.com" /></a>

Note:  I added '[SerializeField]' attribute which allows you to edit private values in the Unity Inspector.  LateUpdate() allows the camera to properly update it's position after the PlayerController script updates the car's position.

The script above updates the Camera's position(every frame this method is called) to the car's position plus a given offset.  It is important to note that position in 3D space is a Vector3 data type.

I then added a speedometer and an RPM guage which are TextMeshProUGUI objects.  These are children of the Canvas which represents the HUD.
<a href="https://imgur.com/kSqsdvz"><img src="https://i.imgur.com/kSqsdvz.png" title="source: imgur.com" /></a>

The following componenets are attached to our vehicle GameObject:  Player Controller script, RigidBody(for physics), and a Mesh Collider(also physics in this case).
<a href="https://imgur.com/gNpOaoQ"><img src="https://i.imgur.com/gNpOaoQ.png" title="source: imgur.com" /></a>

Note:  I assigned the Player Controller script manually a list of WheelCollider objects, the RPM and speedometer text objects from our canvas, and the Center of Mass empty GameObject I created as a child of the vehicle GameObject.  It's a child of the vehicle since we want it's position to be relative to the car's position.

Now what does the Player Controller script do?
<a href="https://imgur.com/mFk8B9V"><img src="https://i.imgur.com/mFk8B9V.png" title="source: imgur.com" /></a>

The Start() method will get a reference to the vehicle's RigidBody component and then assign the RigidBody's property "centerOfMass" the position of our centerOfMass empty GameObject.  All this code is executed once when the game starts.
Then I use FixedUpdate(), which is important to use as opposed to Update() since it is in-step/synced with the physics engine.  In this method, I get the horizontal and vertical inputs which are assigned in the Unity Properties as A,D and W,S.  Then, if the vehicle is grounded(all four wheels touching the plane), then I move the vehicle using force on the RigidBody and rotate using the transform.  When using transform, I use Time.deltaTime and when we use forces, we omit that in the calculation.  Lastly, we calculate the speed and RPM and use the SetText() method on the TextMeshProUGUI objects to update the HUD.

That basically sums up Prototype 1.

Prototype 2 is a top-down game where animals spawn and run forward in the player's direction.  The player can shoot projectiles at the animals which will destroy them.
<a href="https://imgur.com/QVbNwhz"><img src="https://i.imgur.com/QVbNwhz.png" title="source: imgur.com" /></a>

I first created and assigned a Player Controller script.  
<a href="https://imgur.com/QXMvEuC"><img src="https://i.imgur.com/QXMvEuC.png" title="source: imgur.com" /></a>

Variables are declared first of course.  Speed is assigned, we give a boundary range for left and right on the x-coord, and declare a variable for horizontal input and our projectile Prefab we create later.  Only the Update() method is used here.  It checks to see if our Player's position is within the set boundary.  Then it gets the horizontal input(A,D) and uses transform.Translate() to update the position accordingly.  Finally there is a check for spacebar input using Input.GetKeyDown().  In that case, the script utilizes an object pooler which will get an object from the pool, activate it, and then set it's position to the player's position.


This prototype introduced me to prefabs, which is where I can use a GameObject as a template to create new instances whenever I'd like and the Prefab would remember which components were added and data values I've assigned.
<a href="https://imgur.com/LfTM5KO"><img src="https://i.imgur.com/LfTM5KO.png" title="source: imgur.com" /></a>

I created 4 prefabs.  One for each animal and the projectile.  They all have colliders and are set as triggers since we are not using them for physics.  The animals each have the "Move Forward", "Detect Collisions", and the "Destroy Out of Bounds" scripts.  The projectile has the same but without the "Detect Collisions".

For the sake of time, I'll just explain that the Move Forward script simply uses the Update() method with a given speed value using Translate.  The Detect Collision script uses OnTriggerEnter() which then uses SetActive(false) to hide the projectile, and then the animal is destroyed with Destroy().  The Destroy out of bounds script has a topBound and lowerBound variable and the Update() checks if the object's position is outside these values and will use SetActive() or Destroy() based on that.  

Later on, I added an ObjectPooler script to optimize performance.  
<a href="https://imgur.com/svE0emf"><img src="https://i.imgur.com/svE0emf.png" title="source: imgur.com" /></a>

This script inlcudes the Awake() method which is called before any Update() method.  This ensures other scripts that depend on this one are initialized properly.  Here, we are assigning a self reference of the ObjectPooler class to it's own public member.  In the Start() method we initialize our list of objects to pool.  Each object is then hidden by using SetActive(false) and added as a child to the Spawn Manager empty that was created in our hierarchy.  GetPooledObject() returns a reference to a GameObject in the pool that isn't active.

This summarizes the functionality I implemented in prototype 2.

In prototype 3, I created a side-scroller that used a particle effect, sound, and animations.
<a href="https://imgur.com/H4rSJaQ"><img src="https://i.imgur.com/H4rSJaQ.png" title="source: imgur.com" /></a>

There is 1 Prefab created and 4 scripts that are created.  The Prefab is an Obstacle and the scripts are MoveLeft, PlayerController, RepeatBackGround, and SpawnManager.  The MoveLeft script is applied to the Prefab and the Background object in our scene.  This gives the illusion to the player that they are moving in the scene, but the player is actually stationary.  

<a href="https://imgur.com/RB4bfoB"><img src="https://i.imgur.com/RB4bfoB.png" title="source: imgur.com" /></a>

The PlayerController script is applied to our player and does a number of things.  In Start(), we get references to the player object's Rigidbody, Animator, and AudioSource components.  In Update(), we check if the Spacebar is pressed, the player is on the ground, and that the game is not over.  In which case, we apply a force to the Rigidbody(up in this case), play our jump animation, toggle the particle effect, and then lastly play our jump sound.  The OnCollisionEnter() is used to detect when our Collider is colliding with either the ground or the obstacle using collision.gameObject.CompareTag().  In the case we collide with the ground, we toggle the dirt particle effect.  In the case that we collide with the obstacle, we play a death animation by modifying a couple parameters to cause the animation controller to transition to the state we want.  This is all done through the Animator component reference we have.  An explosion particle effect is played, and a crash sound is played.

RepeatBackground script takes half the width of our BoxCollider size on the x-axis, and uses that to determine when to repeat the background by resetting the position.

The SpawnManager spawns our Obstacles using InvokeRepeating() in our Start() method.  We create a method that is invoked by this which checks if the game is over, and if not we instantiate a new Obstable Prefab GameObject using Instantiate().

<a href="https://imgur.com/WIGr3sD"><img src="https://i.imgur.com/WIGr3sD.png" title="source: imgur.com" /></a>

The animation controller is setup on the player by setting a run animation as the default movement state.  We also update the speed value parameter to 1.0.  The other transitions happen with the scripts.

That concludes Prototype 3!

Prototype 4 is a game where the player is a ball and has to stay on the platform.  Enemy balls try to knock the player off the platform, but the player is given powerups that can help them.
<a href="https://imgur.com/W3Kdxlc"><img src="https://i.imgur.com/W3Kdxlc.png" title="source: imgur.com" /></a>

There are two Prefabs created which are the enemy and powerup.  There are 4 scripts which I created:  the Enemy, PlayerController, RotateCamera, and SpawnManager scripts.

I'm going to skip all the repeated details of what I've done here.  The PlayerController uses a co-routine for the powerup countdown.  We check if the powerup was picked up by using OnTriggerEnter() since the powerup Prefab has a collider setup as a trigger.  In the case that it was(check the tag for "powerup"), we show the powerup indicator, destroy the powerup gameobject prefab, and then use StartCoroutine() to start the powerup countdown.
<a href="https://imgur.com/Lx0WN3O"><img src="https://i.imgur.com/Lx0WN3O.png" title="source: imgur.com" /></a>

Finally, we get to Prototype 5.  
<a href="https://imgur.com/4zDnGSI"><img src="https://i.imgur.com/4zDnGSI.png" title="source: imgur.com" /></a>

This prototype is a point and click type game where the player needs to click the right objects and score points.  I learned how to create a user interface using button and text objects.  I created 4 prefabs here, one for each target.  Three scripts were created:  DifficultyButton, GameManager, and Target scripts.

Once again, skipping repetitive details.  The GameManager tracks the state that the game is in, and runs the coroutine where the main gameplay loop takes place.  Scripts communicate with each other by using references.


Thanks for reading!