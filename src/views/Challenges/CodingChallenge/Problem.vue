<template>
  <div>
    <b-breadcrumb :items="ancestors" />
    <b-overlay :show="judging">
      <div>
        <b-tabs content-class="mt-0 p-4 bg-white shadow">
          <b-tab
            title="Problem"
            active
            :title-item-class="['border-left', 'border-right']"
          >
            <loading :show="statement.length == 0">
              <p class="mb-5">{{ statement }}</p>
              <p>
                You can find sample submissions
                <a
                  href="https://github.com/mwashief/ICLP/tree/main/sample_submissions"
                  >here</a
                >
                for testing.
              </p>
              <div v-if="userid" class="max-w-20rem">
                submit solution:
                <b-form @submit.prevent="onsubmit">
                  <FileSubmit id="submission" v-model="submission" />
                  <SolutionLanguages id="language" v-model="language" />
                  <b-button type="submit" variant="primary">Submit</b-button>
                </b-form>
              </div>
              <div v-else>
                <h3>Login/signup to submit solution.</h3>
              </div>
            </loading>
          </b-tab>
          <!--b-tab title="Editor" :title-item-class="['border-right']" lazy>
            <editor buttonAction="Submit" @editorDone="submitFromEditor" />
          </!--b-tab-->
          <b-tab title="Submissions" :title-item-class="['border-right']" lazy>
            <Submissions :problemid="id" />
          </b-tab>
          <b-tab title="Discussions" :title-item-class="['border-right']">
            <div id="disqus_thread"></div>
            <noscript>
              Please enable JavaScript to view the
              <a href="https://disqus.com/?ref_noscript"
                >comments powered by Disqus.</a
              >
            </noscript>
          </b-tab>
        </b-tabs>
      </div>
      <b-modal id="result-modal" centered>
        <b-row
          align-h="center"
          class="text-uppercase"
          :class="{
            'text-success': details === 'correct answer',
            'text-danger': details !== 'correct answer',
          }"
        >
          <h2>{{ details }}</h2>
        </b-row>
        <b-row align-h="center" class="mt-3">Score: {{ score }}</b-row>
      </b-modal>
    </b-overlay>
  </div>
</template>

<script>
import apiUtil from "@/mixins/apiUtil";
import Submissions from "@/components/Submissions";
import FileSubmit from "@/components/FileSubmit";
import SolutionLanguages from "@/components/SolutionLanguages";
import thisuser from "@/mixins/thisuser";
import Loading from "@/components/Loading.vue";

export default {
  name: "Problem",
  mixins: [apiUtil, thisuser],
  props: ["id"],
  data() {
    return {
      ancestors: [
        {
          text: "Coding challenge",
          href: "/problems",
        },
        {
          text: null,
        },
      ],
      statement: "",
      submission: null,
      language: "c++ 17",
      judging: false,
      score: 0,
      details: "",
    };
  },
  methods: {
    onsubmit() {
      this.judging = true;
      let formData = new FormData();
      formData.append("problemid", this.id);
      formData.append("language", this.language);
      formData.append("submission", this.submission);
      for (var pair of formData.entries()) {
        console.log(pair[0] + ", " + pair[1]);
      }
      this.apiPost("/protected/submit", formData, {
        headers: {
          "Content-Type": "multipart/form-data",
        },
      })
        .then(({ score, details }) => {
          this.score = score;
          this.details = details;
          this.$root.$bvModal.show("result-modal");
        })
        .catch((err) => {
          alert("error " + err);
        })
        .finally(() => {
          this.submission = null;
          this.judging = false;
        });
    },
    submitFromEditor(payload) {
      console.log(payload);
      alert("Hasn't implemented yet. See console for the object.");
    },
  },
  components: {
    Submissions,
    FileSubmit,
    SolutionLanguages,
    Loading,
  },
  mounted() {
    /**
     *  RECOMMENDED CONFIGURATION VARIABLES: EDIT AND UNCOMMENT THE SECTION BELOW TO INSERT DYNAMIC VALUES FROM YOUR PLATFORM OR CMS.
     *  LEARN WHY DEFINING THESE VARIABLES IS IMPORTANT: https://disqus.com/admin/universalcode/#configuration-variables    */
    window.disqus_config = function () {
      // this.page.url = PAGE_URL;  // Replace PAGE_URL with your page's canonical URL variable
      this.page.identifier = `problem_${this.id}`; // Replace PAGE_IDENTIFIER with your page's unique identifier variable
    };
    (function () {
      // DON'T EDIT BELOW THIS LINE
      var d = document,
        s = d.createElement("script");
      s.src = "https://iclp.disqus.com/embed.js";
      s.setAttribute("data-timestamp", +new Date());
      (d.head || d.body).appendChild(s);
    })();

    this.apiGet(`/public/problem/${this.id}`).then(([data]) => {
      this.ancestors[this.ancestors.length - 1].text = data.title;
      this.statement = data.statement;
    });
  },
};
</script>

<style lang="scss" scoped></style>
