<template>
    <div>
      <header class="header">
        <h2>Ⅳ. Read and circle（读一读，选择最佳答案，将字母代号写在前面的括号内）：【语法-句法】</h2>

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
                <p style="font-size:18px">题目描述：<span v-html="questionDescriptions[question]"></span></p>
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
              <td style="width: 25%; text-align: center; color: black; font-size:16px">选项</td>
              <td style="width: 25%; text-align: center; color: black; font-size:16px" v-for="(option, optIndex) in options" :key="optIndex">
                <span v-html="highlightTextNew(option.text, questionData[question]?.correctAnswer, staticConOption[question]?.confuse)"></span>
              </td>
            </tr>
            <tr>
              <td style="height: 33.333%; text-align: center; color: black; font-size:16px">选择人数</td>
              <td style="height: 33.333%; text-align: center; color: black; font-size:16px" v-for="(option, optIndex) in options" :key="optIndex">
                {{ questionData[question]?.optionCounts[option.value] || 0 }}
              </td>
            </tr>
            <tr>
              <td colspan=4 style="color: black; font-size:16px"><span class="option-percentages-container">&nbsp;&nbsp;&nbsp;&nbsp;正确率：{{ questionData[question]?.accuracyPercentages || '0' }}%</span></td>
            </tr>
            </tbody>
          </table>
          </div>

          <!--学生端-->
          <div v-if="permission === '1'">
            <table class="custom-border" border="1" cellspacing="0" width=500px height=160px bgcolor=white>
            <tbody>
            <tr>
              <td style="height: 50%; width: 25%; text-align: center; color: black; font-size:16px">选项</td>
              <td style="width: 25%; text-align: center; color: black; font-size:16px" v-for="(option, optIndex) in options" :key="optIndex">
                <span v-html="highlightTextNew(option.text, questionData[question]?.correctAnswer, staticConOption[question]?.confuse)"></span>
              </td>
            </tr>
            <tr>
              <td colspan=4 style="color: black; font-size:16px"><p class="option-percentages-container">&nbsp;&nbsp;&nbsp;&nbsp;学生答案：{{ questionData[question]?.studentAnswer }}</p>
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
            questions: ['PART2_IV_1', 'PART2_IV_2', 'PART2_IV_3', 'PART2_IV_4', 'PART2_IV_5', 'PART2_IV_6', 'PART2_IV_7', 'PART2_IV_8', 'PART2_IV_9'],
            listeningText: '',
            correctAnswer: '',
            studentAnswer: '',
            peerAccuracy: 0,
            permission: this.$route.query.permission,
            name: this.$route.query.name,
            referenceAnalysis: 'Example reference analysis text', // Example reference analysis
            isAnswerCorrect: (question) => this[`studentAnswer_${question}`] === this[`correctAnswer_${question}`],
            options: [
                { text: 'A', value: 'A' },
                { text: 'B', value: 'B' },
                { text: 'C', value: 'C' }
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
                'PART2_IV_1': { difficulty: '★★' },
                'PART2_IV_2': { difficulty: '★★' },
                'PART2_IV_3': { difficulty: '★★' },
                'PART2_IV_4': { difficulty: '★★' },
                'PART2_IV_5': { difficulty: '★★' },
                'PART2_IV_6': { difficulty: '★★' },
                'PART2_IV_7': { difficulty: '★★' },
                'PART2_IV_8': { difficulty: '★★' },
                'PART2_IV_9': { difficulty: '★★' }
            },
            
            staticConOption: {
                'PART2_IV_1': { confuse: 'B' },
                'PART2_IV_2': { confuse: '' },
                'PART2_IV_3': { confuse: '' },
                'PART2_IV_4': { confuse: '' },
                'PART2_IV_5': { confuse: 'B' },
                'PART2_IV_6': { confuse: '' },
                'PART2_IV_7': { confuse: '' },
                'PART2_IV_8': { confuse: '' },
                'PART2_IV_9': { confuse: '' }
            },
            questionDescriptions: {
                'PART2_IV_1': '(   ) 1. I _____ like dolls. They are for girls.A.am not	 		<span class="highlightedCon">B.can’t</span> 			   <span class="highlighted">C.don’t</span>',
                'PART2_IV_2': '(   ) 2. --- What are those  ________ English?    --- They are shoes.<span class="highlighted">A.in</span>		 			B.on			   C.for',
                'PART2_IV_3': '(   ) 3. It’s so cold. Don’t _____ the jacket.A.turn off				<span class="highlighted">B.take off</span>		   C.turn on',
                'PART2_IV_4': '(   ) 4.--- What ____ is it?      	--- It’s a star.A.colour					B.season    	   <span class="highlighted">C.shape</span>',
                'PART2_IV_5': '(   ) 5. In winter, I like ______ in the room.A.Sleep 				<span class="highlightedCon">B.sleepping</span>	   <span class="highlighted">C.sleeping</span>',
                'PART2_IV_6': '(   ) 6. --- Is it summer? 			--- ______________.A.Yes, it’s autumn.<span class="highlighted">B.No, it’s autumn.</span>C.No, it’s summer .',
                'PART2_IV_7': '(   ) 7. My nose __________ small but my feet ___________ big.A.are, is 				<span class="highlighted">B.is, are</span>			   C.is, is',
                'PART2_IV_8': '(   ) 8. --- What ________ you do on Mother’s Day?--- I make a card. <span class="highlighted">A.do</span>					B.can			   C.are',
                'PART2_IV_9': '(   ) 9. There are two Children’s days in ________ .<span class="highlighted">A.Japan</span>			    B.Thailand		   C.Singapore'
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
                    const response = await axios.get('http://localhost:3000/api/question_single', {
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

                        // 获取正确率
                        const accuracyResponse = await axios.get('http://localhost:3000/api/option-accuracy', {
                            params: { question, class: this.selectedClass }
                        });
                        console.log(accuracyResponse);
                        const accuracyPercentages = accuracyResponse.data.accuracyRate;
                        console.log(accuracyPercentages);


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
                            referenceAnalysis,
                            accuracyPercentages
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

        highlightTextNew(optionText, correctAnswer, confusingAnswer)
        {
            if(optionText === correctAnswer)
            {
                return `<span class="highlight-correct">${optionText}</span>`;
            }else if(optionText === confusingAnswer)
            {
                return `<span class="highlight-confusing">${optionText}</span>`;
            }
            return optionText;
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
.highlight-correct {
    background-color: yellow;   /*高亮正确答案*/
}

.highlight-confusing {
    background-color: rgb(148, 211, 92);   /*高亮混淆项答案*/
}

.highlightedCon {
    background-color: rgb(148, 211, 92);
}

.highlight {
    color: red;
}

.class-select {
    margin-bottom: 20px;
}
</style>