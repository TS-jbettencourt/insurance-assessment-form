<template>
  <b-form class="printable-form" id="pForm" @submit.prevent="calcAvg">
    <div class="no-print m-3" v-if="isPrintPreview">
      <b-button @click="exitPrintPreview">Exit Print Preview</b-button>
    </div>
    <h1 class="m-3">Galactic Advisors - Printable Form</h1>
    <div class="no-print p-3" v-if="!isPrintPreview">Please answer all questions below (drag to reorder):</div>
    <b-container fluid>
      <draggable v-model="questions" tag="div">
        <b-list-group-item v-for="q of questions" :key="q.id">
          <b-row>
            <b-col class="reorder-col no-print" sm="1" v-if="!isPrintPreview">
              <p class="reorder h3 mb-2">
                <b-icon icon="list"></b-icon>
              </p>
            </b-col>
            <b-col>
              <b-form-group :label="`${q.label}`">
                <b-form-radio-group
                  class="no-print"
                  v-model="q.value"
                  :options="q.options"
                  v-if="!isPrintPreview"
                >
                </b-form-radio-group>
                <div :class="[ 'answer font-weight-bold print', {'show': isPrintPreview } ]">
                  {{
                    q.value == 1 ? "Yes" : q.value == 0 ? "No" : "Unselected"
                  }}
                </div>
              </b-form-group>
            </b-col>
            <b-col class="delete-col no-print" sm="3" v-if="!isPrintPreview">
              <b-button class="remove-btn" variant="danger" v-b-modal="'modal-' + q.id">Delete</b-button>
              <b-modal :id="'modal-' + q.id" title="Confirm Delete" @ok="deleteQuestion(q.id)">
                <p>Really delete this question?</p>
              </b-modal>
            </b-col>
          </b-row>
        </b-list-group-item>
      </draggable>
    </b-container>
    <b-container fluid class="no-print mt-4 mb-4" v-if="!isPrintPreview">
      <b-row>
        <b-col sm="3" class="fixed-label mt-2">
          <label for="addQuestion">Add a question:</label>
        </b-col>
        <b-col sm="5" class="mb-3">
          <b-form-input v-model="addQuestionText" />
        </b-col>
        <b-col sm="4">
          <b-button @click="addQuestion" :disabled="hasQuestionText"
            >Add question</b-button
          >
        </b-col>
      </b-row>
    </b-container>
    <div class="no-print m-3" v-show="!isPrintPreview">
      <b-button
        type="submit"
        class="mr-2 mb-2"
        variant="primary"
        :disabled="validateForm"
        >Calculate Average</b-button
      >
      <b-button class="mr-2 mb-2" @click="printPreview">Print Preview</b-button>
      <b-button class="mr-2 mb-2" @click="printPage">Print</b-button>
      <b-button class="mr-2 mb-2" variant="danger" v-b-modal.modal-reset>Reset Form</b-button>
      <b-modal id="modal-reset" title="Confirm Reset" @ok="resetForm">
        <p>Really reset the form? This will remove all questions and answers you have added.</p>
      </b-modal>
    </div>
    <div v-if="showResults" class="font-weight-bold p-3">
      Calculated Average: {{ calculatedAverage }}
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
      addQuestionText: "",
      showResults: false,
      calculatedAverage: 0,
      isPrintPreview: false,
      questions: [
        {
          id: 1,
          value: null,
          label: "Are you at least age 21?",
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
          ],
        },
        {
          id: 2,
          value: null,
          label: "Are you a college graduate?",
          type: "radio",
          name: "college",
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
      ],
      dragging: false,
    };
  },
  methods: {
    addQuestion() {
      const newQuestion = {
        id: this.questions.length + 1,
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
        ],
      };
      this.questions.push(newQuestion);
      this.addQuestionText = "";
      this.showResults = false;
      this.calculatedAverage = 0
    },
    deleteQuestion(id) {
      this.questions = this.questions.filter((q) => {
        return q.id !== id;
      });
    },
    calcAvg() {
      let yesCount = 0;
      this.calculatedAverage = 0;
      this.questions.map((q) => {
        if (q.value == 1) {
          yesCount++;
        }
      });
      if (yesCount != 0) {
        this.calculatedAverage = yesCount / this.questions.length;
      } else {
        this.calculatedAverage = 0;
      }
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
    hasQuestionText() {
      return this.addQuestionText == "";
    },
    validateForm() {
      let isValidated = false;
      this.questions.map((q) => {
        if (q.value == null) {
          isValidated = true;
        }
      });
      return isValidated;
    },
  },
};
</script>

<style>
.printable-form {
  max-width:1080px;
}
div.fixed-label {
  max-width: 150px;
}
div.reorder-col {
  max-width:50px;
}
.reorder:hover {
  cursor: move;
}
div.delete-col {
  max-width:100px;
}
div.answer {
  display:none;
}
div.answer.show {
  display:block;
}
.list-group-item:nth-of-type(even) {
  background-color: rgba(0, 0, 0, 0.05);
}
@media print {
  div.no-print {
    display:none;
  }
  div.answer.print {
    display:block;
  }
}
</style>