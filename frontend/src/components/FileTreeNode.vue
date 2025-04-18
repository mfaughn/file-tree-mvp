<!-- FileTreeNode.vue -->
<template>
  <li 
    :class="['tree-node', { 'selected': isSelected }]"
    @click.stop="select"
  >
    <div 
      :class="['node-content', { 'is-folder': node.type === 'folder' }]"
      draggable="true"
      @dragstart="onDragStart"
      @dragover.prevent
      @dragenter.prevent="onDragEnter"
      @dragleave="onDragLeave"
      @drop.prevent="onDrop"
    >
      <span class="node-icon">
        <i :class="node.type === 'folder' ? 'folder-icon' : 'file-icon'"></i>
      </span>
      <span class="node-name">{{ node.name }}</span>
    </div>
    
    <ul v-if="node.type === 'folder' && childNodes.length > 0" class="node-children">
      <file-tree-node
        v-for="child in childNodes"
        :key="child.id"
        :node="child"
        @node-selected="onChildSelected"
        @node-dropped="onChildDropped"
      ></file-tree-node>
    </ul>
  </li>
</template>

<script>
export default {
  name: 'FileTreeNode',
  props: {
    node: {
      type: Object,
      required: true
    }
  },
  inject: ['findNodeById', 'getChildNodes'],
  data() {
    return {
      isSelected: false,
      isDragOver: false
    };
  },
  computed: {
    childNodes() {
      return this.getChildNodes(this.node.id);
    }
  },
  methods: {
    select() {
      this.isSelected = true;
      this.$emit('node-selected', this.node);
    },
    onChildSelected(node) {
      this.isSelected = false;
      this.$emit('node-selected', node);
    },
    onDragStart(event) {
      event.dataTransfer.effectAllowed = 'move';
      event.dataTransfer.setData('text/plain', this.node.id);
      // Add a custom class to the drag image
      event.target.classList.add('dragging');
      // Select the node when starting to drag
      this.select();
    },
    onDragEnter(event) {
      this.isDragOver = true;
      event.currentTarget.classList.add('drag-over');
    },
    onDragLeave(event) {
      this.isDragOver = false;
      event.currentTarget.classList.remove('drag-over');
    },
    onDrop(event) {
      event.currentTarget.classList.remove('drag-over');
      const sourceId = parseInt(event.dataTransfer.getData('text/plain'));
      const targetId = this.node.id;
      
      // Don't allow dropping onto itself
      if (sourceId === targetId) return;
      
      this.$emit('node-dropped', sourceId, targetId);
    },
    onChildDropped(sourceId, targetId) {
      this.$emit('node-dropped', sourceId, targetId);
    }
  }
};
</script>

<style scoped>
.tree-node {
  margin: 5px 0;
  list-style-type: none;
}

.node-content {
  display: flex;
  align-items: center;
  padding: 8px 12px;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.2s;
}

.node-content:hover {
  background-color: #f0f0f0;
}

.node-content.is-folder {
  font-weight: bold;
}

.node-icon {
  margin-right: 8px;
  display: inline-flex;
  align-items: center;
}

.folder-icon::before {
  content: '📁';
}

.file-icon::before {
  content: '📄';
}

.selected > .node-content {
  background-color: #e3f2fd;
}

.node-children {
  padding-left: 20px;
  margin-top: 5px;
}

.dragging {
  opacity: 0.5;
}

.drag-over {
  background-color: #e8f5e9;
  outline: 2px dashed #4caf50;
}
</style>
