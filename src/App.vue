<template>
  <b-form class="printable-form m-4" id="pForm" @submit.prevent="calcAvg">
    <div class="logo">
      <a href="https://www.galacticadvisors.com/" target="_blank">
        <img src="./assets/ga-logo.png" alt="Galactic Advisors" width="204" height="100" />
      </a>
    </div>
    <h1 class="h1 ml-3 mt-3 mb-0">INSURANCE ASSESSMENT</h1>
    <h2 class="h2 ml-3">CLIENT NAME</h2>
    <b-container fluid class="my-4">
      <b-row>
        <b-col sm="3" class="user-label mt-2 font-weight-bold">
          <label for="dateConductedField">Conducted on:</label>
        </b-col>
        <b-col sm="9">
          <b-form-datepicker v-if="!isPrintPreview" class="user-input no-print mb-3" id="dateConductedField" v-model="dateConductedOn" />
          <div :class="['user-input-results mt-2 print', { 'show': isPrintPreview }]">{{ dateConductedOn }}</div> 
        </b-col>
      </b-row>
      <b-row>
        <b-col sm="3" class="user-label mt-2 font-weight-bold">
          <label for="dateAcceptedField">Date Accepted:</label>
        </b-col>
        <b-col sm="9">
          <b-form-datepicker v-if="!isPrintPreview" class="user-input no-print mb-3" id="dateAcceptedField" v-model="dateAccepted" />
          <div :class="['user-input-results mt-2 print', { 'show': isPrintPreview }]">{{ dateAccepted }}</div>
        </b-col>
      </b-row>
      <b-row>
        <b-col sm="3" class="user-label mt-2 font-weight-bold">
          <label for="reviewerNameField">Reviewer Name:</label>
        </b-col>
        <b-col sm="9">
          <b-form-input v-if="!isPrintPreview" class="user-input no-print" id="reviewerNameField" v-model="reviewerName" />
          <div :class="['user-input-results mt-2 print', { 'show': isPrintPreview }]">{{ reviewerName }}</div>
        </b-col>
      </b-row>
    </b-container>
    <div v-if="showResults" class="calc-container p-4">
      <span class="calc-score-red">{{ calculatedAverage }}%</span>
      <span class="calc-score-title">Overall Cyber Insecurity Score: </span>
      <span class="calc-score">{{ calculatedAverage }}% - {{ calcAvgText }}</span>
    </div>
    <div class="no-print p-3 mt-5" v-if="!isPrintPreview">Please answer all questions below (drag to reorder):</div>
    <b-container class="my-5" fluid v-for="s of sections" :key="s.sectionId">
      <h3 class="mt-5">
        <span class="mr-3">{{ s.sectionLabel }}</span>
        <span class="no-print mr-2" v-b-modal="'modal-section-rename-' + s.sectionId" v-if="!isPrintPreview">
          <b-icon class="section-btn" icon="pencil-fill" title="Edit Section"></b-icon>
        </span>
        <span class="no-print" v-b-modal="'modal-section-delete-' + s.sectionId" v-if="!isPrintPreview">
          <b-icon class="section-btn" icon="trash-fill" title="Delete Section"></b-icon>
        </span>
      </h3>
      <b-modal :id="'modal-section-rename-' + s.sectionId" title="Rename Section" @ok="renameSection(s.sectionId)">
        <label :for="'rename-section-' + s.sectionId">Rename section to:</label>
        <b-form-input :id="'rename-section-' + s.sectionId" v-model="s.sectionNewLabel" />
      </b-modal>
      <b-modal :id="'modal-section-delete-' + s.sectionId" title="Delete Section" @ok="deleteSection(s.sectionId)">
        <p>Really delete this section?</p>
      </b-modal>
      <draggable v-model="s.questions" tag="div" class="question-container" :options="{group: 'sections'}">
        <b-list-group-item v-for="q of s.questions" :key="q.id">
          <b-row>
            <b-col :class="[ 'reorder-col', { 'reorder-wide' : isPrintPreview} ]" sm="1">
              <div class="reorder mb-2 no-print" v-if="!isPrintPreview">
                <b-icon icon="list"></b-icon>
              </div>
              <div :class="[ 'answer font-weight-bold print', 
                {'show': isPrintPreview, 'green': q.value == 1, 'red' : q.value == 0  } ]" sm="1">
                <span class="result-icon">
                  <b-icon :icon="q.value == 1 || q.value == 0 ? 'square-fill' : 'slash-square'"></b-icon>
                </span>
                <span class="result">
                  {{q.value == 1 ? "Yes" : q.value == 0 ? "No" : "No Answer" }}
                </span>
              </div>
            </b-col>
            <b-col class="p-0 ml-2" sm="8">
              <b-form-group class="question-text font-weight-bold" :label="`${q.label}`">
                <b-form-radio-group
                  class="no-print font-weight-normal"
                  v-model="q.value"
                  :options="q.options"
                  v-if="!isPrintPreview"
                >
                </b-form-radio-group>
                <b-container class="mt-2">
                  <b-row class="comment-area no-print" v-if="!isPrintPreview">
                    <b-col class="comment-label pt-2 pl-0" sm="3">
                      <label :for="'comment-' + s.sectionId + '-' + q.id">Add comment: </label>
                    </b-col>
                    <b-col class="p-0" sm="9">
                      <b-form-input :id="'comment-' + s.sectionId + '-' + q.id" v-model="q.comment" />
                    </b-col>
                  </b-row>
                  <b-row :class="['comment-label-result font-weight-normal', { 'show': isPrintPreview }]">
                    <b-col>
                      {{ q.comment }}
                    </b-col>
                  </b-row>
                </b-container>
              </b-form-group>
            </b-col>
            <b-col class="delete-col no-print" sm="3" v-if="!isPrintPreview">
              <b-button class="remove-btn" variant="danger" v-b-modal="'modal-' + s.sectionId + '-' + q.id">Delete</b-button>
              <b-modal :id="'modal-' + s.sectionId + '-' + q.id" 
              title="Confirm Delete" @ok="deleteQuestion(s.sectionId, q.id)">
                <p>Really delete this question?</p>
              </b-modal>
            </b-col>
          </b-row>
        </b-list-group-item>
      </draggable>
    </b-container>
    <b-container fluid class="no-print mt-5 mb-5" v-if="!isPrintPreview">
      <h4 class="mb-4">Edit Form Fields</h4>
      <b-row>
        <b-col sm="3" class="fixed-label mt-2">
          <label for="addSection">Add a section:</label>
        </b-col>
        <b-col sm="5" class="mb-3">
          <b-form-input class="addSection" v-model="addSectionText" />
        </b-col>
        <b-col sm="4">
          <b-button @click="addSection" :disabled="hasSectionText">Add section</b-button>
        </b-col>
      </b-row>
      <b-row>
        <b-col sm="3" class="fixed-label mt-2">
          <label for="addQuestion">New question:</label>
        </b-col>
        <b-col sm="5" class="mb-3">
          <b-form-input id="addQuestion" v-model="addQuestionText" />
        </b-col>
      </b-row>
      <b-row>
        <b-col sm="3" class="fixed-label mt-2">
          <label for="addToSection">Add to section:</label>
        </b-col>
        <b-col sm="5" class="mb-3">
          <b-form-select id="addToSection" v-model="selectedSectionId">
            <b-form-select-option v-for="s of sections" :key="s.sectionId" :value="s.sectionId">{{ s.sectionLabel }}</b-form-select-option>
          </b-form-select>
        </b-col>
        <b-col sm="4">
          <b-button @click="addQuestion" :disabled="hasQuestionText">Add question</b-button>
        </b-col>
      </b-row>
    </b-container>
    <div class="no-print m-3" v-show="!isPrintPreview">
      <b-button
        type="submit"
        class="mr-2 mb-2"
        variant="primary"
        >Calculate Average</b-button
      >
      <b-button class="mr-2 mb-2" @click="printPreview">Print Preview</b-button>
      <b-button class="mr-2 mb-2" @click="printPage">Print</b-button>
      <b-button class="mr-2 mb-2" variant="danger" v-b-modal.modal-reset>Reset Form</b-button>
      <b-modal id="modal-reset" title="Confirm Reset" @ok="resetForm">
        <p>Really reset the form? This will remove all questions and answers you have added.</p>
      </b-modal>
    </div>
    <div v-if="showResults" class="calc-container p-4 mb-4">
      <span class="calc-score-red">{{ calculatedAverage }}%</span>
      <span class="calc-score-title">Overall Cyber Insecurity Score: </span>
      <span class="calc-score">{{ calculatedAverage }}% - {{ calcAvgText }}</span>
    </div>
    <div class="no-print my-5 mx-3" v-if="isPrintPreview">
      <b-button @click="exitPrintPreview">Exit Print Preview</b-button>
    </div>
  </b-form>
