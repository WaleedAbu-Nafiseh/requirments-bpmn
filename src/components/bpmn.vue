<template>
  <div ref="container" class="vue-bpmn-diagram-container"></div>
  <button class="save" @click="save">save</button>
</template>

<script>
import BpmnModeler from "waleed-bpmn-js/lib/Modeler";

export default {
  name: "VueBpmn",
  props: {
    url: {
      type: String,
      required: true,
    },
    options: {
      type: Object,
    },
  },
  data: function () {
    return {
      diagramXML: null,
    };
  },
  mounted: function () {
    var container = this.$refs.container;

    var self = this;
    var _options = Object.assign(
      {
        container: container,
      },
      this.options
    );
    this.bpmnViewer = new BpmnModeler(_options);

    this.bpmnViewer.on("import.done", function (event) {
      var error = event.error;
      var warnings = event.warnings;

      if (error) {
        self.$emit("error", error);
      } else {
        self.$emit("shown", warnings);
      }
      var eventBus = self.bpmnViewer.get("eventBus");

      // you may hook into any of the following events
      var events = [
        "element.hover",
        "element.out",
        "element.click",
        "element.dblclick",
        "element.mousedown",
        "element.mouseup",
      ];

      events.forEach(function (event) {
        eventBus.on(event, function (e) {
          // e.element = the model element
          // e.gfx = the graphical element

          console.log(event, "on", e.element.id);
        });
      });

      self.bpmnViewer.get("canvas").zoom("fit-viewport");
    });

    if (this.url) {
      this.fetchDiagram(this.url);
    }
  },
  beforeUnmount: function () {
    this.bpmnViewer.destroy();
  },
  watch: {
    url: function (val) {
      this.$emit("loading");
      this.fetchDiagram(val);
    },
    diagramXML: function (val) {
      this.bpmnViewer.importXML(val);
    },
  },
  methods: {
    save: function () {
      this.bpmnViewer.saveXML().then((xml) => {
        console.log(xml.xml);
      });
    },
    fetchDiagram: function (url) {
      var self = this;

      fetch(url)
        .then(function (response) {
          return response.text();
        })
        .then(function (text) {
          self.diagramXML = text;
        })
        .catch(function (err) {
          self.$emit("error", err);
        });
    },
  },
};
</script>
<style>
.vue-bpmn-diagram-container {
  height: 100%;
  width: 100%;
}
</style>