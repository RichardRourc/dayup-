<template>
  <div class="container" id="container">
    <div class="loading" :style="`--height:${1 - percent}`"></div>
  </div>
</template>

<script>
import waveWorker from '!file-loader!./waveWorker.js';

// import waveWorker from './waveWorker.js';
export default {
  props: {
    percent: {
      default: 0.9,
      type: Number
    }
  },
  created() {
    // console.log(path.dirname, __dirname, __filename, 'dirname');
    // console.log(import.meta.url, 'meximexi');
    // console.log(new URL(__dirname), 'dirname');
    if (CSS.paintWorklet) {
      CSS.paintWorklet.addModule(waveWorker);
      // 路径得用这种绝对路径的？
      //   CSS.paintWorklet.addModule('@/views/personas/consumer/usergrouping/addUserPackage/PreviewWaveBall/waveWorker.js');

      //   CSS.paintWorklet.addModule(
      //     path.resolve(__dirname, 'src/views/personas/consumer/usergrouping/addUserPackage/PreviewWaveBall/waveWorker.js')
      //   );

      //    CSS.paintWorklet.addModule(
      //     path.resolve('waveWorker.js')
      //   );
    }
  }
};
</script>

<style lang="scss" scoped module>
@property --animation-tick {
  syntax: '<number>';
  inherits: false;
  initial-value: 1000;
}

@property --height {
  syntax: '<number>';
  inherits: false;
  initial-value: 1;
}

@property --amplitude {
  syntax: '<number>';
  inherits: false;
  initial-value: 1;
}

.loading {
  width: 140px;
  height: 140px;
  background: paint(waveDraw);
  border: 2px solid #bbb;
  //   --amplitude: 3; // 振幅
  --amplitude: 8; // 振幅
  --gap: 28; // 间距
  --animation-tick: 500; // 运动速度
  //   --height: 0.9; //进度？
  //   --color1: rgba(0, 0, 255, 0.4);
  --color1: rgba(26, 191, 191, 0.85);
  --color2: rgba(26, 191, 191, 0.3);
  --color3: rgba(26, 191, 191, 0.1);

  animation: move 20s infinite linear;
  transition: --amplitude 2s, --height 0s;

  &:hover {
    --amplitude: 10; // 振幅
    // --height: 0; // 进度
  }
}

@keyframes move {
  100% {
    --animation-tick: 0;
  }
}
</style>
