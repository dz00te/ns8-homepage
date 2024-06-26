<template>
  <div class="bx--grid bx--grid--full-width">
    <div class="bx--row">
      <div class="bx--col-lg-16 page-title">
        <h2>{{ $t("about.title") }}</h2>
      </div>
    </div>
    <div class="bx--row">
      <div class="bx--col-lg-16">
        <cv-tile :light="true">
          <NsInlineNotification
            v-if="error.version"
            kind="error"
            :title="$t('error.cannot_retrieve_installed_modules')"
            :description="error.version"
            :showCloseButton="false"
          />
          <cv-skeleton-text
            v-if="loading.moduleInfo"
            :paragraph="true"
            :line-count="12"
            width="80%"
          ></cv-skeleton-text>
          <div v-else-if="error.moduleInfo">
            <NsInlineNotification
              kind="error"
              :title="$t('error.cannot_retrieve_module_info')"
              :description="error.moduleInfo"
              :showCloseButton="false"
            />
          </div>
          <div v-else>
            <div class="logo-and-name">
              <div class="app-logo">
                <img
                  :src="
                    app.logo
                      ? app.logo
                      : require('@/assets/homepage_logo.png')
                  "
                  :alt="app.name + ' logo'"
                />
              </div>
              <div class="app-name">
                <h3>{{ app.name }}</h3>
              </div>
            </div>
            <div class="description">
              {{ getApplicationDescription(app) }}
            </div>
            <div class="key-value-setting">
              <span class="label">{{
                core.$t("software_center.instance")
              }}</span>
              <span class="value">{{ instanceName }}</span>
            </div>
            <div class="key-value-setting">
              <span class="label">{{ core.$t("common.version") }}</span>
              <span class="value">{{ version }}</span>
            </div>
            <div class="key-value-setting">
              <span class="label">{{
                core.$tc("software_center.categories", app.categories.length)
              }}</span>
              <span class="value">{{ getApplicationCategories(app) }}</span>
            </div>
            <div class="key-value-setting">
              <span class="label">{{
                core.$t("software_center.documentation")
              }}</span>
              <span class="value">
                <cv-link :href="app.docs.documentation_url" target="_blank">
                  {{ app.docs.documentation_url }}
                </cv-link>
              </span>
            </div>
            <div class="key-value-setting">
              <span class="label">{{ core.$t("software_center.bugs") }}</span>
              <span class="value">
                <cv-link :href="app.docs.bug_url" target="_blank">
                  {{ app.docs.bug_url }}
                </cv-link>
              </span>
            </div>
            <div class="key-value-setting">
              <span class="label">{{
                core.$t("software_center.source_code")
              }}</span>
              <span class="value">
                <cv-link :href="app.docs.code_url" target="_blank">
                  {{ app.docs.code_url }}
                </cv-link>
              </span>
            </div>
            <div class="key-value-setting">
              <span class="label">{{
                core.$t("software_center.source_package")
              }}</span>
              <span class="value">
                {{ app.source }}
              </span>
            </div>
            <div class="key-value-setting">
              <span class="label">{{
                core.$tc("software_center.authors", app.authors.length)
              }}</span>
              <span class="value">
                <span v-if="app.authors.length == 1"
                  >{{ app.authors[0].name }}
                  <cv-link
                    v-if="app.authors[0].email"
                    :href="'mailto:' + app.authors[0].email"
                    target="_blank"
                    class="email"
                  >
                    {{ app.authors[0].email }}
                  </cv-link>
                </span>
                <ul v-else class="authors">
                  <li
                    v-for="(author, index) in app.authors"
                    :key="index"
                    class="author"
                  >
                    {{ author.name }}
                    <cv-link
                      v-if="author.email"
                      :href="'mailto:' + author.email"
                      target="_blank"
                      class="email"
                    >
                      {{ author.email }}
                    </cv-link>
                  </li>
                </ul>
              </span>
            </div>
          </div>
        </cv-tile>
      </div>
    </div>
  </div>
</template>

<script>
import to from "await-to-js";
import { mapState } from "vuex";
import {
  QueryParamService,
  TaskService,
  UtilService,
} from "@nethserver/ns8-ui-lib";

export default {
  name: "About",
  components: {},
  mixins: [TaskService, QueryParamService, UtilService],
  pageTitle() {
    return this.$t("about.title") + " - " + this.appName;
  },
  data() {
    return {
      q: {
        page: "about",
      },
      urlCheckInterval: null,
      app: null,
      version: "-",
      error: {
        moduleInfo: "",
        version: "",
      },
      loading: {
        moduleInfo: true,
        version: true,
      },
    };
  },
  computed: {
    ...mapState(["core", "appName", "instanceName"]),
  },
  created() {
    this.getModuleInfo();

    // needed to retrieve module version
    this.listInstalledModules();
  },
  beforeRouteEnter(to, from, next) {
    next((vm) => {
      vm.watchQueryData(vm);
      vm.urlCheckInterval = vm.initUrlBindingForApp(vm, vm.q.page);
    });
  },
  beforeRouteLeave(to, from, next) {
    clearInterval(this.urlCheckInterval);
    next();
  },
  methods: {
    getModuleInfo() {
      this.loading.moduleInfo = true;
      const metadata = require("../../public/metadata.json");
      this.app = metadata;
      this.loading.moduleInfo = false;
    },
    getApplicationDescription(app) {
      return this.getAppDescription(app, this.core);
    },
    getApplicationCategories(app) {
      return this.getAppCategories(app, this.core);
    },
    async listInstalledModules() {
      const taskAction = "list-installed-modules";

      // register to task completion
      this.core.$root.$once(
        taskAction + "-completed",
        this.listInstalledModulesCompleted
      );

      const res = await to(
        this.createClusterTaskForApp({
          action: taskAction,
          extra: {
            title: this.core.$t("action." + taskAction),
            isNotificationHidden: true,
          },
        })
      );
      const errApps = res[0];

      if (errApps) {
        console.error("error retrieving installed apps", errApps);
        this.error.version = this.getErrorMessage(errApps);
        this.loading.version = false;
        return;
      }
    },
    listInstalledModulesCompleted(taskContext, taskResult) {
      let apps = [];

      for (let instanceList of Object.values(taskResult.output)) {
        for (let instance of instanceList) {
          apps.push(instance);
        }
      }
      const app = apps.find((el) => el.id == this.instanceName);
      this.version = app.version;
      this.loading.version = false;
    },
  },
};
</script>

<style scoped lang="scss">
@import "../styles/carbon-utils";

.logo-and-name {
  display: flex;
  align-items: center;
  margin-top: $spacing-05;
  margin-bottom: $spacing-07;
}

.app-logo {
  max-width: 4rem;
  max-height: 4rem;
  margin-right: $spacing-05;
  flex-shrink: 0;
}

.app-logo img {
  width: 100%;
  height: 100%;
}

.description {
  margin-bottom: $spacing-07;
}

section {
  margin-bottom: $spacing-05;
}

.section-title {
  font-weight: bold;
}

.author {
  margin-left: $spacing-05;
  margin-bottom: $spacing-02;
}

.authors {
  margin-top: $spacing-02;
}

.email {
  margin-left: $spacing-02;
}
</style>
