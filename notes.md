# 07  CAMERA 
## CAMERA 
* Abstract class 
* Not supposed to use it directly 
* Have to inherit from it to have access to common properties and methods 
* Following classes inherit from the `Camera` class
### ArrayCamera
* Used to render scene multiple times by using multiple cameras 
* Each camera renders a specific area of the canvas 
### StereoCamera
* Used to render the scene through two cameras that mimic the eyes to create a parallax effect that tricks the brain into thinking that there is depth
* Need to have VR headset or red and blue glasses to see the result 
### CubeCamera
* Used to get a render facing each direction (forward, backward, leftward, rightward, upward, and downward) 
* Creates a render of the surrounding
* Can use to create an environment map for reflection or a shadow map
### OrthographicCamera
* Used to create orthographic renders of the scene without perspective 
* Useful for making RTS games like Age of Empire
* Elements will have the same size on the screen regardless of their distance from the camera
### PerspectiveCamera
* Used to simulate a real-life camera with perspective 
## PerspectiveCamera
* needs parameters to be instantiated 
* Adding the third and fourth parameters 
    ```
    const camera = new THREE.PerspectiveCamera(75, sizes.width / sizes.height, 1, 100)
    ```
### FIELD OF VIEW (First Parameter)
* Corresponds to camera views vertical amplitude angle in degrees 
* Using small angle results in a long scope effect 
* Using wide-angle results in a fish eye effect 
* This is due to what the camera sees being stretched of squeezed to fit the canvas
* Field of view between 45 and 75 is usually best 
### ASPECT RATIO (Second Parameter)
* Corresponds to the width divided by the height 
* Usually cavas width/height but not always the case when using Three.js is very specific ways
* Recommending saving values in an object as they're used multiple times
    ```
    const sizes = {
        width: 800,
        height: 600
    }
    ```
### NEAR AND FAR (Third and Fourth Parameters)
* Corresponds to how close and how far the camera can see
* Any object or part of the object closer to the camera than the `near` value of further away from the camera that the `far` value won't show on render
* Using very small or large values like `0.0001` and `999999` you might get a bug called z-fighting where two faces seem to fight for which one will be rendered above the other
* Try to use reasonable values and increase those only if you need it
    * change `near` and `far` parameters to `0.1` and `100`
        ```
        const camera = new THREE.PerspectiveCamera(75, sizes.width / sizes.height, 1, 100)
        ```
## OrthographicCamera 
* Can be useful for specfic projects
* Differs from PerspectiveCamera by lack of perspective (objects will have the same size regardless of distance from camera)
* Parameters very different from PerspectiveCamera
* Need to provide how far the camera can see in each direction (`left`, `right`, `top`, and `bottom`) 
* Then provide `near` and `far`
    * Comment `PerspectiveCamera` and add `OrthographicCamera`
    * keep `position` update and `lookAt(...)`

        ```
        const camera = new THREE.OrthographicCamera(- 1, 1, 1, - 1, 0.1, 100)
        ```
        * can see that there is no perspective and all sides of the cube seem parallel 
        * cube doesn't look cubic
        * a square area was rendered but it is being stretched to fit the rectangle canvas
    * use canvas ratio (width by height) 
    * create variable `aspectRatio` and store ratio in it
        ```
        const aspectRatio = sizes.width / sizes.height
        const camera = new THREE.OrthographicCamera(- 1 * aspectRatio, 1 * aspectRatio, 1, - 1, 0.1, 100)
        ```
        * this results in render area width larger than the render area height b/c canvas width is larger than it's height
## CUSTOM CONTROLS 
* Go back to `PerspectiveCamera`:
    * Comment the `OrthographicCamera`
    * uncomment the `PerspectiveCamera`
    * move `camera` so it faces the cube
    * remove mesh roation in the `tick` function 
        ```
        // Camera
        const camera = new THREE.PerspectiveCamera(75, sizes.width / sizes.height, 1, 1000)

        // const aspectRatio = sizes.width / sizes.height
        // const camera = new THREE.OrthographicCamera(- 1 * aspectRatio, 1 * aspectRatio, 1, - 1, 0.1, 100)

        // camera.position.x = 2
        // camera.position.y = 2
        camera.position.z = 3
        camera.lookAt(mesh.position)
        scene.add(camera)
        ```
