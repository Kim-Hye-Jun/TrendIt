<template>
  <q-card class="keyword-analyze q-pa-lg">
    <q-card-section class="card-title">
      <div class="title q-mt-xs">키워드 분석</div>
      <div class="row justify-center items-center">
        <keyword-line-chart
          :keyword="keyword"
          :keyword_id="$route.params.keyword_id"
        />
      </div>
    </q-card-section>
  </q-card>
  <q-card class="keyword-analyze-result q-pa-lg">
    <q-card-section class="card-title">
      <div class="title q-mt-xs">키워드 분석 결과</div>
    </q-card-section>
    <q-separator inset />
    <div class="flex q-pa-lg justify-around">
      <!-- TODO: img중앙정렬, 글자 폰트 적용 후 위치 조정 -->
      <template v-for="(item, index) of result" :key="index">
        <q-card class="news-card q-my-md">
          <a :href="item.newsLink" target="_blank">
            <div style="height: 190px; overflow: hidden; padding: auto">
              <q-img :src="item.imgLink" height="190px" :alt="item.headline" />
            </div>
            <q-card-section class="flex column">
              <div class="headline-part">
                <strong>{{ contentfilter(item.headline) }}</strong>
              </div>
              <div class="content-part">
                {{ contentfilter(item.newsContent) }}
              </div>
              <div class="cards-agency">
                {{ item.newsAgency }} {{ item.newsDate }}
              </div>
            </q-card-section>
          </a>
        </q-card>
      </template>
    </div>
    <chat-turn-on-button
      class="chat-btn"
      :keywordIde="keyword_id"
    ></chat-turn-on-button>
    <div class="q-pa-lg flex flex-center">
      <q-pagination v-model="page" :max="max_page" input />
    </div>
  </q-card>
</template>

<script>
import { ref } from "vue";
import KeywordLineChart from "../../charts/KeywordLineChart.vue";
import { getKeywordApi } from "boot/keyword.js";
import { searchApi } from "boot/news.js";
import ChatTurnOnButton from "src/components/chat/ChatTurnOnButton.vue";

export default {
  components: { KeywordLineChart, ChatTurnOnButton },
  setup() {
    return {
      keword_id: ref(0),
      keyword: ref(""),

      period: ref(""),
      page: ref(1),
      max_page: ref(99),

      result: ref([]),
    };
  },
  async mounted() {
    this.keyword_id = this.$route.params.keyword_id;
    await getKeywordApi(
      this.keyword_id,
      (response) => {
        this.keyword = response.data.data.keyword;
      },
      () => {
        this.$router.push({ name: "empty_keyword" });
        alert("잘못된 접근입니다.");
      }
    );
    this.period = this.$route.query.period;
    await this.getNewsOfPage(this.page);
  },
  methods: {
    async getNewsOfPage(n) {
      await searchApi(
        {
          keywordId: this.keyword_id,
          period: this.period,
          page: n,
        },
        (response) => {
          this.result = response.data.news.content;
          this.result.forEach((item) => {
            if (item.imgLink.substring(0, 5) == "/asse") {
              item.imgLink = "/src/assets/no-image.png";
            }
            if (item.newsLink == "") {
              item.newsLink = "/404";
            }
          });
          this.max_page = response.data.news.totalPages;
        },
        () => console.warn("failed to load news")
      );
    },
  },
  computed: {
    contentfilter() {
      return function (text) {
        if (text.length > 38) return text.substring(0, 33) + "...";
        else return text;
      };
    },
  },
  watch: {
    page: async function () {
      await this.getNewsOfPage(this.page);
    },
  },
};
</script>

<style scoped>
/* 1 */
.title {
  font-family: "NanumBarunGothicBold";
  font-size: 17px;
  /* margin-top:10px; */
}
.news-card {
  width: 250px;
  height: 345px;
}

.headline-part {
  height: 50px;
}
.content-part {
  height: 60px;
}

/* 2 */
.keyword-analyze {
  width: 1200px;
  margin: 15px 0px;
}

/* 3 */
.keyword-analyze-result {
  /* TODO: "키워드 분석" 글자 폰트, 사이즈, 패딩 마진 등 결정 후 고정된 높이 줘야 할듯 */
  /* height: 1112px; */
  width: 1200px;
  margin: 15px 0px;
}

.cards-agency {
  color: gray;
  font-size: 12px;
}

a {
  color: black;
  text-decoration: none;
}

.chat-btn {
  position: fixed;
  bottom: 0%;
  right: 1%;
}
</style>
