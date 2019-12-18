<template>
  <div style="line-height:1.8">
    <div v-if="qType==1" v-loading="qLoading">
      <div class="q-title" v-html="question.title"/>
      <div class="q-content">
        <el-radio-group v-model="answer.content" @change="modify" >
          <el-radio  v-for="item in question.items"  :key="item.prefix"  :label="item.prefix" class="el-radio-inline" >
            <span class="question-prefix">{{item.prefix}}.</span>
            <span v-html="item.content" class="q-item-span-content"></span>
          </el-radio>
        </el-radio-group>
      </div>
      <el-tag v-if="myflag==1" type="success">正确</el-tag>
      <el-tag v-if="myflag==0" type="danger">错误</el-tag>
      <div v-show="showanalyzeflag">正确答案：{{question.correct}}<br><span v-show="showanalyzeflag">解析：</span><span v-html="question.analyze" class="q-item-content"></span></div>
    </div>
    <div v-else-if="qType==2" v-loading="qLoading">
      <div class="q-title" v-html="question.title"/>
      <div class="q-content">
        <el-checkbox-group v-model="answer.contentArray" @change="answer.completed = true" display:inline-block>
            <el-checkbox v-for="item in question.items" :label="item.prefix" :key="item.prefix">
              <span class="question-prefix">{{item.prefix}}.</span>
              <span v-html="item.content" class="q-item-span-content"></span>
          </el-checkbox>
        </el-checkbox-group>
      </div>
    </div>
    <div v-else-if="qType==3" v-loading="qLoading">
      <div class="q-title" v-html="question.title" style="display: inline;margin-right: 10px"/>
      <span style="padding-right: 10px;">(</span>
      <el-radio-group v-model="answer.content" @change="answer.completed = true" >
        <el-radio  v-for="item in question.items"  :key="item.prefix"  :label="item.prefix" >
          <span v-html="item.content" class="q-item-span-content"></span>
        </el-radio>
      </el-radio-group>
      <span style="padding-left: 10px;">)</span>
    </div>
    <div v-else-if="qType==4" v-loading="qLoading">
      <div class="q-title" v-html="question.title"/>
      <div>
        <el-form-item :label="item.prefix" :key="item.prefix"  v-for="item in question.items"  label-width="50px" style="margin-top: 10px;margin-bottom: 10px;">
          <el-input v-model="answer.contentArray[item.prefix-1]"  @change="answer.completed = true" />
        </el-form-item>
      </div>
    </div>
    <div v-else-if="qType==5" v-loading="qLoading">
      <div class="q-title" v-html="question.title"/>
      <div>
        <el-input v-model="answer.content" type="textarea" rows="5"  @change="answer.completed = true"/>
      </div>
    </div>
    <div v-else>
    </div>
  </div>

</template>

<script>
export default {
  name: 'QuestionShow',
  props: {
    isShow: {
      type: String,
      default: '-1'
    },
    showanalyzeflag: {
      type: Boolean,
      default: false
    },
    question: {
      type: Object,
      default: function () {
        return {}
      }
    },
    answer: {
      type: Object,
      default: function () {
        return { id: null, content: '', contentArray: [] }
      }
    },
    qLoading: {
      type: Boolean,
      default: false
    },
    qType: {
      type: Number,
      default: 0
    }
  },
  data () {
    return {
      myflag: this.isShow
    }
  },
  methods: {
    modify () {
      this.answer.completed = true
      var correct = this.question.correct
      var select = this.$children[0].value
      if (correct === select) {
        this.myflag = 1
      } else {
        this.myflag = 0
      }
    }
  }
}
</script>
