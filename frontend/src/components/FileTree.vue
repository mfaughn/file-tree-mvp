<!-- FileTree.vue -->
<template>
  <div class="file-tree">
    <div class="controls">
      <button @click="addFolder" class="btn">Add Folder</button>
      <button @click="addFile" class="btn">Add File</button>
      <button @click="deleteSelected" class="btn delete" :disabled="!selectedNode">Delete Selected</button>
    </div>
    
    <div class="tree-container">
      <ul class="tree">
        <file-tree-node 
          v-for="node in rootNodes" 
          :key="node.id" 
          :node="node"
          @node-selected="selectNode"
          @node-dropped="handleDrop"
        ></file-tree-node>
      </ul>
    </div>
  </div>
</template>

<script>
import FileTreeNode from './FileTreeNode.vue';

export default {
  name: 'FileTree',
  components: {
    FileTreeNode
  },
  data() {
    return {
      nodes: [],
      selectedNode: null,
      nextId: 1
    };
  },
  computed: {
    rootNodes() {
      return this.nodes.filter(node => !node.parentId);
    }
  },
  created() {
    // Initialize with some sample data
    this.nodes = [
      { id: 1, name: 'Root Folder', type: 'folder', parentId: null },
      { id: 2, name: 'Documents', type: 'folder', parentId: 1 },
      { id: 3, name: 'Images', type: 'folder', parentId: 1 },
      { id: 4, name: 'readme.txt', type: 'file', parentId: 1 },
      { id: 5, name: 'document.doc', type: 'file', parentId: 2 },
      { id: 6, name: 'photo.jpg', type: 'file', parentId: 3 }
    ];
    this.nextId = 7;
  },
  methods: {
    selectNode(node) {
      this.selectedNode = node;
    },
    findNodeById(id) {
      return this.nodes.find(node => node.id === id);
    },
    getChildNodes(parentId) {
      return this.nodes.filter(node => node.parentId === parentId);
    },
    addFolder() {
      const folderName = prompt('Enter folder name:');
      if (!folderName) return;
      
      const parentId = this.selectedNode ? this.selectedNode.id : null;
      const newFolder = {
        id: this.nextId++,
        name: folderName,
        type: 'folder',
        parentId: parentId
      };
      
      this.nodes.push(newFolder);
    },
    addFile() {
      const fileName = prompt('Enter file name:');
      if (!fileName) return;
      
      const parentId = this.selectedNode && this.selectedNode.type === 'folder' 
        ? this.selectedNode.id 
        : this.selectedNode ? this.selectedNode.parentId : null;
      
      const newFile = {
        id: this.nextId++,
        name: fileName,
        type: 'file',
        parentId: parentId
      };
      
      this.nodes.push(newFile);
    },
    deleteSelected() {
      if (!this.selectedNode) return;
      
      // Get all descendant IDs recursively
      const idsToDelete = this.getAllDescendantIds(this.selectedNode.id);
      idsToDelete.push(this.selectedNode.id);
      
      // Filter out the nodes to delete
      this.nodes = this.nodes.filter(node => !idsToDelete.includes(node.id));
      this.selectedNode = null;
    },
    getAllDescendantIds(nodeId) {
      const descendantIds = [];
      const directChildren = this.getChildNodes(nodeId);
      
      directChildren.forEach(child => {
        descendantIds.push(child.id);
        if (child.type === 'folder') {
          descendantIds.push(...this.getAllDescendantIds(child.id));
        }
      });
      
      return descendantIds;
    },
    handleDrop(sourceId, targetId) {
      const sourceNode = this.findNodeById(sourceId);
      const targetNode = this.findNodeById(targetId);
      
      if (!sourceNode || !targetNode) return;
      
      // Prevent dropping a parent into its own descendant
      if (this.isDescendantOf(targetNode.id, sourceNode.id)) return;
      
      // If target is a folder, move the source node into the target folder
      if (targetNode.type === 'folder') {
        sourceNode.parentId = targetNode.id;
      } else {
        // If target is a file, move the source node to the same folder as the target
        sourceNode.parentId = targetNode.parentId;
      }
    },
    isDescendantOf(nodeId, possibleAncestorId) {
      let currentNode = this.findNodeById(nodeId);
      
      while (currentNode && currentNode.parentId) {
        if (currentNode.parentId === possibleAncestorId) {
          return true;
        }
        currentNode = this.findNodeById(currentNode.parentId);
      }
      
      return false;
    }
  },
  provide() {
    return {
      findNodeById: this.findNodeById,
      getChildNodes: this.getChildNodes
    };
  }
};
</script>

<style scoped>
.file-tree {
  width: 100%;
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  font-family: Arial, sans-serif;
}

.controls {
  margin-bottom: 15px;
  display: flex;
  gap: 10px;
}

.btn {
  padding: 8px 15px;
  background-color: #4CAF50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.btn:hover {
  background-color: #45a049;
}

.btn:disabled {
  background-color: #cccccc;
  cursor: not-allowed;
}

.delete {
  background-color: #f44336;
}

.delete:hover {
  background-color: #d32f2f;
}

.tree-container {
  border: 1px solid #ddd;
  border-radius: 4px;
  padding: 15px;
  background-color: #f9f9f9;
}

.tree {
  list-style-type: none;
  padding-left: 0;
}
</style>