</template>

<script>
import draggable from "vuedraggable";

export default {
  components: {
    draggable,
  },
  data() {
    return {
      dateConductedOn: "Date",
      dateAccepted: "Date",
      reviewerName: "Jonathan Bettencourt",
      addQuestionText: "",
      addSectionText: "",
      showResults: false,
      calculatedAverage: 0,
      isPrintPreview: false,
      selectedSectionId: '',
      calcAvgText: 'Low',
      sections: [
        {
          sectionLabel: "First Section",
          sectionNewLabel: "",
          sectionId: 1,
          questions: [
            {
              id: 1,
              value: null,
              label: "Are you age 21 or older?",
              type: "radio",
              comment: "",
              options: [
                {
                  text: "Yes",
                  value: "1",
                },
                {
                  text: "No",
                  value: "0",
                },
              ],
            },
            {
              id: 2,
              value: null,
              label: "Are you a college graduate?",
              type: "radio",
              comment: "",
              options: [
                {
                  text: "Yes",
                  value: "1",
                },
                {
                  text: "No",
                  value: "0",
                },
              ],
            },
          ]
        },
        {
          sectionLabel: "Second Section",
          sectionNewLabel: "",
          sectionId: 2,
          questions: [
            {
              id: 1,
              value: null,
              label: "Were you employed within the last 6 months?",
              type: "radio",
              comment: "",
              options: [
                {
                  text: "Yes",
                  value: "1",
                },
                {
                  text: "No",
                  value: "0",
                },
              ],
            },
            {
              id: 2,
              value: null,
              label: "Are you a U.S. Citizen?",
              type: "radio",
              comment: "",
              options: [
                {
                  text: "Yes",
                  value: "1",
                },
                {
                  text: "No",
                  value: "0",
                },
              ],
            },
          ]
        }
      ],
      dragging: false,
    };
  },
  methods: {
    renameSection(sectionId) {
      this.sections.map((s) => {
        if (s.sectionId == sectionId) {
          s.sectionLabel = s.sectionNewLabel;
          s.sectionNewLabel = "";
        }
      });
    },
    deleteSection(sectionId) {
      this.sections = this.sections.filter((s) => {
        return s.sectionId !== sectionId;
      });
    },
    addSection() {
      this.sections.push({
        sectionLabel: this.addSectionText,
        sectionNewLabel: "",
        sectionId: this.sections.length + 1,
        questions: []
      });

      this.addSectionText = "";
      this.showResults = false;
      this.calculatedAverage = 0;
    },
    addQuestion() {
      this.sections.map((s) => {
        if (s.sectionId == this.selectedSectionId) {
          s.questions.push({
            id: s.questions.length + 1,
            value: null,
            label: this.addQuestionText,
            type: "radio",
            options: [
              {
                text: "Yes",
                value: "1",
              },
              {
                text: "No",
                value: "0",
              },
            ]
          });
        }
      });

      this.addQuestionText = "";
      this.showResults = false;
      this.calculatedAverage = 0;
    },
    deleteQuestion(sectionId, id) {
      this.sections.map((s) => {
        if (s.sectionId == sectionId) {
          s.questions = s.questions.filter((q) => {
            return q.id !== id;
          });
        }
      });
    },
    calcAvg() {
      let yesCount = 0;
      let totalQCount = 0;
      this.calculatedAverage = 0;
      this.sections.map((s) => {
        s.questions.map((q) => {
          if (q.value == 1) {
            yesCount++;
          }
        });
        totalQCount = totalQCount + s.questions.length;
      });
      if (yesCount != 0) {
        this.calculatedAverage = (yesCount / totalQCount) * 100;
      } else {
        this.calculatedAverage = 0;
      }
      this.calcAvgText = this.calculatedAverage <= 25 ? 'Low' : 
        this.calculatedAverage <= 50 ? 'Fair' : 
        this.calculatedAverage <= 75 ? 'Good' :
        this.calculatedAverage <= 100 ? 'Excellent' : 
        'N/A';
      this.showResults = true;
    },
    printPreview() {
      this.isPrintPreview = true;
    },
    exitPrintPreview() {
      this.isPrintPreview = false;
    },
    printPage() {
       window.print();
    },
    resetForm() {
      location.reload();
    }
  },
  computed: {
    hasSectionText() {
      return this.addSectionText == "";
    },
    hasQuestionText() {
      return this.addQuestionText == "";
    }
  }
};
</script>

