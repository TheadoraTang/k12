<template>
    <div>
      <header class="header">
        <h2>Ⅷ.Read and judge（阅读短文，判断正误，用T或F表示）：【语篇-句和句】</h2>

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
      </header>
      <div class="images">
              <img src="images/PART2_8.png" />
            </div>
            
      <div v-for="(question, index) in questions" :key="index">
     <div class="part-background">
      
      <!-- 显示图片和难度 -->
      <div class="question-wrapper">  
        <div class="left-column"> 
          <!--题号+难度-->
          <div class="title">
            <p :class="{highlighted: permission === '1' && questionData[question]?.studentAnswer !== questionData[question]?.correctAnswer }">
              {{ index + 1 }}、 {{ staticData[question]?.difficulty }}</p>
          </div>
      
          <!--题目描述-->
          <div v-if="questionDescriptions[question]">
                <p style="font-size:18px">题目描述：{{ questionDescriptions[question] }}</p>
            </div>

          <!-- 正确答案 -->
          <div class="corrent_answer">
            <p style="font-size:18px">正确答案：<span v-html="highlightText(questionData[question]?.correctAnswer, questionData[question]?.correctAnswer)"></span></p>
          </div>

          <!-- 参考解析 -->
          <div>
            <p style="font-size:18px">参考解析：{{ questionData[question]?.referenceAnalysis }}</p>
          </div>
        </div>
        <div class="right-column table-container"> 

          <!--教师端-->
          <div v-if="permission === '0'">
          <table class="custom-border" border="1" cellspacing="0" width=500px height=160px bgcolor=white>
            <tbody>
            <tr>
              <td style="width: 33.333%; text-align: center; color: black; font-size:16px">选项</td>
              <td style="width: 33.333%; text-align: center; color: black; font-size:16px" v-for="(option, optIndex) in options" :key="optIndex">
                <span v-html="highlightText(option.text, questionData[question]?.correctAnswer)"></span>
              </td>
            </tr>
            <tr>
              <td style="height: 33.333%; text-align: center; color: black; font-size:16px">选择人数</td>
              <td style="height: 33.333%; text-align: center; color: black; font-size:16px" v-for="(option, optIndex) in options" :key="optIndex">
                {{ questionData[question]?.optionCounts[option.value] || 0 }}
              </td>
            </tr>
            <tr>
              <td colspan=3 style="color: black; font-size:16px"><span class="option-percentages-container">&nbsp;&nbsp;&nbsp;&nbsp;正确人数比：{{ questionData[question]?.optionPercentages || '0' }}</span></td>
            </tr>
            </tbody>
          </table>
          </div>

          <!--学生端-->
          <div v-if="permission === '1'">
            <table class="custom-border" border="1" cellspacing="0" width=500px height=160px bgcolor=white>
            <tbody>
            <tr>
              <td style="height: 50%; width: 33.333%; text-align: center; color: black; font-size:16px">选项</td>
              <td style="width: 33.333%; text-align: center; color: black; font-size:16px" v-for="(option, optIndex) in options" :key="optIndex">
                <span v-html="highlightText(option.text, questionData[question]?.correctAnswer)"></span>
              </td>
            </tr>
            <tr>
              <td colspan=3 style="color: black; font-size:16px"><p class="option-percentages-container">&nbsp;&nbsp;&nbsp;&nbsp;学生答案：{{ questionData[question]?.studentAnswer }}</p>
              <p class="option-percentages-container">&nbsp;&nbsp;&nbsp;&nbsp;同伴正确率：{{ questionData[question]?.peerAccuracy }}%</p></td>
            </tr>
            </tbody>
          </table>
          </div>

          <!--管理员端-->
          <div v-if="permission === '2'">
            <table class="custom-border" border="1" cellspacing="0" width="500px" height="160px" bgcolor="white">
              <tbody>
                <tr>
                  <td style="width: 20%; text-align: center; color: black; font-size:16px">选项</td>
                  <td style="width: 40%; text-align: center; color: black; font-size:16px"
                    v-for="(option, optIndex) in options" :key="optIndex">
                    <span v-html="highlightText(option.text, questionData[question]?.correctAnswer)"></span>
                  </td>
                </tr>
                <tr v-if="compareType === 'single'">
                  <td style="height: 33.333%; text-align: center; color: black; font-size:16px">选择人数</td>
                  <td style="height: 33.333%; text-align: center; color: black; font-size:16px"
                    v-for="(option, optIndex) in options" :key="optIndex">
                    {{ questionData[question]?.optionCounts1[option.value] || 0 }}
                  </td>
                </tr>
                <tr v-if="compareType === 'double-class' || compareType === 'double-campus'">
                  <td style="height: 33.333%; text-align: center; color: black; font-size:16px">选择人数 1</td>
                  <td style="height: 33.333%; text-align: center; color: black; font-size:16px"
                    v-for="(option, optIndex) in options" :key="optIndex">
                    {{ questionData[question]?.optionCounts1[option.value] || 0 }}
                  </td>
                </tr>
                <tr v-if="compareType === 'double-class' || compareType === 'double-campus'">
                  <td style="height: 33.333%; text-align: center; color: black; font-size:16px">选择人数 2</td>
                  <td style="height: 33.333%; text-align: center; color: black; font-size:16px"
                    v-for="(option, optIndex) in options" :key="optIndex">
                    {{ questionData[question]?.optionCounts2[option.value] || 0 }}
                  </td>
                </tr>
              
                <tr>
                  <td colspan="3" style="color: black; font-size:16px">
                    <span class="option-percentages-container">
                      &nbsp;&nbsp;&nbsp;&nbsp;正确人数比：
                      <template v-if="compareType === 'single'">
                        {{ questionData[question]?.optionPercentages1 || '0' }}
                      </template>
                      <template v-else-if="compareType === 'double-class'">
                        班级1：{{ questionData[question]?.optionPercentages1 || '0' }}%
                        班级2：{{ questionData[question]?.optionPercentages2 || '0' }}%
                      </template>
                      <template v-else-if="compareType === 'double-campus'">
                        学校1：{{ questionData[question]?.optionPercentages1 || '0' }}%
                        学校2：{{ questionData[question]?.optionPercentages2 || '0' }}%
                      </template>
                  </span>
                </td>
              </tr>
            </tbody>
          </table>
        </div>

        </div>
      </div>
     </div>
     </div>

        
    </div>
