<template>
  <div id="app">
    <div class="inner">

      <div v-if="currentPage === 'before'">
        <div ref="scrollWrapper" class="w-full h-[100vh] overflow-hidden relative">
            <div ref="scrollArea" class="absolute top-0 w-full">
                <div class="grid grid-cols-4 gap-1">
                    <div
                        v-for="card in duplicatedDeck"
                        :key="card.id + '-' + card.dup"
                        class="relative flex justify-center pointer-events-none"
                    >
                        <img
                            :src="`./img/hanafuda-img/${card.id}.webp`"
                            alt=""
                            class="w-[120px] aspect-[2/3] object-cover select-none"
                            draggable="false"
                        />
                        <div class="absolute inset-0 bg-black/40 pointer-events-none"></div>
                    </div>
                </div>
            </div>
        </div>



        <div class="bg-slate-50 text-slate-800 p-4 rounded-lg shadow-md w-[85%] max-w-[400px] m-auto absolute top-1/2 left-1/2 transform -translate-x-1/2 -translate-y-1/2">

          <div v-if="!roomOption">
            <button @click="roomOption = 'create'; devSkip('create');" class="bg-[#3581B8] text-white  block w-1/2 mx-auto p-[10px] rounded">Create a room</button>
            <hr class="my-3">
            <button @click="roomOption = 'join'; retriveCode(); devSkip('join');" class="bg-[#13563B] text-white block w-1/2 mx-auto p-[10px] rounded">Join a room</button>
          </div>
          <template v-if="roomOption && !roomCode">
            <h2>Type your name</h2>
              <div class="flex items-center mb-3 border border-gray-300 rounded overflow-hidden">
              <input
                type="text"
                class="flex-grow p-2 outline-none"
                v-model="username"
                placeholder="Enter your username"
              >
              <button
                class="p-2 text-gray-600 hover:text-gray-900"
                @click="username = getRandomName()"
              >
                <i class="fa-solid fa-shuffle"></i>
              </button>
            </div>

              <div class="w-[95%] mx-auto grid grid-cols-5 gap-3 justify-between mb-5">
                <div v-for="(avatar, index) in avatars" :key="index" @click="randomString = avatar.randomString" v-html="avatar.avatar"  :style="{ opacity:  randomString !== avatar.randomString ? '0.6' : '1' }"></div>

                <div class="text-3xl text-gray-600 flex justify-center items-center text-center" @click="generateAvatars('female')">
                  <i class="fas fa-sync"></i>
                </div>
              </div>
              <div class="flex items-center mb-2" v-if="roomOption === 'join'">
                <input type="number" v-model="tempRoomcode" placeholder="Type room code" class="flex-grow p-2 border border-gray-300 rounded">
              </div>
              <!-- <button v-if="readyToPlay" @click="randomName()" class="add-button">ランダム</button> -->
              <button @click="roomOption = null" class="bg-[#B83A4B] text-white block mx-auto p-[10px] rounded w-1/2 mb-2">Back</button>
              <button @click="username = getRandomName();" class="bg-[black] text-white block mx-auto p-[10px] rounded mb-2 w-1/2">Random Name</button>
              <button v-if="readyToPlay && roomOption === 'create'" @click="createARoom()" class="bg-[#3581B8] text-white  block  mx-auto p-[10px] rounded w-1/2 mb-2">Create</button>
              <button v-if="tempRoomcode >= 10000 && tempRoomcode <= 99999 && readyToPlay && roomOption === 'join'" @click="joinARoom()" class="bg-[#3581B8] text-white  block mx-auto p-[10px] rounded w-1/2">Join</button>
          </template>

          <template v-if="roomOption && roomCode">
            <h2>Welcome {{ username }}!</h2>
            <p>Room code: <strong class="font-size: 2.5em; color: crimson; margin-right: 5px; font-weight: bold;">{{ roomCode }}</strong></p>
            <hr>
            <template v-for="(player, index) in players" :key="index">
              <div class="player-list flex items-center gap-2 my-2">
                <span>{{index +1}}.</span>
                <div v-html="regenerate(player.randomString)"></div>
                <p>{{ player.name }}</p>
              </div>
            </template>
            <button v-if="players?.length == 2 && yourPlayer.isHost"  @click="closeTheRoom()" class="bg-[#3581B8] text-white  block  mx-auto p-[10px] rounded w-1/2 mb-2">Close room</button>
          </template>
        </div>
      </div>

      <div v-if="currentPage === 'game'">
        <div class="main-container">
          <div class="area-container">

            <div class="other-players-area">
              <div class="other-player relative p-2 w-full flex">
                <PlayerInfo
                  :player="otherPlayer"
                  :isActive="otherPlayer == currentPlayer && onlineStatus == 'playing'"
                  :deck="deck"
                />

                <div class="personal-revealed-area">
                  <div class="revealed-container">
                    <template v-for="(card) in capturedCard(otherPlayer.name)" :key="card.id">
                      <div 
                        class="gameCard" 
                        :id="'card-'+card?.id" 
                      >
                        <div class="card-inner">
                          <div class="card-front">
                            <img
                              :src="`./img/hanafuda-img/${card.id}.webp`"
                              alt=""
                              class="object-cover select-none"
                              draggable="false"
                            />
                          </div>
                        </div>
                      </div>
                    </template> 
                  </div>
                </div>
              </div>                 
                
              
            </div>

            <div class="public-area p-2">
              <div class="temp-block">
                <div class="revealed-container">
                  <div 
                    class="draw-pile gameCard"
                    v-if="drawPiles?.length > 0 && !flippedDrawPile"
                    @click="drawPile()"
                    :class="{ 'readyToDraw': canIDraw }"
                    >
                    <div
                        class="player-card-num-container absolute right-0 bottom-0 bg-black rounded-tl-full aspect-square w-[55%] flex items-center justify-center"
                    >
                      <small class="text-sm translate-x-[2.5px] text-white">{{ drawPiles?.length }}</small>
                    </div>
                  </div>
                  <div 
                    class="draw-pile gameCard readyToDraw"
                    v-if="flippedDrawPile"
                    @click="giveUpPlacing()"
                    >
                    <div class="card-inner">
                      <div class="card-front">
                        <img
                          :src="`./img/hanafuda-img/${flippedDrawPile.id}.webp`"
                          alt=""
                          class="object-cover select-none"
                          draggable="false"
                        />
                    </div>  
                    </div>
                  </div>
                  <template v-for="(card) in revealedCards" :key="card.id">
                    <div 
                      class="gameCard" 
                      :id="'card-'+card?.id" 
                      :class="[
                        { 
                          'opacity-0': card.isCaptured,
                          'selected-card': selectedCard?.id === card.id
                        }
                      ]"
                    >

                      <div class="card-inner">
                        <div class="card-front">
                          <img
                              :src="`./img/hanafuda-img/${card.id}.webp`"
                              alt=""
                              class="object-cover select-none"
                              draggable="false"
                          />
                        </div>
                      </div>

                      <!-- Click to select -->
                      <div class="absolute top-0 left-0 w-full h-full cursor-pointer" @click="selectCard(card)"></div>
                    </div>
                    
                  </template>
                </div>
              </div>
            </div>


            <div class="personal-info">
                <!-- <div class="my-player"> -->
                <div class="my-player" :class="{ 'player-mask': currentPlayer !== yourPlayer || onlineStatus !== 'playing' }">
                  <PlayerInfo
                    :player="yourPlayer"
                    :isActive="yourPlayer == currentPlayer && onlineStatus == 'playing'"
                    :deck="deck"
                    :isYourPlayer="true"
                    :currentSelectedPlyaer="selectedPlyaer"
                    />
                  <h5 class="absolute top-full">
                    <i @click="reloadPage()" class="fa fa-refresh" aria-hidden="true"></i>
                    <i v-if="yourPlayer.isHost" @click="resetGame()" class="ml-2 fa-solid fa-trash"></i>
                  </h5>
                </div>

  
                <div class="personal-revealed-area">
                  <div class="revealed-container">
                    <template v-for="(card) in capturedCard(this.yourPlayer.name)" :key="card.id">
                      <div 
                        class="gameCard" 
                        :id="'card-'+card?.id" 
                      >
                        <div class="card-inner">
                          <div class="card-front">
                            <img
                              :src="`./img/hanafuda-img/${card.id}.webp`"
                              alt=""
                              class="object-cover select-none"
                              draggable="false"
                            />
                          </div>
                        </div>
                      </div>
                    </template> 
                  </div>
                </div>
            </div>
            
            <div class="personal-area" :class="{ 'player-mask': currentPlayer !== yourPlayer || onlineStatus !== 'playing' }">
              <div class="cards-container">
                <template v-for="(card,index) in sortedYourPlayerHand" :key="card.id">
                  <div 
                    @click="selectCard(card)"
                    class="gameCard" 
                    :class="[
                      { 'selected-card': selectedCard?.id === card.id, }
                    ]"
                    :style="getCardStyle(index)"
                    :id="'card-'+card?.id"
                  >
                    <div class="card-inner">
                      <div class="card-front">
                        <img
                          :src="`./img/hanafuda-img/${card.id}.webp`"
                          alt=""
                          class="object-cover select-none"
                          draggable="false"
                        />
                      </div>
                    </div>
                  </div>
                </template>
              </div>
            </div>

          </div>
        </div>
      </div>

    </div>
  </div>
