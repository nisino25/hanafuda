<template>
    <div class="playerInfo" :id="'player-'+player.name" ref="playerInfoRef">
        <div
            class="player-box"
            :class="{ active: isActive }"
        >
            <div class="name-container">
                <p>{{ player.name }}</p>
            </div>
            <div class="player-image-container relative">
                <div class="temp-image">
                    <div v-html="regenerate(player.randomString)"></div>
                </div>
                <div
                    class="player-card-num-container absolute right-0 bottom-0 bg-black rounded-tl-full aspect-square w-[35%] flex items-center justify-center"
                >
                    <small class="text-sm translate-x-[2.5px]">{{ countHandsLength() }}</small>
                </div>
            </div>
        </div>
        <div v-if="currentSelectedPlyaer?.name == player.name" class="player-action-mask" ></div>
    </div>
</template>

<script>
export default {
    props: {
        player: { type: Object, required: false },
        isActive: { type: Boolean, required: false },
        deck: { type: Array, required: false },
        isYourPlayer: { type: Boolean, required: false },
        currentSelectedPlyaer: {type: Object, required: false}
    },
    mounted() {
        this.setPlayerBoxWidth();
        window.addEventListener('resize', this.setPlayerBoxWidth);
    },
    beforeUnmount() {
        window.removeEventListener('resize', this.setPlayerBoxWidth);
    },
    methods: {
        regenerate(randomString) {
            return window.multiavatar(randomString);
        },
        setPlayerBoxWidth() {
            if (!this.isYourPlayer) return;

            this.$nextTick(() => {
                const otherPlayer = document.querySelector(
                    '.other-players-area .playerInfo'
                );
                if (otherPlayer) {
                    this.playerBoxWidth = otherPlayer.clientWidth;
                }
            });
        },
        countHandsLength() {
              if (!this.player || !this.deck) return 0;

              return this.deck.reduce((count, card) => {
                    if (card.location === this.player.name && !card.isCaptured) {
                          count++;
                    }
                    return count;
              }, 0);
        },

    }
};
</script>
