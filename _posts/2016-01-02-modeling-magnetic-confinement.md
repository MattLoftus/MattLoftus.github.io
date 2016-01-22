---
layout: post
title: Modeling Magnetic Confinement with JavaScript and D3.js
---

Particle systems are an amazing way to visualize and model important natural phenomena. They give us an experience that provides a different kind of value than simply staring at equations and reading results in an excel document.   The value they give us is tangible, and from observing these systems visually, we can understand the subject at hand  in an intuitive way.

Just below are a couple videos I added to YouTube showing the results of running a couple different variations of the code we'll get into in this post.  If you're not in the mood for a long article, I'd suggest just watching the videos to get an idea of what I mean by a particle system.

[Magnetic Confinement Simulation](https://youtu.be/1fnK0qC0GBU)
[Big Bang Simulation (medium speed)](https://youtu.be/uxJ7BiQ07PE)
[Big Bang Simulation (slow motion)](https://youtu.be/wlInWXpTCAk)

A **particle system** is a simulation in which elements under observation interact with each other and their surroundings according to explicitly defined physical laws.  Think of the planets revolving around the solar system, viruses interacting with and destroying healthy cells.

For each of these we know the exact set of physical laws we will apply to the system.  In other words, this type of system is really a simulation.  We define a set of equations, build a certain number of particles that adhere to these equations, and set them out into the wild (or the current browser tab).  Then we wait and see how the particles interact with each other. 

In this post, I want to talk about one specific implementation of a particle system I made with a partner using JavaScript and D3.js, an impressive library for efficiently manipulating DOM elements based different data sets. In this project, we used the fundamental equations of electrodynamics to build a system of protons and electrons.

To achieve this, we created a set of equations in our main JavaScript file to handle vector operations and force calculations in a 2-dimensional space.  Below I'll go give a brief description of each step of the process.  

**Vector Operations** 

```language(javascript)
////////////////////////////////
/////VECTOR OPERATIONS//////////
////////////////////////////////

//Subtract vectors
function subVectors (vec1, vec2) {
  return [vec1[X] - vec2[X], vec1[Y] - vec2[Y]];
}

//Add 2 vectors
function addVectors (vec1, vec2) {
  return [vec1[X] + vec2[X], vec1[Y] + vec2[Y]];
}

//Scalar multiplication of a vector
function multVector (vec1, multiplier) {
  return [vec1[X] * multiplier, vec1[Y] * multiplier];
}
```
Vectors are useful when you are working with something that has both magnitude and direction.  When we are calculating the velocity, force, and acceleration of different particles, we must keep track of not only the magnitude of these quantities but also the directions, so we can determine how these particles will interact with the other particles in the system.  We will use the vector helper functions in our physics calculations below.

**Force and acceleration functions**

```language(javascript)
/////////////////////////////////
/////PHYSICS EQUATIONS///////////
/////////////////////////////////

//EM force equation: Coulombs Law. Finding 
//the force one charged particle exerts on 
//another. q1, q2 are their respective charges.
//r is the distance between them, and k is 
//Coulomb's constant
function calcForce(q1, q2, r) {
  return k * q1 * q2 / (r * r);
}

//Finding the acceleration. 
//Newton's 2nd Law: F = m * a
function calcAcceleration(force, mass) {
  return force / mass;
}

//Find a new velocity based off of current velocity, 
//current acceleration, and a fixed time interval which 
//is equal to the iteration interval speed of our system loop
function calcVelocity(velocity, acceleration) {
  var deltaV = multVector(acceleration, animSpeed);
  return addVectors(velocity, deltaV);
}

//Find a new position for a particle based off of current position 
//and current velocity and the same time interval as above
function calcPosition(position, velocity) {
  var deltaP = multVector(velocity, animSpeed);
  return addVectors(position, deltaP);
}

//Pythagorean theorem! Finding the distance between two given 
//particles. We will use the result of this as r when we call 
//the Coulomb force equation above
function calcDistance(pos1, pos2) {
  return Math.sqrt(Math.pow(pos1[X]-pos2[X], 2) + Math.pow(pos1[Y]-pos2[Y], 2))
}
```
All the above equations will be used to calculate the new position of a particle based of its current position, velocity, and the force exerted on it by every other particle in the system.  With our main functions in place we can now think about making particles and putting our functions to work.

**Particle generating functions being used in the current configuration**

```language(javascript)
//A function to create an object with all the properties for making a
//circle in html. The arguments are initial position, initial velocity, 
//initial acceleration, mass, charge, radius, whether they should be 
//static or dynamic, and a color. In short, we can use this to make 
//protons, electrons, or any other particle.
function genParticle(pos, v, a , m, q, r, dynamic, color) {
  var isNegative = Math.random() > 0.5;
  pos = pos || [Math.random()*width, Math.random()*height];
  v = v || [0,0];
  a = a || [0,0];
  m = m || 1;
  q = q || (isNegative ? -1 : 1);
  r = r || 10;
  dynamic = dynamic === false ? false : true;
  color = color || (q < 0 ? "#4782E8" : "#F54836");

  var p = {
    pos : pos,
    v : v,
    a : a,
    m : m,
    q : q,
    r : r,
    color : color,
    dynamic : dynamic, 
    id : uuid
  }
  uuid++;
  particles.push(p);
}

//A function that can be used to generate a wall of any given width.
//This wall is typically static, meaning its position in the system is 
//fixed. We specify the charge each point of the wall has explicitly
function makeAWall(startPos, endPos, num) {
  var dPos = subVectors(endPos, startPos);
  var dY = dPos[Y]/num;
  var dX = dPos[X]/num;
  var x = startPos[X];
  var y = startPos[Y];
  for (var i = 0; i < num; i++) {
                //pos   v      a      m   q   r   dynam  color
    genParticle([x, y], [0,0], [0,0], 1, 500, 10, false, "#F54836");
    x += dX;
    y += dY;
  }
}

//This function can be repurposed a number of different ways.
//In its current implementation it used the genParticle function above
//to create 100 particles a randomized intial position and a fixed
//inital velocity. In this case we are making 100 protons
function injector() {
  for (var i = 0; i < 100; i++) {
    genParticle([i*Math.random()*10, 400],[1,0.1], [0,0], 0, 1, 10, true, "#F54836");
  }
}
```
Above we have three main functions.  The first is a general function for making any type of charged particle we want.  The second and third functions are use case specific.  In our current implementation, we are simulating the magnetic confinement of charged particles.  This might be seen in a particle accelerator, or an experimental nuclear fusion reactor, where you have walls or tubes with an extremely strong charge, exerting a large repulsive force from all directions on a narrow beam of particles in the center of the tube.  As a result, the beam of particles is compressed,and often accelerated to tremendous speeds. With this in mind, we see how the makeAWall function can be used to recreate a basic version of one of these repulsive walls.  Our injector function, on the other hand, is what creates our particle beam.

**Main System Logic/Loop**

```language(javascript)
//Use D3 to create our particles in the DOM. Make two positively charged 
//walls above and below where the other particles will be injected. Invoke 
//injector then run renderParticles helper function to render particles 
//based on their current properties
function startSystem() {
  particleSvg = d3.select('body')
                  .append("svg")
                  .attr("width", width)
                  .attr("height", height);
  makeAWall([width * -2, height * 0.25], [width*2, height * 0.25], 200);
  makeAWall([width * -2, height * 0.55], [width*2, height * 0.55], 200);
  injector();
  renderParticles();
}

//This function first starts the system. Then performs the updateParticles 
//helper to get the new acceleration, position, and velocity of each 
//particle based on its interaction with every other particle in the 
//system. This is KEY. After updating the particles' properties we 
//re-render them on the screen. In the current configuration, we perform 
//this update once every 10 ms.  
function systemLoop() {
  startSystem();
  setInterval(function() {
    updateParticles();
    renderParticles();
  }, animSpeed);
}
```
Above is the heart of what makes the particle system run. We initialize our system by creating two positively charged walls, then inject 100 particles in the space between the walls.  The most crucial aspect of this part is what happens inside of the setInterval.  Every time the updateParticles (I've omitted the source for that for the sake of brevity) function gets invoked, every particle that is not static (meaning everything but the walls) has a force exerted on it by every other particle in the system (including the walls).  In the current implementation, we have 500 total particles.  That means for each of the 100 particles made by the injector, we must calculate the influence of the other 499 particles in the system.  That means for one iteration of our game loop we must perform roughly 50,000 operations.  Moreover, this game loop runs on an interval based off of the variable animSpeed, which in this implementation is 10 milliseconds.  That means that every second, this system is performing over 5,000,000 operations.

That in itself is simply amazing.  And what is even more amazing is that it runs in the browser... on a laptop.  This is something that just a decade ago would have seemed unbelievable.  But the fact of the matter is that we can perform meaningful physical simulations with a browser and a software language made for web development.

If you take some time to look at the code, you will probably see how straightforward it would be to modify the code to create a totally different simulation.  (Like the big bang example videos at the top).  We simply change some of the initial conditions, build a couple of new helper functions, and boom! We get a brand new model.  

I hope you've enjoyed reading. Please feel free to contact me with any questions or critiques.