</template>

<script>

import db from './firebase.js';


import { randomNames } from './name.js';
import PlayerInfo from './components/PlayerInfo.vue';
export default {
  components: {
    PlayerInfo,
  },
  data() {
    return {
      // developingMode: false,
      developingMode: true,

      defaultNumber: 2,
      maxPlayerNumber: 6,
      randomNames,


      currentPage: 'before',
      players: [],
      currentPlayerIndex: 0,
      

      deck:[],
      lastSubmitBy: null,
      publicPile: [],

      selectedCard: null,

      tempCards: [],

      selectedPlyaer: null,

      // ----------------
      roomOption: null,
      roomCode: null,
      tempRoomcode: null,
      generalData: null,

      username: null,
      onlineStatus: '',

      processedAudioEvents: [],
      publicAudioEvents: [],

      gameResults: [],
      previousGameResults: [],

      pickedRandomString: null,
      avatars: [],
      randomString: null,

      isCheckingNow: false,
      gameMessage: "",

      canIDraw: false,
      finishedMove: false,

      // -----
      duplicatedDeck: [], 
      speed: .25,
      intervalId: null

    };
  },
  computed: {
    readyToPlay() {
      const namePattern = /^[^\s!@#$%^&*(),.?":{}|<>]+$/;

      // Check if the username is valid
      return namePattern.test(this.username) && this.username?.trim() !== '';
    },
    currentPlayer() {
      return this.players[this.currentPlayerIndex];
    },
    
    readyToSubmit() {

      // If the player's picked hands are empty, return false
      if (this.yourPlayerPickedHands.length === 0) return false;

      // --------------------------------------------------------------------

      // If the public pile is empty
      if (this.publicPile.length === 0) {

        // Single card play
        if (this.yourPlayerPickedHands.length === 1) return true;

        // Check if all cards are the same value
        const sameCardCheck = this.yourPlayerPickedHands.every(
            card => card.value === this.yourPlayerPickedHands[0].value
        );
        if (sameCardCheck) return true;

        // Check for sequence (階段)
        if(this.yourPlayerPickedHands.length < 3) return false
        const tempSuit = this.yourPlayerPickedHands[0].suit;
        let tempValue = this.yourPlayerPickedHands[0].value - 1;

        for (let card of this.yourPlayerPickedHands) {
          if (tempSuit !== card.suit || card.value !== ++tempValue) {
              return false;
          }
        }

        return true;
      }

      // --------------------------------------------------------------------
      let wasPreviousRevolution = this.previousCards.length === 4 && this.previousCards.every(card => card.value === this.previousCards[0].value);

      if(wasPreviousRevolution){
        // Check if all cards are the same value
        const sameCardCheck = this.yourPlayerPickedHands.every(
          card => card.value === this.yourPlayerPickedHands[0].value
        );

        if(sameCardCheck){
          // simple case
          const previousValue = this.isRevolutionGoing ? this.previousCards[0].revolutionValue : this.previousCards[0].value;
          const currentValue = this.isRevolutionGoing ? this.yourPlayerPickedHands[0].revolutionValue : this.yourPlayerPickedHands[0].value;
  
          if (currentValue <= previousValue) {
            return false;
          }

          return true
        }

        // stairs

        // revolution
      }

      // --------------------------------------------------------------------

      // Default regular checking

      // Check stairs
      if (this.yourPlayerPickedHands.length >= this.previousCards.length && this.yourPlayerPickedHands.length >= 3) {
          let stairCheck = true;
          const tempSuit = this.yourPlayerPickedHands[0].suit;
          let tempValue = this.isRevolutionGoing ? this.yourPlayerPickedHands[0].revolutionValue + 1 : this.yourPlayerPickedHands[0].value - 1;

          for (let card of this.yourPlayerPickedHands) {
              const cardValue = this.isRevolutionGoing ? card.revolutionValue : card.value;
              if (tempSuit !== card.suit || (this.isRevolutionGoing ? cardValue !== --tempValue : cardValue !== ++tempValue)) {
                  stairCheck = false;
                  break;
              }
          }

          if (stairCheck) return true;
      }



      // Check if all cards are the same value
      const sameCardCheck = this.yourPlayerPickedHands.every(card => card.value === this.yourPlayerPickedHands[0].value);
      if (!sameCardCheck) return false;

      // Check if the length matches the previous cards
      if (this.yourPlayerPickedHands.length < this.previousCards.length && !wasPreviousRevolution) return false;

      // Check if the current cards' value is higher (or lower in revolution) than the previous cards
      const previousValue = this.isRevolutionGoing || this.isTempRevolutionGoing ? this.previousCards[0].revolutionValue : this.previousCards[0].value;
      const currentValue = this.isRevolutionGoing || this.isTempRevolutionGoing ? this.yourPlayerPickedHands[0].revolutionValue : this.yourPlayerPickedHands[0].value;

      if (currentValue <= previousValue) {
        return false;
      }

      return true;
    },

    drawPiles() {
      // Flatten all players' hands and filter only revealed cards
      return this.deck
        .filter(card => card?.location === "publicPile" && !card.isCaptured)
    },
    
    flippedDrawPile() {
        return this.deck.find(card => card?.location === "publicPile" && card.isFliped);
    },

    revealedCards() {
      // Flatten all players' hands and filter only revealed cards
      return this.deck
          .filter(card => card?.location === "revealedArea" && !card.isCaptured)
        .sort((a, b) => a.number - b.number);
    },

    yourPlayer() {
      return this.players.find(player => player.name === this.username);
    },
    otherPlayer() {
      return this.players.find(player => player.name !== this.username);
    },
    sortedYourPlayerHand() {
      if (!this.yourPlayer) return [];
      return this.deck
        .filter(card => card.location === this.yourPlayer.name && !card.isCaptured)
        .sort((a, b) => a.number - b.number);
    },

  },
  methods: {
    sleep(ms) {
      return new Promise(resolve => setTimeout(resolve, ms));
    },
    reloadPage() {
      if (confirm('Are you sure you want to reload the page?')) {
        location.reload();
      }
    },
    resetGame() {
      if (confirm('Are you sure you want to reset the page?')) {
        this.closeTheRoom()
      }
    },
    shuffleArray(array) {
      for (let i = array.length - 1; i > 0; i--) {
          const j = Math.floor(Math.random() * (i + 1));
          [array[i], array[j]] = [array[j], array[i]];
      }
    },
    getRandomName() {
      let randomName;
      do {
        const randomIndex = Math.floor(Math.random() * this.randomNames.length);
        randomName = this.randomNames[randomIndex];
      } while (this.players.some(player => player.name === randomName));

      // this.generateAvatars();

      return randomName;
    },

    // =======================================================

    preInitalDeck(){
      // Build full deck (1..12, 4 copies each)
      this.deck = [];
      // let id = 1;

      for (let num = 1; num <= 12; num++) {
        for (let i = 1; i < 5; i++) {
          let tmepCard = {
            id: num * 10 + i,
            number: num,
            location: 'publicPile', // everything starts public
            isCaptured: false,
          }

          if([11,31,81,111,121].includes(tmepCard.id)) tmepCard.hikari = true;

          if([21,41,51,61,71,82,91,101,112].includes(tmepCard.id)) tmepCard.tane = true;

          if([12,22,32,42,52,62,72,92,102,113].includes(tmepCard.id)) tmepCard.tanzaku = true;



          this.deck.push(tmepCard);
        }
      }

      console.log(this.deck)
    },
    
    initializeDeck() {
      // Shuffle
      this.deck.sort(() => Math.random() - 0.5);

      // Deal to players (just mark location)
      this.players.forEach(player => {
        for (let i = 0; i < 8; i++) {
          const card = this.deck.find(c => c.location === 'publicPile');
          if (card) card.location = player.name;
        }
      });


      // Deal to revealed area (just mark location)
      for (let i = 0; i < 8; i++) {
        const card = this.deck.find(c => c.location === 'publicPile');
        if (card) {
          card.location = 'revealedArea';
        }
      }
    },

    getOtherPlayers() {
      const currentIndex = this.players.findIndex(player => player.name === this.yourPlayer.name);
      if (currentIndex === -1) return this.players; // Return the full list if currentPlayerName is not found

      const before = this.players.slice(0, currentIndex);
      const after = this.players.slice(currentIndex + 1);

      return after.concat(before);
    },

    async goToNextPlayer() {
        // Move to the next player, wrapping around if necessary
        this.currentPlayer.isRevealing = false
        this.currentPlayerIndex = (this.currentPlayerIndex + 1) % this.players.length;

    },
    async submit(){
      this.lastSubmitBy = this.currentPlayer.name

      await this.updatingData()


      // -----------------------------------------
      await this.updatingData()
      await this.sleep(550)

      for(let card of this.deck){
        if(card.location == 'moving'){
          card.location = 'publicArea'
        }
      }

      



      
      await this.updateAudio('card-submit')
      await this.goToNextPlayer()
      await this.updatingData()



    },
    getPileCardLocation(card){
      if(card.location == 'trash'){
        return `
          top: ${card.verticalPosition}%;
          left: ${card.horizontalPosition - 150}%;
          transform: rotate(${card.rotation}deg) translateX(${card.translateX}px);
          zIndex: ${card.zIndex};
        `;
      }
      return `
        top: ${card.verticalPosition}%;
        left: ${card.horizontalPosition}%;
        transform: rotate(${card.rotation}deg) translateX(${card.translateX}px);
        zIndex: ${card.zIndex};
      `;
    },

    flipCard(cardId) {
      const card = this.publicPile?.find(c => c.id === cardId);
      if (card) card.isFliped = true;
    },

    selectCard(card) {
      if(this.yourPlayer !== this.currentPlayer) return

      if(card.isCaptured){
        return
      }

      if(this.flippedDrawPile && card.location == 'revealedArea'){
        if(this.flippedDrawPile.number == card.number){
          // card.num = 100
          // this.flippedDrawPile.num = 110
          // capture both
          card.isCaptured = true;
          card.location = this.yourPlayer.name
          card.belongsTo = this.yourPlayer.name

          this.flippedDrawPile.isCaptured = true;
          this.flippedDrawPile.belongsTo = this.yourPlayer.name
          this.flippedDrawPile.location = this.yourPlayer.name

          this.goToNextPlayer();
          this.updatingData();
        } return
      }

      if(card.id == this.selectedCard?.id){
        this.selectedCard = null
        return
      }

      if(!this.selectedCard) this.selectedCard = card


      if(card.location == this.selectedCard?.location){
        this.selectedCard = card
        return
      }

      if(card.number == this.selectedCard.number){
        card.isCaptured = true;
        card.belongsTo = this.yourPlayer.name

        this.selectedCard.isCaptured = true;
        this.selectedCard.belongsTo = this.yourPlayer.name

        this.selectedCard = null

        this.finishedMove = true;
        this.canIDraw = true;  
        this.updatingData();
        return
      }

      
    },
    capturedCard(playerName){
      return this.deck
        .filter(card => card.isCaptured && card.belongsTo === playerName)
        .sort((a, b) => a.number - b.number)

    },

    async submitProcess() {
      await this.updateAudio('card-submit')
      this.isCheckingNow = true;

      this.selectedCard = null;
      this.selectedPlyaer = null;
      this.updatingData();

      // check all is same number ---------------
      this.isCheckingNow = false;


      // const allSame = this.allRevealedCards?.every(
      //   card => card.number === this.allRevealedCards[0].number
      // );

      // if (!allSame) return this.cleanAndProceed();




      
      
      // this.updatingData();
      // await this.sleep(3000);



      // // keep the pile location

      // check if the game is over


      // is 7 included then over
      // see if it adds up to 7
      if(this.yourPlayer.captured.includes(7)) return this.updateResult();
      if(this.yourPlayer.captured.length == 3) return this.updateResult();
      
      const capturedA = this.yourPlayer.captured[0]
      const capturedB = this.yourPlayer.captured[1]
      
      if((capturedA + capturedB === 7) || (Math.abs(capturedA - capturedB) === 7)) return this.updateResult();

      // if not then reset then next player
      this.goToNextPlayer();
      this.updatingData();

      


    },

    async cleanAndProceed(){
      this.revealedCards.forEach(card => {
        card.location = "publicPile";
      });

      this.publicPile.forEach(card => {
        card.location = "publicPile";
      });
      await this.updateAudio('card-submit')

      this.goToNextPlayer();
      this.updatingData();
    },

    // ======================================================
    
    generateAvatar() {
      const randomString = Math.random().toString();
      return {
          avatar: window.multiavatar(randomString),
          randomString: randomString
      };
    },
    randomName(){
      this.username = this.getRandomName();
    },
    retriveCode(){
      this.tempRoomcode = localStorage.getItem('latestRoomCode') || 'No room code found'
    },
    async createARoom() {
      if (this.roomCode) return;
      if(!this.username) return;

      let isUnique = false;

      // Generate a unique room code
      while (!isUnique) {
        this.tempRoomcode = Math.floor(10000 + Math.random() * 90000);
        const docRef = db.collection('hanafuda-rooms').doc(`${this.tempRoomcode}`);
        const doc = await docRef.get();
        if (!doc.exists) {
          isUnique = true;
        }
      }

      this.roomCode = this.tempRoomcode
      localStorage.setItem('latestRoomCode', this.roomCode)
      
      this.players.push({name:this.username, isHost:true,randomString: this.randomString, isCaptured: []})

      localStorage.userName = this.userName

      const ref = db.collection('hanafuda-rooms')
      ref.doc(`${this.roomCode}`).set({
        games: JSON.stringify([{ gameStatus: 'waiting' }]),
        players: this.players,
        onlineStatus: 'waiting',
        deck: this.deck,
        publicAudio: [],
        publicAudioEvents: [],
      })

      this.publicAudioEvents = []
      // this.onlineStatus = 'waiting'
      await this.reciveTheData()
    },

    async joinARoom() {

      const docRef = db.collection('hanafuda-rooms').doc(`${this.tempRoomcode}`);

      try {
        const doc = await docRef.get();
        if (doc.exists) {
          if(doc.data().onlineStatus == 'playing') return alert('This room is closed.')

          this.players = doc.data().players;
          this.onlineStatus = doc.data().players;

          if (!this.players.includes(this.username)) this.players.push({name:this.username, isHost:false,randomString:  this.randomString, isCaptured: []});
          this.roomCode = this.tempRoomcode

          await docRef.update({
            players: this.players,
          });
          this.reciveTheData();
        } else {
          console.log('No such document!');
        }
      } catch (error) {
        console.log('Error getting document:', error);
      }
    },

    reciveTheData(){
      
      db.collection("hanafuda-rooms").doc(`${this.roomCode}`)
      .onSnapshot((doc) => {

        this.generalData = doc.data()
        
        // joining room and wait until it closes
        // if(this.currentPage == 'before'){
        this.onlineStatus = doc.data()?.onlineStatus
        this.players = doc.data()?.players

        this.publicAudioEvents = doc.data()?.publicAudioEvents
        if(this.publicAudioEvents?.length > 0){

          this.publicAudioEvents.forEach((event) => {
            if (!this.processedAudioEvents.find(e => e.timeStamp === event.timeStamp)) {
              this.playSound(event.fileName);
              this.processedAudioEvents.push(event);
            }
          });
        }

        if(this.onlineStatus == 'playing' || this.onlineStatus == 'distributing') {
          this.deck = doc.data().deck;


          this.lastSubmitBy = doc.data()?.lastSubmitBy


          // check if the game is overr


          this.currentPlayerIndex = doc.data().currentPlayerIndex
          this.currentPage = 'game'
          localStorage.setItem('latestRoomCode', null);

          
          this.isRevolutionGoing = doc.data().isRevolutionGoing
          this.isTempRevolutionGoing = doc.data().isTempRevolutionGoing

          this.gameResults = doc.data().gameResults

          

        }
      
      })
    },

    async closeTheRoom(){

      if(this.players.length <2) return

      this.shuffleArray(this.players)

      this.currentPlayerIndex = 0;
      this.currentPage = 'game';
      
      this.publicAudioEvents = []
      await this.updateAudio('card-shuffle')
      
      this.deck = []
      this.preInitalDeck()
      await this.initializeDeck()

      this.previousGameResults = this.gameResults
      this.gameResults = []

      this.onlineStatus = 'playing'
      const ref = db.collection('hanafuda-rooms')
      ref.doc(`${this.roomCode}`).update({
        gameResults: this.gameResults,
        previousGameResults: this.previousGameResults,
        players: this.players,
        onlineStatus: this.onlineStatus,
        currentPlayerIndex: this.currentPlayerIndex,
        publicAudioEvents: this.publicAudioEvents,
        deck: this.deck,
      })
    },
    updatingData(){
      const ref = db.collection('hanafuda-rooms')
      ref.doc(`${this.roomCode}`).update({
        // publicPile: this.publicPile,
        deck: this.deck,
        onlineStatus: this.onlineStatus,
        currentPlayerIndex: this.currentPlayerIndex,
        players: this.players,
        gameResults: this.gameResults,
      })
    },

    updateResult(){
      const ref = db.collection('hanafuda-rooms')
      this.gameResults.push(this.yourPlayer.name)
      ref.doc(`${this.roomCode}`).update({
        gameResults: this.gameResults,
        players: this.players,
        publicPile: this.publicPile,
      })
    },

    async updateAudio(fileName){
      await this.playSound(fileName);

      this.processedAudioEvents.push({timeStamp: Date.now(),fileName})

      const ref = db.collection('hanafuda-rooms')
      ref.doc(`${this.roomCode}`).update({
        publicAudioEvents : this.processedAudioEvents
      })
    },

    playSound(fileName) {
      try {
        const audio = new Audio(require(`@/assets/sounds/${fileName}.wav`));
        audio.play().catch(error => {
          console.error('Error playing sound:', error);
        });
      } catch (error) {
        console.error('Error loading sound file:', error);
      }
    },

    generateAvatars() {
      this.tempAvatarCode = null;

      this.avatars = [1, 2, 3, 4, 5,6,7,8,9].map(() => {
        const randomString = Math.random().toString();
        return {
          avatar: window.multiavatar(randomString),
          randomString: randomString
        };
      });
    },


    regenerate(randomString) {
      return window.multiavatar(randomString);
    },

    devSkip(mode){
      if(this.developingMode == false) return
      if(mode == 'create'){
        this.createARoom()
      }else if(mode == 'join'){   
        this.joinARoom();
      }
    },

    getCardStyle(index) {
      const total = this.sortedYourPlayerHand.length;

      if (total <= 1) {
          return { top: '50%', left: '0', position: 'absolute', transform: 'translateX(0%) tanslateY(-50%) translateY(-50%)' };
      }

      // Spread linearly from 0% → 100%
      const leftPercent = (index / (total - 1)) * 100;

      // TranslateX matches the left percent
      const translateXPercent = leftPercent;

      return {
          top: '50%',
          left: `${leftPercent}%`,
          position: 'absolute',
          transform: `translateX(-${translateXPercent}%) translateY(-50%)`,
      };
    },

    drawPile() {
      if(this.yourPlayer !== this.currentPlayer) return

      if(!this.canIDraw)  {
        const confirmGiveUp = confirm("Are you sure you want to skip?");
        if (!confirmGiveUp) return;
        this.goToNextPlayer();
        this.updatingData();
        return;
      }

      if(this.drawPiles.length == 0) return

      this.canIDraw = false;

      const card = this.drawPiles[0]
      card.isFliped = true;
      this.updatingData();
      // card.location = this.yourPlayer.name;  
      
      // this.goToNextPlayer(); 

    },

    giveUpPlacing() {
      if(this.yourPlayer !== this.currentPlayer) return
      if(!this.flippedDrawPile) return
      const confirmGiveUp = confirm("Are you sure you want to give up placing?");
      if (!confirmGiveUp) return;


      const card = this.flippedDrawPile
      card.isFliped = false;
      card.location = 'revealedArea';  
      
      // this.finishedMove = true;
      // this.canIDraw = false;  

      this.goToNextPlayer(); 
      this.updatingData();

    },  

    startAutoScroll() {
        const el = this.$refs.scrollArea
        // const wrapper = this.$refs.scrollWrapper
        this.intervalId = setInterval(() => {
            el.style.top = (parseFloat(el.style.top || '0') - this.speed) + 'px'

            // if scrolled past half → reset
            if (Math.abs(parseFloat(el.style.top)) >= el.scrollHeight / 2) {
                el.style.top = '0px'
            }
        }, 10)
    }
    


  },
  watch: {
    gameResults(newVal) {
      if (newVal && newVal.length > 0) {
        this.isCheckingNow = true;
        this.sleep(1000);
        alert(`${newVal[0]} won the game`);
        this.gameMessage = "GAME IS OVER"
      }
    }
  },
  async mounted(){
    console.clear()


    this.generateAvatars();
    this.randomString = this.avatars[0].randomString;

    this.deck = []
    this.lastSubmitBy = null
    this.currentPlayerIndex = 0

    this.preInitalDeck()
    this.duplicatedDeck = this.deck.concat(
        this.deck.map(c => ({ ...c, dup: true }))
    )
    // this.startAutoScroll()
    this.startAutoScroll()

    this.username = this.getRandomName();
  },
   beforeUnmount() {
      clearInterval(this.intervalId)
   },  
};
</script>

<style>


  :root {

    --game-card-width: 50px;

    --trash-card-width: 60px;
    --trash-card-height: var(--trash-card-width) * 6 / 5;

    --floor-surface: #F1F5F9;
    --floor-surface: #111827; 

    /* --game-card-height: calc(var(--game-card-width) * 6 / 5); */
  }

  * {
    touch-action: manipulation;
    -webkit-user-select: none;

    padding: 0;
    margin: 0;
  }

  html, body {
    overflow: hidden;
    height: 100%;
    
    margin: 0;
    padding: 0;

    font-family: 'Noto Sans JP', sans-serif;

    background: var(--floor-surface);
  }

  img{
    max-width: 100%;
  }

  h2 {
    margin: 0 auto 10px;
    font-size: 1.25em;
    text-align: center;
  }

  h5{
    margin-bottom: 5px;
    font-size: .8em;

    width: 100%;
    text-align: center;
  }

  /* --------------------------------- */

  .player-list svg{
    width: 50px;
    height: 50px;
  }
  /* ---------------------------------------- */

  .main-container{
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%,-50%);

    /* background: red; */


    height: 92.5vh;

    box-sizing: border-box;

    overflow: hidden;

    color: white;

    width: 100%;
    overflow: hidden;
  }

  .area-container{
    height: 100%;
    width: 90vw;

    margin: auto;

    max-width: 425px;
    max-height: 900px;


    display: grid;
    grid-template-rows: calc(18% - 5px) calc(32% - 5px)  calc(18% - 5px) calc(15% - 5px);
    
    align-content: space-between;
  }

  .area-container > *{
    /* overflow: hidden; */
    background: #2C3E50;

    display: flex;
    align-items: center;
    justify-content: center;

    border-radius: 10px;
  }

  .card-actions {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    height: 85%;

    display: flex;
    flex-wrap: wrap;
    align-items: center;
    /* gap: 10px; */
    font-size: 24px;
    color: white;
  }

  .card-actions i {
    width: 30px;
    text-align: center;
    cursor: pointer;
    padding: 8px;
    border-radius: 50%;
    font-size: 15px;
      
  }



  /* ---------------------------------------- */
  .playerInfo{
    position: relative;
    text-align: center;
    font-size: .7em;
  }

  .playerInfo .player-box{
    position: relative;
    border: 1px solid black;
    transition: border-color 0.5s ease-in-out, box-shadow 0.5s ease-in-out;
    
    width: 80px;
  }

  .playerInfo .player-box:before {
    content: '' !important;
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    width: 105%;
    height: 105%;
    box-sizing: border-box; /* Ensure the border is included in the element's size */
    pointer-events: none; /* Ensure the pseudo-element doesn't interfere with interactions */
    transition: opacity .5s ease-in; /* Smooth transition for the pseudo-element */
    background-color: rgba(0, 0, 0, 0.5);

    transition: all .3s ease-in-out;
  }

  .playerInfo .player-box.active:before {
    background-color: rgba(0, 0, 0, 0);
    border: 2px solid #DAA520; /* Change the color as needed */
  }

  .playerInfo .name-container {
    display: flex;
    align-items: center;
    justify-content: center;
    height: 1.5em; /* Adjust based on your font-size and line-height */
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap; /* Prevent line breaks */
    text-align: center; /* Center text horizontally */
    padding: 5px 3px;
  }
  .playerInfo .name-container p {
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap; /* Prevent line breaks */
    width: 100%; /* Ensure the span takes the full width */
    text-align: center; /* Center text horizontally */
  }
  .playerInfo .player-image-container{
    display: block;
    width: 100%;
    aspect-ratio: 1;
    /* background: red; */
  }
  .playerInfo .player-image-container .temp-image{
    display: block;
    margin: auto;
    /* width: calc(100% - 2px); */
    width: 100%;
    padding: 5px;
    aspect-ratio: 1;

    box-sizing: border-box;

    background: #5D6D7E;

    border: 2px solid transparent;
  }

  .playerInfo span{
    font-size: 2em;
    line-height: 1.25;

    font-weight: bold;

    text-shadow: 2px 2px 5px rgba(0, 0, 0, 0.5);
  }

  .draw-pile{
    background: url(/public/img/card-back.webp) center/cover no-repeat, linear-gradient(45deg, gray, #111) !important;
    background-size:
    80% auto,   /* image = 80% width */
    cover;      
    border: 2px solid black;
  }

  /* ---------------------------------------- */

  .revealed-container{
    position: relative;
    margin: auto;

    display: flex;
    flex-wrap: wrap;
    /* grid-template-columns: repeat(2, 1fr); */
    /* justify-content: space-between; */
    gap: 4.4px;
  }

  .public-area{
    overflow: hidden;
    display: block;
  }

  .public-arena > *{
    padding: 5px;
    background: #2C3E50;
    /* border-radius: 10px; */

    width: 100%;
    height: 100%;
  
  }

  .public-area .detailed-area{
    /* width: 80%; */
    padding: 5px;
    display: grid;
    align-content: space-around;
    justify-content: center;
    box-sizing: border-box;

    text-align: center;

    font-size: .9em;
  }
  

  .public-area .detailed-area .status-badge{
    display: block;
    margin: 0 auto;
    padding: 5px 10px;
    background: #DAA520;
    border-radius: 5px;
    color: black;
    transition: all .5s ease-in-out;

    filter: brightness(1.2);
  }

  .public-area .detailed-area .status-badge.undo-badge{
    filter: brightness(.5);
  }
  

  .gameCard{
    display: block;
    position: relative;
    /* border: 2px solid #111827; */
    /* border-radius: 7px; */
    /* position: absolute; */

    overflow: hidden;
    color: black;

    transition: all .3s ease-in-out;

    width: 50px;
    aspect-ratio: 150 / 242;
    /* height: 90px; */

    box-shadow: 0 8px 16px rgba(0, 0, 0, 0.25);
  }

  .selected-card{
    border-color: #DAA520;
    box-shadow: 0 0 15px #DAA520;
    transform: scale(1.1);
    z-index: 10;
  }

  .card-inner {
    width: 100%;
    height: 100%;
    position: relative;
    transform-style: preserve-3d;
    transition: transform 0.5s;
  }

  .gameCard.flipped .card-inner {
    transform: rotateY(-180deg);
  }

  .gameCard img{
    position: absolute;
    width: 100%;
    height: 100%;
  }

  .card-front{
    width: 100%;
    height: 100%;
    background: #fef6ee;
  }

  .card-front span{
    position: absolute;
    top: 46%;
    left: 50%;
    transform: translate(-50%,-50%);
    /* color: #FAEBD7; */
    font-size: 1.8em;
  }

  .card-front,
  .card-back {
    width: 100%;
    height: 100%;
    position: absolute;
    /* backface-visibility: hidden; */
    /* border: 1px solid #ccc; */
    box-sizing: border-box;
    
    /* border: 1px solid #000; */

    box-shadow: 2px 2px 1px #888888;
  }

  .card-back {
    transform: rotateY(-180deg);
  }

  /* ---------------------------------------- */

  .personal-info{
    display: grid !important;
    grid-template-columns: calc(30% - 5px) calc(70% - 5px);
    justify-content: space-between;

    background: unset !important;
    border-radius: unset !important;
  }

  .personal-info > *{
    padding: 5px;
    background: #2C3E50;
    border-radius: 10px;

    width: 100%;
    height: 100%;
  
  }

  .personal-revealed-area {
    position: relative;
    margin: auto;
    overflow: hidden;
    justify-content: space-between;
  }

  /* ---------------------------------------- */

  .personal-area{
    padding: 10px;
    position: relative;
    z-index: 2;
  }
  .my-player{
    position: relative;
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 2;
  }

  .my-player::before, .personal-area::before{
    content: '';
    position: absolute;
    top: 0%;
    left: 0%;

    width: 100%;
    height: 100%;
    background: black;
    opacity: 0;

    z-index: 5;

    border-radius: 10px;

    transition: all .5s ease-in-out;

    pointer-events: none;
  }

  .my-player::before{
    height: 150% !important;
  }

  .my-player::after{
    content: '';
    position: absolute;
    top: 75%;
    left: 0%;

    width: 100%;
    height: 70%;
    background: #2C3E50;
    /* opacity: 0; */

    z-index: -1;
  }


  .player-mask::before{
    opacity: .5 !important;
  }


  .my-player .card-actions{
    height: 75%;
  }

  .player-action-mask{
    position: absolute;
    display: block;
    top: 0%;
    left: 0%;
    width: 100%;
    height: 100%;

    background:black;

    opacity: .5 !important;
    transition: all .5s ease-in-out;
  }

  .personal-area .cards-container{
    position: relative;
    width: 100%;
    /* display: flex;
    flex-wrap: wrap;
    justify-content: center;
    align-items: center;
    gap: 5px; */
/* 
    height: 100%; */
    /* overflow-x: auto; */
  }

  .readyToDraw {
      border-color: #28a745;
      box-shadow: 0 0 15px #28a745;
      animation: pulse 1.5s infinite ease-in-out;
  }

  @keyframes pulse {
      0% {
          box-shadow: 0 0 10px #28a745;
          transform: scale(1);
      }
      50% {
          box-shadow: 0 0 20px #28a745;
          transform: scale(1.05);
      }
      100% {
          box-shadow: 0 0 10px #28a745;
          transform: scale(1);
      }
  }




</style>
