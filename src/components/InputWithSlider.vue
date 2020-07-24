<template>
  <div
    class="flex justify-center items-start w-full"
    :class="labelOnLeft? 'flex-col md:flex-row':'flex-col'"
  >
    <div class="pb-1 text-left w-full font-bold">
      {{label}}
      <img src="icons/info.svg" class="inline cursor-pointer" />
    </div>
    <div class="w-full">
      <div class="flex flex-row justify-between border rounded p-2">
        <money
          :value="intlVal"
          @input="onInput"
          v-bind="moneyMask"
          v-if="type == 'currency'"
          class="w-full oultine-none"
        ></money>
        <input
          :value="intlVal"
          @input="onInput($event.target.value)"
          v-if="type == ''"
          class="oultine-none"
        />
        <span
          class="text-lg text-gray-600 px-2 select-none"
          v-if="layoverText.length>0"
        >{{layoverText}}</span>
      </div>
      <input
        type="range"
        v-model="inRange"
        :min="min"
        :max="max"
        :step="step"
        class="w-full my-1 px-2"
      />
    </div>
  </div>
</template>

<script>
import { Money } from "v-money";

export default {
  name: "input-with-slider",
  components: {
    money: Money,
  },
  props: {
    value: {
      type: Number,
      default: 500,
    },
    label: {
      type: String,
      default: "",
    },
    labelOnLeft: {
      type: Boolean,
      default: false,
    },
    layoverText: {
      type: String,
      default: "",
    },
    min: {
      type: Number,
      default: 0,
    },
    max: {
      type: Number,
      default: 2500,
    },
    step: {
      type: Number,
      default: 100,
    },
    mask: {
      type: String,
      default: "$000,000,000,000",
    },
    type: {
      type: String,
      default: "currency",
    },
  },
  data() {
    return {
      intlVal: 0,
      inRange: 0,
      moneyMask: {
        decimal: ".",
        thousands: ",",
        prefix: "$ ",
        precision: 0,
        masked: false /* doesn't work with directive */,
      },
    };
  },
  mounted() {
    this.intlVal = this.value;
  },
  methods: {
    onInput(oval) {
      let val = oval == "" ? 0 : parseInt(oval);
      if (val >= this.min && val <= this.max) {
        this.intlVal = val;
        this.$emit("input", this.intlVal);
      } else {
        this.intlVal = val > this.max ? this.max : this.min;
        if (this.type == "") {
          this.$nextTick(function () {
            this.intlVal = this.intlVal > this.max ? this.max : this.min;
          });
        } else {
          let x = this.intlVal;
          this.intlVal = 0;
          this.intlVal = x;
        }
      }
    },
  },
  watch: {
    intlVal() {
      console.log("changed");
      if (this.type == "") {
        // only needed for type == '' because'money' comp emits @input on intVal change.
        this.$emit("input", this.intlVal);
      }
      this.inRange = this.intlVal;
    },
    inRange: function () {
      this.intlVal = parseInt(this.inRange);
    },
  },
};
</script>

<style scoped>
img {
  height: 1rem;
  width: 1rem;
}

textarea:focus,
input:focus {
  outline: none;
}

/* input range design */
input[type="range"] {
  height: 23px;
  -webkit-appearance: none;
  margin: 10px 0;
  width: 100%;
}
input[type="range"]:focus {
  outline: none;
}
input[type="range"]::-webkit-slider-runnable-track {
  width: 100%;
  height: 5px;
  cursor: pointer;
  animate: 0.2s;
  box-shadow: 0px 0px 0px #000000;
  background: #17b3e4;
  border-radius: 5px;
  border: 0px solid #000000;
}
input[type="range"]::-webkit-slider-thumb {
  box-shadow: 0px 0px 0px #000000;
  border: 1px solid #17b3e4;
  height: 16px;
  width: 16px;
  border-radius: 16px;
  background: #17b3e4;
  cursor: pointer;
  -webkit-appearance: none;
  margin-top: -6px;
}
input[type="range"]:focus::-webkit-slider-runnable-track {
  background: #17b3e4;
}
input[type="range"]::-moz-range-track {
  width: 100%;
  height: 5px;
  cursor: pointer;
  animate: 0.2s;
  box-shadow: 0px 0px 0px #000000;
  background: #17b3e4;
  border-radius: 5px;
  border: 0px solid #000000;
}
input[type="range"]::-moz-range-thumb {
  box-shadow: 0px 0px 0px #000000;
  border: 1px solid #17b3e4;
  height: 16px;
  width: 16px;
  border-radius: 16px;
  background: #17b3e4;
  cursor: pointer;
}
input[type="range"]::-ms-track {
  width: 100%;
  height: 5px;
  cursor: pointer;
  animate: 0.2s;
  background: transparent;
  border-color: transparent;
  color: transparent;
}
input[type="range"]::-ms-fill-lower {
  background: #17b3e4;
  border: 0px solid #000000;
  border-radius: 10px;
  box-shadow: 0px 0px 0px #000000;
}
input[type="range"]::-ms-fill-upper {
  background: #17b3e4;
  border: 0px solid #000000;
  border-radius: 10px;
  box-shadow: 0px 0px 0px #000000;
}
input[type="range"]::-ms-thumb {
  margin-top: 1px;
  box-shadow: 0px 0px 0px #000000;
  border: 1px solid #17b3e4;
  height: 16px;
  width: 16px;
  border-radius: 16px;
  background: #17b3e4;
  cursor: pointer;
}

/** FF*/
input[type="range"]::-moz-range-progress {
  background-color: #17b3e4;
}
input[type="range"]::-moz-range-track {
  background-color: #e9e9e9;
}
/* IE*/
input[type="range"]::-ms-fill-lower {
  background-color: #17b3e4;
}
input[type="range"]::-ms-fill-upper {
  background-color: #e9e9e9;
}
</style>