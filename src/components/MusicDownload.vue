<template>
  <div class="music-download-container">
    <!-- 背景装饰 -->
    <div class="background-decoration">
      <div class="floating-circle circle-1"></div>
      <div class="floating-circle circle-2"></div>
      <div class="floating-circle circle-3"></div>
      <div class="floating-circle circle-4"></div>
    </div>
    
    <!-- 主内容区域 -->
    <div class="main-content">
      <!-- 头部 - 只在卡片页面显示 -->
      <header v-if="musicDownloadCurrentPage === 'cards'" class="app-header">
        <h1 class="app-title">
          <span class="music-icon">🎵</span>
          <span class="title-text">听歌速下载</span>
        </h1>
        <p class="app-subtitle">🎼 👇音乐分类👇 🎼</p>
      </header>
        
        <!-- 内容区域 -->
        <main class="content-area">
          <!-- 卡片页面 -->
          <div v-if="musicDownloadCurrentPage === 'cards'" class="cards-page">

            <!-- 图片卡片网格 -->
            <div class="cards-grid">
              <div 
                v-for="card in musicCards" 
                :key="card.id"
                class="category-card"
                @click="goToSearchPage(card.type, card.title)"
              >
                <img :src="card.cover" :alt="card.title" class="card-cover">
                <div class="card-overlay">
                  <span class="card-title">{{ card.title }}</span>
                </div>
              </div>
            </div>
          </div>
          
          <!-- 搜索页面 -->
          <div v-else class="search-page">
            <!-- 返回按钮 -->
            <button 
              class="back-btn" 
              @click="goBackToCards"
            >
              ← 返回分类
            </button>
            
            <!-- 播放列表面板 -->
            <div 
              v-if="showPlaylistPanel" 
              :style="{ position: 'absolute', left: playlistPanelFullscreen ? '0px' : '20px', top: playlistPanelFullscreen ? '0px' : '4.6rem', bottom: playlistPanelFullscreen ? '0px' : '4.8rem', right: playlistPanelFullscreen ? '0px' : 'auto', zIndex: '1000', background: 'white', borderRadius: playlistPanelFullscreen ? '0px' : '12px', boxShadow: playlistPanelFullscreen ? 'none' : '0 10px 40px rgba(0,0,0,0.2)', overflow: 'hidden', width: playlistPanelFullscreen ? '100%' : '220px', display: 'flex', flexDirection: 'column', height: playlistPanelFullscreen ? '100vh' : 'auto' }"
            >
              <div style="padding: 15px; border-bottom: 1px solid #eee; flex-shrink: 0;">
                <!-- 标题行 -->
                <div :style="playlistPanelFullscreen ? 'display: flex; align-items: center;' : 'display: flex; justify-content: space-between; align-items: center;'">
                  <!-- 左侧：占位（仅扩展模式下显示，与底部封面位置对应） -->
                  <div :style="playlistPanelFullscreen ? 'width: 140px;' : ''"></div>
                  <!-- 标题 -->
                  <div :style="playlistPanelFullscreen ? 'flex: 1; display: flex; justify-content: center;' : ''">
                    <h4 :style="'margin: 0; color: #333; font-size: ' + (playlistPanelFullscreen ? '20px' : '14px') + '; font-weight: 600;'">🎵 播放列表</h4>
                  </div>
                  <!-- 右侧：按钮组 -->
                  <div :style="'display: flex; gap: ' + (playlistPanelFullscreen ? '10px' : '6px') + '; align-items: center; flex-shrink: 0;'">
                    <button 
                      :style="'background: none; border: 1px solid #ff6b6b; border-radius: ' + (playlistPanelFullscreen ? '5px' : '3px') + '; padding: ' + (playlistPanelFullscreen ? '6px 12px' : '3px 8px') + '; font-size: ' + (playlistPanelFullscreen ? '13px' : '11px') + '; cursor: pointer; color: #ff6b6b; white-space: nowrap;'" 
                      @click="clearMusicPlaylist"
                    >清空</button>
                    <button 
                      :style="'background: none; border: 1px solid #4ecdc4; border-radius: ' + (playlistPanelFullscreen ? '5px' : '3px') + '; padding: ' + (playlistPanelFullscreen ? '6px 12px' : '3px 8px') + '; font-size: ' + (playlistPanelFullscreen ? '13px' : '11px') + '; cursor: pointer; color: #4ecdc4; white-space: nowrap;'" 
                      @click="playlistPanelFullscreen = !playlistPanelFullscreen"
                    >{{ playlistPanelFullscreen ? '收缩' : '扩展' }}</button>
                    <button 
                      :style="'background: none; border: none; font-size: ' + (playlistPanelFullscreen ? '1.5rem' : '1.2rem') + '; cursor: pointer; color: #999; padding: 0;'" 
                      @click="closePlaylistPanel"
                    >×</button>
                  </div>
                </div>
                
              </div>
              <div style="flex: 1; padding: 10px; overflow-y: auto;">
                <div v-if="musicPlaylist.length === 0" style="text-align: center; color: #999; padding: 40px 20px;">
                  播放列表为空<br>点击"批量加播"添加歌曲
                </div>
                <div 
                  v-for="(item, index) in musicPlaylist" 
                  :key="index"
                  :data-playlist-item="index"
                  :style="{ display: 'flex', alignItems: 'center', padding: '8px 10px', borderBottom: '1px solid #f0f0f0', cursor: 'pointer', transition: 'background 0.2s', background: playlistCurrentPlayingIndex === index ? 'rgba(78, 205, 196, 0.35)' : 'transparent', position: 'relative', overflow: 'hidden' }"
                  @click="playFromPlaylist(index)"
                  @mouseenter="$event.currentTarget.style.background = playlistCurrentPlayingIndex === index ? 'rgba(78, 205, 196, 0.45)' : '#f8f9fa'"
                  @mouseleave="$event.currentTarget.style.background = playlistCurrentPlayingIndex === index ? 'rgba(78, 205, 196, 0.35)' : 'transparent'"
                >
                  <!-- 播放提示走马灯 -->
                  <div v-if="playlistCurrentPlayingIndex === index" class="playing-marquee">
                    <span class="marquee-text">🎶 正在播放，请洗耳恭听哦～～～</span>
                  </div>
                  <span style="margin-right: 10px; color: #999; font-size: 11px;">{{ index + 1 }}</span>
                  <div :style="'flex: ' + (playlistPanelFullscreen ? '2' : '1') + '; overflow: hidden; display: flex; align-items: center; gap: ' + (playlistPanelFullscreen ? '40px' : '20px') + ';'">
                    <span :style="'font-size: ' + (playlistPanelFullscreen ? '16px' : '13px') + '; color: #2d3436; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; font-weight: ' + (playlistPanelFullscreen ? '600' : '500') + '; font-family: \'Microsoft YaHei\', \'PingFang SC\', sans-serif;'">🎵 {{ item.name }}</span>
                    <span v-if="playlistPanelFullscreen" style="font-size: 14px; color: #6c5ce7; font-family: 'Microsoft YaHei', 'PingFang SC', sans-serif;">{{ item.artist }}</span>
                  </div>
                  <div :style="playlistPanelFullscreen ? 'display: flex; gap: 30px; align-items: center; flex-shrink: 0; margin-left: -30px;' : 'display: flex; gap: 4px; align-items: center;'">
                    <button 
                      :style="playlistCurrentPlayingIndex === index && !playlistShowStopButton ? ('background: #ff6b6b; border: 1px solid #ff6b6b; border-radius: ' + (playlistPanelFullscreen ? '5px' : '3px') + '; padding: ' + (playlistPanelFullscreen ? '6px 12px' : '2px 6px') + '; font-size: ' + (playlistPanelFullscreen ? '13px' : '10px') + '; cursor: pointer; color: white; white-space: nowrap;') : ('background: none; border: 1px solid #4ecdc4; border-radius: ' + (playlistPanelFullscreen ? '5px' : '3px') + '; padding: ' + (playlistPanelFullscreen ? '6px 12px' : '2px 6px') + '; font-size: ' + (playlistPanelFullscreen ? '13px' : '10px') + '; cursor: pointer; color: #4ecdc4; white-space: nowrap;')" 
                      @click.stop="console.log('Play button clicked, index:', index); playFromPlaylist(index)"
                    >{{ playlistCurrentPlayingIndex === index && !playlistShowStopButton ? '暂停' : '播放' }}</button>
                    <button 
                      v-if="playlistCurrentPlayingIndex === index && playlistShowStopButton"
                      :style="'background: none; border: 1px solid #95a5a6; border-radius: ' + (playlistPanelFullscreen ? '5px' : '3px') + '; padding: ' + (playlistPanelFullscreen ? '6px 12px' : '2px 6px') + '; font-size: ' + (playlistPanelFullscreen ? '13px' : '10px') + '; cursor: pointer; color: #95a5a6; white-space: nowrap;'" 
                      @click.stop="stopPlaylistPlay"
                    >停止</button>
                    <button 
                      :style="'background: none; border: 1px solid #ff6b6b; border-radius: ' + (playlistPanelFullscreen ? '5px' : '3px') + '; padding: ' + (playlistPanelFullscreen ? '6px 12px' : '2px 6px') + '; font-size: ' + (playlistPanelFullscreen ? '13px' : '10px') + '; cursor: pointer; color: #ff6b6b; white-space: nowrap;'" 
                      @click.stop="removeFromPlaylist(index)"
                    >移除</button>
                  </div>
                </div>
              </div>
              
              <!-- 底部播放条 -->
              <div style="border-top: 1px solid #eee; padding: 10px 12px; background: #fafafa; flex-shrink: 0;">
                <!-- 非扩展模式布局 -->
                <div v-if="!playlistPanelFullscreen" style="display: flex; flex-direction: column; gap: 10px;">
                  <!-- 第一行：控制按钮 -->
                  <div style="display: flex; align-items: center; justify-content: space-between;">
                    <!-- 左侧：封面图标 -->
                    <div style="display: flex; align-items: center;">
                      <div style="width: 36px; height: 36px; border-radius: 6px; overflow: hidden;">
                        <img 
                          v-if="musicPlaylist[playlistCurrentPlayingIndex]?.cover" 
                          :src="musicPlaylist[playlistCurrentPlayingIndex].cover" 
                          alt="专辑封面" 
                          style="width: 100%; height: 100%; object-fit: cover;"
                        />
                        <div v-else style="width: 100%; height: 100%; background: linear-gradient(135deg, #4ecdc4, #44a08d); display: flex; align-items: center; justify-content: center;">
                          <span style="font-size: 16px;">🎵</span>
                        </div>
                      </div>
                    </div>
                    
                    <!-- 中间：播放控制按钮组 -->
                    <div style="display: flex; align-items: center; gap: 4px;">
                      <!-- 上一首 -->
                      <button 
                        style="width: 26px; height: 26px; border-radius: 50%; border: none; background: #fff; cursor: pointer; display: flex; align-items: center; justify-content: center; font-size: 13px; color: #666; transition: all 0.2s;" 
                        :style="{ opacity: musicPlaylist.length > 1 && playlistCurrentPlayingIndex >= 0 ? 1 : 0.4, cursor: musicPlaylist.length > 1 && playlistCurrentPlayingIndex >= 0 ? 'pointer' : 'not-allowed' }"
                        title="上一首"
                        @click="playlistPrevSong"
                      >◀</button>
                      
                      <!-- 播放/暂停 -->
                      <button 
                        style="width: 32px; height: 32px; border-radius: 50%; border: none; cursor: pointer; display: flex; align-items: center; justify-content: center; font-size: 15px; color: white; transition: all 0.2s;" 
                        :style="playlistCurrentPlayingIndex >= 0 && !playlistShowStopButton ? 'background: linear-gradient(90deg, #ff6b6b, #ee5a5a);' : 'background: linear-gradient(90deg, #4ecdc4, #44a08d);'"
                        :title="playlistCurrentPlayingIndex >= 0 && !playlistShowStopButton ? '暂停' : '播放'"
                        @click="playlistTogglePlay"
                      >{{ playlistCurrentPlayingIndex >= 0 && !playlistShowStopButton ? '⏸' : '▶' }}</button>
                      
                      <!-- 下一首 -->
                      <button 
                        style="width: 26px; height: 26px; border-radius: 50%; border: none; background: #fff; cursor: pointer; display: flex; align-items: center; justify-content: center; font-size: 13px; color: #666; transition: all 0.2s;" 
                        :style="{ opacity: musicPlaylist.length > 1 && playlistCurrentPlayingIndex >= 0 ? 1 : 0.4, cursor: musicPlaylist.length > 1 && playlistCurrentPlayingIndex >= 0 ? 'pointer' : 'not-allowed' }"
                        title="下一首"
                        @click="playlistNextSong"
                      >▶</button>
                    </div>
                    
                    <!-- 右侧：模式和下载 -->
                    <div :style="'display: flex; align-items: center; gap: ' + (playlistPanelFullscreen ? '8px' : '2px') + ';'">
                      <!-- 播放模式 -->
                      <button 
                        style="width: 26px; height: 26px; border-radius: 50%; border: none; background: #fff; cursor: pointer; display: flex; align-items: center; justify-content: center; font-size: 13px; color: #666; transition: all 0.2s;" 
                        :title="playlistMode === 'single' ? '单曲循环' : playlistMode === 'loop' ? '列表循环' : '随机播放'"
                        @click="playlistToggleMode"
                      >{{ playlistMode === 'single' ? '🔂' : playlistMode === 'loop' ? '🔁' : '🔀' }}</button>
                      
                      <!-- 定位按钮 -->
                      <button 
                        style="width: 26px; height: 26px; border-radius: 50%; border: none; background: #fff; cursor: pointer; display: flex; align-items: center; justify-content: center; font-size: 13px; color: #666; transition: all 0.2s;" 
                        :style="{ opacity: playlistCurrentPlayingIndex >= 0 ? 1 : 0.4, cursor: playlistCurrentPlayingIndex >= 0 ? 'pointer' : 'not-allowed' }"
                        :title="'定位到当前播放歌曲'"
                        @click="scrollToCurrentPlaying"
                      >📍</button>
                      
                      <!-- 下载按钮 -->
                      <button 
                        style="width: 26px; height: 26px; border-radius: 50%; border: none; background: #fff; cursor: pointer; display: flex; align-items: center; justify-content: center; font-size: 13px; color: #666; transition: all 0.2s;" 
                        :style="{ opacity: playlistCurrentPlayingIndex >= 0 ? 1 : 0.4, cursor: playlistCurrentPlayingIndex >= 0 ? 'pointer' : 'not-allowed' }"
                        title="下载当前歌曲"
                        @click="playlistDownloadCurrent"
                      >⬇️</button>
                    </div>
                  </div>
                  
                  <!-- 第二行：进度条 -->
                  <div v-if="playlistCurrentPlayingIndex >= 0 && musicDuration > 0" style="display: flex; align-items: center; gap: 8px;">
                    <span style="font-size: 10px; color: #999; flex-shrink: 0;">{{ musicFormatTimeFunc(musicCurrentTime) }}</span>
                    <div 
                      style="flex: 1; height: 4px; background: #e0e0e0; border-radius: 2px; cursor: pointer; user-select: none;"
                      @click="musicSeekToFunc($event)"
                    >
                      <div 
                        style="height: 100%; background: linear-gradient(90deg, #4ecdc4, #44a08d); border-radius: 2px; transition: width 0.1s ease;"
                        :style="{ width: `${(musicCurrentTime / musicDuration) * 100}%` }"
                      ></div>
                    </div>
                    <span style="font-size: 10px; color: #999; flex-shrink: 0;">{{ musicFormatTimeFunc(musicDuration) }}</span>
                  </div>
                </div>
                
                <!-- 扩展模式布局 -->
                <div v-else style="display: flex; flex-direction: column; gap: 10px;">
                  <!-- 第一行：控制按钮 -->
                  <div style="display: grid; grid-template-columns: 1fr auto 1fr; align-items: center; gap: 20px;">
                    <!-- 左侧：封面 + 歌名歌手 -->
                    <div style="display: flex; align-items: center; gap: 10px; justify-self: flex-start;">
                      <div style="width: 40px; height: 40px; border-radius: 6px; overflow: hidden; flex-shrink: 0;">
                        <img 
                          v-if="musicPlaylist[playlistCurrentPlayingIndex]?.cover" 
                          :src="musicPlaylist[playlistCurrentPlayingIndex].cover" 
                          alt="专辑封面" 
                          style="width: 100%; height: 100%; object-fit: cover;"
                        />
                        <div v-else style="width: 100%; height: 100%; background: linear-gradient(135deg, #4ecdc4, #44a08d); display: flex; align-items: center; justify-content: center;">
                          <span style="font-size: 18px;">🎵</span>
                        </div>
                      </div>
                      <div style="min-width: 0; overflow: hidden; flex-shrink: 1;">
                        <p style="margin: 0; font-size: 13px; color: #333; overflow: hidden; text-overflow: ellipsis; white-space: nowrap;">
                          {{ musicPlaylist[playlistCurrentPlayingIndex]?.name || '暂无歌曲' }}
                        </p>
                        <p style="margin: 2px 0 0 0; font-size: 11px; color: #999; overflow: hidden; text-overflow: ellipsis; white-space: nowrap;">
                          {{ musicPlaylist[playlistCurrentPlayingIndex]?.artist || '-' }}
                        </p>
                      </div>
                    </div>
                    
                    <!-- 中间：播放控制按钮组 -->
                    <div style="display: flex; align-items: center; gap: 25px; justify-self: center;">
                      <!-- 切换歌词显示 -->
                      <button 
                        style="width: 36px; height: 36px; border-radius: 50%; border: none; cursor: pointer; display: flex; align-items: center; justify-content: center; font-size: 18px; transition: all 0.2s; box-shadow: 0 2px 8px rgba(0,0,0,0.1);" 
                        :style="showLyrics ? 'background: linear-gradient(90deg, #ff6b6b, #ee5a5a); color: white;' : 'background: #fff; color: #666;'"
                        :title="showLyrics ? '关闭歌词' : '显示歌词'"
                        @click="showLyrics = !showLyrics"
                      >📝</button>
                      <span style="font-size: 14px; color: #666;">上一首</span>
                      <!-- 上一首 -->
                      <button 
                        style="width: 36px; height: 36px; border-radius: 50%; border: none; background: #fff; cursor: pointer; display: flex; align-items: center; justify-content: center; font-size: 18px; color: #666; transition: all 0.2s; box-shadow: 0 2px 8px rgba(0,0,0,0.1);" 
                        :style="{ opacity: musicPlaylist.length > 1 && playlistCurrentPlayingIndex >= 0 ? 1 : 0.4, cursor: musicPlaylist.length > 1 && playlistCurrentPlayingIndex >= 0 ? 'pointer' : 'not-allowed' }"
                        title="上一首"
                        @click="playlistPrevSong"
                      >◀</button>
                       
                      <!-- 播放/暂停 -->
                      <button 
                        style="width: 44px; height: 44px; border-radius: 50%; border: none; cursor: pointer; display: flex; align-items: center; justify-content: center; font-size: 20px; color: white; transition: all 0.2s; box-shadow: 0 2px 8px rgba(0,0,0,0.15);" 
                        :style="playlistCurrentPlayingIndex >= 0 && !playlistShowStopButton ? 'background: linear-gradient(90deg, #ff6b6b, #ee5a5a);' : 'background: linear-gradient(90deg, #4ecdc4, #44a08d);'"
                        :title="playlistCurrentPlayingIndex >= 0 && !playlistShowStopButton ? '暂停' : '播放'"
                        @click="playlistTogglePlay"
                      >{{ playlistCurrentPlayingIndex >= 0 && !playlistShowStopButton ? '⏸' : '▶' }}</button>
                      
                      <!-- 下一首 -->
                      <button 
                        style="width: 36px; height: 36px; border-radius: 50%; border: none; background: #fff; cursor: pointer; display: flex; align-items: center; justify-content: center; font-size: 18px; color: #666; transition: all 0.2s; box-shadow: 0 2px 8px rgba(0,0,0,0.1);" 
                        :style="{ opacity: musicPlaylist.length > 1 && playlistCurrentPlayingIndex >= 0 ? 1 : 0.4, cursor: musicPlaylist.length > 1 && playlistCurrentPlayingIndex >= 0 ? 'pointer' : 'not-allowed' }"
                        title="下一首"
                        @click="playlistNextSong"
                      >▶</button>
                      <span style="font-size: 14px; color: #666;">下一首</span>
                      <!-- 播放模式 -->
                      <button 
                        style="width: 36px; height: 36px; border-radius: 50%; border: none; background: #fff; cursor: pointer; display: flex; align-items: center; justify-content: center; font-size: 18px; color: #666; transition: all 0.2s; box-shadow: 0 2px 8px rgba(0,0,0,0.1);" 
                        :title="playlistMode === 'single' ? '单曲循环' : playlistMode === 'loop' ? '列表循环' : '随机播放'"
                        @click="playlistToggleMode"
                      >{{ playlistMode === 'single' ? '🔂' : playlistMode === 'loop' ? '🔁' : '🔀' }}</button>
                    </div>
                    
                    <!-- 右侧：定位和下载 -->
                    <div style="display: flex; align-items: center; gap: 20px; justify-self: flex-end;">
                       
                      <!-- 定位按钮 -->
                      <button 
                        style="width: 36px; height: 36px; border-radius: 50%; border: none; background: #fff; cursor: pointer; display: flex; align-items: center; justify-content: center; font-size: 18px; color: #666; transition: all 0.2s; box-shadow: 0 2px 8px rgba(0,0,0,0.1);" 
                        :style="{ opacity: playlistCurrentPlayingIndex >= 0 ? 1 : 0.4, cursor: playlistCurrentPlayingIndex >= 0 ? 'pointer' : 'not-allowed' }"
                        title="定位到当前播放歌曲"
                        @click="scrollToCurrentPlaying"
                      >📍</button>
                       
                      <!-- 下载按钮 -->
                      <button 
                        style="width: 36px; height: 36px; border-radius: 50%; border: none; background: #fff; cursor: pointer; display: flex; align-items: center; justify-content: center; font-size: 18px; color: #666; transition: all 0.2s; box-shadow: 0 2px 8px rgba(0,0,0,0.1);" 
                        :style="{ opacity: playlistCurrentPlayingIndex >= 0 ? 1 : 0.4, cursor: playlistCurrentPlayingIndex >= 0 ? 'pointer' : 'not-allowed' }"
                        title="下载当前歌曲"
                        @click="playlistDownloadCurrent"
                      >⬇️</button>
                    </div>
                  </div>
                  
                  <!-- 第三行：歌词显示 -->
                  <div 
                    v-if="showLyrics && playlistCurrentPlayingIndex >= 0 && currentLyrics.length > 0" 
                    :style="{
                      display: 'flex', 
                      flexDirection: 'column', 
                      alignItems: 'center', 
                      padding: fullscreenMode ? '60px 40px' : (lyricsFullscreen ? '40px 50px' : '25px 30px'), 
                      background: fullscreenMode ? 'linear-gradient(135deg, #0d0d1a 0%, #1a1a3e 30%, #0d0d1a 100%)' : 'linear-gradient(180deg, rgba(255,255,255,0.95) 0%, rgba(248,250,252,0.98) 100%)',
                      backdropFilter: 'blur(20px)',
                      borderRadius: fullscreenMode ? '0px' : (lyricsFullscreen ? '24px' : '16px'),
                      position: fullscreenMode ? 'fixed' : (lyricsFullscreen ? 'absolute' : 'relative'),
                      left: fullscreenMode ? '0px' : (lyricsFullscreen ? '30px' : 'auto'),
                      top: fullscreenMode ? '0px' : (lyricsFullscreen ? '30px' : 'auto'),
                      bottom: fullscreenMode ? '0px' : (lyricsFullscreen ? '30px' : 'auto'),
                      right: fullscreenMode ? '0px' : (lyricsFullscreen ? '30px' : 'auto'),
                      width: fullscreenMode ? '100%' : (lyricsFullscreen ? 'calc(100% - 60px)' : '100%'),
                      height: fullscreenMode ? '100vh' : 'auto',
                      zIndex: fullscreenMode ? '9999' : (lyricsFullscreen ? '1001' : 'auto'),
                      overflowY: fullscreenMode ? 'hidden' : 'auto',
                      justifyContent: fullscreenMode || lyricsFullscreen ? 'center' : 'flex-start',
                      boxShadow: fullscreenMode ? 'none' : '0 25px 50px -12px rgba(0, 0, 0, 0.15), 0 0 0 1px rgba(0,0,0,0.05)',
                      border: fullscreenMode ? 'none' : '1px solid rgba(255,255,255,0.8)'
                    }"

                  >
                    <!-- 浏览器全屏按钮 -->
                    <button 
                      style="position: absolute; top: 15px; right: 15px; background: linear-gradient(135deg, #f87171, #fb923c); border: none; border-radius: 16px; font-size: 14px; cursor: pointer; color: white; padding: 12px 20px; transition: all 0.35s ease; box-shadow: 0 8px 25px rgba(248, 113, 113, 0.45); display: flex; align-items: center; gap: 8px; font-weight: 600; letter-spacing: 1px; z-index: 100;" 
                      onclick="(
                        function() {
                          const event = new CustomEvent('toggleFullscreenMode');
                          document.dispatchEvent(event);
                          const d = document.documentElement;
                          if (!document.fullscreenElement && !document.webkitFullscreenElement && !document.mozFullScreenElement && !document.msFullscreenElement) {
                            if (d.requestFullscreen) d.requestFullscreen();
                            else if (d.webkitRequestFullscreen) d.webkitRequestFullscreen();
                            else if (d.mozRequestFullScreen) d.mozRequestFullScreen();
                            else if (d.msRequestFullscreen) d.msRequestFullscreen();
                          } else {
                            if (document.exitFullscreen) document.exitFullscreen();
                            else if (document.webkitExitFullscreen) document.webkitExitFullscreen();
                            else if (document.mozCancelFullScreen) document.mozCancelFullScreen();
                            else if (document.msExitFullscreen) document.msExitFullscreen();
                          }
                        }
                      )()"
                      title="全屏显示（覆盖整个屏幕）"
                      @mouseenter="$event.currentTarget.style.transform = 'scale(1.08) translateY(-2px)'; $event.currentTarget.style.boxShadow = '0 12px 35px rgba(248, 113, 113, 0.55)';"
                      @mouseleave="$event.currentTarget.style.transform = 'scale(1) translateY(0)'; $event.currentTarget.style.boxShadow = '0 8px 25px rgba(248, 113, 113, 0.45)';"
                    >
                      <span style="font-size: 18px;">🖥️</span>
                      <span style="text-shadow: 0 1px 2px rgba(0,0,0,0.15);">全屏</span>
                    </button>
                    
                    <!-- 歌曲信息 -->
                    <div v-if="lyricsFullscreen" style="margin-bottom: 40px; text-align: center;">
                      <div style="display: flex; align-items: center; justify-content: center; gap: 12px; margin-bottom: 10px;">
                        <div style="width: 12px; height: 12px; background: linear-gradient(135deg, #10b981, #34d399); border-radius: 50%; animation: pulse 2s infinite;"></div>
                        <h3 style="margin: 0; font-size: 28px; background: linear-gradient(135deg, #6366f1, #8b5cf6, #ec4899); -webkit-background-clip: text; -webkit-text-fill-color: transparent; background-clip: text; font-weight: 700; font-family: 'Microsoft YaHei', 'PingFang SC', sans-serif; letter-spacing: 3px;">
                          {{ musicPlaylist[playlistCurrentPlayingIndex]?.name }}
                        </h3>
                      </div>
                      <p style="margin: 0; font-size: 16px; color: #64748b; font-weight: 500;">
                        <span style="color: #8b5cf6;">{{ musicPlaylist[playlistCurrentPlayingIndex]?.artist }}</span> 
                        <span style="color: #cbd5e1;">·</span> 
                        <span style="color: #10b981;">{{ musicPlaylist[playlistCurrentPlayingIndex]?.album }}</span>
                      </p>
                    </div>
                    
                    <!-- 歌词内容 -->
                    <div style="width: 100%; max-width: 800px;">
                      <div 
                        v-for="(lyric, idx) in currentLyrics.slice(Math.max(0, currentLyricIndex - (fullscreenMode ? 6 : 3)), currentLyricIndex + (fullscreenMode ? 7 : 4))" 
                        :key="lyric.time"
                        :style="{
                          fontSize: currentLyrics.indexOf(lyric) === currentLyricIndex ? (fullscreenMode ? '48px' : (lyricsFullscreen ? '36px' : '22px')) : (fullscreenMode ? '32px' : (lyricsFullscreen ? '24px' : '16px')),
                          fontWeight: currentLyrics.indexOf(lyric) === currentLyricIndex ? '700' : '400',
                          textAlign: 'center',
                          padding: fullscreenMode ? '25px 30px' : (lyricsFullscreen ? '20px 30px' : '10px 20px'),
                          transition: 'all 0.5s cubic-bezier(0.4, 0, 0.2, 1)',
                          opacity: currentLyrics.indexOf(lyric) === currentLyricIndex ? 1 : (fullscreenMode ? 0.45 : 0.55),
                          transform: currentLyrics.indexOf(lyric) === currentLyricIndex ? 'translateY(0) scale(1.05)' : 'translateY(0) scale(1)',
                          borderRadius: currentLyrics.indexOf(lyric) === currentLyricIndex ? '20px' : '8px',
                          marginBottom: fullscreenMode ? '20px' : (lyricsFullscreen ? '12px' : '6px'),
                          fontFamily: 'Microsoft YaHei, PingFang SC, sans-serif',
                          letterSpacing: fullscreenMode ? '4px' : (lyricsFullscreen ? '3px' : '1.5px'),
                          position: 'relative',
                          overflow: 'hidden'
                        }"
                      >
                        <!-- 当前歌词渐变背景 -->
                        <div 
                          v-if="currentLyrics.indexOf(lyric) === currentLyricIndex"
                          :style="{ position: 'absolute', inset: 0, background: fullscreenMode ? 'linear-gradient(135deg, rgba(167, 139, 250, 0.2), rgba(232, 121, 249, 0.15), rgba(248, 113, 113, 0.1))' : 'linear-gradient(135deg, rgba(99, 102, 241, 0.15), rgba(139, 92, 246, 0.1), rgba(236, 72, 153, 0.15));', borderRadius: fullscreenMode ? '20px' : '16px;' }"
                        ></div>
                        
                        <!-- 当前歌词发光效果 -->
                        <span 
                          v-if="currentLyrics.indexOf(lyric) === currentLyricIndex"
                          :style="getCurrentLyricStyle()"
                        >{{ lyric.text }}</span>
                        
                        <!-- 非当前歌词 -->
                        <span :style="{ color: fullscreenMode ? '#818cf8' : '#64748b' }">{{ lyric.text }}</span>
                      </div>
                    </div>
                  </div>
                  
                  <!-- 第二行：进度条 -->
                  <div v-if="playlistCurrentPlayingIndex >= 0 && musicDuration > 0" style="display: flex; align-items: center; gap: 12px;">
                    <span style="font-size: 11px; color: #999; flex-shrink: 0;">{{ musicFormatTimeFunc(musicCurrentTime) }}</span>
                    <div 
                      style="flex: 1; height: 5px; background: #e0e0e0; border-radius: 3px; cursor: pointer; user-select: none;"
                      @click="musicSeekToFunc($event)"
                    >
                      <div 
                        style="height: 100%; background: linear-gradient(90deg, #4ecdc4, #44a08d); border-radius: 3px; transition: width 0.1s ease;"
                        :style="{ width: `${(musicCurrentTime / musicDuration) * 100}%` }"
                      ></div>
                    </div>
                    <span style="font-size: 11px; color: #999; flex-shrink: 0;">{{ musicFormatTimeFunc(musicDuration) }}</span>
                  </div>
                </div>
              </div>
            </div>
            
            <!-- 收藏列表面板 -->
            <div 
              v-if="showFavoritesPanel" 
              :style="{ position: 'absolute', right: '20px', top: '4.6rem', bottom: '4.8rem', zIndex: '1000', background: 'white', borderRadius: '12px', boxShadow: '0 10px 40px rgba(0,0,0,0.2)', overflow: 'hidden', width: '220px', display: 'flex', flexDirection: 'column' }"
            >
              <div style="padding: 15px; border-bottom: 1px solid #eee; flex-shrink: 0;">
                <!-- 标题行 -->
                <div style="display: flex; justify-content: space-between; align-items: center;">
                  <h4 style="margin: 0; color: #333; white-space: nowrap; flex: 1; font-size: 14px;">❤️ 收藏列表</h4>
                  <div style="display: flex; gap: 6px; align-items: center;">
                    <button 
                      style="background: none; border: 1px solid #ff6b6b; border-radius: 3px; padding: 3px 8px; font-size: 11px; cursor: pointer; color: #ff6b6b; white-space: nowrap;" 
                      @click="clearFavorites"
                    >清空</button>
                    <button 
                      style="background: none; border: none; font-size: 1.2rem; cursor: pointer; color: #999; padding: 0;" 
                      @click="closeFavoritesPanel"
                    >×</button>
                  </div>
                </div>
                <!-- 操作提示 -->
                <div 
                  v-if="favoritesToastMessage" 
                  :style="{ marginTop: '8px', padding: '6px 10px', borderRadius: '4px', fontSize: '12px', textAlign: 'center', transition: 'all 0.3s', background: favoritesToastType === 'success' ? 'rgba(39, 174, 96, 0.1)' : 'rgba(231, 76, 60, 0.1)', color: favoritesToastType === 'success' ? '#27ae60' : '#e74c3c' }"
                >
                  {{ favoritesToastMessage }}
                </div>
              </div>
              <div style="flex: 1; padding: 10px; overflow-y: auto;">
                <div v-if="favoriteMusicList.length === 0" style="text-align: center; color: #999; padding: 40px 20px;">
                  收藏列表为空<br>点击歌曲旁的"收藏"按钮添加
                </div>
                <div 
                  v-for="(item, index) in favoriteMusicList" 
                  :key="index"
                  :style="{ display: 'flex', alignItems: 'center', padding: '8px 10px', borderBottom: '1px solid #f0f0f0', cursor: 'pointer', transition: 'background 0.2s', background: 'transparent', position: 'relative', overflow: 'hidden' }"
                  @mouseenter="$event.currentTarget.style.background = '#f8f9fa'"
                  @mouseleave="$event.currentTarget.style.background = 'transparent'"
                >
                  <span style="margin-right: 10px; color: #999; font-size: 11px;">{{ index + 1 }}</span>
                  <div style="flex: 1; overflow: hidden;">
                    <span style="font-size: 13px; color: #333; overflow: hidden; text-overflow: ellipsis; white-space: nowrap;">{{ item.name }}</span>
                  </div>
                  <div style="display: flex; gap: 4px; align-items: center;">
                    <button 
                      style="background: none; border: 1px solid #4ecdc4; border-radius: 3px; padding: 2px 6px; font-size: 10px; cursor: pointer; color: #4ecdc4; white-space: nowrap;" 
                      @click.stop="addFavoriteToPlaylist(item)"
                    >加播</button>
                    <button 
                      style="background: none; border: 1px solid #ff6b6b; border-radius: 3px; padding: 2px 6px; font-size: 10px; cursor: pointer; color: #ff6b6b; white-space: nowrap;" 
                      @click.stop="removeFromFavorites(index)"
                    >移除</button>
                  </div>
                </div>
              </div>
            </div>
            
            <!-- 播放列表小圆球 -->
