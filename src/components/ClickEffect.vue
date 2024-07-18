<template>
    
</template>

<script>
export default {
  name: "index",
  methods: {
    playAudio() {
      var audio = new Audio('/1.mp3'); 
      audio.play(); 
    },
    note() {
      const self = this; 
      const shouldPlayAudio = import.meta.env.VITE_CLICK_AUDIO === 'true'; 
      const shouldPrintCiallo = import.meta.env.VITE_CLICK_TEXT === 'true'; 

      (function () {
        var a_idx = 0;
        window.onclick = function (event) {
          if (shouldPrintCiallo) {
            var a = new Array("Ciallo～(∠・ω )⌒☆");

            var heart = document.createElement("b");
            heart.onselectstart = new Function('event.returnValue=false');

            document.body.appendChild(heart).innerHTML = a[a_idx];
            a_idx = (a_idx + 1) % a.length;

            heart.style.cssText = "position: fixed;left:-100%;";

            var f = 16,
                x = event.clientX - f / 2,
                y = event.clientY - f,
                c = randomColor(),
                a = 1,
                s = 1.2;

            var timer = setInterval(function () {
              if (a <= 0) {
                document.body.removeChild(heart);
                clearInterval(timer);
              } else {
                heart.style.cssText = "font-size:16px;cursor: default;position: fixed;color:" +
                    c + ";left:" + x + "px;top:" + y + "px;opacity:" + a + ";transform:scale(" +
                    s + ");";

                y--;
                a -= 0.016;
                s += 0.002;
              }
            }, 15);
          }

          
          if (shouldPlayAudio) {
            self.playAudio();
          }
        };

        function randomColor() {
          return "rgb(" + (~~(Math.random() * 255)) + "," + (~~(Math.random() * 255)) + "," + (~~(Math.random() * 255)) + ")";
        }
      }());
    }
  },
  mounted() {
    this.note();
  }
};
</script>

<style scoped>

</style>
