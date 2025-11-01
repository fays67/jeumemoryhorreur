<template>
  <div class="jeu">
    <!-- 🎥 Fond vidéo -->
    <video autoplay muted loop class="fond-video">
      <source :src="zombie" type="video/mp4" />
    </video>

    <h1 class="titre">🎃 Jeu de mémoire Halloween 🎃</h1>

    <div class="infos">
      <p>Niveau : <span>{{ niveauNom }}</span></p>
      <p>Coups : <span>{{ coups }}</span> | Temps : <span>{{ temps }}</span></p>
    </div>

    <!-- 🃏 Grille -->
    <div class="grille" :class="niveauNom.toLowerCase()">
      <div v-for="(carte, i) in cartes" :key="i" class="carte"
           :class="{ retournee: carte.visible }"
           @click="retourner(i)">
        <div class="interieur">
          <div class="avant"><img :src="citrouille" /></div>
          <div class="arriere" :style="{ backgroundColor: carte.couleur }">
            <img :src="carte.image" />
          </div>
        </div>
      </div>
    </div>

    <!-- 🔘 Boutons -->
    <div class="boutons">
      <button class="btn btn-orange" @click="recommencer">🔄 Rejouer</button>
      <button class="btn btn-rouge" @click="toutRecommencer">🏠 Recommencer du début</button>
    </div>

    <!-- 🧠 Message -->
    <p v-if="victoire" class="message">🎉 Bravo ! Niveau {{ niveauNom }} terminé ! 🎉</p>

    <!-- 😱 Screamer -->
    <div v-if="screamerVisible" class="screamer">
      <img :src="screamerImg" />
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";

// 📦 Imports
import citrouille from "@/asset/citrouille.png";
import sorcier from "@/asset/sorcier.png";
import pingouin from "@/asset/pingouin.png";
import saw from "@/asset/saw.png";
import elmonstro from "@/asset/elmonstro.png";
import monstre from "@/asset/monstre.png";
import chapeau from "@/asset/chapeau.png";
import chat from "@/asset/chat.jpg";
import faucheuse from "@/asset/faucheuse.png";
import momie from "@/asset/momie.png";
import fantome from "@/asset/fantome.png";
import screamerImg from "@/asset/screamer.png";
import screamSon from "@/asset/scream.mp3";
import zombie from "@/asset/zombie.mp4.mp4";
import musiqueFond from "@/asset/fondsonore.mp3";

// 🎧 Musique de fond
const fond = new Audio(musiqueFond);
fond.loop = true;
fond.volume = 0.2;

// 🧩 Variables du jeu
const niveau = ref(0);
const niveauNom = ref("Facile");
const cartes = ref([]);
const visibles = ref([]);
const coups = ref(0);
const secondes = ref(0);
const temps = ref("0:00");
const victoire = ref(false);
const screamerVisible = ref(false);
let chrono = null;
let screamerTimer = null;

// 🔀 Mélanger un tableau
function melanger(tab) {
  return tab.sort(() => Math.random() - 0.5);
}

// 🕒 Chronomètre
function chronoStart() {
  chrono = setInterval(() => {
    secondes.value++;
    const m = Math.floor(secondes.value / 60);
    const s = secondes.value % 60;
    temps.value = `${m}:${s < 10 ? "0" + s : s}`;

    // 😱 Screamer après 15 sec
    if (secondes.value === 30) {
      new Audio(screamSon).play();
      screamerTimer = setTimeout(() => {
        screamerVisible.value = true;
        setTimeout(() => (screamerVisible.value = false), 1000);
      }, 500);
    }
  }, 1000);
}

function chronoStop() {
  clearInterval(chrono);
  clearTimeout(screamerTimer);
}

// 🧠 Créer les cartes selon le niveau
function creerCartes() {
  const niveaux = [
    [ // facile
      { nom: "Pingouin", image: pingouin, couleur: "#6a0dad" },
      { nom: "Sorcier", image: sorcier, couleur: "#0b1e5b" },
    ],
    [ // moyen
      { nom: "Pingouin", image: pingouin, couleur: "#6a0dad" },
      { nom: "Sorcier", image: sorcier, couleur: "#0b1e5b" },
      { nom: "Saw", image: saw, couleur: "#8b0000" },
      { nom: "ElMonstro", image: elmonstro, couleur: "#4b0082" },
    ],
    [ // difficile
      { nom: "Pingouin", image: pingouin, couleur: "#6a0dad" },
      { nom: "Sorcier", image: sorcier, couleur: "#0b1e5b" },
      { nom: "Saw", image: saw, couleur: "#8b0000" },
      { nom: "ElMonstro", image: elmonstro, couleur: "#4b0082" },
      { nom: "Fantome", image: fantome, couleur: "yellow" },
      { nom: "Chapeau", image: chapeau, couleur: "#ff7f00" },
      { nom: "Chat", image: chat, couleur: "#6a0dad" },
      { nom: "Monstre", image: monstre, couleur: "#0b1e5b" },
      { nom: "Faucheuse", image: faucheuse, couleur: "#8b0000" },
      { nom: "Momie", image: momie, couleur: "#4b0082" },
    ],
  ];

  niveauNom.value = ["Facile", "Moyen", "Difficile"][niveau.value];
  const base = niveaux[niveau.value];
  const toutes = base.flatMap((c) => [{ ...c, visible: false }, { ...c, visible: false }]);
  return melanger(toutes);
}

