<template>
  <div class="all-songs-view" 
       @dragover.prevent="isDragOver = true" 
       @dragleave="isDragOver = false" 
       @drop.prevent="handleFileDrop"
       :class="{ 'drag-over': isDragOver }"
  >
    <div class="view-header">
      <h1 class="view-title">All Songs</h1>
      <p class="view-subtitle">
        {{ songs.length }} songs in your library
        <span v-if="selectedItems.length > 0" class="selection-count">
          • {{ selectedItems.length }} selected
        </span>
      </p>
      
      <div class="header-actions">
        <button 
          v-if="selectedItems.length > 0" 
          class="clear-selection-button"
          @click="clearSelection"
        >
          Clear Selection
        </button>
        <button 
          v-if="selectedItems.length === 0" 
          class="select-all-button"
          @click="selectAll"
        >
          Select All
        </button>
        <button class="shuffle-button">
          <svg viewBox="0 0 24 24" fill="currentColor">
            <path d="M10.59 9.17L5.41 4 4 5.41l5.17 5.17 1.42-1.41zM14.5 4l2.04 2.04L4 18.59 5.41 20 17.96 7.46 20 9.5V4h-5.5zm.33 9.41l-1.41 1.41 3.13 3.13L14.5 20H20v-5.5l-2.04 2.04-3.13-3.13z"/>
          </svg>
          <span>Shuffle All</span>
        </button>
        <button class="play-all-button">
          <svg viewBox="0 0 24 24" fill="currentColor">
            <path d="M8 5v14l11-7z"/>
          </svg>
          <span>Play All</span>
        </button>
      </div>
    </div>

    <div class="songs-container">
      <!-- Search and Filter Bar -->
      <div class="controls-bar">
        <div class="search-box">
          <svg viewBox="0 0 24 24" fill="currentColor" class="search-icon">
            <path d="M15.5 14h-.79l-.28-.27C15.41 12.59 16 11.11 16 9.5 16 5.91 13.09 3 9.5 3S3 5.91 3 9.5 5.91 16 9.5 16c1.61 0 3.09-.59 4.23-1.57l.27.28v.79l5 4.99L20.49 19l-4.99-5zm-6 0C7.01 14 5 11.99 5 9.5S7.01 5 9.5 5 14 7.01 14 9.5 11.99 14 9.5 14z"/>
          </svg>
          <input 
            v-model="searchQuery"
            type="text" 
            placeholder="Search songs, artists, albums..."
            class="search-input"
          >
        </div>
        
        <div class="sort-options">
          <button class="sort-button" @click="showSortMenu = !showSortMenu">
            <svg viewBox="0 0 24 24" fill="currentColor">
              <path d="M3 18h6v-2H3v2zM3 6v2h18V6H3zm0 7h12v-2H3v2z"/>
            </svg>
            <span>Sort</span>
          </button>
          
          <transition name="dropdown">
            <div v-if="showSortMenu" class="sort-menu">
              <button 
                v-for="option in sortOptions" 
                :key="option.value"
                @click="setSortBy(option.value)"
                class="sort-option"
                :class="{ active: sortBy === option.value }"
              >
                {{ option.label }}
              </button>
            </div>
          </transition>
        </div>
      </div>

      <!-- Songs List -->
      <div class="songs-list" @click="handleBackgroundClick">
        <div class="list-header">
          <div class="col-check">
            <input 
              type="checkbox" 
              class="select-all-checkbox"
              :checked="selectedItems.length === filteredSongs.length && filteredSongs.length > 0"
              :indeterminate="selectedItems.length > 0 && selectedItems.length < filteredSongs.length"
              @change="selectedItems.length === filteredSongs.length ? clearSelection() : selectAll()"
            >
          </div>
          <div class="col-title">#</div>
          <div class="col-title">Title</div>
          <div class="col-title">Artist</div>
          <div class="col-title">Album</div>
          <div class="col-title">Duration</div>
          <div class="col-title"></div>
        </div>
        
        <div class="list-body">
          <div 
            v-for="(song, index) in filteredSongs" 
            :key="song.id"
            class="song-row"
            :class="{ 
              selected: isSelected(song),
              'drag-source': draggedSong?.id === song.id
            }"
            :draggable="true"
            @dragstart="onDragStart(song, index, $event)"
            @dragend="onDragEnd"
            @click="handleSongClick(song, index, $event)"
            @dblclick="playSong(song)"
          >
            <div class="col-check">
              <input 
                type="checkbox" 
                :checked="isSelected(song)"
                @click.stop
                @change="toggleSelection(song, index, $event)"
              >
            </div>
            <div class="col-number">{{ index + 1 }}</div>
            <div class="col-title">
              <div class="song-info">
                <div class="song-artwork">
                  <svg viewBox="0 0 40 40" class="artwork-placeholder">
                    <rect width="40" height="40" fill="url(#song-gradient)" rx="4"/>
                    <path d="M20 12v8.22c-.47-.34-1.02-.55-1.6-.55-1.77 0-3.2 1.43-3.2 3.2s1.43 3.2 3.2 3.2 3.2-1.43 3.2-3.2V16h3.2v-4H20z" fill="rgba(255,255,255,0.3)"/>
                  </svg>
                </div>
                <span class="song-name">{{ song.title }}</span>
              </div>
            </div>
            <div class="col-artist">{{ song.artist }}</div>
            <div class="col-album">{{ song.album }}</div>
            <div class="col-duration">{{ song.duration }}</div>
            <div class="col-actions">
              <button class="action-button" @click.stop="showSongMenu(song)">
                <svg viewBox="0 0 24 24" fill="currentColor">
                  <path d="M12 8c1.1 0 2-.9 2-2s-.9-2-2-2-2 .9-2 2 .9 2 2 2zm0 2c-1.1 0-2 .9-2 2s.9 2 2 2 2-.9 2-2-.9-2-2-2zm0 6c-1.1 0-2 .9-2 2s.9 2 2 2 2-.9 2-2-.9-2-2-2z"/>
                </svg>
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- SVG Definitions -->
    <svg width="0" height="0">
      <defs>
        <linearGradient id="song-gradient" x1="0%" y1="0%" x2="100%" y2="100%">
          <stop offset="0%" style="stop-color:#4facfe;stop-opacity:1" />
          <stop offset="100%" style="stop-color:#00f2fe;stop-opacity:1" />
        </linearGradient>
      </defs>
    </svg>
  </div>