<div 
  v-if="showPlaylistBall" 
  :style="{
    position: 'absolute', 
    left: '10px', 
    top: '50%', 
    transform: 'translateY(-50%)', 
    zIndex: '999', 
    width: '36px', 
    height: '36px', 
    borderRadius: '50%', 
    display: 'flex', 
    alignItems: 'center', 
    justifyContent: 'center', 
    cursor: 'pointer', 
    boxShadow: '0 4px 12px rgba(78, 205, 196, 0.4)', 
    transition: 'all 0.2s',
    background: playlistCurrentPlayingIndex >= 0 && musicPlaylist[playlistCurrentPlayingIndex]?.cover ? 'transparent' : 'linear-gradient(90deg, #4ecdc4, #44a08d)'
  }"
  @click="togglePlaylistBall"
  @mouseenter="$event.currentTarget.style.transform = 'translateY(-50%) scale(1.1)'"
  @mouseleave="$event.currentTarget.style.transform = 'translateY(-50%) scale(1)'"
>
  <!-- 专辑图片（播放时显示并旋转） -->
  <img 
  v-if="playlistCurrentPlayingIndex >= 0 && musicPlaylist[playlistCurrentPlayingIndex]?.cover" 
  :src="musicPlaylist[playlistCurrentPlayingIndex].cover" 
  alt="专辑封面" 
  class="album-cover-spin"
  style="width: 100%; height: 100%; border-radius: 50%; object-fit: cover;"
