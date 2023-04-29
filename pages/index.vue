<template>
  <div class="content" ref="content">
    <div class="text">
      <Smiley class="smiley top" v-if="!fell" />
      <Frowney class="smiley top" v-else />
      <template v-if="isMobile">
        <p :class="{ fell }">
          {{
            !this.fell ? "THROW YOUR PHONE TO VIEW THIS SITE" : phoneResponse
          }}
        </p>
        <button
          class="permissions-button"
          @click="askPermission"
          v-if="!fell && !hasPermissions && !permissionsError"
        >
          TAP TO GIVE MOTION PERMISSIONS
        </button>
        <p
          v-if="!fell && !hasPermissions && permissionsError"
          class="permissions-text"
        >
          ERROR GETTING MOTION PERMISSIONS
        </p>
        <!-- <p>{{ this.didFall }}</p> -->
      </template>
      <template v-else>
        <p>DEFINITELY DO NOT THROW YOUR COMPUTER. USE A PHONE.</p>
      </template>
      <Smiley class="smiley bottom" v-if="!fell" />
      <Frowney class="smiley bottom" v-else />
    </div>
  </div>
</template>

<script>
import Smiley from "~/assets/img/smiley.svg?inline";
import Frowney from "~/assets/img/frowney.svg?inline";
import { listenCb, tickUpdate } from "~/assets/js/utils";

const PHONE_RESPONSES = [
  "OUCH.",
  "WHY DID YOU THROW ME?",
  "THAT WASN'T NICE",
  "YOU HURT ME",
];