* Control camera with the mouse:
    * get mouse coordinates using js by listening to the `mousemove` event with `addEventListener`
    * coordinates located in the argument of the callback function as `event.clientX` and `event.clientY`
        ```
        // Cursor
        window.addEventListener('mousemove', (event) =>
        {
            console.log(event.clientX, event.clientY)
        })
        ```
        * values are in pixels
    * better to use clean values and have amplitude of 1
    * create `cursor` variable with default `x` and `y` properties and update those properties in the `mousemove` callback:
        ```
        // Cursor
        const cursor = {
            x: 0,
            y: 0
        }

        window.addEventListener('mousemove', (event) =>
        {
            cursor.x = event.clientX / sizes.width - 0.5
            cursor.y = event.clientY / sizes.height - 0.5

            console.log(cursor.x, cursor.y)
        })
        ```
    * update position of the camera in the `tick` function:
        ```
        const tick = () =>
        {
            // ...

            // Update camera
            camera.position.x = cursor.x
            camera.position.y = cursor.y

            // ...
        }
        ```
        * axes movements seem wrong b/c `position.y` is positive when going upwards in Three,js but the `clientY` axis is positive when going downward on page
    * invert `cursorY` by adding `-` in front of the whole formula 
        ```
        window.addEventListener('mousemove', (event) =>
        {
            cursor.x = event.clientX / sizes.width - 0.5
            cursor.y = - (event.clientY / sizes.height - 0.5)
        })
        ```
    * increase amplitude by multiplying `cursor.x` and `cursor.y` and ask camera to look at the mesh using `lookAt(...)` method:
        ```
        const tick = () =>
        {
            // ...

            // Update camera
            camera.position.x = cursor.x * 5
            camera.position.y = cursor.y * 5
            camera.lookAt(mesh.position)

            // ...
        }
        ```
    * do full rotation of camera around the mesh using `Math.sin(...)` and `Math.cos(...)` 
    * to do full roation, angle must have an amplitude of 2 * pi (use `Math.PI`)
    * to increase radius of circle, multiply results of `Math.sin(...)` and `Math.cos(...)`
        ```
        const tick = () =>
        {
            // ...

            // Update camera
            camera.position.x = Math.sin(cursor.x * Math.PI * 2) * 2
            camera.position.z = Math.cos(cursor.x * Math.PI * 2) * 2
            camera.position.y = cursor.y * 3
            camera.lookAt(mesh.position)

            // ...
        }

        tick()
        ```
## BUILT IN CONTROLS   
* Type in "controls" in Three.js docs, there is lots of pre-made controls 
### DeviceOrientationControls 
* automatically retrieves the orientation of your device, if your device, os, and browser allow it and rotates the camera accordingly
* can be used to create immersive universes or VR experiences if you have the right equipment
### FlyControl
* enables moving the camera as if you were of a spaceship
* can rotate on all 3 axis, go forard and backward
### FirstPersonControls
* similar to `FlyContols` but with a fixed axis 
* can see the view of a flying bird, but that bird can't do a barrel roll 
* not the same FirstPerson controls in FPS games
### PointerLockControls
* uses the `pointer lock JavaScript API` 
* this API hides the cursor, keeps in centered, and keeps sending the movements in the `mousemove` event callback
* can create FPS games right in the browser
* it only handles the camera roation when the pointer is locked 
* have the handle the camea position and game physics by yourself 
### OrbitControls 
* rotate around a point with the left mouse
* translates laterally using the right mouse, and zoom in or out using the wheel 
### TrackballControls 
* similar to `Orbitcontrols` but there are no limits in terms of vertical angle 
* can keep rotating and do spins with the camera even if the scene goes upside down 
### TransformControl 
* has nothing to do with the camera
* use it to move objects on a plane facing the camera by dragging and dropping them 

## OrbitControls
* comment out `update camera` in `tick` function
### INSTANTIATING 
* instantiate a variable using `OrbitControl` class 
    * not available by default in the `THREE` variable 
    * this helps reduce weight of the library
    * where Vite template comes in
* import `OrbitControls` from dependencies folder
    * provide path from inside the `node_modules` folder 
    ```
    import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'
    ```
* create camera and the element in the page that will handle the mouse events as parameters 
    ```
    // Controls
    const controls = new OrbitControls(camera, canvas)
    ```
    * can now drag and drop using both the left or right mouse and scroll to zoom
### TARGET
* by default camera is looking at the center of scene, can change with the `target` property 
* property in `Vector3` so can change its `x`, `y`, and `z` properties
* get `OrbitControls` to look above cube by default, increase `y` property and tell `OrbitControls` to update itself
    ```
    controls.target.y = 2
    controls.update()
    ```
* not useful in our case so comment out
### DAMPING
* this smooths the animation by adding some kind of accelerating and friction formula
* to enable damping, switch the `enableDamping` property of `controls` to `true`
* to work properly the controls need to be updated on each frame by calling `controls.update()` do this in `tick` function 
    ```
    // Controls
    const controls = new OrbitControls(camera, canvas)
    controls.enableDamping = true

    // ...

    const tick = () =>
    {
        // ...

        // Update controls
        controls.update()

        // ...
    }
    ```
* can use many other methods and properties to customize your controls such as the rotation speed, zoom speed, zoom limit, angle limit, damping strength, and key bindings

### WHEN TO USE BUILT IN CONTROLS
* if you rely on them too much you might have to change how the class in working in unexpected ways
* list all the features you need from these controls
* then check if the class you're going to use can handle all those features
* if not, you'll have to do it on your own



