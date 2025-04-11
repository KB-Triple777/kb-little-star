<template>
  <div class="select-container">
    <div class="logo-container">
      <img class="logo-img" src="@/assets/images/logo.png" alt="" />
    </div>

    <div class="pokemon-list">
      <img
        v-for="pokemon in startingPokemons"
        :key="pokemon.id"
        :src="pokemon.image"
        :alt="pokemon.name"
        :class="[
          'pokemon-img',
          {
            selected: selectedId === pokemon.id,
            disabled: isConfirmed && selectedId !== pokemon.id,
          },
        ]"
        :style="{
          pointerEvents:
            isConfirmed && selectedId !== pokemon.id ? 'none' : 'auto',
        }"
        @click="!isConfirmed && selectPokemon(pokemon.id)"
      />
    </div>

    <div class="start-prompt">
      <div class="prompt-box" v-html="content"></div>
    </div>

    <div class="bottom-button-container">
      <CustomButton
        v-if="isConfirmed"
        class="bottom-button"
        category="secondary"
        @click="resetSelection"
      >
        다시 선택
      </CustomButton>
      <CustomButton
        v-if="isConfirmed"
        class="bottom-button"
        category="primary"
        @click="confirmSelection"
      >
        확인
      </CustomButton>
      <CustomButton
        v-else
        class="bottom-button"
        category="primary"
        @click="submit"
      >
        선택
      </CustomButton>
    </div>
  </div>
  <TeamRocketAlert :show="showAlert" :message="alertMessage" />
</template>

<script setup>
import { getPokedex } from '@/apis/pokedex';
import { signup } from '@/apis/users';
import CustomButton from '@/components/common/CustomButton.vue';
import TeamRocketAlert from '@/components/common/TeamRocketAlert.vue';
import { useSignupStore } from '@/stores/signupStore';
import { onMounted, ref } from 'vue';
import { useRouter } from 'vue-router';

const startingPokemons = ref([]);
const selectedId = ref(null);
const content = ref('');
const isConfirmed = ref(false);
const signupStore = useSignupStore();
const showAlert = ref(false);
const alertMessage = ref('');

const selectContent = '어떤 포켓몬을 선택할까요?';

const router = useRouter();

const handleNavigate = (routeName) => {
  router.push({ name: routeName });
};

const selectPokemon = (id) => {
  selectedId.value = id;
};

const submit = () => {
  if (!selectedId.value) {
    showTemporaryAlert('포켓몬을 선택해주세요!');
    return;
  }

  const selectedPokemon = startingPokemons.value.find(
    (p) => p.id === selectedId.value
  );

  if (selectedPokemon) {
    content.value = '';
    const confirmMessage = `이 포켓몬으로 하겠습니까? <br> ► ${selectedPokemon.name}`;
    typeText(confirmMessage, content, 30);
    isConfirmed.value = true; // 선택 확정 상태로 진입
  }
};

const resetSelection = () => {
  selectedId.value = null;
  content.value = selectContent;
  isConfirmed.value = false;
};

const confirmSelection = async () => {
  // TODO: 메인포켓몬 설정 api 로직 추가

  // pinia 저장
  signupStore.main_pokemon_id = Number(selectedId.value);
  signupStore.pokemon_ids = [Number(selectedId.value)];

  const user = { ...signupStore.$state };

  const response = await signup(user);

  handleNavigate('home');
  console.log('최종 선택 포켓몬:', selectedId.value);
};

onMounted(async () => {
  const startingIds = [1, 4, 7];
  const responses = await Promise.all(startingIds.map(getPokedex));

  startingPokemons.value = responses.filter(Boolean).map((pokemon) => ({
    id: pokemon.id,
    name: pokemon.name,
    image: pokemon.image_url,
    types: pokemon.types,
  }));
});

onMounted(() => {
  typeText(selectContent, content, 30);
});

function typeText(full, targetRef, speed = 30, delay = 0) {
  let index = 0;
  targetRef.value = '';
  setTimeout(() => {
    const timer = setInterval(() => {
      if (index < full.length) {
        // HTML-safe 처리: <br>은 한꺼번에 넣기
        if (full.slice(index, index + 4) === '<br>') {
          targetRef.value += '<br>';
          index += 4;
        } else {
          targetRef.value += full[index];
          index++;
        }
      } else {
        clearInterval(timer);
      }
    }, speed);
  }, delay);
}

const showTemporaryAlert = (message) => {
  alertMessage.value = message;
  showAlert.value = true;
  setTimeout(() => {
    showAlert.value = false;
  }, 2000);
};
</script>

<style scoped>
.select-container {
  position: relative;
  height: 100%;
}

.logo-container {
  display: flex;
  width: 100%;
  justify-content: center;
}
.logo-img {
  width: 85%;
  max-width: 478px;
  position: relative;
  padding-top: 60px;
}

img {
  position: relative;
}
.pokemon-list {
  display: flex;
  justify-content: center;
  overflow: hidden;
  padding: 60px 15px 0;
  justify-content: space-evenly;
}
.pokemon-img {
  width: 100%;
  transform: scale(1.4);
  transition: transform 0.2s ease;
  cursor: pointer;
  filter: brightness(60%);
}
.pokemon-img:hover {
  filter: brightness(100%);
  transform: scale(1.6); /* 🔍 확대 */
}
.pokemon-img.selected {
  filter: brightness(100%);
  z-index: 1;
}
.pokemon-img.disabled {
  filter: brightness(30%);
  opacity: 0.5;
  cursor: not-allowed;
  pointer-events: none;
}

.start-prompt {
  position: absolute;
  bottom: 5rem;
  width: 100%;
  display: flex;
  justify-content: center;
  z-index: 10;
}

/* 모바일 세로 (해상도 ~ 479px)*/
@media all and (max-width: 479px) {
  .start-prompt {
    bottom: 3rem;
  }
}

/* 모바일 가로 & 태블릿 세로 (해상도 480px ~ 767px)*/
@media all and (min-width: 480px) and (max-width: 767px) {
  .start-prompt {
    bottom: 5rem;
  }
}

.prompt-box {
  position: relative;
  width: 100%;
  max-width: 400px;
  aspect-ratio: 3 / 1; /* 이미지 비율 유지 */
  background: url('@/assets/images/prompt2.png') no-repeat center center;
  background-size: 100% 100%; /* 이미지가 박스에 딱 맞도록 */
  display: flex;
  justify-content: start;
  align-items: start;
  padding: 2rem 2.5rem;
  box-sizing: border-box;
  overflow: hidden; /* 버튼이 삐져나가지 않도록 */
  font-size: 20px;
}

.bottom-button-container {
  position: absolute;
  bottom: 0;
  display: flex;
  width: 100%;
  padding: 0 25px;
  gap: 20px;
  margin-bottom: -55px;
}
.bottom-button {
  width: 100%;
}

/* 배경 몬스터볼 */
.select-container::before {
  content: '';
  position: absolute;
  top: 4%;
  left: 50%;
  max-width: 450px;
  max-height: 450px;
  width: 80%;
  height: 80%;
  background-image: url('@/assets/images/monster-ball-back.png');
  background-size: contain;
  background-repeat: no-repeat;
  background-position: center;
  transform: translate(-50%, -50%);
  z-index: 0;
}
</style>
