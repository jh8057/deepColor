<template>
  <div class="testOneComp">
    <main v-show="question" class="Question">
      <div class="Question__center">
        <h3>
          아래 자극의 <span style="font-weight: bolder">'색상'</span>이
          가벼운가요 무거운가요?
        </h3>
        <img :src="`/deepColor/answer/${num}.png`" class="Question__img" />
        <section class="selectSection">
          <div class="selectSection__item">
            <label for="select1">가벼움</label>
            <input
              type="radio"
              id="select1"
              value="가벼움"
              v-model="selected"
            />
          </div>
          <div class="selectSection__item">
            <label for="select2">무거움</label>
            <input
              type="radio"
              id="select2"
              value="무거움"
              v-model="selected"
            />
          </div>
        </section>
      </div>
    </main>
    <main v-show="showCenter" class="breakTime">
      <div class="centerPoint">
        <img src="/center.png" class="centerPoint__img" />
      </div>
    </main>

    <article v-show="end" class="QuestionEnd">
      <div>
        1차 테스트 종료되었습니다.<br />
        2차 테스트를 진행해주세요.
      </div>
      <div>
        <button @click="goTestTwo" style="font-size: 15px; padding: 25px">
          다음</button
        ><br />
      </div>
    </article>

    <!-- <article v-if="showFin" class="result" id="resultElement">
      <p style="font-weight: bold">전문이 보이도록 세로모드로 캡쳐해주세요.</p>
      <table-result v-model:result="finalResult" :key="finData[0]" />
    </article> -->
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, onMounted, computed } from "vue";
import { useStore } from "vuex";
import { mapMutations } from "vuex";
import tableResult from "./tableResult.vue";
import type { finalResult } from "../types/types";

export default defineComponent({
  components: { tableResult },
  setup() {
    const selected = ref("");
    const question = ref(false);
    const answerList: Array<string> = [];

    const finalResult: Array<finalResult> = [];
    const store = useStore();

    const selectedAnswer = computed(() => store.state.selectedAnswer);
    const num = computed(() => store.state.selectedAnswer.num || 1);
    const answer = computed(() => store.state.selectedAnswer.answer);
    const len = computed(() => store.getters.getAnswerListLength);
    const finData = computed(() => store.state.finalResult);
    const end = computed(() => store.state.testOneEnd);

    //computed
    const showCenter = computed(() => {
      return !question.value && !end.value;
    });

    const doSelect = async () => {
      store.commit("popAnswer");
      return true;
    };

    const resetList = () => {
      store.commit("resetList");
      location.reload();
    };
    const reset = () => {
      store.commit("setStep", 0);
    };
    const saveFinal = () => {
      store.commit("setFinalResult", finalResult);
    };

    const showFin = ref(false);
    const showResult = () => {
      showFin.value = showFin.value ? false : true;
    };

    const goTestTwo = () => {
      store.commit("setStep", 5);
    };

    onMounted(() => {
      // 재시작 방지
      if (finData.value.length > 0) finalResult.push(...finData.value);
    });

    return {
      selected,
      question,
      answerList,
      selectedAnswer,
      num,
      answer,
      doSelect,
      len,
      resetList,
      reset,
      showResult,
      finalResult,
      showFin,
      saveFinal,
      finData,
      showCenter,
      end,
      goTestTwo,
    };
  },
  mounted() {
    // 처음에 조준점으로 시작
    // 검색 끝난 뒤 다시 실행하면 안됨
    if (this.len > 0) this.showCenterPoint();
  },
  methods: {
    ...mapMutations(["setTestOneEnd"]),
    showCenterPoint() {
      this.question = false;
      //question 다음꺼를 여기서 미리 작업 시작
      if (this.len > 0) {
        // 16개중 랜덤으로 하나를 뽑아야 된다.
        // 쓰고 다시 나오면 안되므로 버려야된다.
        this.doSelect();
        setTimeout(this.showQuestion, 1000);
      }
    },
    async showQuestion() {
      this.question = true;
      setTimeout(this.saveResult, 3000);
    },
    saveResult() {
      let result = {
        num: this.num,
        answer: this.selected,
      };
      this.finalResult.push(result);
      this.selected = "";
      // console.log(result);
      this.saveFinal();

      if (this.len > 0) {
        this.showCenterPoint();
      } else {
        this.question = false;
        this.setTestOneEnd(true);
      }
    },
  },
});
</script>

<style>
.testOneComp {
  margin: auto;
  width: 80vw;
  height: 80vh;
  text-align: center;
  align-content: center;
}
.breakTime,
.QuestionEnd {
  width: 100%;
  height: 100%;
}
.Question {
  margin-top: 20px;
  width: 100%;
  height: 100%;
}
.QuestionEnd {
  display: flex;
  flex-direction: column;
  margin: auto;
  text-align: center;
  justify-content: center;
  align-items: center;
  gap: 20px;
}

.Question__img {
  width: 30vh;
  height: 30vh;
}
.centerPoint {
  width: 100%;
  height: 100%;
  display: flex;
  text-align: center;
  justify-content: center;
  align-items: center;
}
.Question__center {
  width: 100%;
  height: 100%;
  display: flex;
  text-align: center;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  gap: 25px;
}

.centerPoint__img {
  width: 30px;
  height: 30px;
}

.selectSection {
  display: flex;
  justify-content: center;
  gap: 30px;
}

.selectSection__item {
  display: flex;
  justify-content: center;
  flex-direction: column;
}
.redfont {
  color: red;
}
</style>