</template>

<script>
import { ref, computed, onMounted, onUnmounted } from 'vue'
import { useDragDropStore } from '@/store/dragddrop'
import { useSelection } from '@/composables/useSelection'

export default {
  name: 'AllSongsView',
  props: {
    songs: Array,
    onFileDrop: Function
  },
  emits: ['song-drag-start'],
  setup(props, { emit }) {
    const searchQuery = ref('')
    const sortBy = ref('title')
    const showSortMenu = ref(false)
    const isDragOver = ref(false)
    const draggedSong = ref(null)
    const dragStore = useDragDropStore()
    
    const sortOptions = [
      { value: 'title', label: 'Title' },
      { value: 'artist', label: 'Artist' },
      { value: 'album', label: 'Album' },
      { value: 'dateAdded', label: 'Date Added' }
    ]
    
    const filteredSongs = computed(() => {
      let filtered = props.songs
      
      if (searchQuery.value) {
        const query = searchQuery.value.toLowerCase()
        filtered = filtered.filter(song => 
          song.title.toLowerCase().includes(query) ||
          song.artist.toLowerCase().includes(query) ||
          song.album.toLowerCase().includes(query)
        )
      }
      
      return [...filtered].sort((a, b) => {
        switch (sortBy.value) {
          case 'title':
            return a.title.localeCompare(b.title)
          case 'artist':
            return a.artist.localeCompare(b.artist)
          case 'album':
            return a.album.localeCompare(b.album)
          default:
            return 0
        }
      })
    })

    // Use selection composable
    const {
      selectedIds,
      selectedItems,
      isSelected,
      clearSelection,
      toggleSelection,
      selectAll: selectAllItems
    } = useSelection(filteredSongs, song => song.id)

    const selectAll = () => {
      selectAllItems()
    }

    const handleSongClick = (song, index, event) => {
      if (event.shiftKey || event.metaKey || event.ctrlKey) {
        toggleSelection(song, index, event)
      } else if (!isSelected(song)) {
        clearSelection()
        toggleSelection(song, index, event)
      }
    }

    const handleBackgroundClick = (event) => {
      if (event.target.classList.contains('songs-list')) {
        clearSelection()
      }
    }
    
    const setSortBy = (value) => {
      sortBy.value = value
      showSortMenu.value = false
    }
    
    const playSong = (song) => {
      console.log('Playing:', song)
    }
    
    const showSongMenu = (song) => {
      console.log('Show menu for:', song)
    }
    
    // Enhanced drag start for multiple items
    const onDragStart = (song, index, event) => {
      draggedSong.value = song
      
      // Check if dragging a selected item
      const itemsToDrag = isSelected(song) ? selectedItems.value : [song]
      
      console.log(`🎵 Dragging ${itemsToDrag.length} songs`)
      
      // Set the allowed effect
      event.dataTransfer.effectAllowed = 'copy'
      
      // Set data for single or multiple songs
      event.dataTransfer.setData('text/plain', 'songs')
      event.dataTransfer.setData('application/json', JSON.stringify(itemsToDrag))
      
      // Store in the drag store
      dragStore.startDrag(itemsToDrag, 'songs')
      
      // Custom drag image for multiple items
      if (itemsToDrag.length > 1) {
        const dragImage = document.createElement('div')
        dragImage.className = 'custom-drag-image'
        dragImage.innerHTML = `
          <div class="drag-count">${itemsToDrag.length}</div>
          <div class="drag-label">songs</div>
        `
        dragImage.style.position = 'absolute'
        dragImage.style.top = '-1000px'
        document.body.appendChild(dragImage)
        event.dataTransfer.setDragImage(dragImage, 50, 25)
        setTimeout(() => document.body.removeChild(dragImage), 0)
      }
      
      emit('song-drag-start', itemsToDrag)
    }
    
    const onDragEnd = () => {
      console.log('🎵 Drag end')
      draggedSong.value = null
      dragStore.endDrag()
    }
    
    const handleFileDrop = (e) => {
      isDragOver.value = false
      const files = e.dataTransfer.files
      if (files && files.length > 0) {
        const filePaths = Array.from(files).map(f => f.path || f.name)
        if (props.onFileDrop) props.onFileDrop(filePaths)
      }
    }

    // Keyboard shortcuts
    const handleKeyboard = (e) => {
      // Select all with Cmd/Ctrl + A
      if ((e.metaKey || e.ctrlKey) && e.key === 'a') {
        e.preventDefault()
        selectAll()
      }
      // Clear selection with Escape
      if (e.key === 'Escape') {
        clearSelection()
      }
    }

    onMounted(() => {
      document.addEventListener('keydown', handleKeyboard)
    })

    onUnmounted(() => {
      document.removeEventListener('keydown', handleKeyboard)
    })
    
    return {
      searchQuery,
      sortBy,
      showSortMenu,
      sortOptions,
      filteredSongs,
      setSortBy,
      playSong,
      showSongMenu,
      onDragStart,
      onDragEnd,
      isDragOver,
      handleFileDrop,
      draggedSong,
      // Selection
      selectedItems,
      isSelected,
      clearSelection,
      toggleSelection,
      selectAll,
      handleSongClick,
      handleBackgroundClick
    }
  }
}
</script>

