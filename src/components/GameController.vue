<template>
  <div v-if="selectedItem" class="mx-auto rounded-lg text-white">
    <div class="grid grid-cols-4 grid-rows-[50px,50px] gap-1">
      <button
        @click.prevent="handleClick('moveFurnitureItem', -increment, 0)"
        @touchstart="handleTouchStart('moveFurnitureItem', -increment, 0)"
        class="w-full font-semibold text-white bg-black rounded-lg active:bg-gray-800 controller-button border border-black text-xs"
      >
        <!-- <i class="fa fa-angle-double-down"></i> -->
        <font-awesome-icon
          :icon="['fas', 'arrow-right']"
          :style="{ transform: 'rotate(225deg)' }"
        />
      </button>

      <button
        @click.prevent="handleClick('moveFurnitureItem', 0, -increment)"
        @touchstart="handleTouchStart('moveFurnitureItem', 0, -increment)"
        class="w-full font-semibold text-white bg-black rounded-lg active:bg-gray-800 controller-button border border-black text-xs"
      >
        <font-awesome-icon
          :icon="['fas', 'arrow-right']"
          :style="{ transform: 'rotate(315deg)' }"
        />
      </button>

      <button
        @click="selectedItem.roomZ += increment"
        class="w-full font-semibold text-white bg-black rounded-lg active:bg-gray-800 controller-button border border-black text-xs"
      >
        <font-awesome-icon :icon="['fas', 'arrow-up']" />
        Up
      </button>
      <button
        @click="rotateFurni"
        class="w-full font-semibold text-white bg-black rounded-lg controller-button border border-black text-xs row-span-2 active:bg-gray-800 controller-button border border-black"
      >
        <font-awesome-icon :icon="['fas', 'arrow-rotate-left']" /> Rotate
      </button>
      <button
        @click.prevent="handleClick('moveFurnitureItem', 0, increment)"
        @touchstart="handleTouchStart('moveFurnitureItem', 0, increment)"
        class="w-full font-semibold text-white bg-black rounded-lg active:bg-gray-800 controller-button border border-black text-xs"
      >
        <font-awesome-icon
          :icon="['fas', 'arrow-right']"
          :style="{ transform: 'rotate(135deg)' }"
        />
      </button>
      <button
        @click.prevent="handleClick('moveFurnitureItem', increment, 0)"
        @touchstart="handleTouchStart('moveFurnitureItem', increment, 0)"
        class="w-full font-semibold text-white bg-black rounded-lg active:bg-gray-800 controller-button border border-black text-xs"
      >
        <font-awesome-icon
          :icon="['fas', 'arrow-right']"
          :style="{ transform: 'rotate(45deg)' }"
        />
      </button>

      <button
        @click="selectedItem.roomZ -= increment"
        class="w-full py-2 font-semibold text-white bg-black rounded-lg active:bg-gray-800 controller-button border border-black text-xs"
      >
        <font-awesome-icon :icon="['fas', 'arrow-down']" />
        Down
      </button>
    </div>

    <div class="text-xs mt-1 bg-black bg-opacity-20 rounded-lg p-3">
      <div class="grid grid-cols-[auto,1fr] gap-3">
        <div class="grid grid-cols-[auto,auto]">
          <img
            :src="getIconUrl(selectedItem.type)"
            class="max-w-full max-h-full object-contain w-[20px] h-[40px] mr-1"
          />
          <div>
            <p>{{ selectedItem.type }}</p>
            <p>
              X{{ selectedItem.roomX.toFixed(2) }} | Y{{
                selectedItem.roomY.toFixed(2)
              }}
              | Z:{{ selectedItem.roomZ.toFixed(2) }} | R{{
                selectedItem.direction
              }}
              | I{{ increment }} | A{{ selectedItem.animation }}
            </p>
          </div>
        </div>
        <!-- <p>Animation: {{ selectedItem.animation }}</p> -->
        <div class="flex-grow flex justify-end">
          <button
            @click="removeRoomItem"
            class="p-2 px-4 font-semibold text-black bg-white border-1 border border-black rounded text-xs hover:bg-gray-800"
          >
            <font-awesome-icon :icon="['fas', 'trash']" />
          </button>
        </div>
      </div>
    </div>

    <!-- Advanced Settings -->

    <button
      @click="isContentVisible = !isContentVisible || false"
      class="text-xs text-white p-2 w-full h-12"
    >
      Advanced Settings
    </button>
    <div
      :class="{ hidden: !isContentVisible }"
      class="rounded-lg text-xs text-white w-full mb-2"
    >
      <label
        class="flex-grow font-semibold text-white rounded-lg text-xs mb-1 mt-0 block"
      >
        <font-awesome-icon :icon="['fas', 'arrows-alt-h']" /> Increment
      </label>
      <div class="w-full flex space-x-1">
        <button
          @click="increment = 0.1"
          class="h-[42px] rounded-lg bg-black p-3 w-full font-semibold text-white text-xs active:bg-gray-800 controller-button border border-black"
        >
          1 Pixel
        </button>
        <button
          @click="increment = 0.5"
          class="h-[42px] rounded-lg bg-black p-3 w-full font-semibold text-white text-xs active:bg-gray-800 controller-button border border-black"
        >
          .5 Block
        </button>
        <button
          @click="increment = 1.0"
          class="h-[42px] rounded-lg bg-black p-3 w-full font-semibold text-white text-xs active:bg-gray-800 controller-button border border-black"
        >
          1 Block
        </button>
        <button
          @click="increment = 2.0"
          class="h-[42px] rounded-lg bg-black p-3 w-full font-semibold text-white text-xs active:bg-gray-800 controller-button border border-black"
        >
          2 Block
        </button>

        <!-- Move Room -->
      </div>
    </div>

    <button
      @click="game.unselectFurniture(selectedItem)"
      class="w-full py-3 mb-8 font-semibold bg-green-600 save-button text-white rounded-lg border border-1 border-b-2 border-black text-xs h-12"
    >
      <font-awesome-icon :icon="['fas', 'star']" /> Save
    </button>
  </div>