export default {
  components: { Smiley, Frowney },
  data: () => {
    return {
      acceleration: { x: 0, y: 0, z: 0 },
      maxAcceleration: 10.5,
      minAcceleration: 8,
      didFall: "",
      isMobile: false,
      isPlateauting: false,
      potentialFallStarted: false,
      fell: false,
      hasPermissions: false,
      permissionsError: false,
      phoneResponse: PHONE_RESPONSES[0],
    };
  },
  mounted() {
    this.checkIsMobile();
    this.startTime = new Date();
    this.unlisten = [];
  },
  beforeDestroy() {
    this.unlisten.forEach((cb) => cb());
  },
  methods: {
    checkIsMobile() {
      let check = false;
      (function (a) {
        if (
          /(android|bb\d+|meego).+mobile|avantgo|bada\/|blackberry|blazer|compal|elaine|fennec|hiptop|iemobile|ip(hone|od)|iris|kindle|lge |maemo|midp|mmp|mobile.+firefox|netfront|opera m(ob|in)i|palm( os)?|phone|p(ixi|re)\/|plucker|pocket|psp|series(4|6)0|symbian|treo|up\.(browser|link)|vodafone|wap|windows ce|xda|xiino/i.test(
            a
          ) ||
          /1207|6310|6590|3gso|4thp|50[1-6]i|770s|802s|a wa|abac|ac(er|oo|s\-)|ai(ko|rn)|al(av|ca|co)|amoi|an(ex|ny|yw)|aptu|ar(ch|go)|as(te|us)|attw|au(di|\-m|r |s )|avan|be(ck|ll|nq)|bi(lb|rd)|bl(ac|az)|br(e|v)w|bumb|bw\-(n|u)|c55\/|capi|ccwa|cdm\-|cell|chtm|cldc|cmd\-|co(mp|nd)|craw|da(it|ll|ng)|dbte|dc\-s|devi|dica|dmob|do(c|p)o|ds(12|\-d)|el(49|ai)|em(l2|ul)|er(ic|k0)|esl8|ez([4-7]0|os|wa|ze)|fetc|fly(\-|_)|g1 u|g560|gene|gf\-5|g\-mo|go(\.w|od)|gr(ad|un)|haie|hcit|hd\-(m|p|t)|hei\-|hi(pt|ta)|hp( i|ip)|hs\-c|ht(c(\-| |_|a|g|p|s|t)|tp)|hu(aw|tc)|i\-(20|go|ma)|i230|iac( |\-|\/)|ibro|idea|ig01|ikom|im1k|inno|ipaq|iris|ja(t|v)a|jbro|jemu|jigs|kddi|keji|kgt( |\/)|klon|kpt |kwc\-|kyo(c|k)|le(no|xi)|lg( g|\/(k|l|u)|50|54|\-[a-w])|libw|lynx|m1\-w|m3ga|m50\/|ma(te|ui|xo)|mc(01|21|ca)|m\-cr|me(rc|ri)|mi(o8|oa|ts)|mmef|mo(01|02|bi|de|do|t(\-| |o|v)|zz)|mt(50|p1|v )|mwbp|mywa|n10[0-2]|n20[2-3]|n30(0|2)|n50(0|2|5)|n7(0(0|1)|10)|ne((c|m)\-|on|tf|wf|wg|wt)|nok(6|i)|nzph|o2im|op(ti|wv)|oran|owg1|p800|pan(a|d|t)|pdxg|pg(13|\-([1-8]|c))|phil|pire|pl(ay|uc)|pn\-2|po(ck|rt|se)|prox|psio|pt\-g|qa\-a|qc(07|12|21|32|60|\-[2-7]|i\-)|qtek|r380|r600|raks|rim9|ro(ve|zo)|s55\/|sa(ge|ma|mm|ms|ny|va)|sc(01|h\-|oo|p\-)|sdk\/|se(c(\-|0|1)|47|mc|nd|ri)|sgh\-|shar|sie(\-|m)|sk\-0|sl(45|id)|sm(al|ar|b3|it|t5)|so(ft|ny)|sp(01|h\-|v\-|v )|sy(01|mb)|t2(18|50)|t6(00|10|18)|ta(gt|lk)|tcl\-|tdg\-|tel(i|m)|tim\-|t\-mo|to(pl|sh)|ts(70|m\-|m3|m5)|tx\-9|up(\.b|g1|si)|utst|v400|v750|veri|vi(rg|te)|vk(40|5[0-3]|\-v)|vm40|voda|vulc|vx(52|53|60|61|70|80|81|83|85|98)|w3c(\-| )|webc|whit|wi(g |nc|nw)|wmlb|wonu|x700|yas\-|your|zeto|zte\-/i.test(
            a.substr(0, 4)
          )
        )
          check = true;
      })(navigator.userAgent || navigator.vendor || window.opera);
      this.isMobile = check;
    },
    handleMotion(e) {
      this.acceleration = {
        x: e.acceleration.x,
        y: e.acceleration.y,
        z: e.acceleration.z,
      };

      const totalAcceleration = Math.sqrt(
        Math.pow(this.acceleration.x, 2) +
          Math.pow(this.acceleration.y, 2) +
          Math.pow(this.acceleration.z, 2)
      );

      // this.didFall = `${this.acceleration.x} ${this.acceleration.y} ${this.acceleration.z} ${totalAcceleration}`;
      // if (Math.floor(totalAcceleration) > 0) {
      //   const statsDiv = document.createElement("div");
      //   statsDiv.innerHTML = `${totalAcceleration}`;
      //   this.$refs.content.appendChild(statsDiv);
      // }

      // TODO: DETECT ANOTHER KIND OF THROW - HARD THROW - GET HIGH ACCELLERATION FOLLOWED BY MOVEMENT OF ZERO IN THE NEXT TWO SECONDS

      if (Math.floor(totalAcceleration) > 15 && !this.potentialFallStarted) {
        this.potentialFallStarted = true;
        this.isPlateauting = false;

        setTimeout(() => {
          this.potentialFallStarted = false;
          this.isPlateauting = false;
        }, 2000);
      } else if (
        this.potentialFallStarted &&
        Math.floor(totalAcceleration) > 0
      ) {
        const now = new Date();
        if (
          totalAcceleration <= this.maxAcceleration &&
          totalAcceleration >= this.minAcceleration &&
          this.isPlateauting &&
          now - this.startTime >= 300
        ) {
          this.fell = true;
          this.phoneResponse = this.selectRandomPhoneResponse();
          this.potentialFallStarted = false;
          this.isPlateauting = false;
          document.documentElement.style.setProperty(
            "--color-bg",
            "var(--color-white)"
          );
          // const statsDiv = document.createElement("div");
          // statsDiv.innerHTML = `FELL`;
          // this.$refs.content.appendChild(statsDiv);
        } else if (
          totalAcceleration <= this.maxAcceleration &&
          totalAcceleration >= this.minAcceleration &&
          !this.isPlateauting
        ) {
          this.startTime = now;
          this.isPlateauting = true;
          // const statsDiv = document.createElement("div");
          // statsDiv.innerHTML = `STARTING TO PLATEAU`;
          // this.$refs.content.appendChild(statsDiv);
        } else if (
          totalAcceleration <= this.maxAcceleration &&
          totalAcceleration >= this.minAcceleration &&
          this.isPlateauting
        ) {
          // const statsDiv = document.createElement("div");
          // statsDiv.innerHTML = `PLATEAUING: ${now - this.startTime}`;
          // this.$refs.content.appendChild(statsDiv);
        } else {
          this.isPlateauting = false;
        }
      }
      // else if (this.potentialFallStarted &&
      //   Math.floor(totalAcceleration) === 0) {

      //   }
    },
    selectRandomPhoneResponse() {
      return PHONE_RESPONSES[
        Math.floor(Math.random() * PHONE_RESPONSES.length)
      ];
    },
    askPermission() {
      // feature detect
      if (
        typeof DeviceMotionEvent !== "undefined" &&
        typeof DeviceMotionEvent.requestPermission === "function"
      ) {
        DeviceMotionEvent.requestPermission()
          .then((response) => {
            if (response == "granted") {
              this.unlisten.push(
                listenCb(window, "devicemotion", tickUpdate(this.handleMotion))
              );
              this.hasPermissions = true;
            }
          })
          .catch((e) => {
            this.permissionsError = true;
            console.log(e);
          });
      } else {
        this.unlisten.push(
          listenCb(window, "devicemotion", tickUpdate(this.handleMotion))
        );
        this.hasPermissions = true;
      }
    },
  },
};
</script>

<style lang="scss" scoped>
@keyframes SPIN {
  0% {
    transform: var(--initial-transform) rotate(0deg);
  }
  100% {
    transform: var(--initial-transform) rotate(360deg);
  }
}
p {
  position: relative;
  color: var(--color-white);
  text-align: center;
  font-size: 24px;
  font-weight: 400;

  &.fell {
    color: var(--color-black);
  }

  @include tablet {
    font-size: 40px;
  }
}

.permissions-text,
.permissions-button {
  font-size: 16px;

  @include tablet {
    font-size: 20px;
  }
}

.permissions-button {
  position: relative;
  display: block;
  border: none;
  background: none;
  color: var(--color-white);
  left: 50%;
  transform: translate3d(-50%, 0, 0);
  margin-bottom: 30px;
}

.text {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate3d(-50%, -50%, 0);
}

.smiley {
  --initial-transform: translate3d(-50%, 0%, 0);
  position: relative;
  left: 50%;
  transform: var(--initial-transform);
  width: 50px;
  height: auto;
  animation: SPIN 5s linear infinite;

  &.bottom {
    animation: SPIN 5s linear infinite reverse;
  }

  @include tablet {
    width: 100px;
  }
}
</style>
