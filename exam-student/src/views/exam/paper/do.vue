<template>
  <div>
    <el-row  class="do-exam-title" style="background-color: rgb(160,160,160,0.4)">
      <el-col :span="24">
        <span :key="item.itemOrder"  v-for="item in answer.answerItems">
             <el-tag :type="questionCompleted(item.completed)" class="do-exam-title-tag" @click="goItem(item.itemOrder)">{{item.itemOrder}}</el-tag>
        </span>
        <el-progress :text-inside="true" :stroke-width="20" :percentage="percentage"></el-progress>
        <span class="do-exam-time">
          <label>剩余时间：</label>
          <label>{{formatSeconds(remainTime)}}</label>
        </span>
      </el-col>
    </el-row>
    <el-row  class="do-exam-title-hidden">
      <el-col :span="24">
        <span :key="item.itemOrder"  v-for="item in answer.answerItems">
             <el-tag  class="do-exam-title-tag" >{{item.itemOrder}}</el-tag>
        </span>
        <span class="do-exam-time">
          <label>剩余时间：</label>
        </span>
      </el-col>
    </el-row>
    <el-container  class="app-item-contain">
      <el-header class="align-center">
        <h1>{{form.name}}</h1>
        <div>
          <span class="question-title-padding">试卷总分：{{form.score}}</span>
          <span class="question-title-padding">考试时间：{{form.suggestTime}}分钟</span>
        </div>
      </el-header>
      <el-main>
        <el-form :model="form" ref="form" v-loading="formLoading" label-width="100px">
          <el-row :key="index"  v-for="(titleItem,index) in form.titleItems">
                <h3>{{titleItem.name}}</h3>
                <el-card class="exampaper-item-box" v-if="titleItem.questionItems.length!==0">
                <el-form-item :key="questionItem.itemOrder" :label="questionItem.itemOrder+'.'"
                            v-for="questionItem in titleItem.questionItems"
                            class="exam-question-item" label-width="50px" :id="'question-'+ questionItem.itemOrder">
                 <QuestionEdit :qType="questionItem.questionType" :question="questionItem" :showanalyzeflag="showanalyzeflag"
                              :answer="answer.answerItems[questionItem.itemOrder-1]"/>
                </el-form-item>
                </el-card>
          </el-row>
           <el-row class="do-align-center">
             <el-button type="primary" @click="submitForm">提交试卷</el-button>
             <el-button  v-show="pageindex!=0" @click="preitem(pageindex)">上一题</el-button>
             <el-button  v-show="pageindex!=(questionItems.length-1)"  @click="nextitem(pageindex)">下一题</el-button>
             <el-button  type="primary" @click="showanalyze">查看解析</el-button>
            <el-button type="danger"  @click="quit">退出</el-button>
           </el-row>
        </el-form>
      </el-main>
    </el-container>
  </div>
</template>

<script>
import { mapState, mapGetters } from 'vuex'
import { formatSeconds } from '@/utils'
import QuestionEdit from '../components/QuestionEdit'
import examPaperApi from '@/api/examPaper'
import examPaperAnswerApi from '@/api/examPaperAnswer'

export default {
  components: { QuestionEdit },
  data () {
    return {
      showanalyzeflag: false,
      form: {},
      questionItems: {},
      formLoading: false,
      answer: {
        questionId: null,
        doTime: 0,
        answerItems: []
      },
      timer: null,
      remainTime: 0,
      pageindex: 0,
      percentage: 0
    }
  },
  created () {
    let id = this.$route.query.id
    let _this = this
    var pageindex = 0
    if (id && parseInt(id) !== 0) {
      _this.formLoading = true
      examPaperApi.select(id).then(re => {
        var questionItems = re.response.titleItems[0].questionItems
        _this.questionItems = questionItems
        var indexque = questionItems[pageindex]
        re.response.titleItems[0].questionItems = []
        re.response.titleItems[0].questionItems[0] = indexque
        _this.form = re.response
        _this.remainTime = re.response.suggestTime * 60
        _this.initAnswer()
        _this.timeReduce()
        _this.formLoading = false
        var exp2 = ((this.pageindex + 1) / this.questionItems.length) * 100
        this.percentage = Math.round(exp2)
      })
    }
  },
  mounted () {

  },
  beforeDestroy () {
    window.clearInterval(this.timer)
  },
  methods: {
    formatSeconds (theTime) {
      return formatSeconds(theTime)
    },
    timeReduce () {
      let _this = this
      this.timer = setInterval(function () {
        if (_this.remainTime <= 0) {
          _this.submitForm()
        } else {
          ++_this.answer.doTime
          --_this.remainTime
        }
      }, 1000)
    },
    questionCompleted (completed) {
      return this.enumFormat(this.doCompletedTag, completed)
    },
    goAnchor (selector) {
      this.$el.querySelector(selector).scrollIntoView({ behavior: 'instant', block: 'center', inline: 'nearest' })
    },
    initAnswer () {
      this.answer.id = this.form.id
      let questionArray = this.questionItems
      for (let qIndex in questionArray) {
        let question = questionArray[qIndex]
        this.answer.answerItems.push({ questionId: question.id, content: null, contentArray: [], completed: false, itemOrder: question.itemOrder })
      }
    },
    submitForm () {
      let _this = this
      window.clearInterval(_this.timer)
      _this.formLoading = true
      examPaperAnswerApi.answerSubmit(this.answer).then(re => {
        if (re.code === 1) {
          _this.$alert('试卷得分：' + re.response + '分', '考试结果', {
            confirmButtonText: '返回考试记录',
            callback: action => {
              _this.$router.push('/record/index')
            }
          })
        } else {
          _this.$message.error(re.message)
        }
        _this.formLoading = false
      }).catch(e => {
        _this.formLoading = false
      })
    },
    preitem (pageindex) {
      pageindex = pageindex - 1
      this.pageindex = pageindex
      this.form.titleItems[0].questionItems[0] = this.questionItems[pageindex]
      this.showanalyzeflag = false
    },
    nextitem (pageindex) {
      pageindex = pageindex + 1
      this.pageindex = pageindex
      this.form.titleItems[0].questionItems[0] = this.questionItems[pageindex]
      var exp2 = ((this.pageindex + 1) / this.questionItems.length) * 100
      this.percentage = Math.round(exp2)
      this.showanalyzeflag = false
    },
    quit () {
      window.history.back()
    },
    showanalyze () {
      this.showanalyzeflag = true
    },
    goItem (order) {
      this.pageindex = order - 1
      this.form.titleItems[0].questionItems[0] = this.questionItems[this.pageindex]
      var exp2 = ((this.pageindex + 1) / this.questionItems.length) * 100
      this.percentage = Math.round(exp2)
      this.showanalyzeflag = false
    }
  },
  computed: {
    ...mapGetters('enumItem', ['enumFormat']),
    ...mapState('enumItem', {
      doCompletedTag: state => state.exam.question.answer.doCompletedTag
    })
  }
}
</script>

<style lang="scss" scoped>
  .align-center {
    text-align: center
  }

  .exam-question-item {
    padding: 10px;

    .el-form-item__label {
      font-size: 15px !important;
    }
  }

  .question-title-padding {
    padding-left: 25px;
    padding-right: 25px;
  }
</style>