</template>

<script>
import { EventBus } from "@/eventBus";
import { FloorFurniture } from "@tetreum/shroom";

export default {
  name: "GameController",
  props: ["game"],
  data() {
    return {
      increment: 1,
      selectedItem: null,
      isContentVisible: false,
      touchLocked: false,
      actionInProgress: false,
    };
  },
  created() {
    EventBus.$on("item-selected", (item) => {
      this.selectedItem = item;
    });

    EventBus.$on("item-unselected", (item) => {
      console.log("item unselected", item);
      this.selectedItem = null;
      this.saveRoomToLocalStorage();
    });

    EventBus.$on("furni-added", () => {
      setTimeout(() => {
        this.saveRoomToLocalStorage();
      }, 1500);
    });
  },
  methods: {
    rotateFurni() {
      this.selectedItem.validDirections.then((directions) => {
        console.log(directions);
        if (directions.length > 0) {
          // Get the index of the current direction in the valid directions array
          const currentIndex = directions.indexOf(this.selectedItem.direction);

          // Check if we are at the last direction, if yes set it to first, else set it to the next direction
          this.selectedItem.direction =
            currentIndex < directions.length - 1
              ? directions[currentIndex + 1]
              : directions[0];
        }
      });
    },
    handleClick(action, ...args) {
      if (this.touchLockedClick) return;
      this[action](...args);
    },
    handleTouchStart(action, ...args) {
      this.touchLocked = true;

      setTimeout(() => (this.touchLocked = false), 500); // Set a delay to unlock.
      this[action](...args);
    },
    saveRoomToLocalStorage() {
      const roomData = this.game.getSerializedRoom();
      localStorage.setItem("savedRoom", JSON.stringify(roomData));
    },
    getIconUrl(type) {
      //Remove from here
      return `https://ducket.net/assets/furni/${type.replace(
        "*",
        "_"
      )}_icon.png`;
    },
    updateItem() {
      EventBus.$emit("update-item", this.selectedItem);
    },
    removeRoomItem() {
      EventBus.$emit("furni-removed", this.selectedItem);
      this.game.room.removeRoomObject(this.selectedItem);
      this.saveRoomToLocalStorage();
      this.selectedItem = null;

      //Emit
    },
    moveFurnitureItem(moveX, moveY) {
      if (this.actionInProgress) return;
      this.actionInProgress = true;

      // Make sure there is a selected furniture item
      if (this.selectedItem) {
        // Access the moveFurnitureItem method from the game object using $refs
        this.game.moveFurnitureItem(this.selectedItem, moveX, moveY);
      }

      window.setTimeout(() => {
        this.actionInProgress = false;
      }, 200);
    },
    clearRoomItems() {
      //Confirm
      if (!confirm("Are you sure you want to clear the room?")) return;
      //Remove all floor furniture
      this.game.room.roomObjects.forEach((object) => {
        if (object instanceof FloorFurniture) {
          this.game.room.removeRoomObject(object);
        }
      });
      this.saveRoomToLocalStorage();
      //reload page
      location.reload();
    },
  },
};
</script>