</template>



<script>
import axios from 'axios';

export default {
    data() {
        return {
            questions: ['PART2_VIII_1', 'PART2_VIII_2', 'PART2_VIII_3', 'PART2_VIII_4'],
            listeningText: '',
            correctAnswer: '',
            studentAnswer: '',
            peerAccuracy: 0,
            permission: this.$route.query.permission,
            name: this.$route.query.name,
            referenceAnalysis: 'Example reference analysis text', // Example reference analysis
            isAnswerCorrect: (question) => this[`studentAnswer_${question}`] === this[`correctAnswer_${question}`],
            options: [
                { text: 'T', value: 'T' },
                { text: 'F', value: 'F' },
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
                'PART2_VIII_1': { difficulty: '★★★★★' },
                'PART2_VIII_2': { difficulty: '★★★★★' },
                'PART2_VIII_3': { difficulty: '★★★★★' },
                'PART2_VIII_4': { difficulty: '★★★★★' },
            },
            questionDescriptions: {
                'PART2_VIII_1': '(    )1. Jack is four years old.',
                'PART2_VIII_2': '(    )2. Jack’s home is near the big park.',
                'PART2_VIII_3': '(    )3. Ben can read a book.',
                'PART2_VIII_4': '(    )4. Jack and Ben are happy.',
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
/* 题目页的页面背景颜色 */
body {  
  background-color: #f5f7fa;   
} 

/*题目整体*/
.question-wrapper {  
  display: flex; /* 启用Flexbox布局 */  
  justify-content: space-between; /* 左右两边内容间隔 */  
}  

/*题目左部分、右部分*/ 
.left-column{  
  flex: 1; /* 使得左右两边都能根据内容自适应宽度，同时保持一定的空间 */  
  padding: 10px; /* 可选，为两边内容添加一些内边距 */  
  margin-left: 100px;
}

.right-column{
  flex: 1; /* 使得左右两边都能根据内容自适应宽度，同时保持一定的空间 */  
  padding: 10px; /* 可选，为两边内容添加一些内边距 */ 
  margin-right: 100px; 
}

/*表格整体*/
.table-container {  
  display: flex;  
  justify-content: center; /* 水平居中 */  
  align-items: center; /* 垂直居中（如果父容器高度足够的话） */  
}  

/*表格边框*/
.custom-border {  
  border-collapse: collapse; /* 确保相邻单元格的边框合并为一个单一的边框 */  
}  
  
.custom-border,  
.custom-border td,  
.custom-border th {  
  border: 1.2px solid black; /* 设置边框宽度、样式和颜色 */  
} 

.highlighted {
  background-color: yellow;
}

.highlight {
  color: red;
}

.class-select {
  margin-bottom: 20px;
}

.question-group {  
  display: flex;  
  justify-content: space-between; /* 根据需要调整间距 */  
  flex-wrap: wrap; /* 如果需要换行显示 */  
}  
  
.question-item {  
  /* 每个问题的样式 */  
  flex: 1; /* 使得每个问题尽可能等宽，但也可以根据需要调整 */  
  margin-right: 20px; /* 右边距，根据需要调整 */  
  /* 其他样式... */  
}  
  
/* 移除最后一个问题项的右边距（如果每行恰好三个问题） */  
.question-group .question-item:nth-child(3n) {  
  margin-right: 0;  
}

/*题目页里题目标题的格式*/
.title{
  font-weight: bold; /*字体加粗*/
  font-size: 24px;   /*字体大小*/
  color: #6DE4C6;    /*字体颜色*/
}

/*题目页里题目图片的格式*/
.image{
  width: 400px;
  height: 200px;
}

/*题目页里整个题目的背景格式*/
.part-background {  
  background-color: white; /* 背景颜色 */  
  padding: 10px; /* 可以根据需要添加内边距 */  
  border-radius: 20px; /* 可以添加圆角边框，使其看起来更柔和 */  
  margin-bottom: 50px;
  margin-left: 50px;
  margin-right: 50px;
}

.highlighted {
    background-color: yellow;
}

.highlight {
    color: red;
}

.class-select {
    margin-bottom: 20px;
}

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