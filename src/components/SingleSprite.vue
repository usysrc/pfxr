<template>
  <div>
    <v-container>
      <canvas id="canvas" width="12px" height="12px"> </canvas>
    </v-container>
    <v-container>
      <v-btn @click="undo" :disabled="history.length <= 1">Undo</v-btn>
      <v-btn @click="generateSprite">Generate</v-btn>
    </v-container>
  </div>
</template>

<script>
export default {
  data: () => {
    return {
      history: [],
      size: 50,
      l: 100,
      r: 200,
      light: 500,
      w: 12,
      h: 12,
    };
  },
  methods: {
    fillData(fillFunc) {
      for (let j = 0; j < this.h; j++) {
        for (let i = 0; i < this.w; i++) {
          fillFunc(i, j, this.w, this.h);
        }
      }
    },

    generateSprite() {
      let rrand = (a, b) => {
        return a + Math.random() * (b - a);
      };

      let canvas = document.getElementById("canvas");
      let ctx = canvas.getContext("2d");
      ctx.globalCompositeOperation = "copy";
      ctx.globalAlpha = 10;

      let imageData = ctx.createImageData(this.w, this.h);

      let vec = [];

      const setPixel = function (x, y, color) {
        let index = (x + y * imageData.width) * 4;
        vec[index + 0] = color.r;
        vec[index + 1] = color.g;
        vec[index + 2] = color.b;
        vec[index + 3] = color.a;
      };

      const getPixel = function (x, y) {
        let index = (x + y * imageData.width) * 4;
        return {
          r: vec[index + 0],
          g: vec[index + 1],
          b: vec[index + 2],
          a: vec[index + 3],
        };
      };

      let CLEAR = { r: 127, g: 127, b: 127, a: 0 };
      let BLACK = { r: 0, g: 0, b: 0, a: 255 };

      this.fillData((x, y) => {
        setPixel(x, y, CLEAR);
      });

      // TODO: Remove magic numbers
      let size = this.size / 25;
      let r = this.r / 1000;
      let l = this.l / 1000;
      let startColor = {
        r: rrand(10, 255),
        g: rrand(10, 255),
        b: rrand(10, 255),
      };

      let light = this.light / 100;

      this.fillData(function (i, j, w, h) {
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

        const c = Math.tan(hull * light);

        setPixel(i, j, {
          r: c * startColor.r,
          g: c * startColor.g,
          b: c * startColor.b,
          a: 255,
        });
      });
      const fillData = this.fillData;
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
      this.fillData(function (i, j) {
        if (
          !isClear(getPixel(i, j)) &&
          // has a clear neigbor
          (isClear(getPixel(i + 1, j)) ||
            isClear(getPixel(i - 1, j)) ||
            isClear(getPixel(i, j + 1)) ||
            isClear(getPixel(i, j - 1)))
        ) {
          setPixel(i, j, BLACK);
        }
      });

      this.fillData((x, y) => {
        let index = (x + y * imageData.width) * 4;
        for (let k = 0; k < 4; k++) {
          imageData.data[index + k] = vec[index + k];
        }
      });
      this.history.push(vec);
      ctx.putImageData(imageData, -1, -1);
    },

    undo() {
      if (this.history.length <= 1) return;
      this.history.pop();
      let canvas = document.getElementById("canvas");
      let ctx = canvas.getContext("2d");
      let imageData = ctx.createImageData(this.w, this.h);
      const vec = this.history[this.history.length - 1];
      this.fillData((x, y) => {
        let index = (x + y * imageData.width) * 4;
        for (let k = 0; k < 4; k++) {
          imageData.data[index + k] = vec[index + k];
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
