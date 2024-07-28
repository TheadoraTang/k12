<template>
  <div>
    <h2>Ⅳ. Listen and number（听一听，用“1 ~5”给下列图片编号）：【词汇-词义】</h2>

    <div v-if="permission === '2'" class="class-select">
      <label for="compare-type-select">选择对比类型：</label>
      <select id="compare-type-select" v-model="compareType" @change="fetchData">
        <option value="single">单个班级</option>
        <option value="double-class">两个班级</option>
        <option value="double-campus">两个校区</option>
      </select>

      <div v-if="compareType === 'single'">
        <label for="class-select">选择班级：</label>
        <select id="class-select" v-model="selectedClass" @change="fetchData">
          <option value="11">张江一班</option>
          <option value="12">张江二班</option>
          <option value="13">张江三班</option>
          <option value="21">滨江一班</option>
          <option value="22">滨江二班</option>
          <option value="23">滨江三班</option>
        </select>
      </div>

      <div v-if="compareType === 'double-class'">
        <label for="class-select-1">选择班级 1：</label>
        <select id="class-select-1" v-model="selectedClass1" @change="fetchData">
          <option value="11">张江一班</option>
          <option value="12">张江二班</option>
          <option value="13">张江三班</option>
          <option value="21">滨江一班</option>
          <option value="22">滨江二班</option>
          <option value="23">滨江三班</option>
        </select>
        <label for="class-select-2">选择班级 2：</label>
        <select id="class-select-2" v-model="selectedClass2" @change="fetchData">
          <option value="11">张江一班</option>
          <option value="12">张江二班</option>
          <option value="13">张江三班</option>
          <option value="21">滨江一班</option>
          <option value="22">滨江二班</option>
          <option value="23">滨江三班</option>
        </select>
      </div>

      <div v-if="compareType === 'double-campus'">
        <label for="campus-select-1">选择校区1：</label>
        <select id="campus-select-1" v-model="selectedCampus1" @change="fetchData">
          <option value="1">张江校区</option>
          <option value="2">滨江校区</option>
        </select>

        <label for="campus-select-2">选择校区2：</label>
        <select id="campus-select-2" v-model="selectedCampus2" @change="fetchData">
          <option value="1">张江校区</option>
          <option value="2">滨江校区</option>
        </select>
      </div>
    </div>

    <div v-if="permission === '0'" class="class-select">
      <label for="class-select">选择班级：</label>
      <select id="class-select" v-model="selectedClass" @change="fetchData">
        <option v-for="cls in classOptions" :key="cls" :value="cls">{{ getClassDisplayName(cls) }}</option>
      </select>
    </div>

    <div v-for="(question, index) in questions" :key="index">

      <div class="text">
        <p>【听力原文】：</p>
        <p>{{ questionData[question]?.listeningText }}</p>
      </div>

      <!-- 显示图片和难度 -->
      <div>
        <img :src="staticData[question]?.image" />
        <!-- 显示选项 -->
        <div v-for="(option, optIndex) in options" :key="optIndex">
          <p v-html="highlightText(option.text, questionData[question]?.correctAnswer)"></p>
          <!-- 显示选项人数，只有 permission === '0' 或 permission === '2' 时显示 -->
          <p v-if="permission === '0'">选择人数：{{
            questionData[question]?.optionCounts[option.value] || 0 }}</p>

          <p v-if="permission === '2' && compareType === 'single'">选择人数：{{
            questionData[question]?.optionCounts1[option.value] || 0 }}</p>

          <p v-if="permission === '2' && compareType !== 'single'">选择人数1：{{
            questionData[question]?.optionCounts1[option.value] || 0 }} 选择人数2：{{
              questionData[question]?.optionCounts2[option.value] || 0 }}</p>
        </div>
        <p>【难度】：{{ staticData[question]?.difficulty }}</p>
      </div>

      <div class="question-container">
        <p
          :class="{ highlighted: permission === '1' && questionData[question]?.studentAnswer !== questionData[question]?.correctAnswer }">
          【题目】： {{ index + 1 }}
        </p>
      </div>


      <!-- 根据 permission 显示不同内容 -->
      <div v-if="permission === '1'" class="peer-accuracy-container">
        <p> 【同伴正确率】：{{ questionData[question]?.peerAccuracy }}%</p>
        <p> 【学生答案】：{{ questionData[question]?.studentAnswer }}</p>
      </div>


      <div v-if="permission === '0'" class="option-percentages-container">
        <p>【正确人数比】</p>
        <div>
          <!-- 直接显示 optionPercentages 的值 -->
          <p>{{ questionData[question]?.optionPercentages || '0' }}</p>
        </div>
      </div>

      <div v-if="permission === '2' && compareType === 'single'" class="option-percentages-container">
          <p>【正确人数比】</p>
          <div>
            <!-- 直接显示 optionPercentages 的值 -->
            <p>{{ questionData[question]?.optionPercentages1 || '0' }}</p>
          </div>
        </div>

        <div v-if="permission === '2' && compareType !== 'single'" class="option-percentages-container">
          <p>【正确人数比】</p>
          <div>
            <!-- 直接显示 optionPercentages 的值 -->
            <p>{{ questionData[question]?.optionPercentages1 || '0' }}</p>
            <p>{{ questionData[question]?.optionPercentages2 || '0' }}</p>
          </div>
        </div>

      <!-- 正确答案 -->
      <div class="corrent_answer">
        <p>【正确答案】</p>
        <p v-html="highlightText(questionData[question]?.correctAnswer, questionData[question]?.correctAnswer)"></p>
      </div>

      <!-- 参考解析 -->
      <div>
        <h3>参考解析</h3>
        <p>{{ questionData[question]?.referenceAnalysis }}</p>
      </div>

      <hr>
    </div>
  </div>