<style scoped>
/* Add to existing styles */

.selection-count {
  color: #4ECDC4;
  font-weight: 600;
}

.clear-selection-button,
.select-all-button {
  padding: 8px 16px;
  background: rgba(78, 205, 196, 0.2);
  border: 1px solid #4ECDC4;
  border-radius: 8px;
  color: #4ECDC4;
  font-size: 13px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s ease;
}

.clear-selection-button:hover,
.select-all-button:hover {
  background: rgba(78, 205, 196, 0.3);
  transform: translateY(-1px);
}

.col-check {
  width: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.select-all-checkbox {
  width: 16px;
  height: 16px;
  cursor: pointer;
}

.list-header {
  display: grid;
  grid-template-columns: 40px 50px 1fr 200px 200px 80px 50px;
  gap: 16px;
  padding: 16px 0;
  border-bottom: 1px solid rgba(255, 255, 255, 0.05);
  font-size: 12px;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  color: rgba(255, 255, 255, 0.5);
}

.song-row {
  display: grid;
  grid-template-columns: 40px 50px 1fr 200px 200px 80px 50px;
  gap: 16px;
  padding: 12px 0;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.2s ease;
  align-items: center;
  user-select: none;
}

.song-row:hover {
  background: rgba(255, 255, 255, 0.05);
}

.song-row.selected {
  background: rgba(78, 205, 196, 0.15);
  border: 1px solid rgba(78, 205, 196, 0.3);
  margin: -1px 0;
}

.song-row.drag-source {
  opacity: 0.5;
}

/* Custom drag image styles - add to global styles */
:global(.custom-drag-image) {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 8px 16px;
  background: rgba(30, 30, 30, 0.95);
  border: 1px solid rgba(78, 205, 196, 0.5);
  border-radius: 8px;
  color: white;
  font-size: 14px;
  font-weight: 500;
}

:global(.custom-drag-image .drag-count) {
  font-size: 18px;
  font-weight: 700;
  color: #4ECDC4;
}

/* All other existing styles remain the same */
.all-songs-view {
  height: 100%;
  display: flex;
  flex-direction: column;
  color: #fff;
}

.view-header {
  padding: 32px;
  border-bottom: 1px solid rgba(255, 255, 255, 0.05);
}

.view-title {
  font-size: 32px;
  font-weight: 700;
  margin-bottom: 8px;
  letter-spacing: -0.5px;
}

.view-subtitle {
  font-size: 16px;
  color: rgba(255, 255, 255, 0.6);
  margin-bottom: 24px;
}

.header-actions {
  display: flex;
  gap: 12px;
}

.shuffle-button,
.play-all-button {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 10px 20px;
  background: rgba(255, 255, 255, 0.1);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 12px;
  color: #fff;
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s ease;
}

.play-all-button {
  background: #fff;
  color: #000;
}

.shuffle-button:hover {
  background: rgba(255, 255, 255, 0.15);
  transform: translateY(-1px);
}

.play-all-button:hover {
  background: rgba(255, 255, 255, 0.9);
  transform: translateY(-1px);
}

.shuffle-button svg,
.play-all-button svg {
  width: 20px;
  height: 20px;
}

.songs-container {
  flex: 1;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.controls-bar {
  padding: 24px 32px;
  display: flex;
  gap: 16px;
  align-items: center;
}

.search-box {
  flex: 1;
  position: relative;
}

.search-icon {
  position: absolute;
  left: 16px;
  top: 50%;
  transform: translateY(-50%);
  width: 20px;
  height: 20px;
  color: rgba(255, 255, 255, 0.5);
}

.search-input {
  width: 100%;
  padding: 12px 16px 12px 48px;
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 12px;
  color: #fff;
  font-size: 14px;
  outline: none;
  transition: all 0.2s ease;
}

.search-input::placeholder {
  color: rgba(255, 255, 255, 0.4);
}

.search-input:focus {
  background: rgba(255, 255, 255, 0.08);
  border-color: rgba(255, 255, 255, 0.2);
}

.sort-options {
  position: relative;
}

.sort-button {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 8px 16px;
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 8px;
  color: rgba(255, 255, 255, 0.8);
  font-size: 14px;
  cursor: pointer;
  transition: all 0.2s ease;
}

.sort-button:hover {
  background: rgba(255, 255, 255, 0.08);
}

.sort-button svg {
  width: 18px;
  height: 18px;
}

.sort-menu {
  position: absolute;
  top: 100%;
  right: 0;
  margin-top: 8px;
  background: rgba(30, 30, 30, 0.95);
  backdrop-filter: blur(20px);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 12px;
  padding: 8px;
  min-width: 150px;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.4);
}

.sort-option {
  display: block;
  width: 100%;
  padding: 8px 16px;
  background: none;
  border: none;
  color: rgba(255, 255, 255, 0.8);
  font-size: 14px;
  text-align: left;
  cursor: pointer;
  border-radius: 8px;
  transition: all 0.2s ease;
}

.sort-option:hover {
  background: rgba(255, 255, 255, 0.1);
  color: #fff;
}

.sort-option.active {
  background: rgba(255, 255, 255, 0.15);
  color: #fff;
}

.songs-list {
  flex: 1;
  overflow-y: auto;
  padding: 0 32px 32px;
}

.list-body {
  padding-top: 8px;
}

.song-row[draggable="true"] {
  cursor: grab;
}

.song-row[draggable="true"]:active {
  cursor: grabbing;
}

.col-number {
  text-align: center;
  color: rgba(255, 255, 255, 0.5);
  font-size: 14px;
}

.song-info {
  display: flex;
  align-items: center;
  gap: 12px;
}

.song-artwork {
  width: 40px;
  height: 40px;
  border-radius: 4px;
  overflow: hidden;
  flex-shrink: 0;
}

.artwork-placeholder {
  width: 100%;
  height: 100%;
}

.song-name {
  font-size: 14px;
  font-weight: 500;
}

.col-artist,
.col-album {
  font-size: 14px;
  color: rgba(255, 255, 255, 0.7);
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.col-duration {
  text-align: center;
  font-size: 14px;
  color: rgba(255, 255, 255, 0.5);
}

.col-actions {
  display: flex;
  justify-content: center;
}

.action-button {
  width: 32px;
  height: 32px;
  background: none;
  border: none;
  color: rgba(255, 255, 255, 0.5);
  cursor: pointer;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s ease;
}

.action-button:hover {
  background: rgba(255, 255, 255, 0.1);
  color: #fff;
}

.action-button svg {
  width: 20px;
  height: 20px;
}

.dropdown-enter-active,
.dropdown-leave-active {
  transition: all 0.2s ease;
}

.dropdown-enter-from,
.dropdown-leave-to {
  opacity: 0;
  transform: translateY(-10px);
}

.all-songs-view.drag-over {
  outline: 2px solid #4ECDC4;
  background: rgba(78, 205, 196, 0.08);
}
</style>