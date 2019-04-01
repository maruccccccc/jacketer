<template>
  <div class="body" @dragover="drop" @dragenter="drop" @drop="drop" @dragleave="drop" @paste="drop">
    <div id="preview-wrapper" class="preview-wrapper">
      <canvas class="preview" id="preview"></canvas>
    </div>
    <div class="columns is-centered button-ctrl">
      <div class="column is-one-fifths">
        <button class="button is-primary is-rounded is-fullwidth" @click="trimLU">LU</button>
      </div>
      <div class="column is-one-fifths">
        <button class="button is-primary is-rounded is-fullwidth" @click="trimC">C</button>
      </div>
      <div class="column is-one-fifths">
        <button class="button is-primary is-rounded is-fullwidth" @click="trimRD">RD</button>
      </div>
      <div class="column">
        <button class="button is-info is-rounded is-fullwidth" @click="marginAdjust">余白調整</button>
      </div>
      <div class="column">
        <button class="button is-grey-lighter is-rounded is-fullwidth" @click="reload">reload</button>
      </div>
    </div>
    <div class="columns is-centered button-ctrl">
      <div class="column">
        <div class="file has-name is-fullwidth is-info">
          <label class="file-label">
            <input
              id="dir-name-input"
              class="file-input"
              type="file"
              name="resume"
              webkitdirectory
              @change="fixDirName"
            >
            <span class="file-cta">
              <span class="file-icon">
                <i class="fas fa-upload"></i>
              </span>
              <span class="file-label">Output Directory..</span>
            </span>
            <span id="dir-name" class="file-name"></span>
          </label>
        </div>
      </div>
      <div class="column">
        <input
          id="file-name"
          class="input"
          value="jacket.png"
          type="text"
          placeholder="Input File Name (png only) [ex. example.png]"
        >
      </div>
    </div>
  </div>
</template>

<script>
const sharp = require("sharp");
const fs = require("fs");
const { clipboard, nativeImage } = require("electron");
function canvasUpdate(canvas, base64Image) {
  const img = document.createElement("img");
  const preview = document.getElementById("preview-wrapper");
  img.src = base64Image;
  img.onload = () => {
    let rate = 1;
    if (canvas.width > preview.clientWidth) {
      rate = preview.clientWidth / canvas.width;
    }

    if (img.width > img.height) {
      if (
        img.height * (preview.clientWidth / img.width) >
        preview.clientHeight
      ) {
        rate =
          preview.clientHeight /
          (img.height * (preview.clientWidth / img.width));
      }
      canvas.width = preview.clientWidth * rate;
      canvas.height = img.height * (preview.clientWidth / img.width) * rate;
    } else {
      if (
        img.width * (preview.clientHeight / img.height) >
        preview.clientWidth
      ) {
        rate =
          preview.clientWidth /
          (img.width * (preview.clientHeight / img.height));
      }
      canvas.height = preview.clientHeight * rate;
      canvas.width = img.width * (preview.clientHeight / img.height) * rate;
    }

    const ctx = canvas.getContext("2d");
    ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
  };
}

function trim(img, left, top, width, height) {
  const canvas = document.getElementById("preview");
  let option = {
    left: left,
    top: top,
    width: width,
    height: height
  };

  sharp(Buffer.from(canvas.orig.split(",")[1], "base64"))
    .extract(option)
    .toBuffer()
    .then(data => {
      canvasUpdate(
        canvas,
        `${canvas.orig.split(";")[0]};base64,${data.toString("base64")}`
      );
      clipboard.writeImage(nativeImage.createFromBuffer(data));
      const dirName = document.getElementById("dir-name-input").files[0];
      const fileName = document.getElementById("file-name");
      if (dirName && dirName.path && fileName) {
        fs.writeFileSync(`${dirName.path}/${fileName.value}`, data, "binary");
      }
    });
}

