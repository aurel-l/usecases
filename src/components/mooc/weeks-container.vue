<template>
    <div id="course-container" class="course-container">
      <uc-list-viewer :item-list="weeks" @selected="selected">
        <template v-slot:title>
          {{ moocInfo.title }}
          <a v-if="moocInfo.course_url" :href="moocInfo.course_url" class="no-link right">
            <i>Link to the course </i>
            <i class="material-icons middle">link</i>
          </a>
        </template>

        <template v-slot:default="slotProps">
          <uc-item v-bind:uc="slotProps.item" v-bind:categories="categories"/>
        </template>

      </uc-list-viewer>
   </div>
</template>

<script>
import UcItem from '@/components/uc/uc-item.vue';
import UcListViewer from '@/components/uc-list-viewer.vue';
import usecases from '@/assets/config_files/usecases.json';
import collabAuthentication from '@/mixins/collabAuthentication';
import mooc from '@/mixins/mooc';

export default {
  name: 'ucContainer',
  components: {
    UcItem,
    UcListViewer,
  },
  data() {
    return {
      weeks: [],
      categories: usecases[1].categories,
      route: {},
      moocInfo: {},
    };
  },
  mixins: [collabAuthentication, mooc],
  methods: {
    selected(uc) {
      if (!uc.disabled) {
        const weekName = uc.title.toLowerCase().replace(/\s/g, '');

        this.$router.push({
          name: uc.next,
          params: {
            week: weekName,
            moocFullWeeks: this.weeks,
            moocFullName: this.moocInfo.title,
          },
          query: this.$route.query,
        });
      }
    },
    prettyfy(name) {
      return name.split('_').map(word => word.charAt(0).toUpperCase() + word.slice(1)).join(' ');
    },
  },
  mounted() {
    const ucSelected = this.compact(this.$route.params.uc_name);
    // get the overall mooc info (title, url, etc)
    this.moocInfo = usecases[0].mooc.find(moocCourse => this.compact(moocCourse.title) === ucSelected);
    document.querySelector('title').innerText = this.prettyfy(this.moocInfo.title);
    // get the external config for the weeks
    this.getMoocFullConfig(ucSelected)
      .then((config) => {
        this.weeks = config;
      });
  },
};
</script>

<style scoped>
   a.no-link,
   .md-theme-default a:not(.md-button) {
      color: #aacff1;
      cursor: pointer;
      text-decoration: none;
   }
   a.no-link:hover,
   .md-theme-default a:not(.md-button):hover {
      color: #000000;
   }
   .middle {
      vertical-align: middle;
   }
   .right {
      float: right;
   }
</style>
