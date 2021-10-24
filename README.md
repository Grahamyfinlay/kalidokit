# KalidoKit - Face, Pose, and Hand Tracking Kinematics

Kalidokit is a blendshape and kinematics solver for Mediapipe/Tensorflow.js face, eyes, pose, and hand tracking models, compatible with [Facemesh](https://github.com/tensorflow/tfjs-models/tree/master/face-landmarks-detection), [Blazepose](https://github.com/tensorflow/tfjs-models/tree/master/pose-detection), [Handpose](https://google.github.io/mediapipe/solutions/hands.html), and [Holistic](https://google.github.io/mediapipe/solutions/holistic.html). It takes predicted 3D landmarks and calculates simple euler rotations and blendshape face values.

As the core to Vtuber web apps, [Kalidoface](https://kalidoface.com) and [Kalidoface 3D](https://3d.kalidoface.com), KalidoKit is designed specifically for rigging 3D VRM models and Live2D avatars!

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/B0B75DIY1)

![Kalidoface Face Tracking](https://cdn.glitch.me/29e07830-2317-4b15-a044-135e73c7f840%2Fkalidoface-face-closeup.gif?v=1633451401758) ![Kalidoface Pose Demo](https://cdn.glitch.me/29e07830-2317-4b15-a044-135e73c7f840%2Fkalidoface-pose-dance.gif?v=1633453098775)

## Install

#### Via NPM

```
npm install kalidokit
```

```js
import * as Kalidokit from "kalidokit";

// or only import the class you need

import { Face, Pose, Hand } from "kalidokit";
```

#### Via CDN

```js
<script src="https://cdn.jsdelivr.net/npm/kalidokit@0.1.0/dist/kalidokit.umd.js"></script>
```

## Methods

Kalidokit is composed of 3 classes for Face, Pose, and Hand calculations. They accept landmark outputs from models like Facemesh, Blazepose, Handpose, and Holistic.

```js
// Accepts an array(468 or 478 with iris tracking) of vectors
Kalidokit.Face.solve(facelandmarkArray, {
    runtime: "tfjs", // `mediapipe` or `tfjs`
    video: HTMLVideoElement,
    imageSize: { height: 0, width: 0 },
    enableLegs: true,
});

// Accepts arrays(33) of Pose keypoints and 3D Pose keypoints
Kalidokit.Pose.solve(poseWorld3DArray, poseLandmarkArray, {
    runtime: "tfjs", // `mediapipe` or `tfjs`
    video: HTMLVideoElement,
    imageSize: { height: 0, width: 0 },
    enableLegs: true,
});

// Accepts array(21) of hand landmark vectors; specify 'Right' or 'Left' side
Kalidokit.Hand.solve(handLandmarkArray, "Right");

// Using exported classes directly
Face.solve(facelandmarkArray);
Pose.solve(poseWorld3DArray, poseLandmarkArray);
Hand.solve(handLandmarkArray, "Right");
```

Additional Utils

```js
// Stabilizes left/right blink + wink by providing blenshapes and head rotation
Kalidokit.Face.stabilizeBlink({ r: 0, l: 1 }, headRotationY);

// The internal vector math class
Kalidokit.Vector();
```

## Remixable Vtuber Template with KalidoKit

![KalidoKit Template on Glitch](https://cdn.glitch.me/29e07830-2317-4b15-a044-135e73c7f840%2Fkalidoface-pose-dance.gif?v=1633453098775)

Quick-start your Vtuber app with this simple remixable example on Glitch. Face, full-body, and hand tracking in under 350 lines of javascript. This demo uses Mediapipe Holistic for body tracking, Three.js + Three-VRM for rendering models, and KalidoKit for the kinematic calculations. This demo uses a minimal amount of easing to smooth animations, but feel free to make it your own!

<a href="https://glitch.com/edit/#!/remix/kalidokit-template"><img alt="Remix on Glitch" src="https://cdn.gomix.com/f3620a78-0ad3-4f81-a271-c8a4faa20f86%2Fremix-button.svg"></a>

## Basic Usage

The implementation may vary depending on what pose and face detection model you choose to use, but the principle is still the same. This example uses Mediapipe Holistic which concisely combines them together.

```js
import Kalidokit from 'kalidokit'
import '@mediapipe/holistic/holistic';

let holistic

// Init Mediapipe Holistic Model
async function initHolistic() {

  holistic = new Holistic({locateFile: (file) => {
    return `https://cdn.jsdelivr.net/npm/@mediapipe/holistic@0.4.1633559476/${file}`;
  }});

  holistic.onResults(results=>{
    // do something with prediction results
    // landmark names may change depending on TFJS/Mediapipe model version
    let facelm = results.faceLandmarks;
    let poselm = results.poseLandmarks;
    let poselm3D = results.ea;
    let rightHandlm = results.rightHandLandmarks;
    let leftHandlm = results.leftHandLandmarks;

    let faceRig = Kalidokit.Face.solve(facelm,{runtime:'mediapipe',video:HTMLVideoElement})
    let poseRig = Kalidokit.Pose.solve(poselm3d,poselm,{runtime:'mediapipe',video:HTMLVideoElement})
    let rightHandRig = Kalidokit.Hand.solve(rightHandlm,"Right")
    let leftHandRig = Kalidokit.Hand.solve(leftHandlm,"Left")

    };
  });
}
initHolistic()