/>

  <!-- 默认图标（没有播放时显示） -->
  <span v-else style="font-size: 16px;">🎵</span>
  <!-- 歌曲数量标记 -->
  <span v-if="musicPlaylist.length > 0" style="position: absolute; top: -4px; right: -4px; background: #ff6b6b; color: white; font-size: 10px; width: 16px; height: 16px; border-radius: 50%; display: flex; align-items: center; justify-content: center; z-index: 1;">{{ musicPlaylist.length > 99 ? '99+' : musicPlaylist.length }}</span>
</div>

            
            <!-- 搜索栏 -->
            <div style="display: flex; gap: 10px; margin-bottom: 1rem; margin-top: 0.5rem; align-items: center;">
              <!-- 左侧按钮组 -->
              <div style="display: flex; gap: 8px; flex-shrink: 0;">
                <button 
                  style="padding: 10px 18px; border-radius: 25px; border: none; background: linear-gradient(90deg, #4ecdc4, #44a08d); color: white; cursor: pointer; font-size: 14px; font-weight: 500; transition: all 0.2s; white-space: nowrap; height: 38px; display: flex; align-items: center; justify-content: center;"
                  @click="showPlaylistPanel = !showPlaylistPanel"
                >
                  播放列表
                </button>
                <button 
                  style="padding: 10px 18px; border-radius: 25px; border: none; background: linear-gradient(90deg, #ffd93d, #ff9500); color: white; cursor: pointer; font-size: 14px; font-weight: 500; transition: all 0.2s; white-space: nowrap; height: 38px; display: flex; align-items: center; justify-content: center;"
                  @click="showBatchAddSelection = true"
                >
                  批量加播
                </button>
                <button 
                  style="padding: 10px 18px; border-radius: 25px; border: none; background: linear-gradient(90deg, #54a0ff, #5f27cd); color: white; cursor: pointer; font-size: 14px; font-weight: 500; transition: all 0.2s; white-space: nowrap; height: 38px; display: flex; align-items: center; justify-content: center;"
                  @click="showMusicBatchDownloadSelection = true"
                >
                  {{ isMusicDownloading ? `批量下载 (${getCompletedDownloadCount()}/${musicDownloadProgress.total})` : '批量下载' }}
                </button>
              </div>
              
              <div style="flex: 1; position: relative; min-width: 0;">
                <input 
                  type="text" 
                  v-model="musicSearchQuery" 
                  placeholder="1.默认展示卡片分类的歌单 2.输入歌名、歌手可查寻歌曲下载（过滤掉不能播放的）～" 
                  style="width: 100%; padding: 10px 35px 10px 15px; border: 1px solid #ddd; border-radius: 25px; font-size: 14px; outline: none; transition: border-color 0.3s; box-sizing: border-box;"
                  @keyup.enter="searchMusic"
                >
                <button 
                  v-if="musicSearchQuery" 
                  @click.stop="clearSearchQuery"
                  style="position: absolute; right: 10px; top: 50%; transform: translateY(-50%); background: #ddd; border: none; border-radius: 50%; width: 20px; height: 20px; cursor: pointer; color: #666; font-size: 12px; display: flex; align-items: center; justify-content: center; line-height: 1;"
                >✕</button>
                <!-- 内联提示 -->
                <span v-if="showSearchHint" style="position: absolute; left: 15px; top: 100%; margin-top: 4px; color: #ff6b6b; font-size: 12px;">
                  请输入歌曲名或歌手名
                </span>
              </div>
              <!-- 音质选择下拉框 -->
              <select 
                v-model="selectedQuality" 
                style="padding: 10px 10px; border: 1px solid #ddd; border-radius: 25px; font-size: 14px; outline: none; cursor: pointer; background: white; min-width: 100px; width: 100px;"
              >
                <option value="标准音质">标准音质</option>
                <option value="极高音质">极高音质</option>
                <option value="无损音质">无损音质</option>
              </select>
              <!-- 音乐平台选择下拉框 -->
              <select 
                v-model="selectedPlatform" 
                style="padding: 10px 18px; border: 1px solid #ddd; border-radius: 25px; font-size: 14px; outline: none; cursor: pointer; background: white; width: auto; min-width: 80px;"
              >
                <option value="kuwo">酷我音乐</option>
                <option value="netease">网易云</option>
              </select>
              <button 
                style="padding: 10px 25px; border-radius: 25px; border: none; background: linear-gradient(90deg, #4ecdc4, #44a08d); color: white; cursor: pointer; font-size: 14px; font-weight: 500; transition: all 0.2s; white-space: nowrap; height: 38px; display: flex; align-items: center; justify-content: center;"
                @click="searchMusic"
              >
                搜索
              </button>
              <button 
                style="padding: 10px 18px; border-radius: 25px; border: none; background: linear-gradient(90deg, #ff6b6b, #ee5a5a); color: white; cursor: pointer; font-size: 14px; font-weight: 500; transition: all 0.2s; white-space: nowrap; height: 38px; display: flex; align-items: center; justify-content: center;"
                @click="showFavoriteList"
              >
                收藏列表
              </button>
            </div>
            
            <!-- 搜索结果列表 -->
            <div style="border-radius: 12px; overflow: hidden; background: #ffffff;">
              <!-- 批量加播选择界面头部 -->
              <div v-if="showBatchAddSelection" style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 1rem; padding: 10px; background: #f8f9fa; border-radius: 8px;">
                <h4 style="margin: 0; color: #333;">选择要添加到播放列表的歌曲</h4>
                <div>
                  <button style="padding: 4px 12px; margin-right: 8px; border-radius: 4px; border: 1px solid #ddd; background: white; cursor: pointer;" @click="selectAllMusicItemsFunc">全选</button>
                  <button style="padding: 4px 12px; margin-right: 8px; border-radius: 4px; border: 1px solid #ddd; background: white; cursor: pointer;" @click="deselectAllMusicItemsFunc">取消全选</button>
                  <button style="padding: 4px 12px; border-radius: 4px; border: 1px solid #4ecdc4; background: #4ecdc4; color: white; cursor: pointer;" @click="confirmAddToPlaylist">确定添加 ({{ selectedMusicItems.length }})</button>
                  <button style="padding: 4px 12px; margin-left: 8px; border-radius: 4px; border: 1px solid #ddd; background: white; cursor: pointer;" @click="cancelBatchAdd">取消</button>
                </div>
              </div>
              
              <!-- 批量下载选择界面 -->
              <div v-if="showMusicBatchDownloadSelection" style="margin-bottom: 1rem;">
                <div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 1rem; padding: 10px; background: #f8f9fa; border-radius: 8px;">
                  <h4 style="margin: 0; color: #333;">选择要下载的歌曲</h4>
                  <div>
                    <template v-if="!isMusicDownloading">
                      <button style="padding: 4px 12px; margin-right: 8px; border-radius: 4px; border: 1px solid #ddd; background: white; cursor: pointer;" @click="selectAllMusicItemsFunc">全选</button>
                      <button style="padding: 4px 12px; margin-right: 8px; border-radius: 4px; border: 1px solid #ddd; background: white; cursor: pointer;" @click="deselectAllMusicItemsFunc">取消全选</button>
                      <button style="padding: 4px 12px; border-radius: 4px; border: 1px solid #4ecdc4; background: #4ecdc4; color: white; cursor: pointer;" @click="batchDownloadMusic">确定下载 ({{ selectedMusicDetails.length }})</button>
                      <button style="padding: 4px 12px; margin-left: 8px; border-radius: 4px; border: 1px solid #ddd; background: white; cursor: pointer;" @click="cancelMusicBatchDownload">取消</button>
                    </template>
                    <template v-else>
                      <span style="color: #4ecdc4; font-weight: 500; margin-right: 8px;">
                        {{ isMusicDownloadPaused ? '⏸ 已暂停' : `正在下载第${musicDownloadProgress.current}首：${currentDownloadSongName} ${currentDownloadProgress}%` }}
                      </span>
                      <button style="padding: 4px 12px; margin-right: 8px; border-radius: 4px; border: 1px solid #4ecdc4; background: #4ecdc4; color: white; cursor: pointer;" @click="toggleMusicDownloadPause">
                        {{ isMusicDownloadPaused ? '▶ 继续' : '⏸ 暂停' }}
                      </button>
                      <button style="padding: 4px 12px; margin-right: 8px; border-radius: 4px; border: 1px solid #95a5a6; background: #95a5a6; color: white; cursor: pointer;" @click="closeMusicDownload">关闭</button>
                      <button style="padding: 4px 12px; border-radius: 4px; border: 1px solid #ff6b6b; background: #ff6b6b; color: white; cursor: pointer;" @click="stopMusicDownload">结束下载</button>
                    </template>
                  </div>
                </div>
              </div>
              
              <!-- 音乐列表 -->
              <div 
                v-for="(item, index) in musicSearchResults" 
                :key="item.id"
                style="display: flex; justify-content: space-between; align-items: center; padding: 12px 15px; border-bottom: 1px solid #eee;"
                :style="{ background: index % 2 === 0 ? 'rgba(78, 205, 196, 0.08)' : 'rgba(84, 160, 255, 0.08)' }"
              >
                <!-- 复选框 - 在批量加播或批量下载模式下显示 -->
                <input 
                  v-if="showBatchAddSelection || showMusicBatchDownloadSelection"
                  type="checkbox" 
                  :checked="selectedMusicItems.includes(item.id.toString())"
                  style="width: 18px; height: 18px; cursor: pointer; margin-right: 12px; flex-shrink: 0;"
                  @change="handleMusicItemSelect(item, $event)"
                >
                <!-- 专辑图片 -->
                <div style="width: 44px; height: 44px; margin-right: 12px; flex-shrink: 0; border-radius: 4px; overflow: hidden; background: #f5f5f5;">
                  <img 
                    :src="getRandomAlbumCover(item.id)" 
                    :alt="item.album"
                    style="width: 100%; height: 100%; object-fit: cover;"
                  />
                </div>
                <div style="flex: 1;">
      <p style="margin: 0; font-weight: 600; font-size: 16px; color: #2d3436; font-family: 'Microsoft YaHei', 'PingFang SC', sans-serif; letter-spacing: 1px;">
          <span style="color: #ff6b6b; margin-right: 5px;">🎶</span>{{ item.name }} <span style="color: #b2bec3;">—</span> <span style="color: #6c5ce7;">{{ item.artist }}</span>
        <span style="margin-left: 10px;">
          <span v-if="selectedQuality === '标准音质'" style="display: inline-block; padding: 4px 12px; font-size: 11px; border-radius: 14px; background: linear-gradient(135deg, #fff5f5, #ffeaa7); color: #e67e22; border: none; font-weight: 600; box-shadow: 0 2px 8px rgba(230, 126, 34, 0.15);">🌟 标准</span>
          <span v-else-if="selectedQuality === '极高音质'" style="display: inline-block; padding: 4px 12px; font-size: 11px; border-radius: 14px; background: linear-gradient(135deg, #f5f0ff, #dfe6e9); color: #6c5ce7; border: none; font-weight: 600; box-shadow: 0 2px 8px rgba(108, 92, 231, 0.15);">✨ 极高</span>
          <span v-else-if="selectedQuality === '无损音质'" style="display: inline-block; padding: 4px 12px; font-size: 11px; border-radius: 14px; background: linear-gradient(135deg, #e8f5f3, #ffeaa7); color: #00b894; border: none; font-weight: 600; box-shadow: 0 2px 8px rgba(0, 184, 148, 0.15);">🎧 无损</span>
        </span>
      </p>
      <p style="margin: 5px 0 0 0; font-size: 13px; color: #74b9ff; font-family: 'Microsoft YaHei', 'PingFang SC', sans-serif;">
          <span style="margin-right: 4px;">💿</span>专辑：{{ item.album }}
        <span v-if="downloadFileSizes[item.id]" style="margin-left: 8px;">
          <template v-if="downloadSongStatus[item.id] === 'completed'">
            <span style="color: #27ae60;">已完成/{{ downloadFileSizes[item.id].sizeText }}</span>
          </template>
          <template v-else-if="downloadSongStatus[item.id] === 'downloading'">
            <span style="color: #4ecdc4;">{{ formatFileSize(downloadFileSizes[item.id].downloaded) }}/{{ downloadFileSizes[item.id].sizeText }}</span>
          </template>
          <template v-else-if="downloadFileSizes[item.id].sizeText">
            <span style="color: #95a5a6;">{{ downloadFileSizes[item.id].sizeText }}</span>
          </template>
        </span>
        <span v-else-if="musicIsPlaying && musicCurrentPlayingIndex === index && musicFileSize" style="color: #4ecdc4; margin-left: 8px;">
          文件：{{ musicFileSize }}
        </span>
      </p>
      
      <!-- 进度条 -->
      <div v-if="(musicIsPlaying || musicShowStopButton) && musicCurrentPlayingIndex === index && musicDuration > 0" style="margin-top: 8px;">
        <div 
          style="height: 6px; background: #e0e0e0; border-radius: 3px; cursor: pointer; user-select: none;"
          @click="musicSeekToFunc($event)"
        >
          <div 
            style="height: 100%; background: linear-gradient(90deg, #4ecdc4, #44a08d); border-radius: 3px; transition: width 0.1s ease;"
            :style="{ width: `${(musicCurrentTime / musicDuration) * 100}%` }"
          ></div>
        </div>
        <div style="display: flex; justify-content: space-between; margin-top: 4px;">
          <span style="font-size: 11px; color: #666;">{{ musicFormatTimeFunc(musicCurrentTime) }}</span>
          <span style="font-size: 11px; color: #999;">{{ musicFormatTimeFunc(musicDuration) }}</span>
        </div>
      </div>
    </div>
    <div style="display: flex; gap: 8px;">
      <!-- 播放/暂停按钮 -->
      <button 
        style="padding: 6px 12px; border-radius: 20px; border: none; color: white; cursor: pointer; font-size: 12px; font-weight: 500; display: flex; align-items: center; gap: 4px;" 
        :style="musicIsPlaying && musicCurrentPlayingIndex === index ? 'background: linear-gradient(90deg, #ff6b6b, #ee5a5a);' : 'background: linear-gradient(90deg, #4ecdc4, #44a08d);'"
        @click="musicTogglePlayFunc(index, item.id, item.source)"
      >
        {{ musicIsPlaying && musicCurrentPlayingIndex === index ? '⏸ 暂停' : '▶ 播放' }}
      </button>
      
      <!-- 停止按钮 -->
      <button 
        v-if="musicShowStopButton && musicCurrentPlayingIndex === index"
        style="padding: 6px 12px; border-radius: 20px; border: none; background: linear-gradient(90deg, #95a5a6, #7f8c8d); color: white; cursor: pointer; font-size: 12px; font-weight: 500; display: flex; align-items: center; gap: 4px;"
        @click="musicStopPlayFunc"
      >
        ⏹ 停止
      </button>
      
      <!-- 下载按钮 - 支持取消功能 -->
      <button 
        v-if="!showMusicBatchDownloadSelection && !isMusicDownloading"
        style="padding: 6px 12px; border-radius: 20px; border: none; font-size: 12px; font-weight: 500; display: flex; align-items: center; gap: 4px;" 
        :style="item.canDownload === false ? 'background: #ddd; color: #999; text-decoration: line-through; cursor: not-allowed;' : (item.canDownload === null ? 'background: #95a5a6; color: white; cursor: wait;' : (downloadSongStatus[item.id] === 'downloading' ? 'background: linear-gradient(90deg, #e74c3c, #c0392b); color: white; cursor: pointer;' : 'background: linear-gradient(90deg, #54a0ff, #5f27cd); color: white; cursor: pointer;'))"
        @click.stop="item.canDownload === false ? () => {} : (item.canDownload === null ? () => {} : (downloadSongStatus[item.id] === 'downloading' ? cancelSingleDownload(item.id) : musicDownloadFunc(item.id, item.actualSource || item.source, item.name, item.artist)))"
        :title="item.canDownload === false ? '下载链接已失效或找不到' : (item.canDownload === null ? '正在检查...' : '')"
      >
        {{ downloadSongStatus[item.id] === 'downloading' ? '✕ 取消' : (item.canDownload === null ? '⏳ 检查中' : '↓ 下载') }}
      </button>
      <!-- 正在批量下载时显示提示 -->
      <span v-else-if="isMusicDownloading" style="font-size: 12px; color: #95a5a6; padding: 6px 12px;">
        批量下载中
      </span>
      
      <!-- 收藏按钮 -->
      <button 
        style="padding: 6px 12px; border-radius: 20px; border: 1px solid #ff6b6b; cursor: pointer; font-size: 12px; font-weight: 500; display: flex; align-items: center; gap: 4px;" 
        :style="isFavorite(item.id) ? 'background: #ff6b6b; color: white;' : 'background: white; color: #ff6b6b;'"
        @click="toggleFavorite(item)"
      >
        {{ isFavorite(item.id) ? '♥ 已收藏' : '♥ 收藏' }}
      </button>
    </div>
  </div>
