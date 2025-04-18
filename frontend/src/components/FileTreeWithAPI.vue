<!-- FileTreeWithAPI.vue -->
<template>
  <div class="file-tree">
    <div class="controls">
      <button @click="addFolder" class="btn">Add Folder</button>
      <button @click="addFile" class="btn">Add File</button>
      <button @click="deleteSelected" class="btn delete" :disabled="!selectedNode">Delete Selected</button>
    </div>
    
    <div class="tree-container">
      <div v-if="loading" class="loading">Loading file tree...</div>
      <div v-else-if="error" class="error">{{ error }}</div>
      <ul v-else class="tree">
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
import axios from 'axios';

const API_URL = process.env.VUE_APP_API_URL || 'http://localhost:5000/api';

export default {
  name: 'FileTree',
  components: {
    FileTreeNode
  },
  data() {
    return {
      nodes: [],
      selectedNode: null,
      loading: true,
      error: null
    };
  },
  computed: {
    rootNodes() {
      return this.nodes.filter(node => !node.parentId);
    }
  },
  created() {
    this.fetchNodes();
  },
  methods: {
    async fetchNodes() {
      try {
        this.loading = true;
        const response = await axios.get(`${API_URL}/nodes`);
        this.nodes = response.data;
        this.loading = false;
      } catch (err) {
        this.error = 'Failed to load file tree. Please try again later.';
        this.loading = false;
        console.error('Error fetching nodes:', err);
      }
    },
    selectNode(node) {
      this.selectedNode = node;
    },
    findNodeById(id) {
      return this.nodes.find(node => node.id === id);
    },
    getChildNodes(parentId) {
      return this.nodes.filter(node => node.parentId === parentId);
    },
    async addFolder() {
      const folderName = prompt('Enter folder name:');
      if (!folderName) return;
      
      const parentId = this.selectedNode ? this.selectedNode.id : null;
      
      try {
        const response = await axios.post(`${API_URL}/nodes`, {
          name: folderName,
          type: 'folder',
          parentId: parentId
        });
        
        this.nodes.push(response.data);
      } catch (err) {
        alert('Failed to create folder');
        console.error('Error creating folder:', err);
      }
    },
    async addFile() {
      const fileName = prompt('Enter file name:');
      if (!fileName) return;
      
      const parentId = this.selectedNode && this.selectedNode.type === 'folder' 
        ? this.selectedNode.id 
        : this.selectedNode ? this.selectedNode.parentId : null;
      
      try {
        const response = await axios.post(`${API_URL}/nodes`, {
          name: fileName,
          type: 'file',
          parentId: parentId
        });
        
        this.nodes.push(response.data);
      } catch (err) {
        alert('Failed to create file');
        console.error('Error creating file:', err);
      }
    },
    async deleteSelected() {
      if (!this.selectedNode) return;
      
      try {
        await axios.delete(`${API_URL}/nodes/${this.selectedNode.id}`);
        
        // Refetch all nodes after deletion to ensure we have the current state
        await this.fetchNodes();
        this.selectedNode = null;
      } catch (err) {
        alert('Failed to delete node');
        console.error('Error deleting node:',