// Predict animation loop
async function predict(){
    if(holistic){
        // send image to holistic prediction
        await holistic.send({image: HTMLVideoElement});
    }
    requestAnimationFrame(predict);
}
predict()
```

## Slight differences with Mediapipe and Tensorflow.js

Due to slight differences in the results from Mediapipe and Tensorflow.js, it is recommended to specify which runtime version you are using as well as the video input/image size as a reference.

```js
Kalidokit.Pose.solve(poselm3D,poselm,{
    runtime:'tfjs', // default is 'mediapipe'
    video: HTMLVideoElement,// specify an html video or manually set image size
    imageSize:{
        width: 640,
        height: 480,
    };
})

Kalidokit.Face.solve(facelm,{
    runtime:'mediapipe', // default is 'tfjs'
    video: HTMLVideoElement,// specify an html video or manually set image size
    imageSize:{
        width: 640,
        height: 480,
    };
})
```

## Outputs

Below are the resting defaults from Kalidokit.

```js
// Kalidokit.Face.solve()
// Head rotations in radians
// Degrees and normalized rotations also available
{
    eye: {l: 1,r: 1},
    mouth: {
        x: 0,
        y: 0,
        shape: {A:0, E:0, I:0, O:0, U:0}
    },
    head: {
        x: 0,
        y: 0,
        z: 0,
        width: 0.3,
        height: 0.6,
        position: {x: 0.5, y: 0.5, z: 0}
    },
    brow: 0,
    pupil: {x: 0, y: 0}
}
```

```js
// Kalidokit.Pose.solve()
// Joint rotations in radians, leg calculators are a WIP
{
    RightUpperArm: {x: 0, y: 0, z: -1.25},
    LeftUpperArm: {x: 0, y: 0, z: 1.25},
    RightLowerArm: {x: 0, y: 0, z: 0},
    LeftLowerArm: {x: 0, y: 0, z: 0},
    LeftUpperLeg: {x: 0, y: 0, z: 0},
    RightUpperLeg: {x: 0, y: 0, z: 0},
    RightLowerLeg: {x: 0, y: 0, z: 0},
    LeftLowerLeg: {x: 0, y: 0, z: 0},
    LeftHand: {x: 0, y: 0, z: 0},
    RightHand: {x: 0, y: 0, z: 0},
    Spine: {x: 0, y: 0, z: 0},
    Hips: {
        position: {x: 0, y: 0, z: 0},
        rotation: {x: 0, y: 0, z: 0},
    }
}
```

```js
// Kalidokit.Hand.solve()
// Joint rotations in radians
// only wrist and thumb have 3 degrees of freedom
// all other finger joints move in the Z axis only
{
    RightWrist: {x: -0.13, y: -0.07, z: -1.04},
    RightRingProximal: {x: 0, y: 0, z: -0.13},
    RightRingIntermediate: {x: 0, y: 0, z: -0.4},
    RightRingDistal: {x: 0, y: 0, z: -0.04},
    RightIndexProximal: {x: 0, y: 0, z: -0.24},
    RightIndexIntermediate: {x: 0, y: 0, z: -0.25},
    RightIndexDistal: {x: 0, y: 0, z: -0.06},
    RightMiddleProximal: {x: 0, y: 0, z: -0.09},
    RightMiddleIntermediate: {x: 0, y: 0, z: -0.44},
    RightMiddleDistal: {x: 0, y: 0, z: -0.06},
    RightThumbProximal: {x: -0.23, y: -0.33, z: -0.12},
    RightThumbIntermediate: {x: -0.2, y: -0.19, z: -0.01},
    RightThumbDistal: {x: -0.2, y: 0.002, z: 0.15},
    RightLittleProximal: {x: 0, y: 0, z: -0.09},
    RightLittleIntermediate: {x: 0, y: 0, z: -0.22},
    RightLittleDistal: {x: 0, y: 0, z: -0.1}
}
```

## Open to Contributions

The current library is a work in progress and contributions to improve it are very welcome. Our goal is to make character face and pose animation even more accessible to creatives regardless of skill level!
