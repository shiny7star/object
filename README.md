# Three.js-Ocean-Scene
This project is an example of lightweight procedural skybox and ocean shaders for browsers using the [Three.js](https://threejs.org/) library. It is mainly designed for mobile, so don't expect too much in terms of graphics, but expect performance (I get a constant 120 FPS on a 2021 mid-range mobile at full resolution).

| ![day](https://github.com/Nugget8/Three.js-Ocean-Scene/assets/78450254/757c8c97-d336-4e9e-8039-13453c2a43f7) | ![sunset](https://github.com/Nugget8/Three.js-Ocean-Scene/assets/78450254/b476220f-293c-461a-8f4c-377c6e889d67) | ![moon_rising](https://github.com/Nugget8/Three.js-Ocean-Scene/assets/78450254/c327680c-264f-478c-a8bb-cd951a3a4c75) |
| :-: | :-: | :-: |
| *Daytime* | *Sunset* | *Moon rising* |

I made a [little demo]() so you can check how it runs and looks. It supports touch input too. Or you can watch [the video](https://youtu.be/xt4Nvrw1EMw).

## How it works
### Skybox
The skybox is basically a cube around the player generated with 3 gradients:
* **Density gradient**, which is used to give a brighter color at horizon
* **Luminosity gradient**, which is used to estimate how much light reaches a given fragment
* **Twilight gradient**, which is used to roughly simulate Rayleigh scattering by multiplying the density and luminosity gradients, offering beautiful dawns and dusks

| ![density](https://github.com/Nugget8/Three.js-Ocean-Scene/assets/78450254/2a5b8df5-7a25-41a0-86a7-39b4efb769b9) | ![density_luminosity](https://github.com/Nugget8/Three.js-Ocean-Scene/assets/78450254/8129b891-367b-4045-8afb-2eef0ef034db) | ![luminosity](https://github.com/Nugget8/Three.js-Ocean-Scene/assets/78450254/97f0b07c-32c4-4127-ade0-8f22c8727b87) |
| :-: | :-: | :-: |
| *Density gradient* | *Twilight gradient* | *Luminosity gradient* |

| ![color](https://github.com/Nugget8/Three.js-Ocean-Scene/assets/78450254/64f1a220-39c0-431b-aef1-ca07cd5256ad) |
| :-: |
| *Twilight gradient with color* |

The sun is drawn by raising the luminosity gradient to some power and multiplying the result so that we get a glow around the sun. The moon is essentially the same thing, but with the inversed luminosity gradient, a higher power and higher multiplier so that the glow is more subtle.

| ![sun](https://github.com/Nugget8/Three.js-Ocean-Scene/assets/78450254/71f5eb89-12f5-4714-9032-aa8697fdec1d) | ![moon](https://github.com/Nugget8/Three.js-Ocean-Scene/assets/78450254/c569a507-1c55-474a-a535-99769cf6f38e) |
| :-: | :-: |
| *The sun* | *The moon* |

The stars are generated by picking random 3d unit vectors mapped to a cube grid alongside with a random offset, size, and color. These vales are packed inside a texture and sent to the gpu. On the gpu, we map the corresponding cube grid values and compute the stars in a similar way to how the sun and moon were calculated. The moon and the stars are only drawn if the sky luminosity is low enough.

| ![stars](https://github.com/Nugget8/Three.js-Ocean-Scene/assets/78450254/4d036c3c-12a2-48d9-8a7f-1be45d8a8522) | ![stars_unclamped](https://github.com/Nugget8/Three.js-Ocean-Scene/assets/78450254/cc778ff6-4fcd-489a-97f0-ecf7e18a9b90) |
| :-: | :-: |
| *Stars* | *Stars with unclamped distance* |

### Water
The water is generated using 3 materials:
* Surface material which is used to render the waviness effect
* Water volume material which is used to avoid skybox rendering underwater
* Object material which is used to render textured geometry

The entire ocean geometry is a giant box that stays centered with the player camera on the xz plane. The ocean surface has a side length of only 4000m to avoid float precision issues. Because the ocean is only 12 vertices and 12 triangles, it is very cheap geometry-wise. It is made from 2 meshes: the surface plane and the volume box, hence the need for surface and volume materials.

The ocean surface seems wavy even though it is geometrically completely flat, thanks to 2 scrolling different normal maps generated with [Blender](https://www.blender.org/)'s ocean modifier ([tutorial](https://www.youtube.com/watch?v=rV6TJ7YDJY8&t=140s)). In an earlier iteration, I also implemented parallax mapping which improved the waviness efect considerably, but I found that it drops the performance way too much on mobile.

### Sea floor
The sea floor is procedurally generated using [Perlin noise](https://en.wikipedia.org/wiki/Perlin_noise). I got heavily inspired by [this video](https://youtu.be/ob3VwY4JyzE?si=jIdOJFmKaLe7LBSI) to generate sea floor height. Also, for performance reasons, it is divided into tiles (or chunks) and only tiles close enough to the player are visible.

## How to edit
I highly recommend using [Visual Studio Code](https://code.visualstudio.com/) to edit projects like this one. If you have never used Three.js before, I recommend installing it with [Node.js](https://nodejs.org/en) using the `npm install three` command in the terminal of your project folder, inside VS Code. This will ensure you get the latest version of Three.js and that the TypeScript file (responsible for coding suggestions and tooltips) can get linked to your project. To link the TS file with the project put this in the head of your index.html:
```html
<script type="importmap">
    {
        "imports":
        {
            "three": "./path/to/three.module.js"
        }
    }
</script>
```
If you have done it properly, you shold get coding suggestions when you write something three.js-related in your scripts.

You can then import Three.js in your JavaScript files using:
```js 
import * as THREE from "three";
```
Or:
```js
import { first, second, third } from "three";
```

To run the project you will need an HTTP server, and the [Live Server](https://marketplace.visualstudio.com/items?itemName=ritwickdey.LiveServer) extension will help you create a local server directly from VS Code.

Coding shaders with code highlights improves readability and workflow, so I highly recommend using [
Shader languages support for VS Code](https://marketplace.visualstudio.com/items?itemName=slevesque.shader) together with [Comment tagged templates](https://marketplace.visualstudio.com/items?itemName=bierner.comment-tagged-templates). Here is an example of how to use them:
```js
export const vertexShader = 
/*glsl*/`
    varying vec2 _uv;

    void main()
    {
        _uv = uv;
        gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
    }
`;

export const fragmentShader = 
/*glsl*/`
    varying vec2 _uv;

    void main() 
    {
        gl_FragColor = vec4(_uv.x, _uv.y, 0.5, 1.0);
    }
`;
```

## Conclusion
I made this project because I wanted to make my own game, but quickly realized how difficult it is to do that by myself. I posted it here because I think it has reached a point which makes it worth sharing.