</div>










            
            <!-- 分页栏 -->
<div style="display: flex; justify-content: center; align-items: center; gap: 8px; margin-top: 1.5rem; flex-wrap: wrap;">
  <button 
    style="padding: 8px 14px; border-radius: 8px; border: 1px solid #ddd; background: white; cursor: pointer; font-size: 13px; transition: all 0.2s;" 
    :disabled="musicSearchCurrentPage === 1"
    :style="{ opacity: musicSearchCurrentPage === 1 ? 0.5 : 1, cursor: musicSearchCurrentPage === 1 ? 'not-allowed' : 'pointer' }"
    @click="changeMusicSearchPage(musicSearchCurrentPage - 1)"
  >
    上一页
  </button>
  
  <!-- 页码按钮 -->
  <template v-if="musicSearchTotalPages <= 10">
    <button 
      v-for="page in musicSearchTotalPages" 
      :key="page"
      style="padding: 8px 14px; border-radius: 8px; border: 1px solid #ddd; cursor: pointer; font-size: 13px; transition: all 0.2s;"
      :style="musicSearchCurrentPage === page ? 'background: linear-gradient(90deg, #4ecdc4, #44a08d); color: white; border: none;' : 'background: white; color: #666;'"
      @click="changeMusicSearchPage(page)"
    >
      {{ page }}
    </button>
  </template>
  
  <!-- 超过10页时的省略显示（修复重复数字bug） -->
  <template v-else>
    <!-- 第一页（当前页不是1时显示） -->
    <button 
      v-if="musicSearchCurrentPage !== 1"
      style="padding: 8px 14px; border-radius: 8px; border: 1px solid #ddd; cursor: pointer; font-size: 13px; transition: all 0.2s;"
      :style="musicSearchCurrentPage === 1 ? 'background: linear-gradient(90deg, #4ecdc4, #44a08d); color: white; border: none;' : 'background: white; color: #666;'"
      @click="changeMusicSearchPage(1)"
    >
      1
    </button>
    
    <span v-if="musicSearchCurrentPage > 3" style="color: #999; padding: 0 5px;">...</span>
    
    <!-- 前一页（只在当前页大于2且不是第1页时显示） -->
    <button 
      v-if="musicSearchCurrentPage > 2"
      style="padding: 8px 14px; border-radius: 8px; border: 1px solid #ddd; background: white; color: #666; cursor: pointer; font-size: 13px; transition: all 0.2s;"
      @click="changeMusicSearchPage(musicSearchCurrentPage - 1)"
    >
      {{ musicSearchCurrentPage - 1 }}
    </button>
    
    <!-- 当前页 -->
    <button 
      style="padding: 8px 14px; border-radius: 8px; background: linear-gradient(90deg, #4ecdc4, #44a08d); color: white; border: none; cursor: pointer; font-size: 13px; transition: all 0.2s;"
    >
      {{ musicSearchCurrentPage }}
    </button>
    
    <!-- 后一页（只在当前页小于总页数-1时显示） -->
    <button 
      v-if="musicSearchCurrentPage < musicSearchTotalPages - 1"
      style="padding: 8px 14px; border-radius: 8px; border: 1px solid #ddd; background: white; color: #666; cursor: pointer; font-size: 13px; transition: all 0.2s;"
      @click="changeMusicSearchPage(musicSearchCurrentPage + 1)"
    >
      {{ musicSearchCurrentPage + 1 }}
    </button>
    
    <span v-if="musicSearchCurrentPage < musicSearchTotalPages - 2" style="color: #999; padding: 0 5px;">...</span>
    
    <!-- 最后一页（当前页不是最后一页时显示） -->
    <button 
      v-if="musicSearchCurrentPage !== musicSearchTotalPages"
      style="padding: 8px 14px; border-radius: 8px; border: 1px solid #ddd; cursor: pointer; font-size: 13px; transition: all 0.2s;"
      :style="musicSearchCurrentPage === musicSearchTotalPages ? 'background: linear-gradient(90deg, #4ecdc4, #44a08d); color: white; border: none;' : 'background: white; color: #666;'"
      @click="changeMusicSearchPage(musicSearchTotalPages)"
    >
      {{ musicSearchTotalPages }}
    </button>
  </template>
  
  <button 
    style="padding: 8px 14px; border-radius: 8px; border: 1px solid #ddd; background: white; cursor: pointer; font-size: 13px; transition: all 0.2s;" 
    :disabled="musicSearchCurrentPage === musicSearchTotalPages"
    :style="{ opacity: musicSearchCurrentPage === musicSearchTotalPages ? 0.5 : 1, cursor: musicSearchCurrentPage === musicSearchTotalPages ? 'not-allowed' : 'pointer' }"
    @click="changeMusicSearchPage(musicSearchCurrentPage + 1)"
  >
    下一页
  </button>
</div>



          </div>
        </main>
        
        <!-- 底部 -->
        <footer class="app-footer">
          <p class="footer-text">🎵 听歌速下载 - 让音乐无处不在</p>
        </footer>
      </div>
    </div>
</template>