export default {
  name: "index-page",
  methods: {
    open(link) {
      this.$electron.shell.openExternal(link);
    },
    reload(e) {
      const orig = document.getElementById("preview").orig;
      if (orig) {
        const canvas = document.getElementById("preview");
        const img = document.createElement("img");
        img.src = orig;
        img.onload = () => {
          let option = {};

          sharp(Buffer.from(canvas.orig.split(",")[1], "base64"))
            .toBuffer()
            .then(data => {
              canvasUpdate(
                canvas,
                `${canvas.orig.split(";")[0]};base64,${data.toString("base64")}`
              );
              clipboard.writeImage(nativeImage.createFromBuffer(data));
              const dirName = document.getElementById("dir-name-input")
                .files[0];
              const fileName = document.getElementById("file-name");
              if (dirName && dirName.path && fileName) {
                fs.writeFileSync(
                  `${dirName.path}/${fileName.value}`,
                  data,
                  "binary"
                );
              }
            });
        };
      }
      e.preventDefault();
    },
    trimLU(e) {
      const orig = document.getElementById("preview").orig;
      if (orig) {
        const img = document.createElement("img");
        img.src = orig;
        img.onload = () => {
          let option = {};
          if (img.width > img.height) {
            option = {
              left: 0,
              top: 0,
              width: img.height,
              height: img.height
            };
          } else {
            option = {
              left: 0,
              top: 0,
              width: img.width,
              height: img.width
            };
          }
          trim(img, option.left, option.top, option.width, option.height);
        };
      }

      e.preventDefault();
    },
    trimC(e) {
      const orig = document.getElementById("preview").orig;
      if (orig) {
        const img = document.createElement("img");
        img.src = orig;
        img.onload = () => {
          let option = {};
          if (img.width > img.height) {
            option = {
              left: Math.ceil((img.width - img.height) / 2),
              top: 0,
              width: img.height,
              height: img.height
            };
          } else {
            option = {
              left: 0,
              top: Math.ceil((img.height - img.width) / 2),
              width: img.width,
              height: img.width
            };
          }
          trim(img, option.left, option.top, option.width, option.height);
        };
      }

      e.preventDefault();
    },
    trimRD(e) {
      const orig = document.getElementById("preview").orig;
      if (orig) {
        const img = document.createElement("img");
        img.src = orig;
        img.onload = () => {
          let option = {};
          if (img.width > img.height) {
            option = {
              left: img.width - img.height,
              top: 0,
              width: img.height,
              height: img.height
            };
          } else {
            option = {
              left: 0,
              top: img.height - img.width,
              width: img.width,
              height: img.width
            };
          }
          trim(img, option.left, option.top, option.width, option.height);
        };
      }

      e.preventDefault();
    },
    marginAdjust(e) {
      const orig = document.getElementById("preview").orig;
      if (orig) {
        const canvas = document.getElementById("preview");
        const img = document.createElement("img");
        img.src = orig;
        img.onload = () => {
          let width = 0;
          let height = 0;
          let option = {
            kernel: sharp.kernel.nearest,
            fit: "contain",
            background: { r: 0, g: 0, b: 0, alpha: 1 }
          };
          if (img.width > img.height) {
            width = height = img.width;
          } else {
            width = height = img.height;
          }

          sharp(Buffer.from(canvas.orig.split(",")[1], "base64"))
            .resize(width, height, option)
            .toBuffer()
            .then(data => {
              canvasUpdate(
                canvas,
                `${canvas.orig.split(";")[0]};base64,${data.toString("base64")}`
              );
              clipboard.writeImage(nativeImage.createFromBuffer(data));
              const dirName = document.getElementById("dir-name-input")
                .files[0];
              const fileName = document.getElementById("file-name");
              if (dirName && dirName.path && fileName) {
                fs.writeFileSync(
                  `${dirName.path}/${fileName.value}`,
                  data,
                  "binary"
                );
              }
            });
        };
      }
      e.preventDefault();
    },
    drop(e) {
      if (e.type == "drop" || e.type == "paste") {
        switch (e.type) {
          case "drop":
            const file = e.dataTransfer.files[0];
            if (file.type.startsWith("image/")) {
              const reader = new FileReader();
              reader.onload = () => {
                const canvas = document.getElementById("preview");
                canvas.orig = reader.result;
                canvasUpdate(canvas, canvas.orig);
              };
              reader.readAsDataURL(file);
            }
            break;
          case "paste":
            const formats = clipboard.availableFormats();
            for (const index in formats) {
              if (formats[index].startsWith("image/")) {
                const cbFile = clipboard.readImage();
                const canvas = document.getElementById("preview");
                canvas.orig = cbFile.toDataURL();
                canvasUpdate(canvas, canvas.orig);
              }
            }
            break;
        }
      }

      e.preventDefault();
    },
    fixDirName(e) {
      const file = document.getElementById("dir-name-input");
      if (file.files.length > 0) {
        document.getElementById("dir-name").innerHTML = file.files[0].path;
      }
    }
  }
};
</script>

<style>
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

div.body {
  width: 100%;
  height: 100%;
}

.preview-wrapper {
  text-align: center;
  height: 75%;
}

.preview {
  display: inline-block;
  text-align: left;
  border: solid 1.5px #999;
  background: #fff;
  background-image: linear-gradient(45deg, #eee 25%, transparent 0),
    linear-gradient(45deg, transparent 75%, #eee 0),
    linear-gradient(45deg, #eee 25%, transparent 0),
    linear-gradient(45deg, transparent 75%, #eee 0);
  background-size: 30px 30px;
  background-position: 0 0, 15px 15px, 15px 15px, 30px 30px;
}

.button-ctrl {
  padding-top: 2%;
  padding-left: 2%;
  padding-bottom: 2%;
  height: 10%;
}
</style>