<style>
@import url(https://fonts.googleapis.com/css?family=Bebas+Neue:regular);
@import url(https://fonts.googleapis.com/css?family=Open+Sans:300,regular,500,600,700,800,300italic,italic,500italic,600italic,700italic,800italic);

.printable-form {
  font-family:'Open Sans';
  max-width:1080px;
  position:relative;
}
.modal-dialog {
  font-family:'Open Sans';
}
.logo {
  position:absolute;
  top:0;
  right:0;
  margin-right:30px;
}
h1.h1 {
  font-family: 'Bebas Neue';
  font-size: 40px;
  color: grey;
  line-height: 1;
  margin-right:250px;
}
h2.h2 {
  font-family: 'Bebas Neue';
  font-size: 60px;
  line-height: 1;
  margin-right:250px;
}
div.user-label {
  max-width:200px;
}
input.user-input,
div.user-input {
  max-width:300px;
}
.section-btn:hover {
  cursor:pointer;
}
div.fixed-label {
  max-width: 150px;
}
div.reorder-col {
  max-width:50px;
  padding-left:0;
}
div.reorder-wide {
  min-width:150px;
  margin-left:10px;
  margin-bottom:10px;
}
.reorder {
  font-size: 32px;
  margin-top: -12px;
}
.reorder:hover {
  cursor: move;
}
div.delete-col {
  max-width:100px;
}
div.answer,
div.user-input-results,
div.comment-label-result {
  display:none;
}
div.answer.show,
div.user-input-results.show,
div.comment-label-result.show {
  display:block;
}
.list-group-item:nth-of-type(even) {
  background-color: rgba(0, 0, 0, 0.05);
}
.calc-container {
  border-top:1px #dedede solid;
  border-bottom:1px #dedede solid;
}
.calc-score-red {
  color:#fff;
  background-color:#ef4265;
  width:50px;
  text-align:center;
  font-size:20px;
  font-weight:bold;
  padding:10px 30px;
  margin-right:20px;
}
.calc-score-title {
  font-weight:bold;
  font-size:20px;
}
.calc-score {
  font-size:20px;
}
.question-container .list-group-item {
  border:none;
}
.result-icon {
  font-size:18px;
  margin-right:10px;
}
.result {
  font-size:12px;
}
.answer {
  color:grey;
}
.green span {
  color:green;
}
.red span {
  color:red;
}
.question-text {
  font-size:16px;
}
div.comment-label {
  font-size:16px;
  font-weight:normal;
  min-width:150px;
  max-width:150px;
}
.comment-label-result {
  background-color:rgba(0, 0, 0, 0.05);
  font-size:16px;
  color:grey;

} 
.list-group-item:nth-of-type(even) .comment-label-result {
  background-color: #fff;
}
@media print {
  div.user-input.no-print,
  div.comment-area,
  div.no-print,
  span.no-print,
  input.no-print {
    display:none;
  }
  div.answer.print,
  div.user-input-results.print,
  div.comment-label-result {
    display:block;
  }
  div.reorder-col {
    min-width:150px;
  }
}
</style>