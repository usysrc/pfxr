<template>
  <div>
    <v-container>
      <canvas id="canvas" width="12px" height="12px"> </canvas>
    </v-container>
    <v-container>
      <v-btn @click="generateSprite">Generate</v-btn>
    </v-container>
  </div>
</template>

<script>
export default {
  data: () => {
    return {
      size: 40,
      l: 100,
      r: 200,
      light: 500,
    };
  },
  methods: {
    generateSprite() {
      let rrand = (a, b) => {
        return a + Math.random() * (b - a);
      };

      let canvas = document.getElementById("canvas");
      let ctx = canvas.getContext("2d");

      let w = 13,
        h = 13;
      let imageData = ctx.createImageData(w, h);

      let setPixel = function (x, y, color) {
        let index = (x + y * imageData.width) * 4;
        imageData.data[index + 0] = color.r;
        imageData.data[index + 1] = color.g;
        imageData.data[index + 2] = color.b;
        imageData.data[index + 3] = color.a || 255;
      };

      let getPixel = function (x, y) {
        let index = (x + y * imageData.width) * 4;
        return {
          r: imageData.data[index + 0],
          g: imageData.data[index + 1],
          b: imageData.data[index + 2],
          a: imageData.data[index + 3],
        };
      };

      let fillData = function (fillFunc) {
        for (let i = 0; i < w; i++) {
          for (let j = 0; j < h; j++) {
            fillFunc(i, j, w, h);
          }
        }
      };

      let CLEAR = { r: 127, g: 127, b: 127, a: 0 };
      let BLACK = { r: 0, g: 0, b: 0 };

      let size = this.size / 25;
      let r = this.r / 1000;
      let l = this.l / 1000;
      let startColor = {
        r: rrand(10, 255),
        g: rrand(10, 255),
        b: rrand(10, 255),
      };

      let light = this.light / 100;

      fillData(function (i, j, w, h) {
        let hullX = 1 - Math.abs(i - w / 2) / (w / 2);
        let hullY = 1 - Math.abs(j - h / 2) / (h / 2);
        let hull = hullX * hullY * size * rrand(l, r);

        // TODO: coming soon, rotation of parts
        let hullcos = hull * Math.cos(i / w);
        let hullsin = hull * Math.sin(j / h);
        hull = hull * 0.5 + hullcos * 0.5;
        hull = hull * 0.5 + hullsin * 0.5;

        if (hull < 0.1) {
          setPixel(i, j, CLEAR);
          return;
        }

        let c = Math.tan(hull * light);

        setPixel(i, j, {
          r: c * startColor.r,
          g: c * startColor.g,
          b: c * startColor.b,
        });
      });

      let mirrorY = function () {
        fillData(function (i, j, w, h) {
          if (j < h / 2) {
            setPixel(i, j, CLEAR);
          } else {
            setPixel(i, h - j, getPixel(i, j));
          }
        });
      };
      let mirrorX = function () {
        fillData(function (i, j, w) {
          if (i < w / 2) {
            setPixel(i, j, CLEAR);
          } else {
            setPixel(w - i, j, getPixel(i, j));
          }
        });
      };

      // Mirror the image by chance
      let mirror = Math.random();

      if (mirror > 0.5) {
        mirrorX();
        if (Math.random() < 0.2) {
          mirrorY();
        }
      } else {
        mirrorY();
        if (Math.random() < 0.2) {
          mirrorX();
        }
      }
      // if (Math.random() < 0.1) {
      //   fillData(function (i, j, w, h) {
      //     if (i < w / 2 || j < h / 2) {
      //       setPixel(i, j, CLEAR);
      //     } else {
      //       setPixel(w - i, h - j, getPixel(i, j));
      //     }
      //   });
      // }
      // if (Math.random() < 0.1) {
      //   fillData(function (i, j, w, h) {
      //     if (i < w / 2 || j > h / 2) {
      //       setPixel(i, j, CLEAR);
      //     } else {
      //       setPixel(w - i, h - j, getPixel(i, j));
      //     }
      //   });
      // }

      let isClear = function (color) {
        return (
          color.r === CLEAR.r && color.g === CLEAR.g && color.b === CLEAR.b
        );
      };

      // Black outline
      fillData(function (i, j) {
        if (
          !isClear(getPixel(i, j)) &&
          (isClear(getPixel(i + 1, j)) ||
            isClear(getPixel(i - 1, j)) ||
            isClear(getPixel(i, j + 1)) ||
            isClear(getPixel(i, j - 1)))
        ) {
          setPixel(i, j, BLACK);
        }
      });

      ctx.putImageData(imageData, -1, -1);
    },
  },
  mounted() {
    this.generateSprite();
  },
};
</script>

<style>
canvas {
  /* pixel perfect nearest neighbor scaling */
  image-rendering: optimizeSpeed; /* Older versions of FF          */
  image-rendering: -moz-crisp-edges; /* FF 6.0+                       */
  image-rendering: -webkit-optimize-contrast; /* Safari                        */
  image-rendering: -o-crisp-edges; /* OS X & Windows Opera (12.02+) */
  image-rendering: pixelated; /* Awesome future-browsers       */
  -ms-interpolation-mode: nearest-neighbor; /* IE                            */
  width: 60px;
  height: 60px;
}
</style>