<style scoped>
/* 容器 */
.music-download-container {
  width: 100%;
  height: 100vh;
  display: flex;
  flex-direction: column;
  background: linear-gradient(135deg, #e0f2fe 0%, #f0fdf4 30%, #fdf4ff 60%, #fef3c7 100%);
  position: relative;
  overflow: hidden;
}

/* 专辑封面旋转动画 */
.album-cover-spin {
  animation: spin 8s linear infinite;
}

@keyframes spin {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}

/* 播放提示走马灯动画 */
.playing-marquee {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  overflow: hidden;
  display: flex;
  align-items: center;
}

.marquee-text {
  font-size: 20px;
  font-weight: 700;
  white-space: nowrap;
  animation: marquee 12s linear infinite, rainbow 3s linear infinite;
  display: inline-block;
  text-shadow: 0 0 12px rgba(255, 255, 255, 0.6);
}

@keyframes marquee {
  0% {
    transform: translateX(100vw);
    opacity: 0;
  }
  5% {
    opacity: 1;
  }
  95% {
    opacity: 1;
  }
  100% {
    transform: translateX(-100%);
    opacity: 0;
  }
}

@keyframes rainbow {
  0% {
    color: #ff6b6b;
    filter: drop-shadow(0 0 8px #ff6b6b);
  }
  20% {
    color: #feca57;
    filter: drop-shadow(0 0 8px #feca57);
  }
  40% {
    color: #48dbfb;
    filter: drop-shadow(0 0 8px #48dbfb);
  }
  60% {
    color: #1dd1a1;
    filter: drop-shadow(0 0 8px #1dd1a1);
  }
  80% {
    color: #5f27cd;
    filter: drop-shadow(0 0 8px #5f27cd);
  }
  100% {
    color: #ff6b6b;
    filter: drop-shadow(0 0 8px #ff6b6b);
  }
}

/* 背景装饰 */
.background-decoration {
  position: absolute;
  width: 100%;
  height: 100%;
  overflow: hidden;
  pointer-events: none;
  z-index: 0;
}

/* 浮动圆形 */
.floating-circle {
  position: absolute;
  border-radius: 50%;
  opacity: 0.25;
  animation: float 15s infinite ease-in-out;
}

.circle-1 {
  width: 350px;
  height: 350px;
  background: linear-gradient(135deg, #60a5fa, #34d399);
  top: -80px;
  left: -80px;
  animation-delay: 0s;
}

.circle-2 {
  width: 280px;
  height: 280px;
  background: linear-gradient(135deg, #f472b6, #fbbf24);
  bottom: -60px;
  right: -60px;
  animation-delay: -4s;
}

.circle-3 {
  width: 200px;
  height: 200px;
  background: linear-gradient(135deg, #a78bfa, #60a5fa);
  top: 30%;
  right: 15%;
  animation-delay: -8s;
}

.circle-4 {
  width: 160px;
  height: 160px;
  background: linear-gradient(135deg, #34d399, #fbbf24);
  bottom: 40%;
  left: 10%;
  animation-delay: -12s;
}

.circle-5 {
  width: 120px;
  height: 120px;
  background: linear-gradient(135deg, #f472b6, #a78bfa);
  top: 60%;
  left: 60%;
  animation-delay: -6s;
}

/* 浮动音符 */
.floating-note {
  position: absolute;
  font-size: 2rem;
  opacity: 0.4;
  animation: noteFloat 8s infinite ease-in-out;
}

.note-1 {
  top: 20%;
  left: 15%;
  animation-delay: 0s;
}

.note-2 {
  top: 45%;
  right: 20%;
  animation-delay: -2s;
}

.note-3 {
  bottom: 30%;
  left: 35%;
  animation-delay: -4s;
}

.note-4 {
  top: 15%;
  right: 35%;
  animation-delay: -6s;
}

.note-5 {
  bottom: 25%;
  right: 15%;
  animation-delay: -1s;
}

.note-6 {
  top: 70%;
  left: 20%;
  animation-delay: -3s;
}

/* 浮动星星 */
.floating-star {
  position: absolute;
  font-size: 1.5rem;
  opacity: 0.5;
  animation: starTwinkle 3s infinite ease-in-out;
}

.star-1 {
  top: 10%;
  left: 25%;
  animation-delay: 0s;
}

.star-2 {
  top: 35%;
  right: 10%;
  animation-delay: -0.5s;
}

.star-3 {
  bottom: 40%;
  left: 60%;
  animation-delay: -1s;
}

.star-4 {
  bottom: 20%;
  right: 30%;
  animation-delay: -1.5s;
}

.star-5 {
  top: 50%;
  left: 10%;
  animation-delay: -2s;
}

@keyframes float {
  0%, 100% {
    transform: translate(0, 0) scale(1) rotate(0deg);
  }
  25% {
    transform: translate(60px, -40px) scale(1.15) rotate(90deg);
  }
  50% {
    transform: translate(-40px, 50px) scale(0.9) rotate(180deg);
  }
  75% {
    transform: translate(30px, 20px) scale(1.1) rotate(270deg);
  }
}

@keyframes noteFloat {
  0%, 100% {
    transform: translate(0, 0) rotate(0deg);
    opacity: 0.4;
  }
  25% {
    transform: translate(30px, -25px) rotate(15deg);
    opacity: 0.6;
  }
  50% {
    transform: translate(-20px, -15px) rotate(-10deg);
    opacity: 0.3;
  }
  75% {
    transform: translate(15px, 20px) rotate(5deg);
    opacity: 0.5;
  }
}

@keyframes starTwinkle {
  0%, 100% {
    opacity: 0.3;
    transform: scale(1);
  }
  50% {
    opacity: 0.8;
    transform: scale(1.3);
  }
}

/* 主内容 */
.main-content {
  position: relative;
  z-index: 1;
  flex: 1;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

/* 头部 */
.app-header {
  padding: 2rem 2rem 1rem;
  text-align: center;
  background: rgba(255, 255, 255, 0.6);
  backdrop-filter: blur(20px);
  border-bottom: 1px solid rgba(255, 255, 255, 0.3);
}

.app-title {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 1rem;
  margin: 0 0 0 -2.3rem;
  font-size: 2.8rem;
  font-weight: 800;
  background: linear-gradient(90deg, #3b82f6, #10b981, #f59e0b, #ec4899, #8b5cf6);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  text-shadow: 0 0 40px rgba(59, 130, 246, 0.3);
}

.music-icon {
  font-size: 3.2rem;
  animation: bounce 1.5s infinite;
}

@keyframes bounce {
  0%, 100% {
    transform: translateY(0) scale(1);
  }
  50% {
    transform: translateY(-12px) scale(1.1);
  }
}

.title-text {
  font-family: 'Microsoft YaHei', 'PingFang SC', cursive;
}

.app-subtitle {
  margin: 0.8rem 0 0;
  color: #374151;
  font-size: 1.4rem;
  font-weight: 600;
  text-align: center;
  font-weight: 500;
}

/* 内容区域 */
.content-area {
  flex: 1;
  overflow-y: auto;
  padding: 1.5rem 2rem;
}

.content-area::-webkit-scrollbar {
  width: 6px;
}

.content-area::-webkit-scrollbar-track {
  background: transparent;
}

.content-area::-webkit-scrollbar-thumb {
  background: rgba(255, 255, 255, 0.2);
  border-radius: 3px;
}

/* 卡片页面 */
.cards-page {
  max-width: 1400px;
  margin: 0 auto;
}

.category-section {
  text-align: center;
  margin-bottom: 2rem;
}

.section-title {
  font-size: 1.8rem;
  color: white;
  margin: 0 0 0.5rem;
  font-weight: 600;
}



.cards-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 2.5rem;
  padding-top: 2rem;
}

.category-card {
  position: relative;
  border-radius: 24px;
  overflow: hidden;
  aspect-ratio: 4/3;
  cursor: pointer;
  transition: all 0.35s cubic-bezier(0.4, 0, 0.2, 1);
  box-shadow: 0 15px 45px rgba(0, 0, 0, 0.15);
  background: white;
}

.category-card:hover {
  transform: translateY(-12px) scale(1.03);
  box-shadow: 0 25px 60px rgba(0, 0, 0, 0.2);
}

.card-cover {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 0.4s ease;
}

.category-card:hover .card-cover {
  transform: scale(1.15);
}

.card-overlay {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  padding: 1.2rem;
  background: linear-gradient(transparent, rgba(0, 0, 0, 0.7));
  display: flex;
  align-items: center;
  justify-content: center;
}

.card-title {
  color: white;
  font-size: 1.25rem;
  font-weight: 700;
  text-shadow: 0 3px 6px rgba(0, 0, 0, 0.6);
}

/* 搜索页面 */
.search-page {
  max-width: 1400px;
  margin: 0 auto;
}

.back-btn {
  padding: 0.8rem 1.6rem;
  border-radius: 28px;
  border: 2px solid #3b82f6;
  background: white;
  color: #3b82f6;
  cursor: pointer;
  font-size: 1rem;
  font-weight: 600;
  transition: all 0.3s ease;
  margin-bottom: 1.5rem;
  box-shadow: 0 4px 15px rgba(59, 130, 246, 0.2);
}

.back-btn:hover {
  background: #3b82f6;
  color: white;
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(59, 130, 246, 0.4);
}

/* 底部 */
.app-footer {
  padding: 1rem;
  text-align: center;
  background: rgba(255, 255, 255, 0.5);
  border-top: 1px solid rgba(0, 0, 0, 0.05);
}

.footer-text {
  margin: 0;
  color: rgba(55, 65, 81, 0.6);
  font-size: 0.95rem;
  font-weight: 500;
}

/* 响应式 */
@media (max-width: 768px) {
  .app-title {
    font-size: 1.8rem;
  }
  
  .music-icon {
    font-size: 2.2rem;
  }
  
  .content-area {
    padding: 1rem;
  }
  
  .cards-grid {
    grid-template-columns: repeat(auto-fill, minmax(180px, 1fr));
    gap: 1.5rem;
  }
}
</style>

<script setup>
import { ref } from 'vue'


// ===== 响应式变量 =====

// 这里需要复制所有相关的函数和变量
// 包括：fetchTopList, searchMusic, musicTogglePlayFunc等
// 音乐下载相关
const showMusicDownloadModal = ref(true)
const showMusicPlaylist = ref(false)
const musicSearchQuery = ref('')
const musicSearchResults = ref([])
const musicSearchCurrentPage = ref(1)
const musicSearchTotalPages = ref(1)
const selectedPlatform = ref('netease')
const selectedQuality = ref('标准音质')
const currentTopListType = ref('')
const currentTopListName = ref('')
const musicIsPlaying = ref(false)
const musicCurrentPlayingUrl = ref('')
const musicCurrentPlayingIndex = ref(-1)
const musicCurrentPlayingSongId = ref('') // 新增的歌曲ID追踪
const musicAudioPlayer = ref(null)
const musicCurrentTime = ref(0)
const musicDuration = ref(0)
const musicShowStopButton = ref(false)
const musicFileSize = ref('')
const downloadFileSizes = ref({})
const downloadSongStatus = ref({})
const lyricsFullscreen = ref(false)
const showLyrics = ref(false) // 默认关闭歌词
const currentLyrics = ref([])
const currentLyricIndex = ref(-1)

const showFavoritesPanel = ref(false)
const showSearchHint = ref(false)
const showMusicBatchDownloadSelection = ref(false)

const isSearching = ref(false)
const isPageLoading = ref(false)
const hasSearched = ref(false)
const searchCache = ref({})
const SEARCH_CACHE_EXPIRE = 5 * 60 * 1000 // 5分钟缓存过期
const musicDownloadCurrentPage = ref('cards')
const selectedMusicItems = ref([])
const selectedMusicDetails = ref([])
const isMusicDownloading = ref(false)
const isMusicDownloadPaused = ref(false)
const musicDownloadProgress = ref({ current: 0, total: 0 })
const currentDownloadSongName = ref('')
const currentDownloadProgress = ref('')
const musicCards = ref([])
const singleDownloadCancelled = ref({})
const favoriteMusicList = ref([])

// 排行榜数据缓存
const toplistCache = ref({})


// 播放列表相关
const musicPlaylist = ref([])
const showPlaylistPanel = ref(false)
const playlistPanelFullscreen = ref(false)
const playlistCurrentPlayingIndex = ref(-1)
const playlistShowStopButton = ref(false)
const playlistMode = ref('loop')
const showPlaylistBall = ref(false)
const showBatchAddSelection = ref(false)
const isFullscreen = ref(false)
const fullscreenMode = ref(false)







// ===== 工具函数和配置 =====
const platformMap = {
  '网易云': 'netease',
  'QQ音乐': 'qq',
  '酷狗': 'kugou',
  '酷我': 'kuwo'
}

// 网易云音乐排行榜ID映射
const topListIdMap = {
  'hot': '3778678',           // 云音乐热歌榜
  'new': '3779629',           // 云音乐新歌榜
  'soaring': '19723756',      // 云音乐飙升榜
  'original': '2884035',      // 网易原创歌曲榜
  'rap': '991319590',         // 云音乐说唱榜
  'electronic': '10520166',   // 云音乐电音榜
  'euro_america': '2006508653', // 欧美新歌榜
  'billboard': '21845217',    // Billboard Hot 100
  'beatport': '1064102070',   // Beatport电子榜
  'korean': '28095774',       // 韩国Melon排行榜周榜
  'uk': '26177625'            // UK排行榜周榜
}

const qualityMap = {
  '标准音质': 128,
  '极高音质': 320,
  '无损音质': 999
}


const proxyConfig = {
  getProxiedUrl: (url) => url // 简单实现，可根据需要修改
}

// 搜索单页（带重试机制）
const searchSinglePage = async (encodedName, source, page, retries = 3) => {
  const url = `https://music-api.gdstudio.xyz/api.php?types=search&source=${source}&name=${encodedName}&pages=${page}`;
  
  for (let i = 0; i < retries; i++) {
    try {
      const response = await fetch(url);
      const data = await response.json();
      if (Array.isArray(data)) return data;
      if (data && data.result && Array.isArray(data.result)) return data.result;
      
      // 如果返回空数组，重试
      if (i < retries - 1) {
        console.log(`第${page}页返回空，第${i + 1}次重试...`);
        await new Promise(resolve => setTimeout(resolve, 500));
        continue;
      }
      return [];
    } catch (e) {
      console.error(`搜索单页失败 (第${i + 1}次尝试):`, e);
      if (i < retries - 1) {
        await new Promise(resolve => setTimeout(resolve, 500));
        continue;
      }
      return [];
    }
  }
  return [];
};

// 并行获取多页搜索结果
const fetchMultiplePagesParallel = async (encodedName, source, pages) => {
  const promises = pages.map(page => searchSinglePage(encodedName, source, page));
  const results = await Promise.all(promises);
  return results;
};

// 获取全量搜索结果（并行获取，带重试）
const fetchAllSearchResults = async (encodedName, source, maxPages = 15) => {
  const allResults = [];
  const seenIds = new Set();
  
  // 并行获取所有页，加快速度
  const pages = Array.from({ length: maxPages }, (_, i) => i + 1);
  const allPageResults = await fetchMultiplePagesParallel(encodedName, source, pages);
  
  // 合并所有页的数据并去重
  for (const pageResults of allPageResults) {
    if (pageResults && pageResults.length > 0) {
      for (const item of pageResults) {
        const itemId = item.id || item.songid || item.songId || '';
        if (itemId && !seenIds.has(itemId)) {
          seenIds.add(itemId);
          allResults.push(item);
        }
      }
    }
  }
  
  // 如果数据较少，尝试另一个平台
  if (allResults.length < 40 && source === 'netease') {
    console.log('网易云数据较少，尝试酷我平台...');
    const kuwoResults = await fetchAllSearchResults(encodedName, 'kuwo', 10);
    for (const item of kuwoResults) {
      const itemId = item.id || item.songid || item.songId || '';
      if (itemId && !seenIds.has(itemId)) {
        seenIds.add(itemId);
        allResults.push(item);
      }
    }
  }
  
  console.log('全量搜索完成，共', allResults.length, '条数据');
  return allResults;
};

// 异步获取封面
const fetchCoversForSearchResults = async (results) => {
  for (const item of results) {
    if (!item.cover || item.cover === '') {
      try {
        const cover = await fetchAlbumPic(item.id, item.source, item.name);
        if (cover) {
          item.cover = cover;
        }
      } catch (e) {
        console.error('获取封面失败:', e);
      }
    }
  }
};

// 获取排行榜（本地分页）
const fetchTopList = async (type, page, limit = 20) => {
  // 缓存数据，避免重复请求
  const cacheKey = `toplist_${type}`;
  
  // 如果没有缓存，先获取全部数据
  if (!toplistCache[cacheKey]) {
    const url = `https://music.xcloudv.top/php/toplist.php?type=${type}`;
    console.log('fetchTopList URL:', url);
    try {
      const response = await fetch(url);
      const data = await response.json();
      
      // 尝试多种数据结构
      let resultData = [];
      if (data && data.result && Array.isArray(data.result)) {
        resultData = data.result;
      } else if (Array.isArray(data)) {
        resultData = data;
      } else if (data && data.data && Array.isArray(data.data)) {
        resultData = data.data;
      } else if (data && data.songs && Array.isArray(data.songs)) {
        resultData = data.songs;
      }
      
      if (resultData.length > 0) {
        // 预处理数据并缓存
        toplistCache[cacheKey] = resultData.map(item => ({
          id: item.id || item.songid || item.songId || '',
          name: item.name || item.title || '未知歌曲',
          artist: Array.isArray(item.artist) ? item.artist.join(', ') : 
                  (item.artist || item.singer || item.ar?.[0]?.name || '未知歌手'),
          album: item.album || item.albumname || item.al?.name || '未知专辑',
          source: 'netease',
          cover: item.pic || item.cover || item.al?.picUrl || ''
        }));
      } else {
        toplistCache[cacheKey] = [];
      }
    } catch (e) {
      console.error('获取排行榜失败:', e);
      toplistCache[cacheKey] = [];
    }
  }
  
  // 本地分页处理
  const allData = toplistCache[cacheKey] || [];
  const total = allData.length;
  musicSearchTotalPages.value = Math.ceil(total / limit);
  
  // 计算分页范围
  const start = (page - 1) * limit;
  const end = start + limit;
  musicSearchResults.value = allData.slice(start, end);
  
  console.log('分页结果:', musicSearchResults.value.length, '条，总页数:', musicSearchTotalPages.value);
};

// 更新播放索引

// 更新播放索引
const updatePlayingIndexAfterDataChange = () => {
  const oldIndex = musicCurrentPlayingIndex.value;
  musicCurrentPlayingIndex.value = -1;
  
  if (musicCurrentPlayingSongId.value && musicSearchResults.value.length > 0) {
    const idx = musicSearchResults.value.findIndex(item => item.id === musicCurrentPlayingSongId.value);
    if (idx !== -1) {
      musicCurrentPlayingIndex.value = idx;
    } else {
      // 当前播放的歌曲不在当前页，停止播放
      if (musicAudioPlayer.value) {
        musicAudioPlayer.value.pause();
        musicAudioPlayer.value.currentTime = 0;
      }
      musicIsPlaying.value = false;
      musicShowStopButton.value = false;
    }
  }
};




// 生成示例图片卡片
const generateMusicCards = () => {
  const cards = [];
  const categories = [
    { name: '热歌榜', type: 'hot' },
    { name: '新歌榜', type: 'new' },
    { name: '飙升榜', type: 'soaring' },
    { name: '原创榜', type: 'original' },
    { name: '说唱榜', type: 'rap' },
    { name: '电音榜', type: 'electronic' },
    { name: '欧美榜', type: 'euro_america' },
    { name: 'Billboard榜', type: 'billboard' },
    { name: 'Beatport榜', type: 'beatport' },
    { name: '韩语榜', type: 'korean' },
    { name: 'UK榜', type: 'uk' }
  ];
  for (let i = 1; i <= 11; i++) {
    cards.push({
      id: i,
      title: categories[i - 1].name || `音乐分类 ${i}`,
      type: categories[i - 1].type || `type${i}`,
      cover: `/music/music-card-${i}.jpg`,
      description: `点击去下载${categories[i - 1].name || `第${i}类`}音乐`
    });
  }
  musicCards.value = cards;
};












// ===== 核心函数 =====// 
// 搜索音乐（先显示结果，后台异步更新可用性）
const searchMusic = async () => {
  // 如果正在搜索，直接返回
  if (isSearching.value) {
    console.log('正在搜索中，请稍候...');
    return;
  }
  
  // 如果搜索框为空，直接调用分类接口返回分类列表
  if (!musicSearchQuery.value.trim()) {
    hasSearched.value = false;
    musicSearchResults.value = [];
    musicSearchCurrentPage.value = 1;
    // 确保处于搜索页面时才跳转
    if (musicDownloadCurrentPage.value === 'search' && currentTopListType.value) {
      await fetchTopList(currentTopListType.value, 1, 20);
    } else if (musicDownloadCurrentPage.value === 'cards') {
      // 如果在卡片页面，不做任何操作
    }
    return;
  }
  
  // 有输入时隐藏提示
  showSearchHint.value = false;
  
  const source = platformMap[selectedPlatform.value] || 'netease';
  const searchCacheKey = `${source}_${musicSearchQuery.value}`;
  
  console.log('开始搜索:', musicSearchQuery.value, '平台:', source);
  
  // 标记正在搜索
  isSearching.value = true;
  
  try {
    // 直接获取全量数据（并行获取，确保缓存完整数据）
    console.log('正在获取全量搜索数据...');
    const encodedName = encodeURIComponent(musicSearchQuery.value);
    
    // 并行获取10页数据
    let allResults = await fetchAllSearchResults(encodedName, source, 10);
    
    // 如果数据较少，尝试另一个平台
    if (allResults.length < 30 && source === 'netease') {
      console.log('网易云数据较少，尝试酷我平台...');
      const kuwoResults = await fetchAllSearchResults(encodedName, 'kuwo', 10);
      const seenIds = new Set(allResults.map(item => item.id || item.songid || item.songId));
      for (const item of kuwoResults) {
        const itemId = item.id || item.songid || item.songId;
        if (itemId && !seenIds.has(itemId)) {
          seenIds.add(itemId);
          allResults.push(item);
        }
      }
    }
    
    if (allResults && allResults.length > 0) {
      console.log('全量数据获取成功，共', allResults.length, '条');
      
      // 缓存全量数据
      searchCache.value[searchCacheKey] = {
        data: allResults,
        timestamp: Date.now()
      };
      
      // 预检查数据，过滤掉无法播放的歌曲（只检查前60首）
      console.log('正在预检查歌曲可用性...');
      let processedResults = processSearchResults(allResults.slice(0, 60), source);
      const checkedResults = await batchPreCheckSongs(processedResults);
      
      // 只保留能播放的歌曲
      const playableResults = checkedResults.filter(item => item.canPlay);
      console.log('预检查完成，可播放:', playableResults.length, '首');
      
      // 如果没有可播放的歌曲
      if (playableResults.length === 0) {
        musicSearchResults.value = [];
        musicSearchTotalPages.value = 1;
        hasSearched.value = true;
        isSearching.value = false;
        alert('抱歉，未找到可播放的歌曲～');
        return;
      }
      
      // 缓存可播放的数据
      searchCache.value[searchCacheKey] = {
        data: playableResults,
        timestamp: Date.now()
      };
      
      // 显示第一页
      musicSearchResults.value = playableResults.slice(0, 20);
      musicSearchCurrentPage.value = 1;
      musicDownloadCurrentPage.value = 'search';
      hasSearched.value = true;
      
      // 计算真实总页数
      musicSearchTotalPages.value = Math.max(Math.ceil(playableResults.length / 20), 1);
      
      // 异步获取封面
      fetchCoversForSearchResults(musicSearchResults.value);
      
      // 重置搜索状态
      isSearching.value = false;
    } else {
      musicSearchResults.value = [];
      musicSearchTotalPages.value = 1;
      hasSearched.value = true;
      isSearching.value = false;
      alert('抱歉，未搜到您喜欢的歌曲条目～');
    }
    
  } catch (error) {
    console.error('搜索音乐失败:', error);
    musicSearchResults.value = [];
    musicSearchTotalPages.value = 1;
    hasSearched.value = true;
    isSearching.value = false;
    alert('搜索失败，请稍后重试');
  }
};


// 快速搜索（优先返回第一页，后台获取其他页）
const searchSinglePlatformEnhancedFast = async (query, source) => {
  const encodedName = encodeURIComponent(query);
  
  // 并行获取第一页和后续几页，加快速度
  console.log('并行获取搜索结果...');
  const promises = [
    searchSinglePage(encodedName, source, 1),
    searchSinglePage(encodedName, source, 2),
    searchSinglePage(encodedName, source, 3)
  ];
  
  const results = await Promise.all(promises);
  
  // 合并结果并去重
  const allResults = [];
  const seenIds = new Set();
  
  for (const pageResults of results) {
    if (pageResults && pageResults.length > 0) {
      for (const item of pageResults) {
        const itemId = item.id || item.songid || item.songId || '';
        if (itemId && !seenIds.has(itemId)) {
          seenIds.add(itemId);
          allResults.push(item);
        }
      }
    }
  }
  
  console.log('快速搜索完成，共', allResults.length, '条数据');
  return allResults;
};

// 后台获取更多搜索结果
const fetchMoreSearchResults = async (query, source, firstPageResults, cacheKey) => {
  try {
    const encodedName = encodeURIComponent(query);
    
    // 合并第一页数据
    const seenIds = new Set();
    const allResults = [];
    
    for (const item of firstPageResults) {
      const itemId = item.id || item.songid || item.songId || '';
      if (itemId && !seenIds.has(itemId)) {
        seenIds.add(itemId);
        allResults.push(item);
      }
    }
    
    // 获取剩余页数的数据（最多获取10页）
    const remainingResults = await fetchAllSearchResults(encodedName, source, 10);
    
    // 合并去重
    for (const item of remainingResults) {
      const itemId = item.id || item.songid || item.songId || '';
      if (itemId && !seenIds.has(itemId)) {
        seenIds.add(itemId);
        allResults.push(item);
      }
    }
    
    // 更新缓存
    searchCache.value[cacheKey] = {
      data: allResults,
      timestamp: Date.now()
    };
    
    // 更新总页数（至少显示10页，如果数据足够）
    const totalPages = Math.max(Math.ceil(allResults.length / 20), 10);
    musicSearchTotalPages.value = totalPages;
    
    console.log('后台获取完成，共', allResults.length, '条数据，总页数:', musicSearchTotalPages.value);
  } catch (e) {
    console.error('后台获取更多结果失败:', e);
  }
};


// 预检查歌曲是否可播放/可下载
const preCheckSongAvailability = async (song) => {
  let canPlay = false;
  let canDownload = false;
  let actualSource = song.source;
  
  // 先尝试当前平台
  const urlData = await getMusicUrlWithSize(song.id, song.source);
  
  if (urlData && urlData.url) {
    canPlay = true;
    canDownload = true;
  } else if (song.source === 'netease') {
    // 当前平台失败，尝试酷我
    const kuwoData = await getMusicUrlWithSize(song.id, 'kuwo');
    if (kuwoData && kuwoData.url) {
      canPlay = true;
      canDownload = true;
      actualSource = 'kuwo';
    }
  }
  
  return { canPlay, canDownload, actualSource };
};

// 批量预检查歌曲可用性（并行处理）
const batchPreCheckSongs = async (songs) => {
  // 一次并行处理所有歌曲
  const promises = songs.map(async (song) => {
    const availability = await preCheckSongAvailability(song);
    return { ...song, ...availability };
  });
  
  const results = await Promise.all(promises);
  return results;
};

// 处理搜索结果数据，统一字段格式
const processSearchResults = (results, source) => {
  return results.map(item => {
    let cover = '';
    if (item.pic_id && item.pic_id !== '') {
      cover = proxyConfig.getProxiedUrl(`https://jx.xshyun.top/music/?type=pic&id=${item.pic_id}`);
    } else if (item.pic) {
      cover = item.pic;
    } else if (item.cover) {
      cover = item.cover;
    }
    return {
      id: item.id || item.songid || item.songId || '',
      name: item.name || item.title || item.songname || '未知歌曲',
      artist: Array.isArray(item.artist) ? item.artist.join(', ') : 
              (item.artist || item.singer || item.artists || item.ar?.[0]?.name || '未知歌手'),
      album: item.album || item.albumname || item.al?.name || '未知专辑',
      source: item.source || source,
      cover: cover,
      canPlay: null,  // null表示未检查，true表示可播放，false表示不可播放
      canDownload: null,  // null表示未检查
      actualSource: item.source || source
    };
  });
};

// 分页处理（先显示数据，后台异步更新可用性）
const changeMusicSearchPage = async (page) => {
  if (page < 1) return;
  
  // 如果正在加载分页，直接返回
  if (isPageLoading.value) return;
  isPageLoading.value = true;
  
  musicSearchCurrentPage.value = page;
  
  // 如果有搜索关键词
  if (musicSearchQuery.value.trim()) {
    const source = platformMap[selectedPlatform.value] || 'netease';
    const searchCacheKey = `${source}_${musicSearchQuery.value}`;
    const cachedItem = searchCache.value[searchCacheKey];
    
    // 优先使用缓存进行本地分页
    if (cachedItem && Date.now() - cachedItem.timestamp < SEARCH_CACHE_EXPIRE) {
      const cachedData = cachedItem.data;
      const totalPages = Math.ceil(cachedData.length / 20);
      
      // 确保页码有效
      if (page > totalPages) {
        musicSearchCurrentPage.value = totalPages;
        isPageLoading.value = false;
        return;
      }
      
      musicSearchTotalPages.value = totalPages;
      const startIdx = (page - 1) * 20;
      const endIdx = startIdx + 20;
      
      // 缓存中的数据已经是预检查过的，直接显示
      musicSearchResults.value = cachedData.slice(startIdx, endIdx);
      
      // 异步获取封面
      fetchCoversForSearchResults(musicSearchResults.value);
      
      // 更新播放索引
      updatePlayingIndexAfterDataChange();
      
      // 重置加载状态
      isPageLoading.value = false;
      
      return;
    }
    
    // 没有缓存，调用接口获取数据
    const encodedName = encodeURIComponent(musicSearchQuery.value);
    
    try {
      // 获取全量数据
      const allResults = await fetchAllSearchResults(encodedName, source, 10);
      
      if (allResults.length > 0) {
        // 预检查并过滤
        let processedResults = processSearchResults(allResults, source);
        const checkedResults = await batchPreCheckSongs(processedResults);
        const playableResults = checkedResults.filter(item => item.canPlay);
        
        if (playableResults.length > 0) {
          searchCache.value[searchCacheKey] = {
            data: playableResults,
            timestamp: Date.now()
          };
          
          const totalPages = Math.ceil(playableResults.length / 20);
          musicSearchTotalPages.value = totalPages;
          
          // 确保页码有效
          const validPage = Math.min(page, totalPages);
          musicSearchCurrentPage.value = validPage;
          
          const startIdx = (validPage - 1) * 20;
          const endIdx = startIdx + 20;
          
          // 显示当前页
          musicSearchResults.value = playableResults.slice(startIdx, endIdx);
          
          // 异步获取封面
          fetchCoversForSearchResults(musicSearchResults.value);
        }
      }
    } catch (error) {
      console.error('分页加载失败:', error);
    }
    
    // 数据更新后，检查并更新播放索引
    updatePlayingIndexAfterDataChange();
    isPageLoading.value = false;
  } else if (currentTopListType.value) {
    // 否则调用分类榜单接口
    await fetchTopList(currentTopListType.value, page, 20);
    
    // 数据更新后，检查并更新播放索引
    updatePlayingIndexAfterDataChange();
    isPageLoading.value = false;
  }
};



// 播放/暂停音乐
const musicTogglePlayFunc = async (index, id, source) => {
  try {
    // 验证歌曲ID是否有效
    if (!id || id === '') {
      showToast('❌ 歌曲ID无效，无法播放');
      return;
    }
    
    // 获取当前歌曲
    const song = musicSearchResults.value[index];
    
    // 如果正在播放当前歌曲，暂停
    if (musicIsPlaying.value && musicCurrentPlayingIndex.value === index) {
      if (musicAudioPlayer.value) {
        musicAudioPlayer.value.pause();
      }
      musicIsPlaying.value = false;
      musicShowStopButton.value = true;
      return;
    }
    
    // 如果正在播放其他歌曲，先停止
    if (musicAudioPlayer.value && musicIsPlaying.value) {
      musicAudioPlayer.value.pause();
      musicShowStopButton.value = false;
    }
    
    // 如果是从暂停状态恢复播放
    if (musicCurrentPlayingIndex.value === index && !musicIsPlaying.value) {
      if (musicAudioPlayer.value) {
        musicAudioPlayer.value.play();
        musicIsPlaying.value = true;
        return;
      }
    }
    
    // 获取音乐链接（尝试多个平台）
    let urlData = await getMusicUrlWithSize(id, source);
    let actualSource = source;
    
    // 如果当前平台失败，尝试另一个平台
    if (!urlData.url && source === 'netease') {
      urlData = await getMusicUrlWithSize(id, 'kuwo');
      actualSource = 'kuwo';
    }
    
    if (urlData.url) {
      // 更新歌曲的可用性状态
      if (song) {
        song.canPlay = true;
        song.canDownload = true;
        song.actualSource = actualSource;
      }
      
      if (!musicAudioPlayer.value) {
        musicAudioPlayer.value = new Audio();
        musicAudioPlayer.value.addEventListener('timeupdate', musicUpdateProgressFunc);
        musicAudioPlayer.value.addEventListener('loadedmetadata', musicUpdateDurationFunc);
        musicAudioPlayer.value.addEventListener('ended', musicOnAudioEndedFunc);
      }
      
      musicAudioPlayer.value.src = urlData.url;
      musicFileSize.value = urlData.size || ''; // 保存文件大小
      musicAudioPlayer.value.play();
      
      musicIsPlaying.value = true;
      musicCurrentPlayingIndex.value = index;
      musicCurrentPlayingSongId.value = id; // 记录当前播放歌曲的ID
      musicShowStopButton.value = false;
    }
  } catch (error) {
    console.error('播放音乐失败:', error);
    musicResetPlayerFunc();
  }
};





// 停止播放
const musicStopPlayFunc = () => {
  if (musicAudioPlayer.value) {
    musicAudioPlayer.value.pause();
    musicAudioPlayer.value.currentTime = 0;
  }
  musicIsPlaying.value = false;
  musicCurrentPlayingIndex.value = -1;
  musicCurrentTime.value = 0;
  musicShowStopButton.value = false;
};



// 获取歌词
const fetchLyrics = async (id, source) => {
  try {
    const url = `https://music-api.gdstudio.xyz/api.php?types=lyric&source=${source}&id=${id}`;
    console.log('Fetching lyrics:', url);
    const response = await fetch(url);
    const data = await response.json();
    console.log('Lyrics response:', data);
    
    if (data && data.lyric) {
      // 解析歌词
      const lyricLines = data.lyric.split('\n');
      const parsedLyrics = [];
      
      lyricLines.forEach(line => {
        const match = line.match(/\[(\d{2}):(\d{2})\.(\d{2,3})\](.+)/);
        if (match) {
          const minutes = parseInt(match[1]);
          const seconds = parseInt(match[2]);
          const milliseconds = parseInt(match[3]);
          const text = match[4];
          
          parsedLyrics.push({
            time: minutes * 60 + seconds + milliseconds / 1000,
            text: text
          });
        }
      });
      
      console.log('Parsed lyrics count:', parsedLyrics.length);
      currentLyrics.value = parsedLyrics;
    } else {
      console.log('No lyrics found');
      currentLyrics.value = [];
    }
  } catch (error) {
    console.error('获取歌词失败:', error);
    currentLyrics.value = [];
  }
};

// 获取专辑封面
const fetchAlbumPic = async (id, source, name) => {
  try {
    const encodedName = encodeURIComponent(name || '');
    const url = `https://music-api.gdstudio.xyz/api.php?types=search&source=${source}&name=${encodedName}`;
    const response = await fetch(url);
    const data = await response.json();
    
    if (data && data.result && data.result.length > 0) {
      const song = data.result[0];
      if (song.pic) {
        return song.pic;
      } else if (song.cover) {
        return song.cover;
      }
    }
    return null;
  } catch (error) {
    console.error('获取专辑封面失败:', error);
    return null;
  }
};

// 获取音乐链接和文件大小（带重试）
const getMusicUrlWithSize = async (id, source, retries = 2) => {
  for (let i = 0; i < retries; i++) {
    try {
      // 根据用户选择的音质获取对应的br参数
      const br = qualityMap[selectedQuality.value] || 192;
      const url = `https://music-api.gdstudio.xyz/api.php?types=url&source=${source}&id=${id}&br=${br}`;
      const response = await fetch(url);
      const data = await response.json();
      
      if (data && data.url && data.url !== '') {
        // 尝试获取文件大小
        let size = '';
        let sizeBytes = 0;
        if (data.size) {
          sizeBytes = data.size;
          // 转换为可读格式
          if (data.size >= 1024 * 1024) {
            size = (data.size / (1024 * 1024)).toFixed(1) + 'M';
          } else if (data.size >= 1024) {
            size = (data.size / 1024).toFixed(1) + 'KB';
          } else {
            size = data.size + 'B';
          }
        } else {
          // 如果API没返回大小，尝试通过HEAD请求获取
          try {
            const headResponse = await fetch(data.url, { method: 'HEAD' });
            const contentLength = headResponse.headers.get('Content-Length');
            if (contentLength) {
              const fileSize = parseInt(contentLength);
              sizeBytes = fileSize;
              if (fileSize >= 1024 * 1024) {
                size = (fileSize / (1024 * 1024)).toFixed(1) + 'M';
              } else if (fileSize >= 1024) {
                size = (fileSize / 1024).toFixed(1) + 'KB';
              } else {
                size = fileSize + 'B';
              }
            }
          } catch (e) {
            console.log('无法获取文件大小');
          }
        }
        
        return { url: data.url, size, sizeBytes };
      }
      
      // 如果没有获取到链接，继续重试
      if (i < retries - 1) {
        await new Promise(resolve => setTimeout(resolve, 300));
      }
    } catch (error) {
      console.error(`获取音乐链接失败 (第${i + 1}次尝试):`, error);
      if (i < retries - 1) {
        await new Promise(resolve => setTimeout(resolve, 300));
      }
    }
  }
  
  return { url: '', size: '', sizeBytes: 0 };
};





//===== 播放控制辅助函数 =====
// 音乐进度更新
const musicUpdateProgressFunc = () => {
  if (musicAudioPlayer.value) {
    musicCurrentTime.value = musicAudioPlayer.value.currentTime;
    
    // 更新歌词索引
    if (currentLyrics.value.length > 0 && musicCurrentTime.value >= 0) {
      for (let i = currentLyrics.value.length - 1; i >= 0; i--) {
        if (currentLyrics.value[i].time <= musicCurrentTime.value) {
          currentLyricIndex.value = i;
          break;
        }
      }
    }
  }
};

// 音乐时长更新
const musicUpdateDurationFunc = () => {
  if (musicAudioPlayer.value) {
    musicDuration.value = musicAudioPlayer.value.duration;
  }
};

// 音乐播放结束
const musicOnAudioEndedFunc = () => {
  musicIsPlaying.value = false;
  musicShowStopButton.value = true;
};

// 音乐重置
const musicResetPlayerFunc = () => {
  if (musicAudioPlayer.value) {
    musicAudioPlayer.value.pause();
    musicAudioPlayer.value.currentTime = 0;
  }
  musicIsPlaying.value = false;
  musicCurrentPlayingIndex.value = -1;
  musicCurrentTime.value = 0;
};

// 音乐跳转
const musicSeekToFunc = (event) => {
  const rect = event.currentTarget.getBoundingClientRect();
  const percent = (event.clientX - rect.left) / rect.width;
  if (musicAudioPlayer.value && musicDuration.value > 0) {
    musicAudioPlayer.value.currentTime = percent * musicDuration.value;
  }
};

// 时间格式化
const musicFormatTimeFunc = (seconds) => {
  const mins = Math.floor(seconds / 60);
  const secs = Math.floor(seconds % 60);
  return `${mins}:${secs.toString().padStart(2, '0')}`;
};

// 文件大小格式化
const formatFileSize = (bytes) => {
  if (bytes >= 1024 * 1024) return (bytes / (1024 * 1024)).toFixed(1) + 'M';
  if (bytes >= 1024) return (bytes / 1024).toFixed(1) + 'KB';
  return bytes + 'B';
};

// 走马灯控制
let marqueeTimer = null;
const startMarquee = () => { stopMarquee(); };
const stopMarquee = () => { if (marqueeTimer) clearInterval(marqueeTimer); };







// 下载音乐（带进度显示）
const musicDownloadFunc = async (id, source, name, artist) => {
  const downloadBtn = event.currentTarget;
  const originalHTML = downloadBtn.innerHTML;
  const originalStyle = {
    background: downloadBtn.style.background,
    color: downloadBtn.style.color
  };
  
  try {
    // 重置取消状态
    singleDownloadCancelled.value[id] = false;
    
    // 使用图标显示状态
    downloadBtn.innerHTML = '⏳';
    downloadBtn.style.opacity = '0.7';
    downloadBtn.disabled = true;
    
    // 使用带进度的下载方式
    console.log(`获取下载链接: id=${id}, source=${source}`);
    let urlData = await getMusicUrlWithSize(id, source);
    console.log('下载链接数据:', urlData);
    
    // 如果当前平台获取失败，尝试另一个平台
    if (!urlData.url && source === 'netease') {
      console.log('网易云获取链接失败，尝试酷我...');
      urlData = await getMusicUrlWithSize(id, 'kuwo');
      console.log('酷我下载链接数据:', urlData);
    }
    
    if (urlData && urlData.url) {
      // 检查是否已取消
      if (singleDownloadCancelled.value[id]) {
        throw new Error('下载已取消');
      }
      
      // 初始化文件大小显示
      if (!downloadFileSizes.value[id]) {
        downloadFileSizes.value[id] = { downloaded: 0, total: urlData.sizeBytes || 0, sizeText: urlData.size || '' };
      }
      downloadSongStatus.value[id] = 'downloading';
      
      // 按钮变为取消按钮
      downloadBtn.innerHTML = '✕';
      downloadBtn.style.opacity = '1';
      downloadBtn.disabled = false;
      downloadBtn.style.background = '#ef4444';
      downloadBtn.style.color = 'white';
      
      await downloadMusicWithProgress(urlData.url, `${name} - ${artist}.mp3`, name, id, true);
      
      downloadSongStatus.value[id] = 'completed';
      
      setTimeout(() => {
        downloadBtn.innerHTML = originalHTML;
        downloadBtn.style.opacity = '1';
        downloadBtn.disabled = false;
        downloadBtn.style.background = originalStyle.background;
        downloadBtn.style.color = originalStyle.color;
        // 使用更好看的提示
        showToast(`✓ ${name} - ${artist}\n下载成功`);
      }, 500);
    } else {
      console.error('获取下载链接失败: url为空');
      throw new Error('获取下载链接失败');
    }
  } catch (error) {
    console.error('下载音乐失败:', error);
    // 重置按钮状态
    downloadBtn.innerHTML = originalHTML;
    downloadBtn.style.opacity = '1';
    downloadBtn.disabled = false;
    downloadBtn.style.background = originalStyle.background;
    downloadBtn.style.color = originalStyle.color;
    downloadSongStatus.value[id] = 'pending';
    
    // 只有非取消错误才显示提示
    if (error.message !== '下载已取消') {
      showToast('❌ 下载失败，请稍后重试');
    }
  }
};

// 获取音质文本（带符号）
const getQualityText = (quality) => {
  const qualityMap = {
    'standard': '🌟 标准',
    'high': '✨ 高清',
    'super': '💎 极高',
    'lossless': '🎧 无损'
  };
  return qualityMap[quality] || quality;
};

// 获取音质颜色
const getQualityColor = (quality) => {
  const colorMap = {
    'standard': 'linear-gradient(90deg, #6b7280, #9ca3af)',
    'high': 'linear-gradient(90deg, #3b82f6, #60a5fa)',
    'super': 'linear-gradient(90deg, #10b981, #34d399)',
    'lossless': 'linear-gradient(90deg, #f59e0b, #fbbf24)'
  };
  return colorMap[quality] || 'linear-gradient(90deg, #6b7280, #9ca3af)';
};

// Toast提示函数
const showToast = (message) => {
  // 创建Toast元素
  const toast = document.createElement('div');
  toast.style.cssText = `
    position: fixed;
    top: 80px;
    left: 50%;
    transform: translateX(-50%);
    background: linear-gradient(135deg, #4ecdc4, #44a08d);
    color: white;
    padding: 18px 30px;
    border-radius: 12px;
    font-size: 16px;
    text-align: center;
    z-index: 99999;
    box-shadow: 0 8px 30px rgba(78, 205, 196, 0.4);
    animation: fadeInDown 0.3s ease;
    max-width: 80%;
    line-height: 1.6;
    font-weight: 500;
  `;
  toast.innerHTML = message.replace(/\n/g, '<br>');
  
  // 添加动画样式
  const style = document.createElement('style');
  style.textContent = `
    @keyframes fadeInUp {
      from {
        opacity: 0;
        transform: translateX(-50%) translateY(20px);
      }
      to {
        opacity: 1;
        transform: translateX(-50%) translateY(0);
      }
    }
    @keyframes fadeOutDown {
      from {
        opacity: 1;
        transform: translateX(-50%) translateY(0);
      }
      to {
        opacity: 0;
        transform: translateX(-50%) translateY(20px);
      }
    }
  `;
  document.head.appendChild(style);
  
  document.body.appendChild(toast);
  
  // 3秒后消失
  setTimeout(() => {
    toast.style.animation = 'fadeOutDown 0.3s ease';
    setTimeout(() => {
      toast.remove();
      style.remove();
    }, 300);
  }, 3000);
};








// ===== 下载辅助函数 =====
// 带进度的下载
const downloadMusicWithProgress = async (url, filename, songName, songId, isSingle = false) => {
  return new Promise(async (resolve, reject) => {
    try {
      const response = await fetch(url);
      if (!response.ok) throw new Error('下载失败');
      
      const contentLength = response.headers.get('content-length');
      const total = contentLength ? parseInt(contentLength) : 0;
      let loaded = 0;
      
      const reader = response.body.getReader();
      const chunks = [];
      
      while (true) {
        // 检查是否需要暂停
        while (isMusicDownloadPaused.value && !isSingle) {
          await new Promise(resolve => setTimeout(resolve, 100));
        }
        
        const { done, value } = await reader.read();
        if (done) break;
        
        chunks.push(value);
        loaded += value.length;
        
        // 更新下载进度
        if (downloadFileSizes.value[songId]) {
          downloadFileSizes.value[songId].downloaded = loaded;
        }
        
        // 更新批量下载进度显示
        if (!isSingle) {
          const percent = total ? Math.round((loaded / total) * 100) : '...';
          currentDownloadProgress.value = percent;
        }
      }
      
      // 重置进度显示
      if (!isSingle) {
        currentDownloadProgress.value = '';
      }
      
      // 合并所有块并创建下载链接
      const blob = new Blob(chunks);
      const downloadUrl = URL.createObjectURL(blob);
      
      const a = document.createElement('a');
      a.href = downloadUrl;
      a.download = filename;
      document.body.appendChild(a);
      a.click();
      document.body.removeChild(a);
      URL.revokeObjectURL(downloadUrl);
      
      resolve();
    } catch (e) {
      reject(e);
    }
  });
};









// 从播放列表播放（独立状态）
const playFromPlaylist = async (index, isSkip = false) => {
  console.log('playFromPlaylist called with index:', index);
  const item = musicPlaylist.value[index];
  console.log('Playlist item:', item);
  if (item) {
    try {
      // 如果点击的是当前选中的歌曲且不是切歌操作
      if (playlistCurrentPlayingIndex.value === index && !isSkip) {
        // 如果处于暂停状态（显示停止按钮），则恢复播放
        if (playlistShowStopButton.value) {
          if (musicAudioPlayer.value) {
            musicAudioPlayer.value.play();
          }
          playlistShowStopButton.value = false;
          startMarquee();
        } else {
          // 当前正在播放，暂停它
          if (musicAudioPlayer.value) {
            musicAudioPlayer.value.pause();
          }
          playlistShowStopButton.value = true;
          stopMarquee();
        }
      } else {
        // 播放新歌曲
        // 如果正在播放其他歌曲，先暂停
        if (musicAudioPlayer.value) {
          musicAudioPlayer.value.pause();
        }
        
        // 获取音乐链接（必须优先）
        const urlData = await getMusicUrlWithSize(item.id, item.source);
        if (urlData.url) {
          // 并行获取歌词和专辑图片（不阻塞播放）
          Promise.all([
            fetchLyrics(item.id, item.source),
            fetchAlbumPic(item.id, item.source, item.name).then(pic => {
              if (pic) {
                item.cover = pic;
                saveMusicPlaylist();
              }
            })
          ]).catch(e => console.error('并行请求失败:', e));

          if (!musicAudioPlayer.value) {
            musicAudioPlayer.value = new Audio();
            musicAudioPlayer.value.addEventListener('timeupdate', musicUpdateProgressFunc);
            musicAudioPlayer.value.addEventListener('loadedmetadata', musicUpdateDurationFunc);
            musicAudioPlayer.value.addEventListener('ended', musicOnAudioEndedFunc);
          }
          
          musicAudioPlayer.value.src = urlData.url;
          startMarquee();
          musicFileSize.value = urlData.size || '';
          musicAudioPlayer.value.play();
          
          // 更新播放列表内部状态
          playlistCurrentPlayingIndex.value = index;
          playlistShowStopButton.value = false; // 播放时不显示停止按钮
        }
      }
    } catch (error) {
      console.error('播放音乐失败:', error);
      playlistCurrentPlayingIndex.value = -1;
      playlistShowStopButton.value = false;
    }
  }
};






// 播放列表上一首
const playlistPrevSong = () => {
  if (musicPlaylist.value.length > 1 && playlistCurrentPlayingIndex.value >= 0) {
    let prevIndex;
    if (playlistMode.value === 'random') {
      // 随机模式：随机选择一首（排除当前）
      do {
        prevIndex = Math.floor(Math.random() * musicPlaylist.value.length);
      } while (prevIndex === playlistCurrentPlayingIndex.value);
    } else {
      // 单曲/列表循环：上一首
      prevIndex = playlistCurrentPlayingIndex.value - 1;
      if (prevIndex < 0) {
        prevIndex = musicPlaylist.value.length - 1;
      }
    }
    playFromPlaylist(prevIndex, true);
  }
};

// 播放列表下一首
const playlistNextSong = () => {
  if (musicPlaylist.value.length > 0 && playlistCurrentPlayingIndex.value >= 0) {
    let nextIndex;
    if (playlistMode.value === 'random') {
      // 随机模式：随机选择一首
      nextIndex = Math.floor(Math.random() * musicPlaylist.value.length);
    } else if (playlistMode.value === 'single') {
      // 单曲循环：继续播放当前
      nextIndex = playlistCurrentPlayingIndex.value;
    } else {
      // 列表循环：下一首
      nextIndex = playlistCurrentPlayingIndex.value + 1;
      if (nextIndex >= musicPlaylist.value.length) {
        nextIndex = 0;
      }
    }
    playFromPlaylist(nextIndex, true);
  }
};

// 切换播放模式
const playlistToggleMode = () => {
  const modes = ['single', 'loop', 'random'];
  const currentIndex = modes.indexOf(playlistMode.value);
  playlistMode.value = modes[(currentIndex + 1) % modes.length];
};



// 从播放列表移除
const removeFromPlaylist = (index) => {
  // 如果移除的是当前正在播放的歌曲，停止播放
  if (playlistCurrentPlayingIndex.value === index) {
    if (musicAudioPlayer.value) {
      musicAudioPlayer.value.pause();
    }
    playlistCurrentPlayingIndex.value = -1;
    playlistShowStopButton.value = false;
  } else if (playlistCurrentPlayingIndex.value > index) {
    // 如果移除的是当前播放歌曲之前的歌曲，调整播放索引
    playlistCurrentPlayingIndex.value--;
  }
  musicPlaylist.value.splice(index, 1);
  saveMusicPlaylist();
};








// 清空播放列表
const clearMusicPlaylist = () => {
  if (confirm('确定要清空播放列表吗？')) {
    musicPlaylist.value = [];
    showPlaylistBall.value = false;
    saveMusicPlaylist();
  }
};







// 从localStorage读取播放列表
const loadMusicPlaylist = () => {
  try {
    const saved = localStorage.getItem('music_playlist');
    if (saved) {
      return JSON.parse(saved);
    }
  } catch (e) {
    console.error('加载播放列表失败:', e);
  }
  return [];
};




// 保存播放列表到localStorage
const saveMusicPlaylist = () => {
  try {
    localStorage.setItem('music_playlist', JSON.stringify(musicPlaylist.value));
  } catch (e) {
    console.error('保存播放列表失败:', e);
  }
};




// ===== UI交互函数 =====
// 获取随机专辑封面
const getRandomAlbumCover = (seed) => {
  const totalCovers = 80;
  // 使用seed生成固定的随机索引，相同歌曲显示相同封面
  const hash = seed.toString().split('').reduce((acc, char) => acc + char.charCodeAt(0), 0);
  const index = Math.abs(hash) % totalCovers + 1; // 1-80
  return `/public/music/album/专辑封面图${index}.jpg`;
};

// 页面导航
const goToSearchPage = (type, name) => {
  currentTopListType.value = type;
  currentTopListName.value = name;
  musicDownloadCurrentPage.value = 'search';
  musicSearchQuery.value = '';
  fetchTopList(type, 1, 20);
};

const goBackToCards = () => {
  musicDownloadCurrentPage.value = 'cards';
  currentTopListType.value = '';
  currentTopListName.value = '';
};

// 弹窗控制
const closeMusicDownloadModal = () => { 
  showMusicDownloadModal.value = false; 
};

const closePlaylistPanel = () => { 
  showPlaylistPanel.value = false; 
  // 如果播放列表有歌曲，显示小圆球
  if (musicPlaylist.value.length > 0) {
    showPlaylistBall.value = true;
  }
};

// 全屏切换函数
const toggleFullscreen = () => {
  const doc = document.documentElement;
  // 检查是否已经全屏
  const isFull = !!(document.fullscreenElement || document.webkitFullscreenElement || document.mozFullScreenElement || document.msFullscreenElement);
  
  if (!isFull) {
    // 进入全屏
    if (doc.requestFullscreen) {
      doc.requestFullscreen();
    } else if (doc.webkitRequestFullscreen) {
      doc.webkitRequestFullscreen();
    } else if (doc.mozRequestFullScreen) {
      doc.mozRequestFullScreen();
    } else if (doc.msRequestFullscreen) {
      doc.msRequestFullscreen();
    }
  } else {
    // 退出全屏
    if (document.exitFullscreen) {
      document.exitFullscreen();
    } else if (document.webkitExitFullscreen) {
      document.webkitExitFullscreen();
    } else if (document.mozCancelFullScreen) {
      document.mozCancelFullScreen();
    } else if (document.msExitFullscreen) {
      document.msExitFullscreen();
    }
  }
};

defineExpose({
  toggleFullscreen
});

// 监听全屏模式事件
document.addEventListener('toggleFullscreenMode', () => {
  fullscreenMode.value = true;
  playlistPanelFullscreen.value = true;
});

// 监听浏览器全屏退出事件
document.addEventListener('fullscreenchange', () => {
  if (!document.fullscreenElement) {
    fullscreenMode.value = false;
  }
});

// 获取当前歌词样式
const getCurrentLyricStyle = () => {
  return {
    position: 'relative',
    zIndex: 1,
    background: fullscreenMode.value ? 'linear-gradient(135deg, #a855f7, #ec4899, #f472b6)' : 'linear-gradient(135deg, #6366f1, #8b5cf6, #ec4899)',
    '-webkit-background-clip': 'text',
    '-webkit-text-fill-color': 'transparent',
    'background-clip': 'text',
    textShadow: fullscreenMode.value ? '0 0 50px rgba(168, 85, 247, 0.7), 0 0 100px rgba(236, 72, 153, 0.5), 0 0 150px rgba(244, 114, 182, 0.3)' : '0 0 30px rgba(99, 102, 241, 0.4), 0 0 60px rgba(139, 92, 246, 0.2)'
  };
};

const togglePlaylistBall = () => {
  showPlaylistPanel.value = !showPlaylistPanel.value;
  showPlaylistBall.value = !showPlaylistBall.value;
};

// 播放列表控制
const playlistTogglePlay = () => {
  if (playlistCurrentPlayingIndex.value >= 0) {
    playFromPlaylist(playlistCurrentPlayingIndex.value);
  }
};

const stopPlaylistPlay = () => {
  if (musicAudioPlayer.value) {
    musicAudioPlayer.value.pause();
    musicAudioPlayer.value.currentTime = 0;
  }
  playlistCurrentPlayingIndex.value = -1;
  playlistShowStopButton.value = false;
};

const scrollToCurrentPlaying = () => {
  const el = document.querySelector(`[data-playlist-item="${playlistCurrentPlayingIndex.value}"]`);
  if (el) {
    el.scrollIntoView({ behavior: 'smooth', block: 'center' });
  }
};

const playlistDownloadCurrent = () => {
  const item = musicPlaylist.value[playlistCurrentPlayingIndex.value];
  if (item) {
    musicDownloadFunc(item.id, item.source, item.name, item.artist);
  }
};

// 搜索控制
const clearSearchQuery = () => { 
  musicSearchQuery.value = '';
  // 重置播放状态，避免清空后分类列表显示播放状态
  if (musicAudioPlayer.value) {
    musicAudioPlayer.value.pause();
    musicAudioPlayer.value.currentTime = 0;
  }
  musicIsPlaying.value = false;
  musicCurrentPlayingIndex.value = -1;
  musicCurrentPlayingSongId.value = '';
  musicShowStopButton.value = false;
  musicCurrentTime.value = 0;
  musicDuration.value = 0;
  
  // 如果在搜索页面且有当前分类，返回分类列表第一页
  if (musicDownloadCurrentPage.value === 'search' && currentTopListType.value) {
    musicSearchCurrentPage.value = 1;
    fetchTopList(currentTopListType.value, 1, 20);
  }
};

// 批量操作
const selectAllMusicItemsFunc = () => {
  // 跨页累加选择：保留之前已选的，添加当前页未选的
  musicSearchResults.value.forEach(item => {
    const id = item.id.toString();
    if (!selectedMusicItems.value.includes(id)) {
      selectedMusicItems.value.push(id);
      selectedMusicDetails.value.push(item);
    }
  });
};

const deselectAllMusicItemsFunc = () => {
  // 只取消当前页的选择，保留其他页已选的
  const currentPageIds = musicSearchResults.value.map(item => item.id.toString());
  selectedMusicItems.value = selectedMusicItems.value.filter(id => !currentPageIds.includes(id));
  selectedMusicDetails.value = selectedMusicDetails.value.filter(item => !currentPageIds.includes(item.id.toString()));
};

const handleMusicItemSelect = (item, event) => {
  const id = item.id.toString();
  if (event.target.checked) {
    if (!selectedMusicItems.value.includes(id)) {
      selectedMusicItems.value.push(id);
      selectedMusicDetails.value.push(item);
    }
  } else {
    selectedMusicItems.value = selectedMusicItems.value.filter(i => i !== id);
    selectedMusicDetails.value = selectedMusicDetails.value.filter(i => i.id !== item.id);
  }
};

const confirmAddToPlaylist = () => {
  selectedMusicDetails.value.forEach(item => {
    const exists = musicPlaylist.value.some(p => p.id === item.id);
    if (!exists) {
      musicPlaylist.value.push({ ...item });
    }
  });
  saveMusicPlaylist();
  showBatchAddSelection.value = false;
  alert(`已添加 ${selectedMusicDetails.value.length} 首歌曲到播放列表`);
};

const cancelBatchAdd = () => { 
  showBatchAddSelection.value = false; 
};

// 显示收藏列表
const showFavoriteList = () => {
  showFavoritesPanel.value = true;
};

// 关闭收藏列表面板
const closeFavoritesPanel = () => {
  showFavoritesPanel.value = false;
};

// 清空收藏列表
const clearFavorites = () => {
  if (confirm('确定要清空收藏列表吗？')) {
    favoriteMusicList.value = [];
    saveFavorites();
    showToast('收藏列表已清空');
  }
};

// 将收藏歌曲添加到播放列表
const addFavoriteToPlaylist = (item) => {
  const exists = musicPlaylist.value.some(p => p.id === item.id);
  if (!exists) {
    musicPlaylist.value.push({ ...item });
    saveMusicPlaylist();
    showToast(`✓ "${item.name}" 已添加到播放列表`);
  } else {
    showToast('该歌曲已在播放列表中');
  }
};

// 从收藏列表移除
const removeFromFavorites = (index) => {
  const item = favoriteMusicList.value[index];
  favoriteMusicList.value.splice(index, 1);
  showToast(`✓ "${item.name}" 已从收藏移除`);
  // 保存到本地存储
  saveFavorites();
};

// 检查是否已收藏
const isFavorite = (songId) => {
  return favoriteMusicList.value.some(item => item.id === songId);
};

// 切换收藏状态
const toggleFavorite = (item) => {
  const index = favoriteMusicList.value.findIndex(i => i.id === item.id);
  if (index >= 0) {
    favoriteMusicList.value.splice(index, 1);
    showToast(`✓ "${item.name}" 已取消收藏`);
  } else {
    favoriteMusicList.value.push({ ...item });
    showToast(`✓ "${item.name}" 已添加到收藏`);
  }
  // 保存到本地存储
  saveFavorites();
};

// 统计已完成下载数量
const getCompletedDownloadCount = () => {
  return Object.keys(downloadSongStatus.value).filter(id => 
    downloadSongStatus.value[id] === 'completed'
  ).length;
};

// 批量下载
const batchDownloadMusic = async () => {
  console.log('batchDownloadMusic called');
  console.log('selectedMusicDetails length:', selectedMusicDetails.value.length);
  console.log('selectedMusicDetails:', selectedMusicDetails.value);
  
  if (selectedMusicDetails.value.length === 0) {
    showToast('请先选择要下载的歌曲');
    return;
  }
  
  isMusicDownloading.value = true;
  isMusicDownloadPaused.value = false;
  musicDownloadProgress.value = { current: 0, total: selectedMusicDetails.value.length };
  
  for (let i = 0; i < selectedMusicDetails.value.length; i++) {
    // 检查是否已停止
    if (!isMusicDownloading.value) break;
    
    // 检查是否暂停
    while (isMusicDownloadPaused.value) {
      await new Promise(resolve => setTimeout(resolve, 100));
      if (!isMusicDownloading.value) break;
    }
    
    if (!isMusicDownloading.value) break;
    
    const item = selectedMusicDetails.value[i];
    currentDownloadSongName.value = item.name;
    musicDownloadProgress.value.current = i + 1;
    
    try {
      // 获取下载链接（带平台备用）
      let urlData = await getMusicUrlWithSize(item.id, item.source);
      
      // 如果当前平台获取失败，尝试另一个平台
      if (!urlData.url && item.source === 'netease') {
        console.log(`网易云获取 ${item.name} 链接失败，尝试酷我...`);
        urlData = await getMusicUrlWithSize(item.id, 'kuwo');
      }
      
      if (urlData && urlData.url) {
        // 初始化文件大小显示
        if (!downloadFileSizes.value[item.id]) {
          downloadFileSizes.value[item.id] = { 
            downloaded: 0, 
            total: urlData.sizeBytes || 0, 
            sizeText: urlData.size || '' 
          };
        }
        downloadSongStatus.value[item.id] = 'downloading';
        
        await downloadMusicWithProgress(urlData.url, `${item.name} - ${item.artist}.mp3`, item.name, item.id, false);
        
        downloadSongStatus.value[item.id] = 'completed';
      } else {
        // 两个平台都获取失败
        console.log(`两个平台都无法获取 ${item.name} 的下载链接`);
        downloadSongStatus.value[item.id] = 'failed';
      }
    } catch (error) {
      console.error(`下载 ${item.name} 失败:`, error);
      downloadSongStatus.value[item.id] = 'failed';
    }
  }
  
  const completedCount = getCompletedDownloadCount();
  isMusicDownloading.value = false;
  showMusicBatchDownloadSelection.value = false; // 下载完成后关闭选择界面
  selectedMusicItems.value = []; // 清空已选项目
  selectedMusicDetails.value = []; // 清空已选详情
  
  if (completedCount > 0 || musicDownloadProgress.value.current > 0) {
    showToast(`✓ 已完成 ${completedCount} 首歌下载`);
  }
};

const cancelMusicBatchDownload = () => { 
  showMusicBatchDownloadSelection.value = false; 
};

const toggleMusicDownloadPause = () => {
  isMusicDownloadPaused.value = !isMusicDownloadPaused.value;
};

const closeMusicDownload = () => {
  // 关闭下载界面，但保持下载继续在后台
  showMusicBatchDownloadSelection.value = false;
};

const stopMusicDownload = () => {
  isMusicDownloading.value = false;
  isMusicDownloadPaused.value = false;
  showMusicBatchDownloadSelection.value = false;
  showToast('已停止批量下载');
};

// 取消单个下载
const cancelSingleDownload = (id) => { 
  singleDownloadCancelled.value[id] = true; 
};

// 从localStorage读取收藏列表
const loadFavorites = () => {
  try {
    const saved = localStorage.getItem('music_favorites');
    if (saved) {
      return JSON.parse(saved);
    }
  } catch (e) {
    console.error('加载收藏列表失败:', e);
  }
  return [];
};

// 保存收藏列表到localStorage
const saveFavorites = () => {
  try {
    localStorage.setItem('music_favorites', JSON.stringify(favoriteMusicList.value));
  } catch (e) {
    console.error('保存收藏列表失败:', e);
  }
};

// ===== 组件初始化 =====
// 生成音乐卡片
generateMusicCards();
// 加载保存的播放列表
musicPlaylist.value = loadMusicPlaylist();
// 加载保存的收藏列表
favoriteMusicList.value = loadFavorites();








</script>