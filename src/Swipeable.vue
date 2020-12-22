<template>
  <div
    ref="interactElement"
    :style="{
      transform: transformString,
      transition: transitionString,
      touchAction: 'none',
    }"
  >
    <slot />
  </div>
</template>

<script>
import interact from "interactjs";

const INTERACT_ON_START = "start";
const INTERACT_ON_MOVE = "move";
const INTERACT_ON_END = "end";

const SWIPE_LEFT = "swipe-left";
const SWIPE_RIGHT = "swipe-right";
const SWIPE_TOP = "swipe-top";
const SWIPE_BOTTOM = "swipe-bottom";
const SWIPE_ANY = "swipe";

export default {
  name: "Swipeable",
  props: {
    transition: {
      type: String,
      default: "transform 0.5s cubic-bezier(0.2, 0.8, 0.4, 1.2)",
      required: false,
    },
    maxRotation: {
      type: Number,
      default: 15,
      required: false,
    },
    outOfSightXOffset: {
      type: Number,
      default: 500,
      required: false,
    },
    outOfSightYOffset: {
      type: Number,
      default: 500,
      required: false,
    },
    thresholdX: {
      type: Number,
      default: 50,
      required: false,
    },
    thresholdY: {
      type: Number,
      default: 70,
      required: false,
    },
  },
  data() {
    return {
      isDragging: true,
      interactPosition: {
        x: 0,
        y: 0,
        rotation: 0,
      },
    };
  },
  computed: {
    transformString() {
      const { x, y, rotation } = this.interactPosition;
      return `translate3D(${x}px, ${y}px, 0) rotate(${rotation}deg)`;
    },
    transitionString() {
      return !this.isDragging && this.$props.transition;
    },
  },
  mounted() {
    const element = this.$refs.interactElement;
    interact(element).draggable({
      onstart: () => {
        this.$emit(INTERACT_ON_START);
        this.isDragging = true;
      },
      onmove: (event) => {
        this.$emit(INTERACT_ON_MOVE);
        const { maxRotation, thresholdX } = this.$props;
        const x = this.interactPosition.x + event.dx;
        const y = this.interactPosition.y + event.dy;
        let rotation = maxRotation * (x / thresholdX);
        if (rotation > maxRotation) rotation = maxRotation;
        else if (rotation < -maxRotation) rotation = -maxRotation;

        this.setPosition({ x, y, rotation });
      },
      onend: () => {
        this.$emit(INTERACT_ON_END);
        const { x, y } = this.interactPosition;
        const { thresholdX, thresholdY } = this.$props;
        this.isDragging = false;

        if (x > thresholdX) this.onThresholdReached(SWIPE_RIGHT);
        else if (x < -thresholdX) this.onThresholdReached(SWIPE_LEFT);
        else if (y < -thresholdY) this.onThresholdReached(SWIPE_TOP);
        else if (y > thresholdY) this.onThresholdReached(SWIPE_BOTTOM);
        else this.setPosition({ x: 0, y: 0, rotation: 0 });
      },
    });
  },
  beforeDestroy() {
    this.unsetInteractElement();
  },
  methods: {
    onThresholdReached(interaction) {
      const { outOfSightXOffset, outOfSightYOffset, maxRotation } = this.$props;
      this.unsetInteractElement();
      switch (interaction) {
        case SWIPE_RIGHT:
          this.setPosition({
            x: outOfSightXOffset,
            rotation: maxRotation,
          });
          this.$emit(SWIPE_RIGHT);
          break;
        case SWIPE_LEFT:
          this.setPosition({
            x: -outOfSightXOffset,
            rotation: -maxRotation,
          });
          this.$emit(SWIPE_LEFT);
          break;
        case SWIPE_TOP:
          this.setPosition({
            y: -outOfSightYOffset,
          });
          this.$emit(SWIPE_TOP);
          break;
        case SWIPE_BOTTOM:
          this.setPosition({
            y: outOfSightYOffset,
          });
          this.$emit(SWIPE_BOTTOM);
          break;
      }
      this.$emit(SWIPE_ANY, interaction);
    },
    setPosition(position) {
      const { x = 0, y = 0, rotation = 0 } = position;
      this.interactPosition = { x, y, rotation };
    },
    unsetInteractElement() {
      interact(this.$refs.interactElement).unset();
    },
  },
};
</script>
