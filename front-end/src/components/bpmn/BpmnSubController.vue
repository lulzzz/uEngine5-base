<template>
  <div>
    <sub-controller
      v-if="clone.intermediate"
      cloneable
      :image="'event_intermediate.png'"
      v-on:clone="intermediateClone">
      <circle-element :width="30" :height="30"></circle-element>
    </sub-controller>

    <sub-controller
      v-if="clone.end"
      cloneable
      :image="'event_end.png'"
      v-on:clone="endClone">
      <circle-element :width="30" :height="30"></circle-element>
    </sub-controller>

    <sub-controller
      v-if="clone.gateway"
      cloneable
      :image="'gateway_exclusive.png'"
      v-on:clone="exclusiveGatewayClone">
      <circle-element :width="50" :height="50"></circle-element>
    </sub-controller>

    <sub-controller
      v-if="clone.task"
      cloneable
      :image="'task.png'"
      v-on:clone="taskClone">
      <circle-element :width="100" :height="100"></circle-element>
    </sub-controller>

    <sub-controller
      v-if="clone.annotaion"
      cloneable
      :image="'annotation.png'"
      v-on:clone="annotaionClone">
      <circle-element :width="100" :height="30"></circle-element>
    </sub-controller>

    <sub-controller
      v-if="clone.wrench"
      :image="'wrench.png'"
      v-on:click="showComponentChange">
    </sub-controller>
  </div>

</template>

<script>
  import BpmnVueFinder from './BpmnVueFinder'
  import BpmnComponentFinder from './BpmnComponentFinder'
  export default {
    mixins: [BpmnVueFinder, BpmnComponentFinder],
    name: 'bpmn-sub-controller',
    props: {
      type: String
    },
    computed: {},
    data: function () {
      return {
        clone: {
          intermediate: false,
          end: false,
          gateway: false,
          task: false,
          annotaion: false,
          wrench: false
        }
      }
    },
    watch: {},
    mounted: function () {
      //종료 이벤트인 경우 어노테이션, 도형바꾸기만 가능.
      if (this.type == 'EndEvent') {
        this.clone.annotaion = true;
        this.clone.wrench = true;
      }

      //풀은 도형바꾸기 불가능
      else if (this.type == 'Pool') {
        this.clone.intermediate = true;
        this.clone.end = true;
        this.clone.gateway = true;
        this.clone.task = true;
        this.clone.annotaion = true;
      }

      //데이터나 롤은 어노테이션만 가능
      else if (this.type == 'Data' || this.type == 'Role') {
        this.clone.annotaion = true;
      }

      //나머지는 모두 가능
      else {
        this.clone.intermediate = true;
        this.clone.end = true;
        this.clone.gateway = true;
        this.clone.task = true;
        this.clone.annotaion = true;
        this.clone.wrench = true;
      }
    },
    /**
     * clone : 컨트롤러에 의해 신규 bpmn 이 생성되었을 경우
     * showComponentChange : 컨트롤러중 렌치 모양을 클릭하여 도형 변경 창을 여는 경우
     */
    methods: {
      createEdgeAndElement: function (edgeElement, sourceElement, targetElement, component) {
        var newTracingTag = this.bpmnVue.createNewTracingTag();
        var edgeInfo = {
          from: sourceElement.id,
          to: newTracingTag,
          vertices: '[' + edgeElement.shape.geom.vertices.toString() + ']',
          component: 'bpmn-relation'
        }

        let boundary = targetElement.shape.geom.getBoundary();
        var targetInfo = {
          x: boundary.getCentroid().x,
          y: boundary.getCentroid().y,
          width: boundary.getWidth(),
          height: boundary.getHeight(),
          component: component
        }

        this.bpmnVue.canvas.removeShape(edgeElement);
        this.bpmnVue.canvas.removeShape(targetElement);
        this.bpmnVue.addComponenet(targetInfo, newTracingTag);
        this.bpmnVue.addComponenet(edgeInfo);
      },
      intermediateClone: function (edgeElement, sourceElement, targetElement) {
        this.createEdgeAndElement(edgeElement, sourceElement, targetElement, 'bpmn-intermediate-event');
      },
      endClone: function (edgeElement, sourceElement, targetElement) {
        this.createEdgeAndElement(edgeElement, sourceElement, targetElement, 'bpmn-end-event');
      },
      exclusiveGatewayClone: function (edgeElement, sourceElement, targetElement) {
        this.createEdgeAndElement(edgeElement, sourceElement, targetElement, 'bpmn-exclusive-gateway');
      },
      taskClone: function (edgeElement, sourceElement, targetElement) {
        this.createEdgeAndElement(edgeElement, sourceElement, targetElement, 'bpmn-task');
      },
      annotaionClone: function (edgeElement, sourceElement, targetElement) {
        this.createEdgeAndElement(edgeElement, sourceElement, targetElement, 'bpmn-annotaion');
      },
      showComponentChange: function (event, opengraphComponent) {
        if (this.bpmnComponent) {
          console.log('showComponentChange', this.bpmnComponent.activity.elementView);
          var canvasEl = $(this.bpmnVue.canvas._CONTAINER);
          var x = opengraphComponent.x;
          var y = opengraphComponent.y;
          var width = opengraphComponent.width;
          var height = opengraphComponent.height;

          var pageX = x * this.bpmnVue.canvas.getScale() + canvasEl.offset().left - canvasEl[0].scrollLeft;
          var pageY = y * this.bpmnVue.canvas.getScale() + canvasEl.offset().top - canvasEl[0].scrollTop;
//          var pageX = x + canvasEl.offset().left - canvasEl[0].scrollLeft;
//          var pageY = y + canvasEl.offset().top - canvasEl[0].scrollTop;
          var top = pageY - 70;
          var left = pageX + (width / 2 + 10);
          this.bpmnComponent.openComponentChanger(top, left);
        }
      }
    }
  }
</script>


<style scoped lang="scss" rel="stylesheet/scss">

</style>