// 🎮 Initialiser le jeu
function init() {
  cartes.value = creerCartes();
  visibles.value = [];
  coups.value = 0;
  secondes.value = 0;
  temps.value = "0:00";
  victoire.value = false;
  chronoStop();
  fond.play().catch(() => console.log("Clique pour activer le son"));
}

// 🎴 Retourner une carte
function retourner(i) {
  const c = cartes.value[i];
  if (c.visible || visibles.value.length === 2) return;
  if (coups.value === 0 && secondes.value === 0) chronoStart();

  c.visible = true;
  visibles.value.push(c);
  if (visibles.value.length === 2) {
    coups.value++;
    if (visibles.value[0].nom === visibles.value[1].nom) {
      visibles.value = [];
      if (cartes.value.every((x) => x.visible)) {
        victoire.value = true;
        chronoStop();
        setTimeout(() => {
          if (niveau.value < 2) niveau.value++;
          init();
        }, 2000);
      }
    } else {
      setTimeout(() => {
        visibles.value.forEach((x) => (x.visible = false));
        visibles.value = [];
      }, 1200);
    }
  }
}

function recommencer() { init(); }
function toutRecommencer() { niveau.value = 0; init(); }

init();
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Creepster&display=swap');

.jeu {
  text-align: center;
  color: #ff7518;
  font-family: 'Creepster', cursive;
  text-shadow: 2px 2px 8px #000;
  min-height: 100vh;
  overflow: hidden;
}

.fond-video {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
  z-index: -1;
  filter: brightness(0.4);
}

.titre {
  font-size: 2.5em;
  margin: 20px 0;
  color: #ff7518;
  text-shadow: 3px 3px 10px #b30000;
}

.infos {
  font-size: 1.3em;
  margin-bottom: 20px;
  color: #fff;
}

/* Grille de cartes */
.grille { display: grid; gap: 12px; justify-content: center; }
.grille.facile { grid-template-columns: repeat(2, 110px); }
.grille.moyen { grid-template-columns: repeat(4, 110px); }
.grille.difficile { grid-template-columns: repeat(5, 110px); }

.carte {
  width: 110px;
  height: 110px;
  perspective: 1000px;
}
.interieur {
  width: 100%; height: 100%;
  position: relative;
  transform-style: preserve-3d;
  transition: 0.6s;
}
.carte.retournee .interieur { transform: rotateY(180deg); }
.avant, .arriere {
  position: absolute;
  width: 100%; height: 100%;
  border-radius: 10px;
  backface-visibility: hidden;
  overflow: hidden;
}
.avant { background: #ff7518; box-shadow: 0 0 15px #ff7518; }
.arriere {
  transform: rotateY(180deg);
  display: flex; justify-content: center; align-items: center;
  box-shadow: 0 0 20px #000 inset;
}
img { width: 90%; height: 90%; object-fit: contain; }

/* Boutons */
.boutons {
  margin-top: 25px;
  display: flex;
  justify-content: center;
  gap: 15px;
}

.btn {
  font-family: 'Creepster', cursive;
  font-size: 1.3em;
  padding: 10px 25px;
  border: none;
  border-radius: 8px;
  color: #fff;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 0 10px #000;
}

.btn-orange {
  background: linear-gradient(145deg, #ff7518, #e85c00);
}
.btn-orange:hover {
  transform: scale(1.05);
  box-shadow: 0 0 20px #ff7518;
}

.btn-rouge {
  background: linear-gradient(145deg, #b30000, #800000);
}
.btn-rouge:hover {
  transform: scale(1.05);
  box-shadow: 0 0 20px red;
}

.message {
  margin-top: 20px;
  color: #00ff00;
  font-size: 1.8em;
  text-shadow: 2px 2px 10px black;
  animation: pulse 1s infinite alternate;
}

@keyframes pulse {
  from { transform: scale(1); color: #00ff00; }
  to { transform: scale(1.05); color: #aaffaa; }
}

/* Screamer */
.screamer {
  position: fixed;
  top: 0; left: 0;
  width: 100%; height: 100%;
  background: black;
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 999;
}
</style>
