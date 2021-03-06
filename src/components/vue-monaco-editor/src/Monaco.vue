<template>
  <div :style="style"></div>
</template>

<script>
import monacoLoader from './MonacoLoader'
const debounce = require('lodash.debounce');

export default {
  props: {
    // 编辑器的宽，默认 100%
    width: { type: [String, Number], default: '100%' },
    // 编辑器的高，默认 100%
    height: { type: [String, Number], default: '100%' },
    // 传进来的代码，一段字符串
    code: { type: String, default: '// code \n' },
    // 资源路径
    srcPath: { type: String },
    // 默认使用 js
    language: { type: String, default: 'javascript' },
    // 主题 默认 vs-dark
    theme: { type: String, default: 'vs-dark' }, // vs, hc-black
    // 一些 monaco 配置参数
    options: { type: Object, default: () => {} },
    // 截流
    changeThrottle: { type: Number, default: 0 }
  },
  // 加载资源
  created() {
    this.fetchEditor();
  },
  // 销毁
  destroyed() {
    this.destroyMonaco();
  },
  computed: {
    style() {
      const { width, height } = this;
      const fixedWidth = width.toString().indexOf('%') !== -1 ? width : `${width}px`;
      const fixedHeight = height.toString().indexOf('%') !== -1 ? height : `${height}px`;
      return {
        width: fixedWidth,
        height: fixedHeight,
      };
    },
    editorOptions() {
      return Object.assign({}, this.defaults, this.options, {
        value: this.code,
        language: this.language,
        theme: this.theme
      });
    }
  },
  data() {
    return {
      defaults: {
        selectOnLineNumbers: true,
        roundedSelection: false,
        readOnly: false,
        cursorStyle: 'line',
        automaticLayout: false,
        glyphMargin: true
      }
    }
  },
  methods: {
    editorHasLoaded(editor, monaco) {
      this.editor = editor;
      this.monaco = monaco;
      this.editor.onDidChangeModelContent(event =>
        this.codeChangeHandler(editor, event)
      );
      this.$emit('mounted', editor);
    },
    codeChangeHandler: function(editor) {
      if (this.codeChangeEmitter) {
        this.codeChangeEmitter(editor);
      } else {
        this.codeChangeEmitter = debounce(
          function(editor) {
            this.$emit('codeChange', editor);
          },
          this.changeThrottle
        );
        this.codeChangeEmitter(editor);
      }
    },
    fetchEditor() {
      monacoLoader.load(this.srcPath, this.createMonaco);
    },
    createMonaco() {
      this.editor = window.monaco.editor.create(this.$el, this.editorOptions);
      this.editorHasLoaded(this.editor, window.monaco);
    },
    destroyMonaco() {
      if (typeof this.editor !== 'undefined') {
        this.editor.dispose();
      }
    }
  }
};
</script>