</template>



<script>
import axios from 'axios';

export default {
  data() {
    return {
      questions: ['PART1_IV_1', 'PART1_IV_2', 'PART1_IV_3', 'PART1_IV_4', 'PART1_IV_5'],
      listeningText: '',
      correctAnswer: '',
      studentAnswer: '',
      peerAccuracy: 0,
      permission: this.$route.query.permission,
      name: this.$route.query.name,
      referenceAnalysis: 'Example reference analysis text', // Example reference analysis
      isAnswerCorrect: (question) => this[`studentAnswer_${question}`] === this[`correctAnswer_${question}`],
      options: [
        { text: '1', value: '1' },
        { text: '2', value: '2' },
        { text: '3', value: '3' },
        { text: '4', value: '4' },
        { text: '5', value: '5' },
      ],
      questionData: {},
      questionData2: {},
      selectedClass: '11',
      selectedClass1: '',
      selectedClass2: '',
      selectedCampus1: '',
      selectedCampus2: '',
      compareType: 'single',
      optionCounts: {},
      optionPercentages: {},
      staticData: {
        'PART1_IV_1': { image: '/images/PART1_IV_1.png', difficulty: '两颗星' },
        'PART1_IV_2': { image: '/images/PART1_IV_2.png', difficulty: '两颗星' },
        'PART1_IV_3': { image: '/images/PART1_IV_3.png', difficulty: '两颗星' },
        'PART1_IV_4': { image: '/images/PART1_IV_4.png', difficulty: '两颗星' },
        'PART1_IV_5': { image: '/images/PART1_IV_5.png', difficulty: '两颗星' },
        'PART1_IV_6': { image: '/images/PART1_IV_6.png', difficulty: '两颗星' }
      },
      classOptions: []
    };
  },
  methods: {
    async fetchClassOptions() {
      if (this.permission === '0') {
        console.log("permission===0能进来吗");
        try {
          console.log(this.name);
          const response = await axios.get('http://localhost:3000/api/teacher-classes', {
            params: { name: this.name }
          });
          this.classOptions = response.data.classes;
          this.selectedClass = this.classOptions[0];
          console.log(this.selectedClass);
          this.fetchData();
        } catch (error) {
          console.error('Error fetching class options:', error);
        }
      } else {
        this.fetchData();
      }
    },

    async fetchData() {
      try {
        const questions = this.questions;
        const questionData = {};

        for (const question of questions) {
          console.log(question);
          const response = await axios.get('http://localhost:3000/api/question', {
            params: { question }
          });

          const { correctAnswer, listeningText } = response.data;
          console.log(listeningText);
          console.log(question);
          console.log(correctAnswer);
          console.log("per", this.permission);
          if (this.permission === '1') {
            // 获取学生答案和同伴正确率
            console.log("hi");
            const studentResponse = await axios.get('http://localhost:3000/api/student-answer', {
              params: { name: this.name, question }
            });
            const studentAnswer = studentResponse.data.answer;
            console.log("stu_ans:", studentAnswer);

            const peerResponse = await axios.get('http://localhost:3000/api/peer-accuracy', {
              params: { name: this.name, question }
            });
            const peerAccuracy = peerResponse.data.accuracy * 100;

            const referenceAnalysisResponse = await axios.get('http://localhost:3000/api/analysis', {
              params: { question, options: studentAnswer }
            });
            const referenceAnalysis = referenceAnalysisResponse.data.analysis;
            questionData[question] = {
              correctAnswer,
              listeningText,
              studentAnswer,
              peerAccuracy,
              referenceAnalysis
            };
          } else if (this.permission === '2') {
            // 获取选项人数比
            if (this.compareType === 'single') {
              const optionResponse = await axios.get('http://localhost:3000/api/option-accuracy', {
                params: { question, class: this.selectedClass }
              });
              console.log(optionResponse);
              const optionPercentages = optionResponse.data.accuracyRate;
              console.log(optionPercentages);

              const optionCountsResponse = await axios.get('http://localhost:3000/api/option-counts', {
                params: { question, class: this.selectedClass }
              });
              const optionCounts = optionCountsResponse.data.optionCounts;

              console.log("能请求到解析吗");
              const referenceAnalysisResponse = await axios.get('http://localhost:3000/api/analysis', {
                params: { question, options: correctAnswer }
              });
              const referenceAnalysis = referenceAnalysisResponse.data.analysis;

              questionData[question] = {
                correctAnswer,
                listeningText,
                optionCounts1: optionCounts,
                optionCounts2: '0',
                optionPercentages1: optionPercentages,
                optionPercentages2: '0',
                referenceAnalysis
              };

              this.optionCounts = optionCounts;
              console.log.optionCounts;
            } else if (this.compareType === 'double-class') {
              console.log("可以进来这里吗");
              console.log("第一个班级", this.selectedClass1);
              console.log("第二个班级", this.selectedClass2);
              const optionCounts1Response = await axios.get('http://localhost:3000/api/option-counts', {
                params: { question, class: this.selectedClass1 }
              });
              const optionCounts2Response = await axios.get('http://localhost:3000/api/option-counts', {
                params: { question, class: this.selectedClass2 }
              });

              const optionPercentages1Response = await axios.get('http://localhost:3000/api/option-accuracy', {
                params: { question, class: this.selectedClass1 }
              });
              const optionPercentages2Response = await axios.get('http://localhost:3000/api/option-accuracy', {
                params: { question, class: this.selectedClass2 }
              });

              const referenceAnalysisResponse = await axios.get('http://localhost:3000/api/analysis', {
                params: { question, options: correctAnswer }
              });
              const referenceAnalysis = referenceAnalysisResponse.data.analysis;

              questionData[question] = {
                correctAnswer,
                listeningText,
                optionCounts1: optionCounts1Response.data.optionCounts,
                optionCounts2: optionCounts2Response.data.optionCounts,
                optionPercentages1: optionPercentages1Response.data.accuracyRate,
                optionPercentages2: optionPercentages2Response.data.accuracyRate,
                referenceAnalysis
              };
              console.log("所需要展示的所有数据", questionData);
            } else if (this.compareType === 'double-campus') {
              console.log("可以进来这里吗");
              console.log("第一个校区", this.selectedCampus1);
              console.log("第二个校区", this.selectedCampus2);
              const optionCounts1Response = await axios.get('http://localhost:3000/api/option-counts-campusCom', {
                params: { question, school: this.selectedCampus1 }
              });
              const optionCounts2Response = await axios.get('http://localhost:3000/api/option-counts-campusCom', {
                params: { question, school: this.selectedCampus2 }
              });

              const optionPercentages1Response = await axios.get('http://localhost:3000/api/option-accuracy-campusCom', {
                params: { question, school: this.selectedCampus1 }
              });
              const optionPercentages2Response = await axios.get('http://localhost:3000/api/option-accuracy-campusCom', {
                params: { question, school: this.selectedCampus2 }
              });

              const referenceAnalysisResponse = await axios.get('http://localhost:3000/api/analysis', {
                params: { question, options: correctAnswer }
              });
              const referenceAnalysis = referenceAnalysisResponse.data.analysis;

              questionData[question] = {
                correctAnswer,
                listeningText,
                optionCounts1: optionCounts1Response.data.optionCounts,
                optionCounts2: optionCounts2Response.data.optionCounts,
                optionPercentages1: optionPercentages1Response.data.accuracyRate,
                optionPercentages2: optionPercentages2Response.data.accuracyRate,
                referenceAnalysis
              };
              console.log("所需要展示的所有数据", questionData);
            }
          } else if (this.permission === '0') {
            // 获取选项人数比
            const optionResponse = await axios.get('http://localhost:3000/api/option-percentages', {
              params: { question, class: this.selectedClass }
            });
            console.log(optionResponse);
            const optionPercentages = optionResponse.data.optionPercentages;
            console.log(optionPercentages);

            const optionCountsResponse = await axios.get('http://localhost:3000/api/option-counts', {
              params: { question, class: this.selectedClass }
            });
            const optionCounts = optionCountsResponse.data.optionCounts;

            const referenceAnalysisResponse = await axios.get('http://localhost:3000/api/analysis', {
              params: { question, options: correctAnswer }
            });
            const referenceAnalysis = referenceAnalysisResponse.data.analysis;

            questionData[question] = {
              correctAnswer,
              listeningText,
              optionPercentages,
              optionCounts,
              referenceAnalysis
            };

            this.optionCounts = optionCounts;
            console.log.optionCounts;
          }
        }
        this.questionData = questionData;
        console.log(questionData);

      } catch (error) {
        console.error('Error fetching data:', error);
      }
    },

    highlightText(text, highlightText) {
      if (!text || !highlightText) return text;

      return Array.from(text).map(char => {
        if (highlightText.includes(char)) {
          return `<span class="highlighted">${char}</span>`;
        }
        return char;
      }).join('');
    },

    getClassDisplayName(classCode) {
      const classMap = {
        '11': '张江一班',
        '12': '张江二班',
        '13': '张江三班',
        '21': '滨江一班',
        '22': '滨江二班',
        '23': '滨江三班'
      };
      return classMap[classCode] || classCode;
    }
  },

  created() {
    this.fetchClassOptions();
  }
};
</script>

<style>
.highlighted {
  background-color: yellow;
}

.highlight {
  color: red;
}

.class-select {
  margin-bottom: 20px;
}
</style>
